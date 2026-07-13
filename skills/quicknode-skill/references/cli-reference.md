# Quicknode CLI Reference

The `qn` CLI manages Quicknode product APIs from a terminal, script, CI job, or agent shell. Use it for endpoints, Streams, Webhooks, Key-Value Store, SQL Explorer, teams, usage, billing, metrics, security workflows, and making JSON-RPC calls to any supported network via Tooling Access, with no endpoint setup required.

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

In CLI v0.4.0 and later, `qn auth whoami` validates the API key and returns account metadata such as account id, account name, plan, plan status, and plan interval.

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

Use `qn auth whoami --format json` when an agent needs structured account data:

```json
{
  "source": "config file",
  "key": "****abcd",
  "validated": true,
  "account_id": 123456,
  "account_name": "Example Account",
  "plan": "Accelerate (active, monthly)",
  "plan_status": "active",
  "plan_interval": "monthly"
}
```

### RPC (Tooling Access)

Requires CLI v0.5.0+. `qn rpc` calls your account's Tooling Access endpoint — a single multichain, read-only endpoint Quicknode provisions automatically, with short-lived JWTs minted and refreshed automatically. No manual endpoint setup required.

```bash
qn auth login
qn rpc call eth_getBlockByNumber '["latest", false]' --network base-mainnet
```

Params: positional (JSON array), by name (JSON object), from a file, or from stdin.

```bash
qn rpc call eth_blockNumber
qn rpc call eth_getBalance '["0xabc...", "latest"]'
qn rpc call getSlot --network solana-mainnet
qn rpc call eth_call --params-file params.json
echo '[...]' | qn rpc call eth_call -
cat params.json | qn rpc call eth_call -f -
```

`--network <KEY>` selects a network on the multichain endpoint (e.g. `base-mainnet`, `solana-mainnet`, `polygon`, `btc`). Omit it for the default network. `qn rpc list-networks` (alias `ls`) lists available keys — no RPC call made.

```bash
qn rpc list-networks
qn rpc list-networks -o json
```

`--endpoint-url <URL>` sends the call to a self-authenticating URL instead of the Tooling Access endpoint, overriding `[rpc] endpoint_url` in `~/.config/qn/config.toml`. Mutually exclusive with `--network`.

First-run `qn rpc call` prompts `[y/N]` to enable Tooling Access if it isn't provisioned yet. Pass `-y`/`--yes` to auto-confirm non-interactively.

```bash
qn rpc call eth_blockNumber --network base-mainnet -y
```

### Tooling Access

```bash
qn tooling-access status
qn tooling-access enable
qn tooling-access disable
```

`status` shows whether Tooling Access is enabled and the endpoint URL. `enable` provisions the endpoint (idempotent, admin role required). `disable` is idempotent and account-wide — it cuts off blockchain access for all Quicknode developer tooling, not just the current session. Confirm with the user before running it.

### Endpoints

```bash
qn endpoint list --limit 20
qn endpoint show <endpoint-id>
qn endpoint urls <endpoint-id>
qn endpoint logs <endpoint-id> --from 1h --to now --limit 50
qn endpoint log-details <endpoint-id> <request-id>
qn endpoint metrics <endpoint-id> --metric method_calls_over_time --period day
```

Commands that create, update, pause, resume, archive, enable or disable multichain, or change endpoint security/rate limits mutate endpoint state.

```bash
qn endpoint create --chain ethereum --network mainnet
qn endpoint update <endpoint-id> --label "production-mainnet"
qn endpoint pause <endpoint-id>
qn endpoint resume <endpoint-id>
qn endpoint archive <endpoint-id>
qn endpoint enable-multichain <endpoint-id>
qn endpoint disable-multichain <endpoint-id>
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
qn endpoint security options <endpoint-id>
qn endpoint security set-options <endpoint-id> --tokens enabled --ips enabled
qn endpoint security token create <endpoint-id>
qn endpoint security token delete <endpoint-id> <token-id>
qn endpoint security referrer add <endpoint-id> https://example.com
qn endpoint security referrer remove <endpoint-id> <referrer-id>
qn endpoint security ip add <endpoint-id> 203.0.113.10
qn endpoint security ip remove <endpoint-id> <ip-id>
qn endpoint security jwt add <endpoint-id> --kid key-1 --name "prod jwt" --public-key-file public.pem
qn endpoint security jwt remove <endpoint-id> <jwt-id>
qn endpoint security domain-mask add <endpoint-id> rpc.example.com
qn endpoint security domain-mask remove <endpoint-id> <domain-mask-id>
qn endpoint security request-filter create <endpoint-id> --method eth_getBalance
qn endpoint security request-filter update <endpoint-id> <request-filter-id> --methods eth_getBalance,eth_call
qn endpoint security request-filter remove <endpoint-id> <request-filter-id>
qn endpoint security ip-header set <endpoint-id> X-Forwarded-For
qn endpoint security ip-header remove <endpoint-id>

qn endpoint rate-limit get <endpoint-id>
qn endpoint rate-limit set <endpoint-id> --rps 100 --rpm 5000
qn endpoint rate-limit delete-override <endpoint-id> <override-id>
qn endpoint rate-limit method-list <endpoint-id>
qn endpoint rate-limit method-create <endpoint-id> --interval minute --method eth_getLogs --rate 120
qn endpoint rate-limit method-update <endpoint-id> <method-rate-limit-id> --status disabled
qn endpoint rate-limit method-delete <endpoint-id> <method-rate-limit-id>
```

Security removals, disabling protection toggles, and rate-limit deletes are destructive or protection-lowering operations. Inspect `qn endpoint security show <endpoint-id>` and `qn endpoint rate-limit get <endpoint-id>` first.

### Team Management

Read team state with a Viewer or Admin API key:

```bash
qn team list
qn team show <team-id>
qn team endpoints <team-id>
```

Team create, delete, endpoint association, and member management commands mutate account access. Use an Admin API key and get explicit confirmation before running them.

```bash
qn team create --name "Protocol Ops"
qn team set-endpoints <team-id> <endpoint-id-1> <endpoint-id-2>
qn team delete <team-id>
```

`qn team set-endpoints` replaces the full endpoint association set for the team. Pass every endpoint that should remain associated; omitting an existing endpoint removes it from that team.

Manage team members with `qn team member`:

```bash
qn team member invite <team-id> --email alice@example.com --full-name "Alice Example" --role viewer
qn team member resend <team-id> <user-id>
qn team member remove <team-id> <user-id>
qn team member remove <team-id> <user-id> --destroy-user
```

Invite roles are `admin`, `viewer`, and `billing`. `--full-name` is required when inviting a new user. `--destroy-user` deletes the user account instead of only removing team membership; treat it as destructive.

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
qn stream update <stream-id> --name "renamed-stream" --notification-email ops@example.com
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

Webhook lifecycle and update commands:

```bash
qn webhook update <webhook-id> --name "wallet alerts v2" --url https://example.com/new-webhook --compression none
qn webhook update-template <webhook-id> --template evm-contract-events --contract 0xContract --event-hash 0xEventHash
qn webhook pause <webhook-id>
qn webhook activate <webhook-id>
qn webhook delete <webhook-id>
```

`qn webhook update` changes name, notification email, destination, security token, or compression without changing the filter template. `qn webhook update-template` changes template arguments and accepts the same template/list flags as `create`.

### Key-Value Store

Sets store one value per key:

```bash
qn kv set put mykey myvalue
qn kv set get mykey
qn kv set list
qn kv set delete mykey
qn kv set bulk --add threshold=750000 --delete old_threshold
```

Lists store ordered values under one key:

```bash
qn kv list list
qn kv list create allowlist 0xabc 0xdef
qn kv list append allowlist 0x123
qn kv list contains allowlist 0xabc
qn kv list get allowlist
qn kv list remove-item allowlist 0x123
qn kv list update allowlist --add 0x456 --remove 0xabc
qn kv list delete allowlist
```

### SQL Explorer

```bash
qn sql query "SELECT * FROM hyperliquid_trades LIMIT 10" --cluster-id hyperliquid-core-mainnet
qn sql schema hyperliquid-core-mainnet
qn sql query "SELECT * FROM hyperliquid_trades LIMIT 100" --cluster-id hyperliquid-core-mainnet --format json
qn sql query --file query.sql --cluster-id hyperliquid-core-mainnet
qn sql query --file - --cluster-id hyperliquid-core-mainnet
```

### Account Operations

```bash
qn usage summary --from 7d
qn usage by-endpoint --from 7d
qn usage by-method --from 7d
qn usage by-chain --from 7d
qn usage by-tag --from 7d
qn metrics account --period day --metric credits_over_time
qn metrics endpoint <endpoint-id> --period day --metric response_status_breakdown
qn chain list
qn chain credits ethereum
qn billing invoices
qn billing payments
qn completions zsh
```

`qn chain credits <chain>` returns the per-method API credit costs for that chain, using the same chain slugs returned by `qn chain list`. Use it when an agent needs account-aware credit costs instead of hardcoding method multipliers.

## Confirmation Behavior

Commands that delete, archive, pause, revoke, or otherwise change resources prompt before acting. In non-interactive environments, keep examples read-only by default or pass the documented prompt bypass only after explicit user confirmation. This includes the first-run Tooling Access auto-enable prompt on `qn rpc call` and `qn tooling-access disable`; pass `-y`/`--yes` to proceed non-interactively.

## Documentation

- **CLI Docs**: https://www.quicknode.com/docs/quicknode-cli
- **CLI Examples**: https://www.quicknode.com/docs/quicknode-cli/examples
