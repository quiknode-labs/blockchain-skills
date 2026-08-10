# Quicknode MCP Reference

The Quicknode MCP server exposes Quicknode Admin API capabilities to AI assistants and coding tools via the Model Context Protocol (MCP). Quicknode is available as a native connector in Claude, ChatGPT, and Codex, and it can also be configured in generic MCP clients that support Streamable HTTP.


**Docs:** https://www.quicknode.com/docs/build-with-ai/quicknode-mcp

## Overview

| Property | Value |
|----------|-------|
| **Protocol** | Model Context Protocol (MCP) |
| **Transport** | HTTP (Streamable HTTP) |
| **Auth** | OAuth 2.1 for interactive setup; bearer API key for CI/non-interactive setup |
| **Directory** | Native Claude connector; native OpenAI app/plugin surface for ChatGPT and Codex |
| **Capabilities** | Endpoint management, usage monitoring, rate limits, security rules, billing |

## Client Setup

### Claude (Web, Desktop, and Claude Code)

Quicknode is listed in Claude's built-in Anthropic connector directory.

**Claude Web / Desktop:**
1. Open Claude settings → **Customize** → **Connectors**
2. Find **Quicknode** in the directory listing
3. Click **Connect**
4. Log in to Quicknode in the browser window, select **Admin** or **Viewer**, and approve access

**Claude Code (CLI):**

```bash
claude mcp add --transport http quicknode https://mcp.quicknode.com/mcp
```

By default, Claude Code installs in local scope for the current project. Use `--scope project` to share via project MCP config, or `--scope user` to make it available across your projects:

```bash
claude mcp add --transport http --scope user quicknode https://mcp.quicknode.com/mcp
```

### OpenAI (ChatGPT / Codex)

Install Quicknode directly from the ChatGPT Apps catalog or via `/plugin` in Codex when available. The interactive flow opens a Quicknode OAuth browser window where the user selects **Admin** or **Viewer** before approving access.

### Cursor / VS Code / Windsurf / Zed / Generic MCP Clients

For any MCP-compatible client that supports HTTP transport, configure:

```text
https://mcp.quicknode.com/mcp
```

If a client does not support native OAuth or HTTP transport, use `mcp-remote` as a bridge:

```json
{
  "mcpServers": {
    "quicknode": {
      "command": "npx",
      "args": ["-y", "mcp-remote", "https://mcp.quicknode.com/mcp"]
    }
  }
}
```

## Authentication and Roles

Interactive setup uses OAuth 2.1. The user logs in to Quicknode, chooses the account to connect, then selects an API key role:

| Role | Access |
|------|--------|
| **Viewer** | Read-only access to endpoints, metrics, logs, usage, billing, and chains |
| **Admin** | Viewer access plus create/delete endpoints, security changes, and rate-limit updates |

Use **Viewer** for inspection-only workflows. Use **Admin** only when the assistant needs to make Quicknode account changes.

## Non-Interactive / CI Setup

For CI, remote machines, or clients where browser OAuth is unavailable, pass a Quicknode API key as a bearer token. The key's Quicknode role controls MCP permissions.

### Claude Code

```bash
claude mcp add quicknode --transport http https://mcp.quicknode.com/mcp --header "Authorization: Bearer YOUR_API_KEY"
```

### Codex

```bash
codex mcp add quicknode \
  --url https://mcp.quicknode.com/mcp \
  --bearer-token-env-var QUICKNODE_API_KEY
```

### Generic HTTP MCP Config

```json
{
  "mcpServers": {
    "quicknode": {
      "url": "https://mcp.quicknode.com/mcp",
      "headers": {
        "Authorization": "Bearer YOUR_API_KEY"
      }
    }
  }
}
```

## Tool Surface

The Quicknode MCP server exposes tools covering the Admin API surface:

### Endpoint Management

| Tool | Description |
|------|-------------|
| `list-endpoints` | List all RPC endpoints in your account |
| `get-endpoint` | Retrieve details for a specific endpoint |
| `create-endpoint` | Create a new RPC endpoint |
| `delete-endpoint` | Delete an endpoint |
| `update-endpoint-rate-limits` | Configure global request rate limits for an endpoint |

### Per-Method Rate Limits

| Tool | Description |
|------|-------------|
| `list-endpoint-method-rate-limits` | List per-method rate limits for an endpoint |
| `create-endpoint-method-rate-limit` | Set a rate limit for a specific RPC method |
| `update-endpoint-method-rate-limit` | Update an existing method-level rate limit |
| `delete-endpoint-method-rate-limit` | Remove a method rate limit |

### Security

| Tool | Description |
|------|-------------|
| `list-endpoint-security` | List all security rules for an endpoint |
| `create-security-rule` | Add a security rule (allowlist, JWT auth, etc.) |
| `update-endpoint-security-options` | Update endpoint-level security settings |
| `delete-security-rule` | Remove a security rule |

### Usage & Monitoring

| Tool | Description |
|------|-------------|
| `get-endpoint-metrics` | Retrieve performance metrics for an endpoint |
| `list-endpoint-logs` | List request logs for an endpoint |
| `get-endpoint-log-details` | Get details for a specific log entry |
| `get-rpc-usage` | View RPC usage statistics across your account |

### Account & Billing

| Tool | Description |
|------|-------------|
| `list-chains` | List all supported blockchain networks |
| `get-billing` | Retrieve billing information |

## MCP Resources

Quicknode MCP also exposes endpoint details as MCP resources in clients that support resource browsing. Reference `@quicknode` in the client UI to attach endpoint context such as `quicknode://endpoints/<endpoint-id>` without first calling a tool.

## Example Prompts

- "List all my Quicknode endpoints"
- "Create a new Ethereum mainnet endpoint"
- "What's my RPC usage for this month?"
- "Add an IP allowlist rule to my endpoint"
- "Show me the per-method rate limits on my endpoint"
- "Connect my Quicknode workspace to Claude via MCP"

## When to Use MCP vs. CLI vs. SDK vs. Admin API

| Interface | Best For |
|-----------|----------|
| **MCP** | Conversational management inside Claude, Cursor, or other AI tools |
| **CLI (`qn`)** | Shell scripts, CI/CD pipelines, direct terminal workflows |
| **SDK** | Application or agent code that coordinates Quicknode product APIs |
| **Admin API** | Programmatic infrastructure-as-code from application code |

## Documentation

- **MCP Docs**: https://www.quicknode.com/docs/build-with-ai/quicknode-mcp
- **Build with AI Overview**: https://www.quicknode.com/docs/build-with-ai
- **SDK Docs**: https://www.quicknode.com/docs/sdk
