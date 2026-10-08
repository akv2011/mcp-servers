# MCP servers

MCP servers I have built, with what each one does, how it is secured, and how to connect it. A printable overview of all of them is in [mcp-servers.pdf](mcp-servers.pdf). Each server's README says which clients it was checked with on 2026-10-08; Claude Code connected to every API-key and stdio server and reached Outreach Guard's Google sign-in, whose full OAuth flow was run with an MCP client against the live URL.

## Servers

| Server | What it does | MCP | Transport and auth | Guardrails |
|---|---|---|---|---|
| [outreach-guard-mcp](https://github.com/akv2011/outreach-guard-mcp) | Email outreach tools over the Instantly API v2, on demo data by default, with a web chat and a live decision log. Live at https://outreach-guard-mcp.vercel.app | 2026-07-28, falls back to 2025-11-25 | Streamable HTTP, OAuth 2.1 with Google sign-in (PKCE, protected resource metadata, CIMD) | Per-call policy on every tool call: recipient domain and block list checks across to, cc and bcc, a daily send cap, and approval before any send using the 2026-07-28 input-required round trip |
| [warehouse-analytics-mcp](https://github.com/akv2011/warehouse-analytics-mcp) | Analytics questions over a DuckDB warehouse through a metrics layer and guarded SQL, on generated mobile app data | 2026-07-28 | stdio, or Streamable HTTP on localhost | Read-only connection with external access off, a parser that admits one SELECT over the published tables, a 1,000 row cap and a 10 second timeout |
| [adnetwork-reporting-mcp](https://github.com/akv2011/adnetwork-reporting-mcp) | Spend, installs, CPI, ROAS, anomalies and budget pacing across three ad networks mapped to one schema | 2026-07-28 | stdio, or Streamable HTTP on localhost | Read-only tools, inputs from fixed lists, a 92 day range cap, non-USD reports refused rather than mixed into totals |
| [okta-mcp-poc](https://github.com/akv2011/okta-mcp-poc) | OAuth resource server where Okta signs people in and Okta groups decide which tools each person sees | 2026-07-28 | Streamable HTTP, Okta access tokens checked for signature, issuer, audience and expiry | A scope per tool; the admin tool is hidden from readers, not just refused |
| [API_mcp_servers_builds](https://github.com/akv2011/API_mcp_servers_builds) | DeFi data and unsigned transaction builders for Aave V3, Morpho, Hyperliquid, tokens and yields across six EVM chains | 2025-06-18 | Streamable HTTP and SSE, API key | All 16 tools are read-only and return unsigned transactions; keys never reach the logs |
| [MCP_Automated_Siri_cursor_control](https://github.com/akv2011/MCP_Automated_Siri_cursor_control) | Text a phone number and an agent acts in Cursor; an MCP bridge exposes the editor actions | 2026-07-28 | stdio for the MCP bridge; Twilio webhook for SMS | Twilio signature and sender allowlist, admin token, SMS only to allowlisted numbers, read-only command allowlist run without a shell; 35 security tests |
| [Market_intelligence_MCP](https://github.com/akv2011/Market_intelligence_MCP) | Market intelligence backend that calls a financial data MCP server for prices, statements, news and filings | 2026-07-28 | stdio | Read-only tools; the backend talks to the server through an MCP client |

## Clients

| Project | What it does |
|---|---|
| [slack-analytics-bot](https://github.com/akv2011/slack-analytics-bot) | Slack bot that answers analytics questions by calling warehouse-analytics-mcp over stdio with Gemini; every reply shows the SQL behind the number and any refused query |

## Other projects

| Project | What it is |
|---|---|
| [AgentWeaver](https://github.com/akv2011/AgentWeaver) | Multi-agent orchestration with a supervisor, routing, shared state and a live dashboard |
| [agent-instruction-conflict](https://github.com/akv2011/agent-instruction-conflict) | Study of what a coding agent does when a repository file contradicts the user: 271 rollouts, two models |
| [Cryptocurrency_Matching_Engine](https://github.com/akv2011/Cryptocurrency_Matching_Engine) | Order matching engine with price-time priority, trade-through protection, REST and WebSocket feeds |

## Connecting

Each server's README has the commands for the clients it supports. Gemini CLI lists a server as "Disabled" when the current folder is not trusted; trust the folder or start it with `--skip-trust`. In short:

```sh
# remote server with OAuth: Claude Code finds the sign-in from the 401 challenge; finish it with /mcp
claude mcp add --transport http outreach-guard https://outreach-guard-mcp.vercel.app/mcp

# remote server with an API key
claude mcp add --transport http defi-mcp http://localhost:3000/mcp -H "Authorization: Bearer $MCP_API_KEY"

# local stdio server
claude mcp add market-intel -- uv --directory /path/to/Market_intelligence_MCP/mcp-server-main run server.py
```
