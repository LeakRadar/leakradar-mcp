# LeakRadar MCP connector

The LeakRadar MCP (Model Context Protocol) server connects your assistant to [LeakRadar](https://leakradar.io) to search, monitor and unlock results with your account. It works with Claude (web, Desktop and Claude Code), ChatGPT, Codex, Cursor, VS Code and any other client that supports remote MCP with OAuth.

```
https://mcp.leakradar.io/mcp
```

No API key to copy: you add the URL, sign in to your LeakRadar account and approve the access. You need a LeakRadar account. Features depend on your LeakRadar plan and your assistant or workspace settings.

- [Full guide, in 13 languages](https://leakradar.io/en/mcp)
- [API reference](https://docs.leakradar.io)
- [Python SDK](https://github.com/LeakRadar/leakradar-wrapper)

## What this repository contains

The server is hosted by LeakRadar at `mcp.leakradar.io`. This repository holds the files that let clients and directories install it.

| File | Purpose |
|---|---|
| `.claude-plugin/plugin.json` and `.mcp.json` | Claude Code plugin that adds the connector |
| `.claude-plugin/marketplace.json` | Lets Claude Code install the plugin from this repository |
| `lhm.plugin.json` | Listing manifest for the LobeHub MCP marketplace |

## Connect your assistant

When your client redirects you, sign in to LeakRadar, review the requested access and approve, then return to your assistant. Never paste your password or a token into the conversation.

### Claude Code

Install the plugin from this repository, then open `/mcp` in Claude Code and authenticate LeakRadar:

```
claude plugin marketplace add LeakRadar/leakradar-mcp
claude plugin install leakradar@leakradar
```

Or add the server without the plugin:

```
claude mcp add --transport http leakradar https://mcp.leakradar.io/mcp
```

[Official client guide](https://code.claude.com/docs/en/mcp)

### Claude (web and Desktop)

Open Customize > Connectors > Add custom connector. Name it LeakRadar and paste the server URL. Choose OAuth and, if asked for an OAuth client, Register automatically. Connect, then enable the connector in your conversation.

[Official client guide](https://support.claude.com/en/articles/11175166-get-started-with-custom-connectors-using-remote-mcp)

### ChatGPT

In ChatGPT on the web, open Plugins, then + > Add custom MCP server. Name it LeakRadar, paste the server URL and choose OAuth. Create the plugin, install it, then select it with @ in your conversation. Workspace permissions may restrict this option.

[Official client guide](https://developers.openai.com/api/docs/guides/custom-mcp-server)

### Codex

Run the commands below in your terminal. The login command opens the LeakRadar authorization page. Use `codex mcp list` or `/mcp` in the Codex CLI to check the server.

```
codex mcp add leakradar --url https://mcp.leakradar.io/mcp
codex mcp login leakradar
```

[Official client guide](https://learn.chatgpt.com/docs/extend/mcp?surface=cli)

### Cursor

Add the entry below to `.cursor/mcp.json` in your project, preserving any existing servers. Save, restart Cursor and complete the sign-in prompt. Enable LeakRadar in the agent tools.

```json
{
  "mcpServers": {
    "leakradar": {
      "url": "https://mcp.leakradar.io/mcp"
    }
  }
}
```

[Official client guide](https://prod.cursor.com/help/customization/mcp)

### VS Code

In the Command Palette, run MCP: Add Server, choose HTTP, paste the URL and name it `leakradar`. Alternatively merge the entry below into `.vscode/mcp.json`. Start the server, authorize it and enable its tools in agent mode.

```json
{
  "servers": {
    "leakradar": {
      "type": "http",
      "url": "https://mcp.leakradar.io/mcp"
    }
  }
}
```

[Official client guide](https://code.visualstudio.com/docs/agent-customization/mcp-servers)

## Try your first request

Check your plan, points and quotas first, without unlocking results:

> Use LeakRadar to show my plan, points balance and current quotas. Do not unlock or change anything.

More requests to try. Replace `example.com` with a domain you are authorized to assess.

> With LeakRadar, summarize exposure for my authorized domain example.com by category, without unlocking results.

> With LeakRadar, list my monitors and summarize their latest alerts, without changing them or spending points.

> With LeakRadar, prepare to unlock at most 10 results from the last search on my authorized domain. Show the selected results and cost, then wait for my approval before unlocking them.

Unlocks may use points. Confirmation depends on your assistant's settings.

## What the connection allows

The connector acts on your behalf, within your account permissions. Access is granted as a whole; the categories below are explanations, not separate permission switches.

| Tool | Category | What it does |
|---|---|---|
| `account_status` | Search and read | Plan, points, subscription dates, usage against the limits |
| `domain_overview` | Search and read | Sampled exposure counts of a domain by category |
| `list_monitors` | Search and read | The account's monitors and their channels |
| `monitoring_alerts` | Search and read | Alert runs, or the records of one run |
| `job_status` | Search and read | Unlock task status, export jobs and download links |
| `api_catalog` | Search and read | List the API operations, filtered by text or class |
| `api_describe` | Search and read | Parameters and body of one operation |
| `api_read` | Search and read | Any read operation of the API: free, masked unless already unlocked |
| `api_spend` | Unlock with points | Unlock records, with a mandatory maximum |
| `api_export` | Export and download | Queue an export, get a download link |
| `api_create` | Create and modify | Monitors, alert channels, lists |
| `api_update` | Create and modify | Monitors, channels, profile, lists, statuses |
| `api_delete` | Delete and revoke | Delete, revoke, clear |
| `api_send` | Send messages | Test a channel, send a team invitation |

- **Search and read.** Reads spend no points, remain subject to quotas and show masked credentials unless already unlocked.
- **Unlock with points.** Unlocks need an explicit maximum. Points may be used depending on your plan; reads never unlock automatically.
- **Export and download.** Export and raw download allowances depend on your plan, unlocked data and any applicable GB balance.
- **Create and modify.** Auto-unlock monitors require a points budget.
- **Delete and revoke.** Where your account permits it.
- **Send messages.** These actions can notify other people.

The connector instructs the assistant to ask before spending, exporting, changing, deleting or sending. Actual approval prompts depend on your client settings; the server does not enforce a separate human approval for every action.

Revoke access at any time from LeakRadar > [Settings > MCP](https://members.leakradar.io/en/settings?tab=mcp) > Connected applications. Data already shared with your assistant remains subject to that provider's policies.

## Help with the connection

- **The connection failed or expired.** Reconnect from your assistant and sign in again. A 401 can indicate expired or revoked access.
- **A feature is unavailable or results are masked.** Check your LeakRadar plan with `account_status`. Masked results need unlocking; a 402 indicates insufficient points or GB, and a 403 can indicate an account or plan restriction.
- **A request is rate limited.** On a 429, respect `Retry-After` before retrying and reduce concurrent requests. Use smaller pages. Do not repeatedly replay failing requests.
- **You cannot add the server or see its tools.** Check client and workspace permissions, enable LeakRadar in the conversation and refresh its tools.

More in the [full guide](https://leakradar.io/en/mcp), or [contact us](https://leakradar.io/en/contact-us).

## License

The files in this repository are released under the [MIT License](LICENSE). Use of the LeakRadar service is subject to its [Terms of Service](https://leakradar.io/en/terms) and [Privacy Policy](https://leakradar.io/en/privacy).
