# Changelog

## 0.1.0 (2026-10-28)

- First release: a forwarding client for the hosted snap screenshot MCP server, serving the two built tools locally for tools/list and forwarding every tools/call upstream over a streamable-HTTP JSON-RPC session (initialize, notifications/initialized, tools/call, honouring the Mcp-Session-Id header when issued).
- Tools: `take_screenshot` and `get_quota`.
- Upstream results pass through verbatim; upstream failures surface as `pagesnap error:` text results.
- One bin serves the CLI (one subcommand per tool, plus `--version`) and the stdio MCP server (`mcp` subcommand). Upstream endpoint overridable via `SNAP_MCP_URL`; when `SNAP_API_KEY` is set it is sent as an `Authorization: Bearer` header.
