# @pipeworx/indiana-court-rules

Indiana's statewide court rules by citation (Trial Rule 60(B), Appellate Rule 9, Evidence Rule 401, Professional Conduct 1.6 and the rest of the Indiana Rules of Court), plus the text of each county's local court rules.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1865+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Tools

- `in_court_rule(set, rule)` — full text of one rule: heading, effective date, text with its subdivisions, and the published version history where there is one. A subdivision such as `60(B)` also returns just that part. Answers "what does Trial Rule 60(B) say, verbatim".
- `in_court_rules_list(set?)` — every rule in a set with its number and title (to find the right citation), or, with no set, all 24 rule sets with their rule counts.
- `in_local_rules(county?, query?, rule?, document?, offset?)` — the text of a county's local rules (circuit/superior court rules, district governance plan, city-court rules). `query` returns the best-matching passages ("filing motions"), `rule` returns one rule by the county's own citation ("LR41-TR-102"), and neither reads a document from the start. Every response lists the county's documents with their links, and carries `data_as_of`. A document with no machine-readable text (a scanned image, or one that cannot be parsed) is reported as such with its link, never as an empty answer. With no county, returns all 92 counties with their coverage.

## Auth

Keyless.

Statewide rules are returned live from the publisher. County local rules are published by each court as documents; their text is extracted into a stored copy, so `in_local_rules` answers from that copy and says when it was taken.

## Data sources

- Indiana Supreme Court, Indiana Rules of Court — the current version of each statewide rule.
- Indiana Supreme Court, Indiana Judicial Branch — each county's local court rules page.

Statewide rule calls go to the publisher; the rule index is held in memory for a day. County local rules are a stored copy of the publisher's documents (public Indiana court material), refreshed by re-running `scripts/bake-local-rules.mjs` (needs poppler's text extractor) and committing the regenerated module; the deploy loads it into KV, not the bundle. Proposed (not yet in effect) local rules are not included.

Coverage at capture (2026-10-26): all 92 counties have at least one document with extractable text. 13 of the 184 listed documents (mostly district governance plans published as scanned images, plus one small city-court document and one appendix file) have no text layer; those are reported with their link.

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

Both URLs reach the same gateway and the same 1865+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/in_court_rule \
  -H 'Content-Type: application/json' \
  -d '{"set":"trial","rule":"60(B)"}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/in_court_rule`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Run it from npm

`@pipeworx/mcp-indiana-court-rules` is a local stdio bridge to the Pipeworx gateway: your MCP client
starts it with `npx`, and it forwards every call to the same
`https://gateway.pipeworx.io/indiana-court-rules/mcp` endpoint shown above. It needs network access;
there is no offline mode.

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

Without a key it calls anonymously, capped at 50 calls a day. A free account
raises that to 200 a day: get a key at https://pipeworx.io/signup and add
`"env": { "PIPEWORX_API_KEY": "<your key>" }` to the server entry.
The key is sent as a Bearer token, the same way a hosted connection sends it,
so calls are metered and billed exactly as they are there.

It lists **only** this pack's tools by default. Set `PIPEWORX_META_TOOLS=1` to
also list `ask_pipeworx` and the other gateway-wide tools.

Behind an HTTPS proxy: Node's built-in `fetch` does not read `HTTPS_PROXY` by
default. Newer Node releases honour it when `NODE_USE_ENV_PROXY=1` is set; check
your Node version's documentation.

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
