# Quicknode SDK Reference

The Quicknode SDK is a unified client for Quicknode product APIs. Use it when an application, script, or agent needs typed access to Admin API, Streams, Webhooks, Key-Value Store, and SQL Explorer from one shared Quicknode API key.

**Docs:** https://www.quicknode.com/docs/sdk

## When to Use the SDK

- Use the SDK for Quicknode product workflows: provision endpoints, inspect usage, manage Streams/Webhooks, store KV state, and run SQL Explorer queries.
- Use the SDK's `rpc` client (Node 3.7.0+, other languages 0.7.0+) for blockchain RPC calls to any supported network via Tooling Access — no endpoint setup required, JWTs auto-mint and refresh. Chain-specific libraries (`ethers`, `viem`, `@solana/kit`, `@solana/web3.js`, Bitcoin JSON-RPC helpers, etc.) still apply for typed chain abstractions or a dedicated endpoint.
- Use the SDK's micropayment lane (Node 3.8.0+, other languages 0.8.0+) to pay for RPC calls with stablecoins through x402 or MPP, with no Quicknode account or API key. See [RPC Micropayments](#rpc-micropayments).
- Use the CLI for terminal workflows and CI scripts where shell commands are simpler than application code.

## Packages

| Language | Package | Install |
|----------|---------|---------|
| Node.js / TypeScript | `@quicknode/sdk` 3.7.0+ (3.8.0+ for micropayments) | `npm install @quicknode/sdk` |
| Python | `quicknode-sdk` 0.7.0+ (0.8.0+ for micropayments) | `pip install quicknode-sdk` |
| Rust | `quicknode-sdk` 0.7.0+ (0.8.0+ for micropayments) | `cargo add quicknode-sdk --features rust` |
| Ruby | `quicknode_sdk` 0.7.0+ (0.8.0+ for micropayments) | `gem install quicknode_sdk` |

## Authentication

Create an API key in the Quicknode dashboard, then set:

```bash
export QN_SDK__API_KEY="YOUR_API_KEY"
```

Optional base URL overrides:

| Environment variable | Default |
|----------------------|---------|
| `QN_SDK__ADMIN__BASE_URL` | `https://api.quicknode.com/v0/` |
| `QN_SDK__STREAMS__BASE_URL` | `https://api.quicknode.com/streams/rest/v1/` |
| `QN_SDK__WEBHOOKS__BASE_URL` | `https://api.quicknode.com/webhooks/rest/v1/` |
| `QN_SDK__KVSTORE__BASE_URL` | `https://api.quicknode.com/kv/rest/v1/` |
| `QN_SDK__SQL__BASE_URL` | `https://api.quicknode.com/sql/rest/v1/` |
| `QN_SDK__HTTP__TIMEOUT_SECS` | `30` |

## Product Clients

| Client | Purpose |
|--------|---------|
| `admin` | Account info, per-chain API credits, endpoints, teams, usage, logs, billing, metrics, security, rate limits, tags |
| `rpc` | Direct JSON-RPC calls to any supported network via Tooling Access (auto-minted, auto-refreshed JWTs), or via x402/MPP micropayments |
| `streams` | Stream listing, creation, lifecycle, and filter testing |
| `webhooks` | Webhook listing, template creation, lifecycle, and enabled counts |
| `kvstore` | Sets and lists for persisted state, watchlists, cursors, and filters |
| `sql` | SQL Explorer queries and schema retrieval |

## Node.js / TypeScript Examples

### Admin API

```typescript
import { QuicknodeSdk } from "@quicknode/sdk";

const qn = QuicknodeSdk.fromEnv();

const endpoints = await qn.admin.getEndpoints({
  limit: 20,
  sortBy: "created_at",
  sortDirection: "desc",
});

for (const endpoint of endpoints.data) {
  console.log(endpoint.id, endpoint.name, endpoint.status);
}
```

### RPC & Tooling Access

Tooling Access provisions one multichain, read-only endpoint per account, with short-lived (10-minute) JWTs minted and refreshed automatically. `qn.rpc` calls it directly — no endpoint URL or token to manage.

```typescript
import { QuicknodeSdk, RpcError } from "@quicknode/sdk";

const qn = QuicknodeSdk.fromEnv();

// Enable once (admin role + eligible plan). Idempotent.
const status = await qn.admin.toolingAccessStatus();
if (!status.enabled) {
  await qn.admin.enableToolingAccess();
}

// params defaults to []; pass an array (positional) or object (by name).
const blockNumber = await qn.rpc.call("eth_blockNumber");
const balance = await qn.rpc.call("eth_getBalance", ["0xabc...", "latest"]);

// Multichain: seed the network map from admin.getEndpointUrls, then select
// a network by its multichain_urls key as the 3rd argument.
const endpointId = "your-endpoint-id";
const urls = await qn.admin.getEndpointUrls(endpointId);
const networks = Object.fromEntries(
  Object.entries(urls.data?.multichainUrls ?? {}).map(([key, url]) => [
    key,
    url.httpUrl,
  ]),
);
qn.rpc.setNetworks(networks);
const slot = await qn.rpc.call("getSlot", [], "solana-mainnet");

// Custom endpoint URL (4th arg): bypasses Tooling Access and the JWT
// entirely. Mutually exclusive with network — a custom URL isn't multichain-routed.
const block = await qn.rpc.call(
  "eth_blockNumber",
  [],
  undefined,
  "https://example.solana-mainnet.quiknode.pro/TOKEN/"
);

// A JSON-RPC error member throws as RpcError (.code, .message).
try {
  await qn.rpc.call("eth_getBalance", ["bad"]);
} catch (e) {
  if (e instanceof RpcError) console.error(e.code, e.message);
}

// Disable
// Account-wide — cuts off blockchain access for all Quicknode developer
// tooling. Confirm with the user first.
// await qn.admin.disableToolingAccess();
```

### Account Info and API Credits

```typescript
import { QuicknodeSdk } from "@quicknode/sdk";

const qn = QuicknodeSdk.fromEnv();

const account = await qn.admin.accountInfo();
console.log(account.data.id, account.data.name, account.data.billingVersion);

const credits = await qn.admin.getApiCredits("ethereum");
for (const row of credits.data.slice(0, 5)) {
  console.log(row.method, row.credits);
}
```

`accountInfo()` calls the Admin API account-info endpoint and returns details such as account id, name, creation timestamp, billing version, and current subscription. `getApiCredits(chain)` calls the per-chain API credits endpoint and returns `{ method, credits }` rows for the supplied chain slug. Use `listChains()` to discover valid chain slugs.

### Streams

```typescript
import { QuicknodeSdk, StreamDataset } from "@quicknode/sdk";

const qn = QuicknodeSdk.fromEnv();

const streams = await qn.streams.listStreams({ limit: 10 });
const enabled = await qn.streams.getEnabledCount();

const filterFunction = Buffer.from(`
function main(data) {
  return data;
}
`).toString("base64");

const test = await qn.streams.testFilter({
  network: "ethereum-mainnet",
  dataset: StreamDataset.Block,
  block: "17811625",
  filterFunction,
});

console.log({ streams: streams.pageInfo.total, enabled: enabled.total, result: test.result });
```

### Webhooks

```typescript
import { QuicknodeSdk } from "@quicknode/sdk";

const qn = QuicknodeSdk.fromEnv();

const webhooks = await qn.webhooks.listWebhooks({ limit: 10, offset: 0 });
const enabled = await qn.webhooks.getEnabledCount();

console.log({ count: webhooks.data.length, enabled: enabled.total });
```

### Key-Value Store

```typescript
import { QuicknodeSdk } from "@quicknode/sdk";

const qn = QuicknodeSdk.fromEnv();

const sets = await qn.kvstore.getSets({ limit: 10 });
const lists = await qn.kvstore.getLists({ limit: 10 });

if (sets.data.length > 0) {
  const value = await qn.kvstore.getSet(sets.data[0].key);
  console.log(value.value);
}

console.log({ sets: sets.data.length, lists: lists.data.keys.length });
```

### SQL Explorer

```typescript
import { QuicknodeSdk } from "@quicknode/sdk";

const qn = QuicknodeSdk.fromEnv();

const result = await qn.sql.query(
  "SELECT action_type, user FROM hyperliquid_system_actions LIMIT 3",
  "hyperliquid-core-mainnet"
);

const schema = await qn.sql.getSchema("hyperliquid-core-mainnet");

console.log(`${result.rows} rows, ${result.credits} credits`);
console.log(schema);
```

## RPC Micropayments

Node 3.8.0+, other languages 0.8.0+. The `rpc` client can pay for calls with stablecoins through the x402 or MPP gateway instead of an API key. Set a payment wallet and the SDK handles the HTTP `402 Payment Required` challenge, signs the payment, and resends the request. This lane needs no Quicknode account, API key, or provisioned endpoint — construct the SDK without an API key.

| Payment path | Entry point | Best for |
|--------------|-------------|----------|
| x402 per request | `rpc.call` / `rpc.callWithReceipt` with `scheme: "x402"` | One-off EVM or Solana payments |
| MPP per request | `rpc.call` / `rpc.callWithReceipt` with `scheme: "mpp"` | One-off payments on Tempo |
| x402 credit drawdown | `gatewayAuthenticate` → `gatewayBuyCredits` → `gatewayDrawdownCall` | Prepay once, then spend credits over many calls |
| MPP payment channel | `mppOpen` → `mppSessionCall` | Deposit once, then authorize calls with off-chain vouchers |

This lane moves real funds. Get explicit user confirmation first, keep only the funds needed for RPC payments in the payment wallet, and never log the payment config — it holds the raw private key, and the binding object's own `Debug`/inspect output is not redacted.

The payment network is independent of the chain you query: Base Sepolia USDC can pay for an Ethereum Mainnet call.

### Payment Configuration

Set these fields under `rpc.payment`. Node uses camelCase; Python, Rust, and Ruby use snake_case (`pay_network`, `max_amount`, `svm_rpc_url`).

| Field | Description |
|-------|-------------|
| `scheme` | `"x402"` or `"mpp"` |
| `key` | Raw private key. EVM/Tempo: hex, with or without `0x`. Solana: base58 64-byte secret key |
| `payNetwork` | CAIP-2 payment network, e.g. `eip155:84532` (x402/EVM), `solana:5eykt4…` (x402/Solana), `eip155:42431` (MPP/Tempo testnet) |
| `asset` | Token address or Solana mint matching an offered payment option |
| `maxAmount` | **Required.** Spend ceiling in the asset's integer base units, as a string |
| `svmRpcUrl` | Optional. Solana RPC used to build x402/Solana payments. Set this at any real volume — the public default rate-limits aggressively |

```typescript
import { QuicknodeSdk } from "@quicknode/sdk";

// No API key needed for the payment lane.
const qn = new QuicknodeSdk({
  rpc: {
    payment: {
      scheme: "x402",
      key: process.env.QN_PAYMENT_KEY!,
      payNetwork: "eip155:84532", // Base Sepolia
      asset: "0x036CbD53842c5426634e7929541eC2318f3dCF7e", // Base Sepolia USDC
      maxAmount: "10000", // 0.01 USDC ceiling, not the amount sent
    },
  },
});

// Payment is automatic on 402.
const blockNumber = await qn.rpc.call("eth_blockNumber", [], "ethereum-mainnet");

// The wallet address the lane pays from.
console.log(qn.rpc.paymentAddress());

// Settlement metadata: { result, paymentReceipt }. paymentReceipt is
// { method, status, timestamp, reference } on MPP and null on x402.
const withReceipt = await qn.rpc.callWithReceipt("eth_blockNumber", [], "ethereum-mainnet");
```

Switch to MPP by setting `scheme: "mpp"` with an offered Tempo payment network and asset. `maxAmount` is a ceiling, not the amount sent: the SDK skips offers above it and refuses to sign one.

Rust gates payments behind cargo features — `payments` for x402 on EVM, plus `payments-svm` for x402 on Solana and `payments-tempo` for MPP. The Tempo feature needs Rust 1.93+. The Node, Python, and Ruby packages ship with payments compiled in.

```toml
quicknode-sdk = { version = "0.8", features = ["payments", "payments-svm", "payments-tempo"] }
```

### Generate A Payment Wallet

`generatePaymentWallet` creates a keypair offline and returns `{ address, chain, key }`. The key is returned once and is never stored or recoverable — persist it securely before continuing.

```typescript
import { generatePaymentWallet } from "@quicknode/sdk";

const wallet = generatePaymentWallet("evm"); // "evm" or "svm"
console.log("Fund this address:", wallet.address);
// Persist wallet.key now. It cannot be recovered later.
```

Use `evm` for x402 on EVM and for MPP on Tempo; use `svm` for x402 on Solana. Other bindings expose it as `generate_payment_wallet`.

### x402 Credit Drawdown

Authenticate the wallet once, buy prepaid credits, then spend one credit per successful response — no per-call signing. Persist the session between processes.

```typescript
const session = await qn.rpc.gatewayAuthenticate();

// Buy credits (signs a payment), then check the balance.
await qn.rpc.gatewayBuyCredits(session, "ethereum-mainnet");
const balance = await qn.rpc.gatewayCredits(session);
console.log("Credits:", balance.credits);

const blockNumber = await qn.rpc.gatewayDrawdownCall(
  "eth_blockNumber",
  session,
  "ethereum-mainnet"
);
```

`gatewayDrip(session)` funds the payment wallet from the Base Sepolia faucet, once per account, and returns the funding transaction hash rather than a credit balance. Fund Solana wallets separately.

### MPP Payment Channel

A channel deposits into escrow once, then authorizes calls with cumulative off-chain vouchers. The caller owns the channel state: advance `cumulativeSpent` only after a successful call and persist it immediately.

```typescript
const channel = await qn.rpc.mppOpen("1000000"); // deposit in base units

const newTotal = (
  BigInt(channel.cumulativeSpent) + BigInt(channel.perCall)
).toString();

const blockNumber = await qn.rpc.mppSessionCall(
  "eth_blockNumber",
  "ethereum-mainnet",
  channel,
  newTotal
);

// On success, persist the channel with cumulativeSpent set to newTotal.
```

`mppTopUp(channel, additionalDeposit)` adds funds. `mppClose(channel)` settles on-chain and refunds the unused deposit. `mppStatus(channel)` returns the gateway's view but consumes one request unit, because the gateway prices every session request — persist its updated cumulative spend too.

The gateway has no read-only channel endpoint, so it cannot reconstruct a lost local channel record. Losing the record means opening a new channel and forfeiting the old deposit until it is closed.

### Payment Errors

| Error | Meaning |
|-------|---------|
| `PaymentError` | Base class for the payment lane |
| `PaymentUnsupportedError` | The requested payment operation isn't available in this build (for example, a Rust binary compiled without the matching feature) |
| `PaymentRejectedError` | The gateway refused the payment; nothing settled |
| `PaymentIndeterminateError` | Outcome unknown — the payment may have settled. Check the wallet before retrying; never blind-retry a paid call |

## Other Languages

Python, Rust, and Ruby expose the same product-client model through language-native method names:

- Python: `QuicknodeSdk.from_env()`
- Rust: `QuicknodeSdk::from_env()?`
- Ruby: `QuicknodeSdk::SDK.from_env`

Use the official SDK examples for current syntax in each language. The account-info and API-credit methods are available as language-native equivalents of `account_info` / `accountInfo` and `get_api_credits` / `getApiCredits`.

## Platform Notes

The SDK uses a shared Rust core with native language bindings. Published prebuilt targets include Linux glibc, Linux musl, and Apple Silicon macOS. Browsers, Windows native runtime, Intel macOS, and older Linux distributions are not supported by the native packages. Use WSL2 on Windows.

## Documentation

- **SDK Overview**: https://www.quicknode.com/docs/sdk
- **QuickStart**: https://www.quicknode.com/docs/sdk/quick-start
- **RPC Micropayments**: https://www.quicknode.com/docs/sdk/micropayments
- **Examples**: https://www.quicknode.com/docs/sdk/examples
- **npm Package**: https://www.npmjs.com/package/@quicknode/sdk
