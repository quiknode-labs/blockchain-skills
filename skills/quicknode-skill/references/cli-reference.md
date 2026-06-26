# Quicknode CLI Reference

The `qn` CLI provides command-line access to Quicknode infrastructure — endpoints, Streams, Webhooks, Key-Value Store, and SQL Explorer. Available on all paid plans.

**Docs:** https://www.quicknode.com/docs/quicknode-cli

## Installation

### Homebrew (macOS / Linux)

```bash
brew install quiknode-labs/tap/qn
qn --version
```

### Docker

```bash
docker pull quiknode/qn-cli:latest
docker run --rm quiknode/qn-cli:latest qn --version
```

## Authentication

The CLI authenticates using a Quicknode API key. Two ways to provide it:

### `--api-key` Flag

Pass the key inline on every command:

```bash
qn --api-key YOUR_API_KEY endpoints list
```

### `--config-file` Flag

Store credentials in a YAML config file (default location: `~/.qn/config.yaml`):

```yaml
api_key: YOUR_API_KEY
```

Override the config file path:

```bash
qn --config-file /path/to/config.yaml endpoints list
```

**Note:** The CLI reads the API key from `--api-key` or the config file only. It does not read from environment variables.

### API Key vs. JWT

| Credential | Source | Use Case |
|------------|--------|----------|
| API key (`QN_*`) | Quicknode dashboard | General CLI use, Admin API |
| JWT | Issued per-session by some services | x402 credit drawdown (handled automatically, not the CLI `--api-key`) |

## Global Flags

| Flag | Description |
|------|-------------|
| `--api-key` | Quicknode API key |
| `--config-file` | Path to config file (default: `~/.qn/config.yaml`) |
| `--format` | Output format: `json` or `table` (default: `table`) |
| `--yes` | Skip all confirmation prompts |

## Command Groups

### `qn admin endpoints` — Admin API

Manage RPC endpoints via the Admin API:

```bash
# List all endpoints
qn admin endpoints list

# Get details for a specific endpoint
qn admin endpoints get --id <endpoint-id>

# Create a new endpoint
qn admin endpoints create --chain ethereum --network mainnet

# Delete an endpoint
qn admin endpoints delete --id <endpoint-id>
```

### `qn streams` — Streams CRUD

```bash
# List streams
qn streams list

# Create a stream
qn streams create --name my-stream --chain ethereum --network mainnet

# Get stream details
qn streams get --id <stream-id>

# Update a stream
qn streams update --id <stream-id> --name new-name

# Delete a stream
qn streams delete --id <stream-id>
```

### `qn webhooks` — Webhooks CRUD

```bash
# List webhooks
qn webhooks list

# Create a webhook
qn webhooks create --name my-webhook --url https://your-server.example.com/hook

# Get webhook details
qn webhooks get --id <webhook-id>

# Delete a webhook
qn webhooks delete --id <webhook-id>
```

### `qn kv` — Key-Value Store

```bash
# List KV stores
qn kv list

# Get a value by key
qn kv get --store <store-name> --key <key>

# Set a value
qn kv set --store <store-name> --key <key> --value <value>

# Delete a key
qn kv delete --store <store-name> --key <key>
```

### `qn sql` — SQL Explorer

Run queries and retrieve schema information from SQL Explorer clusters:

```bash
# Run a SQL query (returns JSON output)
qn sql query --cluster hyperliquid-core-mainnet --format json \
  "SELECT timestamp, coin, price, size FROM hyperliquid_trades LIMIT 10"

# Retrieve cluster schema
qn sql schema --cluster hyperliquid-core-mainnet
```

The `--cluster` flag is required and specifies the SQL Explorer cluster ID (e.g., `hyperliquid-core-mainnet`). Use `--format json` for machine-readable output suitable for scripting and pipelines.

## RBAC & Members/Teams

Team and member management (RBAC) is available on **all paid plans**. Use `qn admin` subcommands to list and manage team members, teams, and permissions:

```bash
qn admin members list
qn admin teams list
```

## Output Formats

Use `--format json` for scripting — pipe to `jq` for filtering:

```bash
qn streams list --format json | jq '.[] | {id: .id, name: .name}'
qn sql query --cluster hyperliquid-core-mainnet --format json "SELECT COUNT(*) FROM hyperliquid_trades"
```

## Documentation

- **CLI Docs**: https://www.quicknode.com/docs/quicknode-cli
