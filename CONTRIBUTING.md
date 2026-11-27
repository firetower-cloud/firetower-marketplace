# Adding something to this marketplace

**Working out whether a server can be added at all is
[docs/adding-a-server.md](docs/adding-a-server.md)** — the probe, the folder,
and the four things that have caught us. This file is what the pull request
has to show once you have one.

Open a pull request. The folder is the easy part; the review is about whether
anybody should install it.

## What the pull request has to show

**What it is for.** One or two sentences on what somebody gets. Not "Linear
integration" — what an agent can now do that it could not before, in the words
somebody would use asking for it.

**The cases you expect people to use it for**, each written as something a
person would actually type at an agent. "Move the ticket I just fixed to In
Review and write what changed." Three or four is plenty. One is not enough to
tell whether the thing is useful or merely connected.

**A screenshot of each case working in Firetower**, on Claude Code or Codex.
The agent's own output, in a real session, with the tools being called. Not the
vendor's marketing, not a terminal running `curl`, not the connect screen.

That last one is the whole review. An MCP server that handshakes, authorizes
and lists forty tools can still be useless — the tools may need ids nobody has,
or return a shape the model cannot use, or quietly fail on an empty account.
A screenshot of it doing the job is the only evidence that survives contact
with a person. If you cannot produce one for a case, drop the case; if you
cannot produce one at all, the entry is not ready.

## The folder

```
mcp/<name>/
  .claude-plugin/plugin.json     name, displayName, description, version, author, homepage
  .codex-plugin/plugin.json      the same bytes
  .mcp.json                      the server, and what it asks for
  assets/logo.png                square, 180px or better, under 96KB
```

`skills/<name>/` for an entry whose point is instructions rather than a server.
Then add it to `.agents/plugins/marketplace.json` with that path.

Four things get this wrong often enough to list:

- **Both manifests, byte for byte**, and `name` equal to the folder's name. An
  engine that reads one and not the other has to see the same entry.
- **`displayName` is not optional in practice.** `name` is a namespacing key —
  lowercase, hyphens — and without a display name the library draws `cal-com`
  at people.
- **The logo travels inside the entry.** It is stored with the bundle rather
  than fetched, so a row draws without reaching anybody's server.
- **Say what it reaches, not what it is.** The description is read by somebody
  deciding whether to install it, and by an agent deciding whether to use it.

## Scopes, and asking for less

An entry may carry a `scopes` list, and it can only ever ask for **less** than
the server advertises — anything not on the server's own list is dropped.

Use it. Gmail advertises twenty scopes including the whole mailbox and delete,
where reading, composing and sending is the job. Google also refuses an entire
authorization over one scope that is not on the consent screen somebody
configured, so narrowing is often what makes an entry work at all rather than
merely what makes it polite.

## Servers that will not hand out a client

Some publish their OAuth metadata and register no clients — GitHub, Slack,
every Google one. Firetower has a form for pasting an application somebody
creates once, and your entry should say where to create it:

```json
"setup": [
  { "label": "Google Cloud console", "url": "https://console.cloud.google.com/auth/clients" },
  { "label": "Google's MCP setup guide", "url": "https://developers.google.com/workspace/guides/configure-mcp-servers" }
]
```

The labels are yours and so is the number of them — one page, two, or five.
**Open every link before you write it down.** A dead link on the one screen
somebody reaches when they are already stuck is worse than no link.

## What gets turned down

- An entry nobody has run. See the screenshots above.
- A skill that repeats what the tool list already says. A connector gives the
  agent tools with names and descriptions; instructions on top are worth the
  context they cost only when they encode something the tool list does not.
- A server whose address answers but publishes no RFC 9728 metadata. There is
  nothing to discover, so nothing can authorize.
- A manifest that asks to reuse a credential granted for something else.

## Before you open it

```sh
cargo test -p ft-worker --test catalogue -- --ignored --nocapture
```

From a Firetower checkout with this repository beside it. It hands every entry
to Claude Code and to Codex and asks each what it got, and it checks what no
engine can: that the manifests match, that the catalogue row and the folder
agree in both directions, and that there is exactly one server behind a row
promising one.
