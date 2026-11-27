# Firetower marketplace

The skills and MCP servers Firetower publishes. Add this repository as a
marketplace and everything in it appears in your library, ready to turn on.

```
https://github.com/firetower-cloud/firetower-marketplace
```

Nothing here is specific to Firetower. The layout is the one Claude Code and
Codex already read, so a folder in this repository works in all of them —
which is the only way we find out it is broken before you do.

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
An entry carrying both would go where its point is; none does today.

The manifest is a label. Everything an agent actually reads is the body —
`skills/`, `commands/`, `agents/` and `.mcp.json` — and that body is identical
whichever engine runs it. Firetower hands the folder to Claude Code as a
plugin directory, and takes it apart for Codex, Kimi and Cursor, which read
skills and MCP servers from their own places.

## What is here

Seventy-nine servers and no skills. That is an MCP directory, not yet a
marketplace — see `skills/README.md` for what is missing and why it matters.

**"One click" means the server registers Firetower itself**, so nobody creates
an OAuth application and nobody pastes a client id: you press Connect, you
authorize in your browser, and the agent has the tools. Fifty-nine of these do
that. Nineteen want an application made by hand once — that is their choice,
not a property of this repository, and Firetower has a form for it, with a
link to the vendor's console and to their own instructions. One needs nothing
at all.

Every row below was measured against the live endpoint, not read off a
documentation page. **Including the registration itself**, which is the part
that is easy to get wrong: reading `registration_endpoint` out of a server's
metadata says it *offers* to hand out clients, not that it will. Figma offers
and answers `403`. `tools/probe-registration.py` in the Firetower repository
posts a real registration to every entry here.

### What an entry says about authorizing

Beside `url`, a server in `.mcp.json` may carry three things. **None of them
has a fallback compiled into Firetower** — a server nobody told us about shows
no link rather than a guess, which is the whole reason these live here:

| key | what it is |
| --- | --- |
| `scopes` | what to ask for, narrowed (see below) |
| `setup` | where somebody goes to come back with a client id |

```json
"setup": [
  { "label": "Google Cloud console", "url": "https://console.cloud.google.com/auth/clients" },
  { "label": "Google's MCP setup guide", "url": "https://developers.google.com/workspace/guides/configure-mcp-servers" }
]
```

`setup` appears on the form Firetower shows when a server will not hand out a
client of its own. **The labels are yours, and so is the number of them.**
Google splits the job across a Cloud console and a twelve-step guide; Twilio
has one page; somebody else might have three. An earlier version of this had
two fields called `console` and `guide`, which was Google's shape read as a
schema — every other vendor got Google's vocabulary and a null, and the screen
said "Console" at people whose vendor calls it a developer portal.

Links need no ceiling the way scopes do: a link grants nothing.

#### Scopes

An entry may carry a `scopes` list in its `.mcp.json`, and it can only ask for
**less** than the server advertises — anything not on the server's own list is
dropped. That exists for the lists that are too big to request honestly:
Gmail advertises twenty scopes including the whole account and delete, where
reading, composing and sending is the job. Google also refuses an entire
authorization over one scope that is not on the consent screen somebody
configured, so narrowing is what makes it work at all rather than merely
polite.

### Automation and glue

| Entry | What it reaches | Authorizing |
| --- | --- | --- |
| `mcp/make` | Scenarios and the modules inside them, run and inspected | one click |
| `mcp/pipedream` | Workflows and the apps they connect, called as tools | one click |
| `mcp/zapier` | Thousands of apps as tools, through the automation hub itself | one click |

### Email and messaging

| Entry | What it reaches | Authorizing |
| --- | --- | --- |
| `mcp/customer-io` | Campaigns, segments and the messages a person was actually sent | one click |
| `mcp/discord` | Servers, channels and message history, searched and read | a client id, created once |
| `mcp/gmail` | Read a mailbox, search it, draft and send — not delete, and not settings | a client id, created once |
| `mcp/google-chat` | Spaces, memberships and messages, read and posted to | a client id, created once |
| `mcp/loops` | Lifecycle email — contacts, campaigns and the events behind them | a client id, created once |
| `mcp/resend` | Send transactional email and read back what happened to it | one click |
| `mcp/slack` | Read a channel, post, and search your workspace history | a client id, created once |
| `mcp/twilio` | SMS, voice calls and WhatsApp messages, sent and looked up | a client id, created once |
| `mcp/vapi` | Voice agents that place and take calls, and the transcripts after | one click |

### Reading the web

| Entry | What it reaches | Authorizing |
| --- | --- | --- |
| `mcp/apify` | Ready-made scrapers, run and their results read back | one click |
| `mcp/brightdata` | Web data collection that gets through — proxies, unblocking, feeds | one click |
| `mcp/exa` | Search that returns page contents rather than a list of links | one click |
| `mcp/firecrawl` | Scrape a page, crawl a site, and get markdown an agent can actually read | nothing at all |
| `mcp/hyperbrowser` | Headless browsers that run a page and hand back what it rendered | one click |
| `mcp/scrapingbee` | Fetch a rendered page without being blocked | one click |
| `mcp/steel` | Browser sessions an agent drives, with the page state back | a client id, created once |
| `mcp/tavily` | Search and extraction shaped for an agent to act on | one click |

### Records, forms and CRM

| Entry | What it reaches | Authorizing |
| --- | --- | --- |
| `mcp/airtable` | Bases, tables and records — the database most operations run on | one click |
| `mcp/attio` | People, companies and deals, read and written as records | one click |
| `mcp/cal-com` | Bookings, availability and event types, read and scheduled | one click |
| `mcp/front` | Shared inboxes, conversations and the drafts waiting in them | a client id, created once |
| `mcp/google-calendar` | Events, availability and invitations, read and scheduled | a client id, created once |
| `mcp/google-docs` | Documents read, drafted and edited in place | a client id, created once |
| `mcp/google-sheets` | Spreadsheets, ranges and formulas, read and written back | a client id, created once |
| `mcp/hubspot` | Contacts, companies and deals, as context for every answer | a client id, created once |
| `mcp/intercom` | Conversations, contacts and help-centre articles | one click |
| `mcp/notion` | Search Notion, read pages and databases, and write back to them | one click |
| `mcp/tally` | Forms and the responses submitted to them | one click |

### Social and marketing

| Entry | What it reaches | Authorizing |
| --- | --- | --- |
| `mcp/buffer` | Queue a post to every channel and read how the last ones did | one click |
| `mcp/typefully` | Draft, schedule and measure posts across social accounts | one click |
| `mcp/zernio` | Marketing for agents — social, ads, inbox, WhatsApp, SMS and calls behind one connection | one click |

### Money

| Entry | What it reaches | Authorizing |
| --- | --- | --- |
| `mcp/mercury` | Accounts, transactions and payments for a business's banking | one click |
| `mcp/paypal` | Invoices, orders, payments and refunds | one click |
| `mcp/polar` | Products, subscriptions and checkouts for software being sold | one click |
| `mcp/ramp` | Cards, spend, bills and the receipts attached to them | one click |
| `mcp/square` | Payments, catalogue, orders and customers of a storefront | one click |
| `mcp/stripe` | Customers, subscriptions, invoices and payments | one click |

### Shipping software

| Entry | What it reaches | Authorizing |
| --- | --- | --- |
| `mcp/asana` | Tasks, projects and portfolios — read the work and write back what was done | a client id, created once |
| `mcp/buildkite` | Pipelines, builds and the logs of a job that failed | one click |
| `mcp/circleci` | Pipelines, workflows and the test output of a failing job | one click |
| `mcp/cloudflare` | Workers, KV, R2, D1 and the bindings between them | one click |
| `mcp/context7` | Current documentation for a library, at the version in use | one click |
| `mcp/fly-io` | Apps, machines and volumes, across regions | one click |
| `mcp/github` | GitHub issues, pull requests, Actions and code search, as tools the agent can call | a client id, created once |
| `mcp/google-tasks` | Task lists and the things on them, read and ticked off | a client id, created once |
| `mcp/linear` | Linear issues, projects and cycles — read a ticket, move it, and write back what the agent actually did | one click |
| `mcp/monday` | Boards, items and columns of the work tracked on them | one click |
| `mcp/netlify` | Sites, deploys and environment variables, as tools the agent can call | one click |
| `mcp/railway` | Projects, services, deployments and the logs of the last one | one click |
| `mcp/render` | Services, deploys and the logs of the one that failed | a client id, created once |
| `mcp/shortcut` | Stories, epics and iterations of a team's work | one click |
| `mcp/vercel` | Projects, deployments and logs — read what shipped and why a build failed | one click |

### The stack underneath

| Entry | What it reaches | Authorizing |
| --- | --- | --- |
| `mcp/algolia` | Search indices, records and the analytics behind a query | one click |
| `mcp/clerk` | Users, organisations and sessions of an application's sign-in | one click |
| `mcp/datadog` | Metrics, monitors, logs and traces, as an agent can query them | one click |
| `mcp/grafana` | Dashboards, the queries behind them, and the alerts that fired | one click |
| `mcp/neon` | Postgres with branches: open one for a change, query it, throw it away | one click |
| `mcp/posthog` | Events, funnels, feature flags and session replays | one click |
| `mcp/prisma` | Postgres databases, their schema and the migrations applied | one click |
| `mcp/sentry` | Find the issue, read the stacktrace, and the release that introduced it | one click |
| `mcp/supabase` | Postgres, auth and storage — query a branch, read a schema, check a policy | one click |
| `mcp/upstash` | Serverless Redis, Kafka and vector indexes | one click |
| `mcp/workos` | SSO, directory sync and the organisations behind enterprise sign-in | one click |

### Content and media

| Entry | What it reaches | Authorizing |
| --- | --- | --- |
| `mcp/canva` | Designs, folders and brand assets, read and created from a conversation | one click |
| `mcp/figma` | Read a Figma file as the agent builds the UI — frames, components, variables and styles | one click |
| `mcp/google-slides` | Presentations and the slides inside them, read and built | a client id, created once |
| `mcp/higgsfield` | Generate and edit images, video and voice with Higgsfield's creative suite | one click |
| `mcp/mintlify` | A documentation site's pages, searched and updated | one click |
| `mcp/mux` | Video assets, uploads and the playback metrics of each one | one click |
| `mcp/sanity` | Documents, datasets and the schema of a content lake | one click |
| `mcp/webflow` | Sites, collections and CMS items, published from a conversation | one click |

### Google Workspace

| Entry | What it reaches | Authorizing |
| --- | --- | --- |
| `mcp/google-drive` | Files, folders and shared drives — find them, read them, create and move them | a client id, created once |

### Models

| Entry | What it reaches | Authorizing |
| --- | --- | --- |
| `mcp/elevenlabs` | Speech, voice cloning and dubbing, with the audio back | a client id, created once |
| `mcp/fal` | Run image, video and audio models and get the file back | one click |
| `mcp/hugging-face` | Models, datasets and spaces, searched and read | one click |
| `mcp/replicate` | Run any published model and collect its output | one click |

## Adding an entry

**`CONTRIBUTING.md` is the one to read** — it says what a pull request has to
show, which is mostly evidence that somebody ran the thing. The rest of this
section is the shape of the folder.


Make the folder under `mcp/` or `skills/`, add it to
`.claude-plugin/marketplace.json` with that path, and open a pull request.
Three things to get right:

**Both manifests, byte for byte.** `.claude-plugin/plugin.json` and
`.codex-plugin/plugin.json` hold the same bytes, and the `name` in them is the
folder's name. An engine that reads one and not the other has to see the same
entry.

**The logo travels inside the entry.** Put it at `assets/logo.png` — square,
180px or better, under 96KB. It is stored with the bundle rather than fetched,
so a row draws without reaching anybody's server and never shows a broken
image.

**Say what it reaches, not what it is.** "Read a ticket, move it, and write
back what the agent actually did" tells somebody whether they want it. "Linear
integration" does not.

### How it is checked

Firetower's own test suite reads this repository and delivers every entry to
both engines, then asks each engine what it got:

```
cargo test -p ft-worker --test catalogue -- --ignored --nocapture
```

Claude Code has to report every entry as a loaded plugin and a registered
server in its `init` frame; Codex has to list every server as
`streamable_http` at the right address with the credential recognised as one.
It also checks what no engine can: that the manifests match, that the
catalogue row and the folder agree in both directions, and that there is
exactly one server behind a row that promises one.

**Skills are the exception, not the rule.** A connector already gives the
agent tools with names and descriptions. A skill on top is only worth the
context it costs when it encodes something the tool list does not say — so no
entry here ships one.

### Authorizing

Every entry authorizes for itself, per person. A manifest does not get to say
which credential it would like — reusing a token somebody granted for
something else is the one thing a repository nobody here wrote must not be
able to ask for.
