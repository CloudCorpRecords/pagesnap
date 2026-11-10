# BUILD REPORT — pagesnap

Rebuild of `com.i2dev/snap` v1.0.1 (remote-only)
as `@reneturcios/pagesnap` 0.1.0 / `io.github.CloudCorpRecords/pagesnap`.

## Fixture conflicts

**None.** The brief, the spec prose, and the fixture agree on the two
tool names and their required fields. Required fields as served,
taken from the fixture
(`harness/fixtures/i2dev-snap-tools-original.json`):

- `take_screenshot` — required: `url`
- `get_quota` — required: **none — the captured inputSchema carries NO
  `required` key**; served verbatim with no `required` key invented
  (asserted by a dedicated test: the served schema is exactly
  `{type: "object", properties: {}}`).

The fixture's tool literals (name, title, description, inputSchema —
including the `title` keys, the `$schema` key on `take_screenshot`,
and every property constraint) are served verbatim from
`src/forward.ts`; the literals were generated programmatically from
the fixture JSON (parsed and re-serialised, not retyped) and verified
by the registration-integrity test (served tools/list deep-equals the
fixture's 2 tool objects, in order) and again by the parity harness.

## What was built

All 19 factory files under `builds/pagesnap/`:

- `package.json` (`@reneturcios/pagesnap` 0.1.0, bin
  `pagesnap` → `dist/cli.js`, deps exactly `@modelcontextprotocol/sdk ^1`
  + `zod ^3`; devDeps `@types/node ^20`, `tsx ^4`, `typescript ^5`)
- `server.json` (description exactly the prescribed 86-char string
  "Screenshot any public web page as an image, and check your remaining screenshot quota.";
  npm entry carries `"version": "0.1.0"`; OCI entry
  `ghcr.io/cloudcorprecords/pagesnap:0.1.0` carries **no** version field)
- `Dockerfile` (node:20-alpine both stages; server-name label
  `io.modelcontextprotocol.server.name="io.github.CloudCorpRecords/pagesnap"`
  matches `server.json`)
- `tsconfig.json` (fleet-identical, 318 bytes), `.gitignore`,
  `package-lock.json`, `README.md` (fresh prose), `LICENSE` (MIT,
  Copyright (c) 2026 Rene Turcios), `CHANGELOG.md` (single 0.1.0 entry,
  2026-10-28)
- `src/index.ts`, `src/cli.ts`, `src/mcp.ts`, `src/types.ts`,
  `src/forward.ts` (core; exports `VERSION = "0.1.0"`)
- `test/forward.test.ts`
- `.github/workflows/{ci.yml, publish.yml, publish-container.yml,
  registry-publish.yml}` (container workflow: IMAGE
  `ghcr.io/cloudcorprecords/pagesnap`, smoke assertion on server name
  `pagesnap`)

Plus `BUILD-REPORT.md` (this file) and the `parity-out/` harness output.

## Design

- Forwarding client to `https://snap.i2dev.com/mcp`
  (pinned), overridable via `SNAP_MCP_URL` (labelled
  reconstruction — see below). tools/call is forwarded as a
  streamable-HTTP JSON-RPC client: initialize (protocolVersion
  `2025-03-26`, clientInfo `pagesnap`) → notifications/initialized →
  tools/call, honouring `Mcp-Session-Id` when issued; session
  established lazily and reused per client. Plain-JSON and SSE
  replies both parsed; the matching-id message wins; upstream
  results pass through verbatim (including `isError` and extra
  fields).
- When `SNAP_API_KEY` is set, an `Authorization: Bearer <key>` header
  is sent on every upstream request. When unset, no key header is
  sent. No `X-API-Key` header is sent in either case. Both branches
  are covered by tests.
- Upstream failures (JSON-RPC error, non-2xx status, malformed body,
  throwing/timing-out fetch) surface as `pagesnap error:` text results.
- Registration via the MCP SDK **low-level** `Server` API
  (`ListToolsRequestSchema` / `CallToolRequestSchema` handlers) serving
  the fixture literals exactly; no `McpServer` + zod registration
  (per the task brief, overriding TEMPLATE.md §2 item 12).
  `src/mcp.ts` also exports `createServer` / `startMcpServer` /
  `handleCallTool` so the MCP layer is testable without a transport;
  the CLI's `mcp` subcommand starts it over stdio.
- Core takes an injectable fetch (`ForwardDeps.fetchFn`), so all tests
  run against a stubbed JSON-RPC upstream. No live network calls in
  tests; no live call claimed anywhere beyond the coordinator's
  tools/list capture. The built surface screenshots a caller-supplied
  public URL and reads the caller's own remaining quota; no purchase,
  top-up or account tool exists in the surface — none was added.

## Labelled reconstructions

Per the spec, the following are this rebuild's own reconstructions,
not pinned by the original's record or capture:

- The `SNAP_MCP_URL` env override for the pinned upstream endpoint
  `https://snap.i2dev.com/mcp`.
- The env name `SNAP_API_KEY` (the record pins no package env name),
  the `Bearer` form, and the forwarding of the key from that env var
  (the header NAME `Authorization` is pinned by the registry record's
  remote declaration, which declares a secret `Authorization` header,
  not marked required; the capture was taken keyless).
- The `pagesnap error:` rendering for upstream failures.

## Lockfile (Route 3, clean from birth)

Generated in the build dir with no `node_modules` present and no
pre-existing lockfile:
`npm install --package-lock-only --ignore-scripts --offline`.
Verified: parses, `name == @reneturcios/pagesnap`,
**128 package entries**, sha256
`caeae2421eb8cf553fe9f5b7d1f46378ac26ed0feaeba02dbdf59a2e0e27269b`.
Then `npm install --offline --ignore-scripts` in the build dir for
`node_modules`.

## Verification (all in the build dir unless noted)

- `npx tsc --noEmit` → exit 0
- `npm run build` → real `dist/` (cli.js, forward.js, mcp.js, all
  non-empty, with `.d.ts` + source maps)
- `node dist/cli.js --version` → `0.1.0`; unknown subcommand → usage on
  stderr, exit 2
- `npm test` → **52 tests, 52 pass, 0 fail** (8 suites: registration
  integrity — served tools deep-equal the fixture's full 2 tool
  objects, names in order, exactly the fixture's keys incl. title,
  required fields per tool incl. the `get_quota` no-`required`
  assertion, every property key in order, screenshot constraint
  literals incl. `$schema`, format enum/defaults and the
  9007199254740991 maxima, isKnownTool; session
  establishment/reuse/session-id handling, clientInfo/protocol
  version; env URL override, explicit url override,
  explicit-over-env precedence, blank-env fallback; Authorization
  Bearer header present on every request when the key is set /
  absent when unset / no X-API-Key header in either case; verbatim
  argument forwarding for both tools incl. a fully-specified
  screenshot and an empty-arguments quota call; verbatim result
  pass-through incl. upstream isError and SSE; JSON-RPC/init/
  malformed/no-result/HTTP/throw/timeout failures →
  `pagesnap error:`; parseRpcBody JSON + SSE + matching-id + [DONE]
  cases; MCP-layer MethodNotFound / InvalidParams validation; CLI
  --version / -v / unknown exit 2 / bad-JSON exit 2 / `mcp extra`
  exit 2)
- Parity: `python3 harness/parity.py --original
  harness/fixtures/i2dev-snap-tools-original.json --rebuild
  "stdio:node builds/pagesnap/dist/cli.js mcp" --out
  builds/pagesnap/parity-out` (run from `~/workspace/mcp-rebuild-100`)
  → **PASS, parity 100.0, 2/2 identical, no missing/extra/schema
  diffs** (see `parity-out/parity.json`)
- Consistency sweep: package.json version = server.json top-level =
  npm-entry version = OCI tag = core `VERSION` = CHANGELOG heading
  (0.1.0); package.json `mcpName` = server.json `name` = Dockerfile
  label; server.json description measured at 86 chars; OCI entry
  confirmed to have no `version` field.

Clean-room: all prose and code written fresh for pagesnap; only the
mechanics were patterned on the sibling carddata build, and only the
fixture's tool literals are served verbatim.
