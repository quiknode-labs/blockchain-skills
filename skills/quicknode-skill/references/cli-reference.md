# Quicknode CLI Reference

The `qn` CLI manages Quicknode product APIs from a terminal, script, CI job, or agent shell. Use it for endpoints, Streams, Webhooks, Key-Value Store, SQL Explorer, teams, usage, billing, metrics, and security workflows.

**Docs:** https://www.quicknode.com/docs/quicknode-cli

## Installation

### Homebrew (macOS / Linux)

```bash
brew install quicknode/tap/qn
qn --version
```

## Authentication

Create an API key in the Quicknode dashboard, then log in and verify access:

```bash
qn auth login
qn auth whoami
```

For non-interactive environments, pass a key directly or write a config file outside the repository:

```bash
qn auth login --api-key <KEY>
qn --api-key <KEY> endpoint list --format json
```

Credential precedence:

1. `--api-key <KEY>`
2. `--config-file <PATH>`
3. `~/.config/qn/config.toml`

The CLI does not read API keys directly from environment variables. In CI, read the environment secret in the shell and pass it through `--api-key`, or write a temporary config file and pass `--config-file`.

## Agent Context

Use `qn agent context` to print a self-contained CLI usage guide for AI agents. It does not require authentication and makes no network request.

```bash
qn agent context
qn agent context -o json
```

## Output Formats

Set output with `--format` or `-o`.

| Format | Use |
|--------|-----|
| `table` | Human-readable terminal output |
| `json` | Scripts, tests, and `jq` pipelines |
| `yaml` | Readable structured output |
| `md` | Markdown tables for issues and docs |
| `toon` | Compact LLM-oriented structured output |

Examples:

```bash
qn endpoint list --format json
qn usage summary --from 7d -o yaml
qn endpoint list --wide
```

## Command Groups

### Auth

```bash
qn auth login
qn auth login --api-key <KEY>
qn auth whoami
qn auth status
qn auth logout
```

### Endpoints

```bash
qn endpoint list --limit 20
qn endpoint show <endpoint-id>
qn endpoint urls <endpoint-id>
qn endpoint logs <endpoint-id> --from 1h --to now --limit 50
qn endpoint metrics <endpoint-id> --metric method_calls_over_time --period day
```

Commands that create, pause, resume, archive, or change endpoint security/rate limits prompt before acting.

```bash
qn endpoint create --chain ethereum --network mainnet
qn endpoint pause <endpoint-id>
qn endpoint resume <endpoint-id>
qn endpoint archive <endpoint-id>
```

### Endpoint Tags

```bash
qn endpoint tag list
qn endpoint tag add <endpoint-id> production
qn endpoint tag remove <endpoint-id> <tag-id>
qn endpoint tag rename <tag-id> mainnet-production
qn endpoint tag delete <tag-id>
```

Tag add, rename, remove, and delete commands change endpoint metadata.

### Bulk Endpoint Operations

```bash
qn endpoint bulk pause <endpoint-id-1> <endpoint-id-2>
qn endpoint bulk resume <endpoint-id-1> <endpoint-id-2>
qn endpoint bulk tag add --label production <endpoint-id-1> <endpoint-id-2>
qn endpoint bulk tag remove --tag-id <tag-id> <endpoint-id-1> <endpoint-id-2>
```

Bulk operations can change multiple endpoints in one command.

### Endpoint Security and Rate Limits

```bash
qn endpoint security show <endpoint-id>
qn endpoint security token create <endpoint-id>
qn endpoint security ip add <endpoint-id> 203.0.113.10
qn endpoint security request-filter create <endpoint-id> --method eth_getBalance

qn endpoint rate-limit get <endpoint-id>
qn endpoint rate-limit set <endpoint-id> --rps 100 --rpm 5000
qn endpoint rate-limit method-list <endpoint-id>
qn endpoint rate-limit method-create <endpoint-id> --interval minute --method eth_getLogs --rate 120
```

### Streams

```bash
qn stream list --limit 20
qn stream show <stream-id>
qn stream enabled-count
qn stream test-filter \
  --network ethereum-mainnet \
  --dataset block \
  --block 17811625 \
  --filter-file filter.js
```

Create and lifecycle commands change Stream state and may prompt:

```bash
qn stream create \
  --name blocks \
  --network ethereum-mainnet \
  --dataset block \
  --start 17811625 \
  --end=-1 \
  --region usa-east \
  --webhook https://example.com/webhook

qn stream pause <stream-id>
qn stream activate <stream-id>
qn stream delete <stream-id>
```

### Webhooks

```bash
qn webhook list --limit 20
qn webhook show <webhook-id>
qn webhook enabled-count
qn webhook create \
  --name "wallet alerts" \
  --network ethereum-mainnet \
  --url https://example.com/webhook \
  --compression none \
  --template evm-wallet \
  --wallet 0xa0b86991c6218b36c1d19d4a2e9eb0ce3606eb48
```

### Key-Value Store

Sets store one value per key:

```bash
qn kv set put mykey myvalue
qn kv set get mykey
qn kv set ls
qn kv set delete mykey
```

Lists store ordered values under one key:

```bash
qn kv list create allowlist 0xabc 0xdef
qn kv list append allowlist 0x123
qn kv list contains allowlist 0xabc
qn kv list get allowlist
qn kv list delete allowlist
```

### SQL Explorer

```bash
qn sql query "SELECT * FROM hyperliquid_trades LIMIT 10" --cluster-id hyperliquid-core-mainnet
qn sql schema hyperliquid-core-mainnet
qn sql query "SELECT * FROM hyperliquid_trades LIMIT 100" --cluster-id hyperliquid-core-mainnet --format json
```

### Account Operations

```bash
qn team list
qn team show <team-id>
qn team endpoints <team-id>
qn usage summary --from 7d
qn usage by-endpoint --from 7d
qn metrics account --period day --metric credits_over_time
qn chain list
qn billing invoices
qn billing payments
```

## Confirmation Behavior

Commands that delete, archive, pause, revoke, or otherwise change resources prompt before acting. In non-interactive environments, keep examples read-only by default or pass the documented prompt bypass only after explicit user confirmation.

## Documentation

- **CLI Docs**: https://www.quicknode.com/docs/quicknode-cli
- **CLI Examples**: https://www.quicknode.com/docs/quicknode-cli/examples
