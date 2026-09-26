# @pipeworx/uk-planning-data

UK planning designations from planning.data.gov.uk, the MHCLG national planning
data platform — look up any point in England and get the conservation areas,
listed buildings, article 4 directions, green belt, brownfield land, flood risk
zones and tree preservation zones that cover it.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1684+ live data sources.

## Tools

- `ukplanning_datasets(query?, typology?, headline_only?)` — the dataset
  catalogue: the slug you pass to every other tool, plus row counts and licence.
- `ukplanning_entities_at_point(latitude, longitude, dataset?, limit?, include_geometry?)`
  — every designation whose boundary covers one lat/lon. The site due-diligence
  call: "is this address in a conservation area / green belt / flood zone".
- `ukplanning_search(dataset?, typology?, reference?, curie?, organisation_entity?, entry_date_year?, limit?, offset?, include_geometry?)`
  — filter entities. Deliberately takes NO free-text query; see below.
- `ukplanning_entity(entity, include_geometry?)` — one entity by numeric id.

## Auth

Keyless.

## Data sources

- <https://www.planning.data.gov.uk/dataset.json> — the dataset catalogue.
- <https://www.planning.data.gov.uk/entity.json> — entity search and point lookup.
- <https://www.planning.data.gov.uk/entity/{entity}.json> — one entity.
- <https://www.planning.data.gov.uk/openapi.json> — the parameter list.

Things worth knowing:

- **`q` is documented and does nothing.** `?q=Trafalgar%20Square&limit=3`
  returns `count: 25355431` — the whole platform — and the same three rows as
  the unfiltered call. There is no free-text search on this API. This pack does
  not accept or forward one, because forwarding it returns 25 million rows
  wearing the shape of a result set. Use `ukplanning_entities_at_point` to
  search by place, and `dataset`/`curie`/`reference`/`organisation_entity` to
  filter. Those four genuinely filter (verified: `reference=CA18` → 31 rows).
- **Geometry is enormous.** Each row carries a WKT `MULTIPOLYGON`; one
  conservation area can be 60 KB. Every list call here sends
  `exclude_field=geometry&exclude_field=point` unless the caller asks for it.
- **Point lookup needs no relation parameter** — `?longitude=&latitude=`
  defaults to `intersects`. Omit `dataset` to get every layer at once.
- **`reference` is not unique.** It is the publisher's own id, so "CA18" matches
  31 councils. Pair it with `organisation_entity`, or use the `prefix:reference`
  curie.
- **Response fields are hyphenated, query parameters are underscored**
  (`entry-date` in the body, `entry_date_year` in the query). Both are upstream's.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "uk-planning-data": {
      "url": "https://gateway.pipeworx.io/uk-planning-data/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/uk-planning-data/mcp` returns the tools in the table
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

Both URLs reach the same gateway and the same 1684+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/ukplanning_datasets \
  -H 'Content-Type: application/json' \
  -d '{"query":"conservation"}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/ukplanning_datasets`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Standalone (no gateway account)

This package also runs as a local stdio MCP server — no Pipeworx account, no
gateway round-trip:

```json
{
  "mcpServers": {
    "uk-planning-data": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-uk-planning-data"]
    }
  }
}
```

Or run it directly to confirm it starts:

```bash
npx -y @pipeworx/mcp-uk-planning-data
```

It speaks MCP over stdin/stdout and answers `initialize`/`tools/list`/`tools/call`
for **only** this pack's tools — none of the shared meta-tools the gateway
connection above adds. Same source, same tools, no ask_pipeworx routing.

## Using with ask_pipeworx

Instead of calling tools directly, you can ask questions in plain English —
this works on the pack endpoint above as well as on the full gateway:

```
ask_pipeworx({ question: "your question about Uk Planning Data data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
