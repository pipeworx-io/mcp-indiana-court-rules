# @pipeworx/indiana-court-rules

Indiana's statewide court rules by citation (Trial Rule 60(B), Appellate Rule 9, Evidence Rule 401, Professional Conduct 1.6 and the rest of the Indiana Rules of Court), plus where each county publishes its local court rules.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1881+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Tools

- `in_court_rule(set, rule)` — full text of one rule: heading, effective date, text with its subdivisions, and the published version history where there is one. A subdivision such as `60(B)` also returns just that part. Answers "what does Trial Rule 60(B) say, verbatim".
- `in_court_rules_list(set?)` — every rule in a set with its number and title (to find the right citation), or, with no set, all 24 rule sets with their rule counts.
- `in_local_rules(county?)` — the current local-rules documents a county's courts publish (county local rules, district governance plan, city-court rules), as links. With no county, the 92 counties.

## Auth

Keyless.

Local rules are published by each court as documents, so `in_local_rules` returns the document links and does not extract their text. Statewide rules are returned as text.

## Data sources

- Indiana Supreme Court, Indiana Rules of Court — the current version of each statewide rule.
- Indiana Supreme Court, Indiana Judicial Branch — each county's local court rules page.

Every call goes to the publisher; nothing is stored. The rule index is held in memory for a day.

Not the Indiana Code (statutes): use `indiana-code`.

## Notes

- The version history table only exists on some rule pages; where the publisher gives none, `version_history` is null and `effective` is the only date.
- Rule numbers use a dot for decimals (`2.1`, `60.5`). Set names such as "trial rules" and "professional conduct" are accepted as well as the ids.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "indiana-court-rules": {
      "url": "https://gateway.pipeworx.io/indiana-court-rules/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/indiana-court-rules/mcp` returns the tools in the table
above **plus the shared Pipeworx meta-tools** — `ask_pipeworx`,
`discover_tools`, `search_within`, `remember`/`recall` and the rest of the
gateway-wide set. So the tool count you see is larger than this table: a
single-pack endpoint currently lists roughly 30 shared tools alongside the
pack's own. The connection's `initialize` response states its exact scope, and
is the authoritative answer for a given day.

This is deliberate, not multiplexing by accident. The meta-tools are what let a
scoped connection answer a question this pack does not cover — via
`ask_pipeworx`, which routes across the whole catalog — without you adding a
second MCP server. There is currently no way to mount a pack endpoint without
them; if the extra schemas cost you more context than the routing is worth,
connect to the full gateway once rather than to several pack endpoints.

Or connect to the full Pipeworx gateway to get every pack's tools listed
directly, instead of just this one's:

```json
{
  "mcpServers": {
    "pipeworx": {
      "url": "https://gateway.pipeworx.io/mcp"
    }
  }
}
```

Both URLs reach the same gateway and the same 1881+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/in_court_rule \
  -H 'Content-Type: application/json' \
  -d '{"set":"trial","rule":"60(B)"}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/in_court_rule`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Standalone (no gateway account)

This package also runs as a local stdio MCP server — no Pipeworx account, no
gateway round-trip:

```json
{
  "mcpServers": {
    "indiana-court-rules": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-indiana-court-rules"]
    }
  }
}
```

Or run it directly to confirm it starts:

```bash
npx -y @pipeworx/mcp-indiana-court-rules
```

It speaks MCP over stdin/stdout and answers `initialize`/`tools/list`/`tools/call`
for **only** this pack's tools — none of the shared meta-tools the gateway
connection above adds. Same source, same tools, no ask_pipeworx routing.

## Using with ask_pipeworx

Instead of calling tools directly, you can ask questions in plain English —
this works on the pack endpoint above as well as on the full gateway:

```
ask_pipeworx({ question: "your question about Indiana Court Rules data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
