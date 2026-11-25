# Skills

Entries that are instructions rather than a connection.

A server's tool names say *what* it can do. They do not say *how* — which
project board the team actually uses, what a good triage note looks like, how
to map a Figma frame onto a design system that already exists. That is what
belongs here, and it is the only thing a curated list adds that pasting an
address does not.

| | |
| --- | --- |
| `security-review` | read a pending change for bugs and holes, then prove a fix stops the one you found |
| `release-notes` | write the commit a changelog is cut from, and the note that goes out |

Both of these used to ship inside the Firetower binary. They came out for the
same reason `github` did: what Firetower compiles in is what Firetower itself
knows how to do, and everything else is chosen rather than arriving.

## Which directory

| | |
| --- | --- |
| `mcp/` | the entry's point is a server to call |
| `skills/` | the entry's point is instructions to read |

An entry carrying both goes where its point is. The layout is identical in
either; the split is for people reading the repository, not for anything that
parses it.
