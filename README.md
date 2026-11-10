# pagesnap

pagesnap lets an agent turn any public web page into
a picture. Give it a URL and it renders the page in
Chromium upstream and hands back the screenshot —
as a viewport capture or as the whole scrollable
page, in webp, jpeg or png, at the viewport size you
ask for. You can hold the capture until a selector
appears, pick which navigation event to wait for,
and set the time zone and locale the page sees, which
matters when a site renders differently by region or
language. A second tool reports how many screenshots
you have left, so a workflow can check its allowance
before it starts spending it.

This package renders nothing itself. It is a thin
local bridge to the hosted snap service: it answers
the tool list locally, exactly as captured from the
live endpoint, and forwards each call upstream,
returning whatever comes back untouched. The service
is free without signup; supplying your own API key
raises the allowance from the free daily amount to
your organisation's monthly amount, which is what
`get_quota` then reports.

## Tools

- `take_screenshot` — render a public web page and
  return a screenshot. Required: `url` (a public
  http(s) URL). Optional: `format` (`webp`, `jpeg`
  or `png`, default `webp` — webp and jpeg are much
  smaller than png), `width` (viewport width in CSS
  pixels, default 1280) and `height` (default 720;
  values above the service maximum are clamped),
  `fullPage` (capture the full scrollable page,
  default false), `waitUntil` (`load`,
  `domcontentloaded` or `networkidle`, default
  `load`), `waitForSelector` (a CSS selector that
  must appear before capturing), `timezoneId` (an
  IANA time zone such as Europe/Berlin) and `locale`
  (a BCP 47 locale such as de-DE). Very tall full-page
  captures come back as a link instead of an inline
  image, and a fullPage WebP capture of a page taller
  than 16383px is clipped to that height (reported as
  `fullPageClip`) — use jpeg or png for pages that
  tall.
- `get_quota` — how many screenshots are left. Takes
  no arguments. Returns the free daily allowance when
  no key is in play, or the organisation's monthly
  allowance when one is. This tool only reads your
  own allowance; nothing in this package buys,
  tops up or manages an account.

## Configuration

- `SNAP_MCP_URL` (optional) — override the upstream
  endpoint. Defaults to `https://snap.i2dev.com/mcp`.
  (The override hook is this rebuild's own labelled
  reconstruction.)
- `SNAP_API_KEY` (optional) — when set, every upstream
  request carries an `Authorization: Bearer <key>`
  header with your snap API key. When it is not set,
  no key header is sent and the free allowance applies.
  (The header name is pinned by the original's registry
  record, which declares a secret `Authorization`
  header; the record pins no package env name, so the
  env name, the Bearer form and the forwarding itself
  are labelled reconstructions — the original surface
  was captured keyless.)

### Forwarding mechanics

On the first tool call the client opens a streamable
HTTP session upstream: a JSON-RPC `initialize`
request (protocolVersion `2025-03-26`, clientInfo
name `pagesnap`), then a `notifications/initialized`
notification, then the `tools/call` request carrying
the caller's arguments unchanged. When the upstream
answers `initialize` with an `Mcp-Session-Id` header,
that value rides on every later request of the
session, which is reused for the life of the process.
Replies may arrive as plain JSON or as server-sent
events; both are read.

Successful upstream results pass through verbatim,
including any `isError` flag the upstream sets.
Failures — a JSON-RPC error, a non-2xx status, an
unreadable response, or a request that cannot be
completed — come back as an `isError` result whose
text starts with `pagesnap error:` (this rendering
is this rebuild's own labelled reconstruction).

## CLI

Each tool is also a subcommand taking one JSON
argument string:

```bash
npx @reneturcios/pagesnap take_screenshot '{"url":"https://example.com"}'
npx @reneturcios/pagesnap take_screenshot '{"url":"https://example.com","fullPage":true,"format":"png"}'
npx @reneturcios/pagesnap get_quota '{}'
npx @reneturcios/pagesnap --version
```

Exit codes: `0` on success, `1` on an `isError`
result, `2` on a usage error.

## MCP server

```bash
npx -y @reneturcios/pagesnap mcp
```

MCP client config:

```json
{
  "mcpServers": {
    "pagesnap": {
      "command": "npx",
      "args": ["-y", "@reneturcios/pagesnap", "mcp"]
    }
  }
}
```

## Docker

```bash
docker run -i ghcr.io/cloudcorprecords/pagesnap
docker run --rm ghcr.io/cloudcorprecords/pagesnap --version
```

The container's default command is `mcp`, so the first
form is a stdio MCP server; any other arguments reach
the CLI normally.

## Development

```bash
npm install
npm run build
npm test
npm run dev
```

The test suite runs entirely against a stubbed
JSON-RPC upstream — no live network calls.

## License

MIT — see `LICENSE`.
