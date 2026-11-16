# restockradar

Is that sold-out card, kit or pressing coming back, or is the resale premium the only way in? restockradar is a local forwarding client for the hosted Drop Index MCP service: it answers `tools/list` locally from the captured tool definitions and forwards every call to the service, returning the answers exactly as the service sends them back.

All three tools are reads over published collectible restock data. One lists the niches the index covers, one returns the live restock index for a niche — what is restocking now, what dropped recently, and what is running low across the indie shops being polled — and one predicts, for a single item, whether it is likely to restock, from whatever restock-cadence history has accrued. That prediction is shopping information about an item: restockradar executes no transaction, holds no payment method, and forwards the query only.

All 3 captured tools are built; there are no omissions.

## The tools

- **`list_niches`** — The collectible niches the index covers. Call this first to pick a valid `niche` value. No required parameters.
- **`drop_index`** — The live restock index for one niche: tracked items restocking now, recently dropped, or low on stock, across the indie shops being polled. `niche` is required (one of `tcg`, `lego`, `funko`, `keebs`, `vinyl`, `aqua`).
- **`will_it_restock`** — A per-item restock prediction: accrued cadence stats, a probability, and a WAIT versus BUY-RESALE verdict — or an honest "no history yet" when there is nothing to go on. `niche` and `query` (the item name) are both required.

## Install and run

As an MCP server via npm:

```bash
npx -y @reneturcios/restockradar mcp
```

MCP client configuration:

```json
{
  "mcpServers": {
    "restockradar": {
      "command": "npx",
      "args": ["-y", "@reneturcios/restockradar", "mcp"]
    }
  }
}
```

As a container:

```bash
docker run -i ghcr.io/cloudcorprecords/restockradar
docker run --rm ghcr.io/cloudcorprecords/restockradar drop_index '{"niche":"tcg"}'
```

## CLI

One binary serves both the command line and the MCP server. Each tool is a subcommand taking a single JSON string with that tool's arguments:

```bash
restockradar list_niches '{}'
restockradar drop_index '{"niche":"lego"}'
restockradar will_it_restock '{"niche":"tcg","query":"Cynthia'"'"'s Garchomp"}'
restockradar --version
```

Exit codes: `0` when the forwarded call succeeds, `1` when the upstream result is an error, and `2` for usage problems such as an unknown subcommand or a malformed JSON argument.

## Configuration

- `DROPINDEX_MCP_URL` — Overrides the upstream endpoint. The default is the pinned hosted endpoint, `https://index.dropwatchhq.com/mcp`. (The override variable is a reconstruction added by this rebuild.)

The upstream registry record declares no headers, so no authentication header is ever sent; the tool surface was captured keyless.

Upstream failures — a JSON-RPC error from the service, a non-2xx response, an unreadable body, or a request that fails outright — are reported to the caller as an error result whose text starts with `restockradar error:`.

## How forwarding works

The first forwarded call opens a session: an `initialize` request (protocol version `2025-03-26`), followed by a `notifications/initialized` notification, then the `tools/call` itself. If the service answers the initialize request with an `Mcp-Session-Id` header, that value is carried on every later request, and the open session is reused for the rest of the process. Replies are accepted either as plain JSON or as server-sent events. Successful results pass through verbatim, including any `isError` flag and any extra fields the service includes.

## Development

```bash
npm install
npm run build   # tsc -> dist/ (dist/cli.js is the bin)
npm test        # tsx --test test/*.test.ts — stubbed upstream, no network
npm run dev     # run the CLI from source
```

The test suite checks the served tool list against the captured surface (all 3 tools, deep-equal, in order, with no titles invented and no `required` key added to the tool that carries none), exercises the session handshake and its reuse, verifies that no `Authorization` header is ever sent under any environment, and verifies that arguments go upstream untouched and results come back untouched.

## License

MIT — see `LICENSE`. Copyright (c) 2026 Rene Turcios.
