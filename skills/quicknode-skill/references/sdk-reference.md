# Quicknode SDK Reference

The Quicknode SDK is a unified client for Quicknode product APIs. Use it when an application, script, or agent needs typed access to Admin API, Streams, Webhooks, Key-Value Store, and SQL Explorer from one shared Quicknode API key.

**Docs:** https://www.quicknode.com/docs/quicknode-sdk

## When to Use the SDK

- Use the SDK for Quicknode product workflows: provision endpoints, inspect usage, manage Streams/Webhooks, store KV state, and run SQL Explorer queries.
- Use direct endpoint URLs plus chain libraries for blockchain RPC calls (`ethers`, `viem`, `@solana/kit`, `@solana/web3.js`, Bitcoin JSON-RPC helpers, etc.).
- Use the CLI for terminal workflows and CI scripts where shell commands are simpler than application code.

## Packages

| Language | Package | Install |
|----------|---------|---------|
| Node.js / TypeScript | `@quicknode/sdk` 3.6.0+ | `npm install @quicknode/sdk` |
| Python | `quicknode-sdk` 0.6.0+ | `pip install quicknode-sdk` |
| Rust | `quicknode-sdk` 0.6.0+ | `cargo add quicknode-sdk --features rust` |
| Ruby | `quicknode_sdk` 0.6.0+ | `gem install quicknode_sdk` |

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

## Other Languages

Python, Rust, and Ruby expose the same product-client model through language-native method names:

- Python: `QuicknodeSdk.from_env()`
- Rust: `QuicknodeSdk::from_env()?`
- Ruby: `QuicknodeSdk::SDK.from_env`

Use the official SDK examples for current syntax in each language. The account-info and API-credit methods are available as language-native equivalents of `account_info` / `accountInfo` and `get_api_credits` / `getApiCredits`.

## Platform Notes

The SDK uses a shared Rust core with native language bindings. Published prebuilt targets include Linux glibc, Linux musl, and Apple Silicon macOS. Browsers, Windows native runtime, Intel macOS, and older Linux distributions are not supported by the native packages. Use WSL2 on Windows.

## Documentation

- **SDK Overview**: https://www.quicknode.com/docs/quicknode-sdk
- **QuickStart**: https://www.quicknode.com/docs/quicknode-sdk/quick-start
- **Examples**: https://www.quicknode.com/docs/quicknode-sdk/examples
- **npm Package**: https://www.npmjs.com/package/@quicknode/sdk
