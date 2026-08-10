# Quicknode CLI Reference

The `qn` CLI manages Quicknode product APIs from a terminal, script, CI job, or agent shell. Use it for endpoints, Streams, Webhooks, Key-Value Store, SQL Explorer, teams, usage, billing, metrics, security workflows, and making JSON-RPC calls to any supported network via Tooling Access, with no endpoint setup required.

It also has a keyless paid lane: `qn wallet` plus `qn rpc call --x402`/`--mpp` pay for RPC calls with stablecoins, with no Quicknode account, API key, login, Tooling Access, or provisioned endpoint. See [Paid RPC (x402 and MPP)](#paid-rpc-x402-and-mpp).

**Docs:** https://www.quicknode.com/docs/cli

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

### Paid RPC (x402 and MPP)

Requires CLI v0.6.0+. `qn rpc call` can pay for a call with stablecoins through the x402 or MPP gateway instead of using the account's API key. This lane requires no Quicknode account, API key, login, Tooling Access, or provisioned endpoint — only a funded local wallet.

| Payment path | Command or flag | Best for |
|--------------|-----------------|----------|
| x402 per request | `qn rpc call --x402` | One-off EVM or Solana payments |
| MPP per request | `qn rpc call --mpp` | One-off payments on Tempo |
| x402 credit drawdown | `qn rpc x402` + `--x402-drawdown` | Prepay once, then spend credits over many calls |
| MPP payment channel | `qn rpc mpp` + `--mpp-session` | Deposit once, then authorize calls with off-chain vouchers |

These commands move real funds. Get explicit user confirmation first, use a dedicated minimally funded wallet, and set `--max-amount` on every signing action. Paid calls cannot use `--endpoint-url`.

#### Payment Wallets

`qn wallet` manages the local key store for the paid lane. No API key or login required.

```bash
qn wallet generate --vm evm --name payer
qn wallet generate --vm svm --name sol-payer
qn wallet list
qn wallet show payer
qn wallet rm payer
```

`--vm evm` generates a secp256k1 wallet for x402 on EVM and for MPP on Tempo. `--vm svm` generates an ed25519 wallet for x402 on Solana. `--force` overwrites an existing wallet of the same name.

`generate` prints the address (with a QR to fund it on a terminal) and writes the raw key at `0600` under `<config-dir>/qn/wallets/`. Quicknode does not hold, back up, or recover it. `list` and `show` never print the key. `rm` is gated and irreversible — the stored key is the only copy.

The CLI never accepts a private key as a flag value, environment variable, or inline config value. To pay with an existing key, put it in a file and pass `--payment-key-file <PATH>` (or `-` to read stdin). Key precedence is `--payment-key-file` > `--payment-wallet` > `key_file` > `wallet` under `[rpc.payment]`.

#### Discover Networks And Payment Options

Both gateways expose discovery commands that need no API key (aliases: `networks`, `payments`).

```bash
qn rpc x402 supported-networks
qn rpc x402 supported-payments
qn rpc mpp supported-networks
qn rpc mpp supported-payments
```

Use a slug from `supported-networks` as `--network`. Use a row from `supported-payments` — it returns the payment network, asset symbol, and token address — as `--payment-network` and `--payment-asset`. Query these instead of hardcoding a network or asset list.

The query network and the payment network are independent. `--network ethereum-mainnet` with `--payment-network base-sepolia` pays on Base Sepolia for an Ethereum Mainnet read.

#### Pay Per Request

`--x402` signs a stablecoin payment per call on an EVM or Solana payment network:

```bash
qn rpc call eth_getBlockByNumber '["latest", false]' \
  --network ethereum-mainnet \
  --x402 \
  --payment-wallet payer \
  --payment-network base-sepolia \
  --payment-asset USDC \
  --max-amount 1000
```

`--mpp` settles on Tempo using the same EVM key format. `--receipt` wraps stdout as `{"result": ..., "payment_receipt": ...}`:

```bash
qn rpc call eth_getBlockByNumber '["latest", false]' \
  --network ethereum-mainnet \
  --mpp \
  --receipt \
  --payment-wallet payer \
  --payment-network tempo-testnet \
  --payment-asset pathUSD \
  --max-amount 1000
```

The receipt carries the payment method, status, timestamp, and the settlement transaction hash in `reference`. It is non-null only on MPP; x402 returns `null`. Payment happens either way — `--receipt` only changes the output shape.

`--x402` and `--mpp` are mutually exclusive, and both require `--network` (the query chain, as the gateway's path slug). `--payment-asset` accepts a symbol like `USDC`, an EVM contract address, or a Solana mint.

`--max-amount` is a per-call spend ceiling in integer base units of the asset (`10000` = 0.01 USDC), not the amount sent. It has no built-in default. Offers above the ceiling are never signed; among those at or under it, the cheapest is paid.

For x402 on Solana, use an `svm` wallet with an offered Solana payment network and asset. Pass `--svm-rpc-url` at any real volume — the public Solana RPC the CLI otherwise falls back to rate-limits aggressively.

#### x402 Credit Drawdown

Drawdown authenticates the wallet once, buys prepaid credits, and spends one credit per successful response — no per-call signing.

```bash
qn rpc x402 drip \
  --payment-wallet payer \
  --payment-network base-sepolia

qn rpc x402 buy-credits \
  --network ethereum-mainnet \
  --payment-wallet payer \
  --payment-network base-sepolia \
  --payment-asset USDC \
  --max-amount 10000000

qn rpc x402 balance \
  --payment-wallet payer \
  --payment-network base-sepolia

qn rpc call eth_blockNumber \
  --network ethereum-mainnet \
  --x402-drawdown \
  --payment-wallet payer
```

`drip` requests testnet credits from the Base Sepolia faucet, once per account; fund Solana wallets out of band. `drip` and `balance` (alias `credits`) authenticate the wallet but sign no payment, so they need only the wallet and payment network. `buy-credits` signs the purchase, requires the full payment flags, and is gated — pass `-y` in a script.

`qn rpc call --x402-drawdown` needs only the query network and the wallet; it takes no asset or spend ceiling, and the pay network defaults to the query network. Credits are not scoped to the network they were bought against and can pay for calls to any supported query network. The authenticated gateway session is cached at `0600` and refreshed automatically. Running out of credits surfaces an exit-2 error pointing back at `buy-credits`.

#### MPP Payment Channel

A channel deposits into a Tempo escrow once, then authorizes calls with cumulative off-chain EIP-712 vouchers.

```bash
qn rpc mpp open \
  --deposit 1000000 \
  --payment-wallet payer \
  --payment-network tempo-testnet \
  --payment-asset pathUSD \
  --max-amount 1000000

qn rpc call eth_blockNumber \
  --network ethereum-mainnet \
  --mpp-session \
  --payment-wallet payer \
  --payment-network tempo-testnet \
  --payment-asset pathUSD

qn rpc mpp status \
  --payment-wallet payer \
  --payment-network tempo-testnet \
  --payment-asset pathUSD

qn rpc mpp top-up \
  --deposit 500000 \
  --payment-wallet payer \
  --payment-network tempo-testnet \
  --payment-asset pathUSD \
  --max-amount 500000

qn rpc mpp close \
  --payment-wallet payer \
  --payment-network tempo-testnet \
  --payment-asset pathUSD
```

The lifecycle commands identify the channel by `--payment-network` and `--payment-asset` and take no query `--network`. `open`, `top-up`, and `close` sign on-chain actions, so they are gated and take `--max-amount`. `close` settles on-chain and refunds the unused deposit.

`status` reads the local record only. `status --verify` asks the gateway instead and spends one request unit from the channel. The channel record lives at `<config-dir>/qn/channels.toml`; the gateway has no read-only channel endpoint, so a lost record means opening a new channel.

`--mpp-session` needs an open channel plus the query `--network`. One channel can fund calls to any supported query network.

#### Payment Defaults In Config

Store reusable payment values under `[rpc.payment]` in `~/.config/qn/config.toml`:

```toml
[rpc.payment]
wallet = "payer"
payment_network = "base-sepolia"
payment_asset = "USDC"
max_amount = "10000"
```

`key_file` and `svm_rpc_url` are also accepted. Config supplies values but never activates a payment mode — every call still needs `--x402`, `--mpp`, `--x402-drawdown`, or `--mpp-session`.

#### Retry And Exit-Code Safety

Paid calls never auto-retry; `--retries` does not apply to them. On a per-request paid call, exit 2 means the gateway refused and nothing settled, while **exit 3 means the outcome is unknown and the wallet may already have been charged**. On exit 3, check the wallet before re-running — never blind-retry a paid call.

A drawdown call is single-attempt and draws one credit only on success; the one exception is a transparent re-auth when the cached session token expired, which draws nothing. A session call signs one cumulative voucher and is single-attempt.

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

Commands that delete, archive, pause, revoke, spend funds, or otherwise change resources prompt before acting. In non-interactive environments, keep examples read-only by default or pass the documented prompt bypass only after explicit user confirmation. This includes the first-run Tooling Access auto-enable prompt on `qn rpc call`, `qn tooling-access disable`, `qn wallet rm`, `qn rpc x402 buy-credits`, and `qn rpc mpp open`/`top-up`/`close`; pass `-y`/`--yes` to proceed non-interactively.

In a non-TTY, a gated command without `--yes` exits 5 before any request is sent, so nothing changes. `--no-input` forces fail-fast non-interactive behavior everywhere.

## Documentation

- **CLI Docs**: https://www.quicknode.com/docs/cli
- **CLI Examples**: https://www.quicknode.com/docs/cli/examples
- **RPC Micropayments**: https://www.quicknode.com/docs/cli/micropayments
