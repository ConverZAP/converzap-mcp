# ConverZAP MCP server

[![license](https://img.shields.io/badge/license-MIT-blue.svg)](./LICENSE)
![WhatsApp official API](https://img.shields.io/badge/WhatsApp-official%20API-25D366)

Connect AI assistants and agents (Claude, ChatGPT, Cursor, Cline, agent frameworks…) to **WhatsApp via ConverZAP** — the official WhatsApp Business Platform (Meta Cloud API), as a **Meta-verified Tech Provider**.

The server exposes a ConverZAP workspace as [MCP](https://modelcontextprotocol.io) tools: read and manage contacts, read conversations and send WhatsApp messages and approved templates, manage the CRM pipeline, run campaigns, enroll contacts into automations and read reports. Every tool runs through the [ConverZAP public API (v1)](https://developers.converzap.com) internally, so **every action honors the same permissions, scopes and rate limits as your account** — the connection never acts above the member's role.

> This repository holds the **documentation and connection settings** for the hosted server. The server itself runs on ConverZAP's infrastructure — there is nothing to install or self-host.

## Tools

41 tools in three groups. Read tools are safe to call freely. Write tools only appear with the write permissions granted on the connection, support a `dry_run` mode that runs every rule without executing, and the ones that reach a real person (messages, campaigns, turning an agent or automation on) are marked **external** so the assistant confirms first. Text written by contacts comes back marked as untrusted content.

### Read

| Tool | What it does |
|---|---|
| `whoami` | Account, your role, granted permissions, plan, connected WhatsApp numbers, pipelines and team |
| `find_contacts` | Search contacts by name (accent-insensitive), phone (with or without the Brazilian 9th digit), e-mail or label |
| `get_contact` | Full contact card with recent conversations and open deals |
| `list_conversations` | List inbox conversations (open, awaiting a human, waiting for a reply…) |
| `read_conversation` | Read a conversation's messages and state, incl. the 24-hour window |
| `list_templates` | WhatsApp templates with their variables and the numbers (`channels`) each one works on |
| `get_results` | Inbox report, campaign and automation results |
| `get_approval` | Check an action waiting for a person to approve it in the app |

### Act

| Tool | Type | What it does |
|---|---|---|
| `send_message` | write · external | Send text or an approved template — in a conversation, or to a contact with no conversation yet |
| `schedule_message` | write · external | Schedule a message for later |
| `update_conversation` | write | Assign, resolve, reopen, snooze, label, add notes, hand over to/from the AI agent |
| `upsert_contact` | write | Create or update a contact |
| `manage_deal` | write | Create, update or move a CRM deal across stages |
| `create_campaign_draft` | write | Create a campaign as a draft (the dry run warns about template variables that are empty in the audience) |
| `start_campaign` | write · external | Start, pause, resume or cancel a campaign (starting always waits for approval in the app) |
| `enroll_in_automation` | write · external | Enroll a contact into an automation (or remove it) |
| `draft_internal_agent` | write | Draft an internal agent for recurring team chores (the user reviews and publishes it in the app) |

### Set up (AI agents, briefing, pipeline)

Lets the assistant set the account up for the owner: research the business, write the briefing, build and test the WhatsApp AI agent, design the pipeline and create templates and automations. Changes to a live agent go to a draft until published.

| Tool | What it does |
|---|---|
| `get_setup_status` | Setup status and the next step |
| `research_business` · `get_briefing` · `update_briefing` | Research the business and read/update the briefing the agents use |
| `get_rollout_plan` | Rollout plan from the briefing to the agents |
| `create_agent` · `get_agent` · `update_agent` | Create, read and adjust an AI agent (draft) — incl. hand-back to the agent after the team |
| `test_agent` | Test the agent in the simulator |
| `publish_agent` · `discard_draft` · `activate_agent` | Publish or discard the draft, turn the agent on or off |
| `set_agent_followup` · `set_agent_scheduling` | Follow-up when the customer goes quiet; scheduling and meeting reminders |
| `get_agent_report` | AI agents' results |
| `get_pipeline_context` · `apply_pipeline` · `set_operator` | Design and apply the pipeline; turn the pipeline operator on or off |
| `create_template` | Create a WhatsApp template (sent to Meta for approval) |
| `create_automation` · `set_automation_status` | Create an automation (draft) and turn it on or off |
| `connect_whatsapp` · `get_app_link` · `get_plans` | Link to connect WhatsApp, link to an app page, plans and subscribe link |

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
- "Set up a WhatsApp agent for my clinic and test it with three questions."
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
