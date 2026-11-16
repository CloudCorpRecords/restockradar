# BUILD REPORT — restockradar

Clean-room rebuild of `com.dropwatchhq/drop-index` v1.0.0
(remote-only MCP server at `https://index.dropwatchhq.com/mcp`),
built 2026-10-28 as a forwarding client. Package:
`@reneturcios/restockradar` 0.1.0, registry name
`io.github.CloudCorpRecords/restockradar`.

## Surface built

Full surface: **all 3 captured tools built**, no omissions.

| Tool | Required fields as served |
|------|---------------------------|
| `list_niches` | NO `required` key in the capture — served verbatim, none invented |
| `drop_index` | `niche` |
| `will_it_restock` | `niche`, `query` |

Each served tool carries exactly the keys `name`, `description`,
`inputSchema`, exactly as captured: the capture carries NO `title`
keys, so none are served. The tool literals were generated
programmatically from
`harness/fixtures/dropindex-tools-original.json` by the script at
`/tmp/b46-restockradar-scratch/gen_forward.py` (parse + re-serialise
into `src/forward.ts`) — never retyped — and a post-generation check
(a Python deep-equality comparison of the served literals against
the fixture, the registration-integrity test suite, plus the parity
harness) confirmed the served literals deep-equal the fixture's 3
tool objects, in order. Tools are registered via the MCP SDK
low-level Server API (`ListToolsRequestSchema` /
`CallToolRequestSchema` handlers), not McpServer+zod.

## Fixture conflicts

**None.** The spec and the fixture agree on all 3 tools' names,
descriptions, required fields (including `list_niches` with no
`required` key) and property shapes. The spec's note that the
capture carries no `title` keys matches the fixture exactly.

## Labelled reconstructions (from the spec's Wire section)

- Upstream endpoint pinned: `https://index.dropwatchhq.com/mcp`.
- Env override `DROPINDEX_MCP_URL` — labelled reconstruction.
- The registry record declares NO headers on the remote: no auth
  header is ever sent. Tests assert that no `Authorization` header
  is sent under any environment — including with key-like env vars
  (`DROPINDEX_API_KEY`, `API_KEY`, `AUTHORIZATION`) set — and that
  no `X-API-Key` header is ever sent; upstream requests carry only
  `Content-Type`, `Accept` and, when issued, `Mcp-Session-Id`.
- Error rendering `restockradar error:` text for upstream
  failures — labelled reconstruction.
- Upstream results pass through verbatim, including `isError` and
  extra fields. tools/call is forwarded as streamable-HTTP
  JSON-RPC: initialize (protocolVersion `2025-03-26`) →
  notifications/initialized → tools/call, honouring
  `Mcp-Session-Id` when issued; plain-JSON and SSE replies are
  both parsed. The forwarding core takes an injectable fetch;
  tests stub the upstream — no live network calls in tests.
- `will_it_restock` judgment (from the spec, recorded by the
  coordinator on the firsthand capture read): its WAIT vs
  BUY-RESALE verdict is a prediction about a PRODUCT's restock
  likelihood from accrued cadence stats — shopping information
  about an item, not a purchase. The rebuild executes no
  transaction, holds no payment method and forwards the query
  only; no payment, metering, contact, signup or send tool
  exists in the surface.

## Deviations

None.

## Lockfile

- Route: generated clean from birth in the build dir with no
  `node_modules` and no pre-existing lockfile:
  `npm install --package-lock-only --ignore-scripts --offline`
  (Route 3).
- Parses as JSON; `name` == `@reneturcios/restockradar`.
- Package entries: **128**.
- sha256: `d4309bac77c029f06eb6e73cb6c2a573a0f6f17d68d5bfdfcfc2a8724c4aa77f`
  (re-verified unchanged after the subsequent offline `npm ci`).

## server.json

- Description exactly: "Collectible restock index by niche, with
  per-item will-it-restock predictions." — **78 characters**.
- npm package entry carries `"version": "0.1.0"`; the OCI entry
  (`ghcr.io/cloudcorprecords/restockradar:0.1.0`) carries **no**
  `version` field. Both entries carry packageArguments
  `[{type: positional, value: mcp}]` and transport stdio. Dockerfile
  (node:20-alpine, both stages) carries the label
  `io.modelcontextprotocol.server.name="io.github.CloudCorpRecords/restockradar"`.
  Container workflow `IMAGE` is `ghcr.io/cloudcorprecords/restockradar`
  and its smoke test asserts the server name `restockradar`.

## Verification (all run 2026-10-28)

1. `npx tsc --noEmit` → exit 0.
2. `npm run build` → exit 0; real `dist/` produced (cli, forward,
   index, mcp, types — .js, .d.ts, .js.map each).
3. `node dist/cli.js --version` → `0.1.0`.
4. `npm test` → **54 tests, 54 pass, 0 fail** (registration-integrity
   suite deep-equals the served tools/list against the fixture's 3
   tools, including the no-title and no-required assertions, plus
   session handling/reuse, env override, the
   no-Authorization-header assertions under any environment,
   verbatim argument forwarding and result pass-through including
   upstream isError, SSE parsing, error rendering, MCP-layer
   validation, and CLI behaviour).
5. Parity: `python3 /home/hatch/workspace/mcp-rebuild-100/harness/parity.py --original /home/hatch/workspace/mcp-rebuild-100/harness/fixtures/dropindex-tools-original.json --rebuild "stdio:node /home/hatch/workspace/mcp-rebuild-100/builds/restockradar/dist/cli.js mcp" --out /home/hatch/workspace/mcp-rebuild-100/builds/restockradar/parity-out` → **verdict PASS, parity 100.0**
   (3/3 identical, 0 missing, 0 extra, no schema diffs). Output in
   `parity-out/`.
