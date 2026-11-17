# @pipeworx/dc-corporations

The District of Columbia corporate register: every corporation, LLC, limited partnership, nonprofit, cooperative and foreign registration the DC Corporations Division holds, active and dissolved, searchable by name, by file number and by registration date.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 2014+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Tools

- `dc_corp_search(name, match_mode?, status?, limit?)` -- is this company registered in DC, and is it in good standing. Matches on the normalised name, so Inc / Incorporated, LLC / L.L.C., Co / Company, a trailing full stop and `&` vs `and` are one name; each row carries `match` (`exact`, `legal_form_variant`, `spacing_variant`). `match_mode: "starts_with"` returns every name that begins with the text instead.
- `dc_corp_entity(file_number)` -- one registration: type, domestic or foreign, status, DC registration date, the date a foreign entity was organised elsewhere, business address, next annual-report year, last report filed.
- `dc_corp_new_entities(since, locale?, entity_type?, name_query?, limit?)` -- entities newly registered in DC since a date, newest first. Registrations dated in the future (delayed-effective filings) are left out.

Every response carries `data_as_of` (the register's latest record change), the publisher and dataset, and the licence and attribution line.

## Auth

Keyless.

## What is not served

Registered-agent names and addresses are never returned, because an agent can be an individual. No officers or other individuals' names are served either: the register publishes none, and the names-and-roles ruling applies to the sos-entities pack alone. A miss (`found: false`) says what was searched and what to try next, and an upstream that fails to answer is an error, never an empty list.

## Data sources

- DC Department of Licensing and Consumer Protection (DLCP), Corporations Division (the Office of Corporate Registrar), "Corporate Registration" open data, licensed Creative Commons Attribution 4.0. Contains all statuses (active, dissolved, revoked, withdrawn, merged and others), about half a million records.

Things the next person would otherwise rediscover:

- Dates are stored as midnight Eastern, as epoch milliseconds. Reading them as UTC dates shifts every one by a day in some cases; the pack reads the Eastern calendar day.
- Foreign companies (formed elsewhere) are in the register only if they registered to do business in DC. A miss is not proof a company does not exist.
- The registration date for a foreign entity is its DC registration, not its formation; `organized_elsewhere_date` is the formation date where the register holds one.
- File numbers appear in two shapes: a plain number (`223996`) and a prefixed one (`US-DC-FN112602492`). Look up with the exact string.
- A cold first call to this host has timed out once and answered in under a second on retry, so each query makes two short attempts.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "dc-corporations": {
      "url": "https://gateway.pipeworx.io/dc-corporations/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/dc-corporations/mcp` returns the tools in the table
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

Both URLs reach the same gateway and the same 2014+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/dc_corp_search \
  -H 'Content-Type: application/json' \
  -d '{"name":"Booz Allen Hamilton Inc","status":"active","limit":3}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/dc_corp_search`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Run it from npm

`@pipeworx/mcp-dc-corporations` is a local stdio bridge to the Pipeworx gateway: your MCP client
starts it with `npx`, and it forwards every call to the same
`https://gateway.pipeworx.io/dc-corporations/mcp` endpoint shown above. It needs network access;
there is no offline mode.

```json
{
  "mcpServers": {
    "dc-corporations": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-dc-corporations"]
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
ask_pipeworx({ question: "your question about Dc Corporations data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
