# Working out whether a server can be added, and adding it

Everything below is measurement. A vendor's documentation will tell you their
MCP server supports OAuth; it will not tell you whether their registration
endpoint actually hands out a client, and that is the difference between an
entry that works and a row with a button that always fails.

Budget twenty minutes. Most of it is the probe.

## 1. Find the address

Almost always `https://mcp.<vendor>.com/mcp`. Known exceptions in this
catalogue: GitHub is `api.githubcopilot.com/mcp/`, HubSpot ends `/anthropic`,
Pipedream is `remote.mcp.pipedream.net`, Google is
`https://<product>mcp.googleapis.com/mcp/v1`.

If `/mcp` 404s, try `/sse` and the vendor's docs, then stop. A server you
cannot find in five minutes is one your users cannot either.

## 2. Does it answer

```sh
curl -s -i -X POST "$URL" \
  -H 'content-type: application/json' \
  -H 'accept: application/json, text/event-stream' \
  -d '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"protocolVersion":"2025-06-18","capabilities":{},"clientInfo":{"name":"probe","version":"0"}}}'
```

- **401 with a `WWW-Authenticate` header** — the ordinary case. Go on.
- **200** — it needs no authorization at all. Firecrawl is the one here. Skip
  to step 5.
- **404, or nothing** — there is no server. Stop.

## 3. Follow the metadata

The 401 carries `resource_metadata="…"`. Fetch it; if the header has no
pointer, try the default address — the well-known segment goes after the
origin, with the resource's path *after* it:

```sh
curl -s "https://$HOST/.well-known/oauth-protected-resource/$PATH" | python3 -m json.tool
```

You want `authorization_servers` and `scopes_supported`. No metadata means
nothing can discover how to authorize, and the entry is not addable — that is
Composio, Telegram and Atlassian today.

Then the authorization server's own document. **Two addresses, and the spec's
is not the obvious one:** RFC 8414 inserts the well-known segment between host
and path, OpenID Connect Discovery appends it. Try both.

```sh
curl -s "https://$ISSUER_HOST/.well-known/oauth-authorization-server/$ISSUER_PATH"
curl -s "https://$ISSUER/.well-known/oauth-authorization-server"
```

Stripe publishes only the first form. Reading only the second made it look
dead for a day.

## 4. Does registration actually work

`registration_endpoint` in that document means the server *offers* to hand out
clients. It does not mean it will.

```sh
curl -s -X POST "$REGISTRATION_ENDPOINT" -H 'content-type: application/json' -d '{
  "client_name": "Firetower",
  "redirect_uris": ["http://127.0.0.1:14400/api/v1/connectors/callback"],
  "grant_types": ["authorization_code", "refresh_token"],
  "response_types": ["code"],
  "token_endpoint_auth_method": "none",
  "application_type": "native"
}' -w '\n[%{http_code}]\n'
```

- **200 or 201 with a `client_id`** — one press to connect. Nothing more to do.
- **401, 403** — it offers and refuses. Figma does exactly this. Treat it as
  the next case.
- **No `registration_endpoint` at all** — somebody has to create an OAuth
  application once. GitHub, Slack, every Google one. Your entry needs `setup`
  links; see step 6.

`tools/probe-registration.py` in the Firetower repository does all of the
above for every entry at once, which is how you check the catalogue has not
rotted.

## 5. Write the folder

```
mcp/<name>/
  .claude-plugin/plugin.json
  .codex-plugin/plugin.json      the same bytes
  .mcp.json
  assets/logo.png
```

```json
{
  "name": "resend",
  "displayName": "Resend",
  "description": "Send transactional email and read back what happened to it.",
  "version": "1.0.0",
  "author": { "name": "Firetower", "url": "https://firetower.cloud" },
  "homepage": "https://resend.com"
}
```

`name` is the folder's name and a namespacing key — lowercase and hyphens.
`displayName` is what people read. Neither is derivable from the other:
`cal-com` is Cal.com and `workos` is WorkOS.

**The logo.** The vendor's own `apple-touch-icon` is usually best; their
GitHub organisation avatar (`https://github.com/<org>.png?size=256`) is a
good fallback and often better than a 32px favicon. Square, 180px or more,
under 96KB, and the extension must match the bytes.

## 6. What the server asks for

```json
{
  "mcpServers": {
    "resend": {
      "type": "http",
      "url": "https://mcp.resend.com/mcp",
      "scopes": ["…"],
      "setup": [{ "label": "…", "url": "https://…" }]
    }
  }
}
```

**`scopes` can only narrow.** Anything not on the server's own
`scopes_supported` is dropped, so this is for asking for *less*. Gmail
advertises twenty including the whole mailbox and delete; our entry asks for
four. Check each one you write against the list from step 3 — a scope that is
not advertised is silently discarded and the entry quietly asks for the
ceiling instead.

**`setup` is for servers that hand out no client.** As many links as the
vendor's setup takes, each labelled in their words — "Google Cloud console",
not "Console". Open every one before writing it down.

## 7. Add the row

In `.agents/plugins/marketplace.json`: `name`, `displayName`, `description`,
`author`, `category`, `source` (`./mcp/<name>`), `homepage`. The description
must match the manifest's exactly; a test checks it.

## 8. Prove it

```sh
cargo test -p ft-worker --test catalogue -- --ignored --nocapture
```

From a Firetower checkout with this repository beside it. It delivers every
entry to Claude Code and Codex and asks each what it got.

Then connect it in Firetower and use it, because none of the above proves the
tools are any good. `CONTRIBUTING.md` says what the pull request has to show.

## The four that have caught us

- **A server name with a dot** in `.mcp.json` takes Codex's entire config
  down — `[mcp_servers.cal.com]` is TOML for `com` nested under `cal`. Use a
  key with no dots.
- **Google needs `access_type=offline`** or the connection dies in an hour.
  Firetower sends it; you do not have to, but know that is why.
- **A one-click server can stop being one.** Figma advertises registration
  and refuses it. Re-probe before trusting an old classification.
- **`tools/list` is not the job.** Forty tools that need ids nobody has is
  not a working entry.
