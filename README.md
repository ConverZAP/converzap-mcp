# ConverZAP MCP server

[![license](https://img.shields.io/badge/license-MIT-blue.svg)](./LICENSE)
![WhatsApp official API](https://img.shields.io/badge/WhatsApp-official%20API-25D366)

Connect AI assistants and agents (Claude, ChatGPT, Cursor, Cline, agent frameworks…) to **WhatsApp via ConverZAP** — the official WhatsApp Business Platform (Meta Cloud API), as a **Meta-verified Tech Provider**.

The server exposes a ConverZAP workspace as [MCP](https://modelcontextprotocol.io) tools: read and manage contacts, read conversations and send WhatsApp messages and approved templates, manage the CRM pipeline, run campaigns, enroll contacts into automations and read reports. Every tool runs through the [ConverZAP public API (v1)](https://developers.converzap.com) internally, so **every action honors the same permissions, scopes and rate limits as your account** — the connection never acts above the member's role.

> This repository holds the **documentation and connection settings** for the hosted server. The server itself runs on ConverZAP's infrastructure — there is nothing to install or self-host.

## Tools

15 tools. Read tools are safe to call freely; write tools support a `dry_run` mode that validates without executing, and `send_message` / `start_campaign` are marked **destructive** so the assistant asks for confirmation first.

| Tool | Type | What it does |
|---|---|---|
| `whoami` | read | Account, your role, granted permissions, plan and connected WhatsApp numbers |
| `find_contacts` | read | Search contacts by phone, name, e-mail or label |
| `get_contact` | read | Full contact card with recent conversations and open deals |
| `list_conversations` | read | List inbox conversations (open, awaiting a human, waiting for a reply…) |
| `read_conversation` | read | Read a conversation's messages and state (incl. the 24-hour window) |
| `list_templates` | read | List approved WhatsApp templates |
| `get_results` | read | Inbox report and campaign results |
| `send_message` | write · destructive | Send text or an approved template |
| `schedule_message` | write | Schedule a message for later |
| `update_conversation` | write | Assign, resolve, reopen, snooze, label, add notes, hand over to/from the AI agent |
| `upsert_contact` | write | Create or update a contact |
| `manage_deal` | write | Create, update or move a CRM deal across stages |
| `create_campaign_draft` | write | Create a campaign as a draft |
| `start_campaign` | write · destructive | Start a campaign |
| `enroll_in_automation` | write | Enroll a contact into an automation (or remove it) |

**Message rate limit:** 30 messages/minute and 300/day per connection (on top of your plan's own limits).

## Requirements

- A ConverZAP account on a plan that includes the public API.
- Either an **API key** (`czk_live_…`) created in the app under **Settings → Public API**, or an account to sign in with via **OAuth**. Access is scoped to the member's role; on the free trial it is **read-only**.

## Connect

This is a **remote MCP server** (Streamable HTTP) at:

```
https://mcp.converzap.com
```

It supports **two authentication methods** — pick whichever your client makes easier.

### Option A — OAuth (recommended for clients with remote-MCP support)

Clients like **Claude** (as a custom connector) and others that support remote MCP handle OAuth automatically: just add the server by URL and sign in. You'll log into ConverZAP, pick the workspace and check the permissions to grant — **write permissions start unchecked**.

```
Server URL: https://mcp.converzap.com
Auth:       OAuth (the client opens the login automatically)
```

No API key to copy or store — the client keeps the token.

### Option B — API key (works everywhere, incl. stdio-only clients)

Send your key as a bearer header. For clients that only speak stdio (some desktop setups), bridge the remote server with [`mcp-remote`](https://www.npmjs.com/package/mcp-remote):

```json
{
  "mcpServers": {
    "converzap": {
      "command": "npx",
      "args": [
        "-y", "mcp-remote", "https://mcp.converzap.com",
        "--header", "Authorization: Bearer ${CONVERZAP_API_KEY}"
      ],
      "env": { "CONVERZAP_API_KEY": "czk_live_…" }
    }
  }
}
```

Clients with native remote-MCP support can instead add the URL `https://mcp.converzap.com` directly and set the header `Authorization: Bearer czk_live_…`.

## Example prompts

- "List my open WhatsApp conversations waiting for a human."
- "Send the approved `lembrete` template to a contact."
- "Create a contact and move their deal to the Negotiation stage."
- "How many messages were delivered this week?"

## Security

- Access is limited to the member's role and scopes — on OAuth, grant the minimum permissions (writes are unchecked by default); on API keys, create the key with only the scopes you need.
- Treat an API key like a password and keep it server-side. Revoke it anytime in **Settings → Public API**, or disconnect an OAuth connection — both take effect immediately.
- Found a vulnerability? Email **contato@converzap.com** — don't open a public issue. See [SECURITY.md](./SECURITY.md).

## Links

- API docs: https://developers.converzap.com
- OpenAPI: https://api.converzap.com/v1/openapi.json
- SDK: https://github.com/converzap/converzap-sdk
- Website: https://converzap.com

## License

[MIT](./LICENSE)
