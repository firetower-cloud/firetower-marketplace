# Firetower plugins

The plugins Firetower publishes. Add this repository as a marketplace and its
plugins appear in your library, ready to turn on.

```
https://github.com/firetower-cloud/firetower-plugins
```

Nothing here is specific to Firetower. The layout is the one Claude Code and
Codex already read, so a folder in this repository works in all three — which
is the only way we find out it is broken before you do.

## How it is laid out

```
mcp/                           entries whose point is a server to call
  linear/
    .claude-plugin/plugin.json   name, description, version, author
    .codex-plugin/plugin.json    the same bytes, so Codex finds it too
    .mcp.json                    the server, and the address it calls
    assets/logo.png              the mark shown on the row
    skills/<name>/SKILL.md       instructions, where they earn their place
skills/                        entries whose point is instructions to read
```

**The split is for people, not for parsers.** An entry is the same shape in
either directory; `marketplace.json` names its path and nothing else cares.
An entry carrying both goes where its point is — `mcp/github` ships a skill
and still lives under `mcp/`, because somebody adding it wants the tools.

The manifest is a label. Everything an agent actually reads is the body —
`skills/`, `commands/`, `agents/` and `.mcp.json` — and that body is identical
whichever engine runs it. Firetower hands the folder to Claude Code as a
plugin directory, and takes it apart for Codex, Kimi and Cursor, which read
skills and MCP servers from their own places.

## What is here

| Entry | Connects to | Authorizing |
| --- | --- | --- |
| `mcp/figma` | Figma files, components and variables | one click |
| `mcp/github` | issues, pull requests, Actions, code search | a client id, created once |
| `mcp/higgsfield` | image, video and voice generation | one click |
| `mcp/linear` | issues, projects and cycles | one click |
| `mcp/notion` | pages and databases | one click |
| `mcp/zernio` | social, ads, inbox, WhatsApp, SMS, calls | one click |

Six servers and one skill between them. That is an MCP directory, not yet a
marketplace — see `skills/README.md` for what is missing and why it matters.

"One click" means the server registers Firetower itself, so nobody creates an
OAuth application and nobody pastes a client id. You press Connect, you
authorize in your browser, and the agent has the tools. `github` is the
exception: GitHub publishes its metadata but registers no clients, so it
wants one made by hand.

**Skills are the exception, not the rule.** A connector already gives the
agent tools with names and descriptions. A skill on top is only worth the
context it costs when it encodes something the tool list does not say — so
most plugins here ship none, and `github` ships one because opening a decent
pull request is a judgement, not an API call.

## Adding a plugin

Make the folder under `mcp/` or `skills/`, add it to
`.claude-plugin/marketplace.json` with that path, and open a pull request.
Two things to get right:

**The logo travels inside the plugin.** Put it at `assets/logo.png` — a
square, 180px or better. It is stored with the bundle rather than fetched, so
a row draws without reaching anybody's server and never shows a broken image.

**Say what it does, not what it is.** "Read a ticket, move it, and write back
what the agent actually did" tells somebody whether they want it. "Linear
integration" does not.

### Authorizing

Every entry authorizes for itself, per person. A manifest does not get to
say which credential it would like — reusing a token somebody granted for
something else is the one thing a repository nobody here wrote must not be
able to ask for.

Most servers register a client automatically, so Connect is one press.
`github`, `slack` and Google's do not, and need a client id created once and
pasted in; that is their choice, not a property of this repository.
