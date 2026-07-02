# Quicknode MCP Reference

The Quicknode MCP server exposes Quicknode Admin API capabilities to AI assistants and coding tools via the Model Context Protocol (MCP). Quicknode is available as a native connector in Claude, ChatGPT, and Codex, and it can also be configured in generic MCP clients that support Streamable HTTP.


**Docs:** https://www.quicknode.com/docs/build-with-ai/quicknode-mcp

## Overview

| Property | Value |
|----------|-------|
| **Protocol** | Model Context Protocol (MCP) |
| **Transport** | HTTP (Streamable HTTP) |
| **Auth** | Quicknode API key (provided at setup) |
| **Directory** | Native Claude connector; native OpenAI app/plugin surface for ChatGPT and Codex |
| **Capabilities** | Endpoint management, usage monitoring, rate limits, security rules, billing |

## Client Setup

### Claude (Web, Desktop, and Claude Code)

Quicknode is listed in Claude's built-in Anthropic connector directory.

**Claude Web / Desktop:**
1. Open Claude settings → **Customize** → **Connectors**
2. Find **Quicknode** in the directory listing
3. Click **Connect** and enter your Quicknode API key when prompted

**Claude Code (CLI):**

```bash
claude mcp add --transport http https://mcp.quicknode.com
```

> **Important:** The correct argument order is `--transport http <server-url>`. Do **not** use the older `mcp-remote` flow — Claude's native HTTP transport handles the connection directly.

### OpenAI (ChatGPT / Codex)

Install Quicknode directly from the ChatGPT Apps catalog or via `/plugin` in Codex when available. ChatGPT and Codex setup is consolidated under the **OpenAI** tab on the Quicknode MCP setup page.

### Cursor / Windsurf / Generic MCP Clients

For any MCP-compatible client that supports HTTP transport, configure the server URL in the client's MCP settings. Refer to each client's documentation for the exact configuration format and field names.

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
- **SDK Docs**: https://www.quicknode.com/docs/quicknode-sdk
