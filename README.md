# @pipeworx/it-admin-courts

Search and read the published rulings of Italy's administrative courts (Consiglio di Stato and the regional TAR tribunals): public-procurement awards, planning and permits, licensing, anti-mafia certification appeals, public-sector disputes.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1856+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Tools

- `search_rulings(all_words?, phrase?, exclude_words?, type?, seat?, limit?)` — find rulings by words or exact phrase, narrowed by type (sentenza, ordinanza, decreto, parere) and court seat (e.g. Brescia). Returns type, seat, section, number, ECLI, case number, a snippet and a link to the record.
- `get_ruling(schema, case_key, file, max_chars?)` — full text of one ruling found by search (rulings the court publishes only as PDF return a link instead).

## Auth

Keyless.

## Data sources

- Giustizia Amministrativa — decisions and opinions of the Consiglio di Stato and the TAR. Called live per request; nothing is stored. Covers administrative justice only: not civil or criminal courts, and not the company register (Registro Imprese is a paid, account-based service and is not included). The court itself pseudonymises some personal details ("OMISSIS") and responses pass the published text through unchanged.

Italian search terms work best. Results are the first page only; narrow with phrase, type or seat.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "it-admin-courts": {
      "url": "https://gateway.pipeworx.io/it-admin-courts/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/it-admin-courts/mcp` returns the tools in the table
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

Both URLs reach the same gateway and the same 1856+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/search_rulings \
  -H 'Content-Type: application/json' \
  -d '{"all_words":"appalto","seat":"Brescia","type":"sentenza","limit":20}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/search_rulings`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Run it from npm

`@pipeworx/mcp-it-admin-courts` is a local stdio bridge to the Pipeworx gateway: your MCP client
starts it with `npx`, and it forwards every call to the same
`https://gateway.pipeworx.io/it-admin-courts/mcp` endpoint shown above. It needs network access;
there is no offline mode.

```json
{
  "mcpServers": {
    "it-admin-courts": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-it-admin-courts"]
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
ask_pipeworx({ question: "your question about It Admin Courts data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
