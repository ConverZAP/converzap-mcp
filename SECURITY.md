# Security policy

## Reporting a vulnerability

Please report security issues privately by email to **contato@converzap.com**. Do **not** open a public GitHub issue for security reports.

Include what you found, how to reproduce it and the potential impact. We will acknowledge your report and keep you informed while we investigate and fix it.

## Scope of this repository

This repository contains only **documentation and connection settings** for the hosted ConverZAP MCP server (`https://mcp.converzap.com`). It contains no server code, no database access and no credentials. The security boundary — OAuth, API key validation, per-workspace isolation, the member's role and scopes, plan checks and rate limits — is enforced **server-side** by the ConverZAP platform, which is not part of this repository.

Never commit API keys (`czk_live_…`). If a key is exposed, revoke it immediately in the app under **Settings → Public API**; OAuth connections can be disconnected in the same screen.
