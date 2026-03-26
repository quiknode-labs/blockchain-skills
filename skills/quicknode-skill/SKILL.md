---
name: quicknode-skill
description: Quicknode blockchain infrastructure including RPC endpoints (80+ chains), Streams (real-time data), Webhooks, IPFS storage, Marketplace Add-ons (Token API, NFT API, DeFi tools), Solana DAS API (Digital Asset Standard), Key-Value Store, gRPC streaming (Yellowstone for Solana, Hypercore for Hyperliquid), SQL Explorer (direct SQL access to indexed blockchain data), and x402 pay-per-request RPC. Use when setting up blockchain infrastructure, configuring real-time data pipelines, processing blockchain events, storing data on IPFS, using Quicknode-specific APIs, querying Solana NFTs/tokens/compressed assets, persisting state with Key-Value Store, querying blockchain data with SQL, analyzing trading data, or building low-latency gRPC streams. Triggers on mentions of Quicknode, Streams, qn_ methods, IPFS pinning, Quicknode add-ons, DAS API, Digital Asset Standard, compressed NFT, cNFT, getAssetsByOwner, searchAssets, Key-Value Store, KV store, qnLib, Yellowstone, gRPC, Geyser, Hypercore, Hyperliquid, HYPE, SQL Explorer, SQL query, blockchain data, trading data, indexed data, evm, rpc, ethereum, blockchain, solana, or x402.
---

# Quicknode Blockchain Infrastructure

## Intake Questions
- Which chain and network should Quicknode target?
- Is this read-only or should I create infrastructure (streams, webhooks, IPFS writes)?
- Does this require real-time streaming (gRPC/Yellowstone/Hypercore) or standard RPC?
- What endpoint or API key should I use (default: `QUICKNODE_RPC_URL`, optional `QUICKNODE_WSS_URL` / `QUICKNODE_API_KEY`)?
- Any constraints (latency, regions, throughput, destinations)?

## Safety Defaults
- Default to testnet/devnet when a network is not specified.
- Prefer read-only operations and dry runs before creating resources.
- Never ask for or accept private keys or secret keys.

## Confirm Before Write
- Require explicit confirmation before creating or modifying Streams, Webhooks, or IPFS uploads.
- If confirmation is missing, return the exact API payload for review.

## Quick Reference

| Product | Description | Use Case |
|---------|-------------|----------|
| **RPC Endpoints** | High-performance blockchain access | dApp backend, wallet interactions |
| **Streams** | Real-time & historical blockchain data pipelines | Event monitoring, analytics, indexing |
| **Webhooks** | Event-driven notifications | Alerts, transaction monitoring |
| **IPFS** | Decentralized file storage | NFT metadata, asset hosting |
| **Add-ons** | Enhanced blockchain APIs | Token balances, NFT data, DeFi |
| **DAS API** | Solana Digital Asset Standard (add-on) | NFT/token queries, compressed NFTs, asset search |
| **Yellowstone gRPC** | Solana Geyser streaming (add-on) | Real-time account, transaction, slot data |
| **Hypercore** | Hyperliquid gRPC/JSON-RPC/WS (beta) | Trades, orders, book updates, blocks, TWAP, events, writer actions |
| **SQL Explorer** | Direct SQL access to indexed blockchain data | Trading analytics, historical queries, market analysis |
| **Admin API** | REST API for account management | Endpoint CRUD, usage monitoring, billing |
| **Key-Value Store** | Serverless key-value and list storage (beta) | Persistent state for Streams, dynamic address lists |
| **x402** | Pay-per-request RPC via USDC micropayments | Keyless RPC access, AI agents, pay-as-you-go |

## RPC Endpoints

Quicknode provides low-latency RPC endpoints for 80+ blockchain networks.

### Endpoint Setup

```typescript
// EVM chains (ethers.js)
import { JsonRpcProvider } from 'ethers';
const provider = new JsonRpcProvider(process.env.QUICKNODE_RPC_URL!);

// EVM chains (viem)
import { createPublicClient, http } from 'viem';
import { mainnet } from 'viem/chains';
const client = createPublicClient({
  chain: mainnet,
  transport: http(process.env.QUICKNODE_RPC_URL!),
});

// Solana
import { createSolanaRpc } from '@solana/kit';
const rpc = createSolanaRpc(process.env.QUICKNODE_RPC_URL!);
```

### Authentication

Quicknode endpoints include authentication in the URL:
```
https://{ENDPOINT_NAME}.{NETWORK}.quiknode.pro/{API_KEY}/
```

For additional security, enable JWT authentication or IP allowlisting in the Quicknode dashboard.

### Supported Networks

| Category | Networks |
|----------|----------|
| **EVM** | Ethereum, Polygon, Arbitrum, Optimism, Base, BSC, Avalanche, Fantom, zkSync, Scroll, Linea, Hyperliquid EVM (HyperEVM) |
| **Non-EVM** | Solana, Bitcoin, NEAR, Stacks, Cosmos, Sei, Aptos, Sui, TON, Hyperliquid (HyperCore) |

Not exhaustive. Full list: https://www.quicknode.com/chains

### Rate Limits & Plans

As of 2026-02-02. Verify current limits in Quicknode docs before sizing a production system.

| Plan | Requests/sec | Credits/month |
|------|-------------|---------------|
| Free Trial | 15 | 10M |
| Build | 50 | 80M |
| Accelerate | 125 | 450M |
| Scale | 250 | 950M |
| Business | 500 | 2B |

See [references/rpc-reference.md](references/rpc-reference.md) for complete RPC documentation including method tables for EVM, Solana, and Bitcoin chains, WebSocket patterns, and batch request examples.

## Streams

Real-time & historical blockchain data pipelines that filter, transform, and deliver data to your destinations.

### Stream Types

| Type | Data | Use Case |
|------|------|----------|
| **Block** | Full block data | Block explorers, analytics |
| **Transaction** | Transaction details | Tx monitoring, indexing |
| **Logs** | Contract events | DeFi tracking, NFT sales, token transfers |
| **Receipt** | Transaction receipts | Gas analysis, status tracking |

### Quick Setup

1. Create stream in Quicknode dashboard
2. Select network and data type
3. Add filter function (JavaScript)
4. Configure destination (webhook, S3, database)

### Basic Filter Function

See [references/streams-reference.md](references/streams-reference.md) for filter examples and full Streams documentation.

## Webhooks

Event-driven notifications for blockchain activity.

### Webhooks vs Streams

| Feature | Webhooks | Streams |
|---------|----------|---------|
| **Setup** | Simple | More configuration |
| **Filtering** | Address/event-based | Custom JavaScript |
| **Destinations** | HTTP endpoint only | Webhook, S3, Postgres, Azure |
| **Processing** | Basic | Full transformation |
| **Use Case** | Simple alerts | Complex pipelines |

### Webhook Setup

See [references/webhooks-reference.md](references/webhooks-reference.md) for API examples and full Webhooks documentation.

## IPFS Storage

Decentralized file storage with Quicknode's IPFS gateway.

See [references/ipfs-reference.md](references/ipfs-reference.md) for upload examples, metadata examples, and complete IPFS documentation.


## Marketplace Add-ons

Enhanced APIs available through Quicknode's marketplace.

### Token API (Ethereum)

```javascript
// Get all token balances for an address
const response = await fetch(process.env.QUICKNODE_RPC_URL!, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    method: 'qn_getWalletTokenBalance',
    params: [{ wallet: '0x...', contracts: [] }]
  })
});
```

### NFT API (Ethereum)

```javascript
// Fetch NFTs owned by address
const response = await fetch(process.env.QUICKNODE_RPC_URL!, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    method: 'qn_fetchNFTs',
    params: [{ wallet: '0x...', page: 1, perPage: 10 }]
  })
});
```

### Solana Priority Fee API

```javascript
// Get recommended priority fees
const response = await rpc.request('qn_estimatePriorityFees', {
  last_n_blocks: 100,
  account: 'YOUR_ACCOUNT'
}).send();
```

### Metis - Jupiter Swap API

```typescript
// Using Metis - Jupiter Swap API
import { createJupiterApiClient } from '@jup-ag/api';

const jupiterApi = createJupiterApiClient({
  basePath: `${process.env.QUICKNODE_METIS_URL}`
});

const quote = await jupiterApi.quoteGet({
  inputMint: 'So11111111111111111111111111111111111111112',
  outputMint: 'EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v',
  amount: 1000000000,
  slippageBps: 50
});

const swapResult = await jupiterApi.swapPost({
  swapRequest: {
    quoteResponse: quote,
    userPublicKey: 'YourPubkey...'
  }
});
```

See [references/marketplace-addons.md](references/marketplace-addons.md) for complete Add-ons documentation.

## Solana DAS API (Digital Asset Standard)

Comprehensive API for querying Solana digital assets — standard NFTs, compressed NFTs (cNFTs), fungible tokens, MPL Core Assets, and Token 2022 Assets. Available as a Marketplace add-on (Metaplex DAS API).

**Docs:** https://www.quicknode.com/docs/solana/solana-das-api

### Quick Setup

```javascript
// Get all assets owned by a wallet
const response = await fetch(process.env.QUICKNODE_RPC_URL, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    jsonrpc: '2.0',
    id: 1,
    method: 'getAssetsByOwner',
    params: {
      ownerAddress: 'E645TckHQnDcavVv92Etc6xSWQaq8zzPtPRGBheviRAk',
      limit: 10,
      options: { showFungible: true, showCollectionMetadata: true }
    }
  })
});
const { result } = await response.json();
// result.total — total assets
// result.items — array of asset metadata
// result.cursor — for pagination
```

### Available Methods

| Method | Description |
|--------|-------------|
| `getAsset` | Get metadata for a single asset |
| `getAssets` | Get metadata for multiple assets |
| `getAssetProof` | Get Merkle proof for a compressed asset |
| `getAssetProofs` | Get Merkle proofs for multiple assets |
| `getAssetsByAuthority` | List assets by authority |
| `getAssetsByCreator` | List assets by creator |
| `getAssetsByGroup` | List assets by group (e.g., collection) |
| `getAssetsByOwner` | List assets by wallet owner |
| `getAssetSignatures` | Transaction signatures for compressed assets |
| `getTokenAccounts` | Token accounts by mint or owner |
| `getNftEditions` | Edition details of a master NFT |
| `searchAssets` | Search assets with flexible filters |

See [references/solana-das-api-reference.md](references/solana-das-api-reference.md) for complete DAS API documentation with all methods, parameters, and examples.

## Yellowstone gRPC (Solana)

High-performance Solana Geyser plugin for real-time blockchain data streaming via gRPC. Available as a Marketplace add-on.

### Quick Setup

```typescript
import Client, { CommitmentLevel } from "@triton-one/yellowstone-grpc";

// Derive from HTTP URL: https://example.solana-mainnet.quiknode.pro/TOKEN/
const client = new Client(
  "https://example.solana-mainnet.quiknode.pro:10000",
  "TOKEN",
  {}
);

const stream = await client.subscribe();
stream.on("data", (data) => {
  if (data.transaction) console.log("Tx:", data.transaction);
});

stream.write({
  transactions: {
    txn_filter: {
      vote: false,
      failed: false,
      accountInclude: ["PROGRAM_PUBKEY"],
      accountExclude: [],
      accountRequired: [],
    },
  },
  accounts: {},
  slots: {},
  blocks: {},
  blocksMeta: {},
  transactionsStatus: {},
  entry: {},
  accountsDataSlice: [],
  commitment: CommitmentLevel.CONFIRMED,
});
```

### Filter Types

| Filter | Description |
|--------|-------------|
| **accounts** | Account data changes by pubkey, owner, or data pattern |
| **transactions** | Transaction events with vote/failure/account filters |
| **transactionsStatus** | Lightweight transaction status updates |
| **slots** | Slot progression and status changes |
| **blocks** | Full block data with optional tx/account inclusion |
| **blocksMeta** | Block metadata without full contents |
| **entry** | PoH entry updates |

See [references/yellowstone-grpc-reference.md](references/yellowstone-grpc-reference.md) for complete Yellowstone gRPC documentation.

## HyperCore (Hyperliquid)

Quicknode's data delivery infrastructure for the Hyperliquid L1 chain. Provides gRPC, JSON-RPC, WebSocket, and Info API access. Currently in public beta.

### Access Methods

| Method | Path / Port | Use Case |
|--------|-------------|----------|
| **Info API** | `/info` (POST) | 50+ methods for market data, positions, orders |
| **JSON-RPC** | `/hypercore` (POST) | Block queries (`hl_getBlock`, `hl_getBatchBlocks`) |
| **WebSocket** | `/hypercore/ws` | Real-time subscriptions (`hl_subscribe`) |
| **gRPC** | Port 10000 | Lowest-latency streaming for trades, orders, books |

### gRPC Stream Types

| Stream | Volume | Description |
|--------|--------|-------------|
| **TRADES** | High | Execution data: coin, price, size, side, fees |
| **ORDERS** | Very High | Order lifecycle with 18+ status types |
| **BOOK_UPDATES** | Very High | L2 order book diffs |
| **TWAP** | Low | Time-weighted average price order updates |
| **EVENTS** | High | Ledger updates, funding, deposits, withdrawals |
| **BLOCKS** | Extreme | Raw HyperCore blocks (gRPC only) |
| **WRITER_ACTIONS** | Low | System-level token transfers |

### HyperEVM

| Path | Debug/Trace | Archive | Use Case |
|------|-------------|---------|----------|
| `/evm` | No | Partial | Standard EVM operations |
| `/nanoreth` | Yes | Extended | Debug, trace, WebSocket subscriptions |

See [references/hypercore-hyperliquid-reference.md](references/hypercore-hyperliquid-reference.md) for complete HyperCore and Hyperliquid documentation.

## SQL Explorer

Direct SQL access to indexed blockchain data without requiring infrastructure. Query billions of rows of on-chain data using standard SQL syntax and receive results in seconds.

**Current Coverage:** Hyperliquid (HyperCore) with 37 tables and 371.9 billion+ rows

**Docs:** https://www.quicknode.com/docs/sql-explorer

### What It Is

SQL Explorer provides infrastructure-free SQL access to indexed blockchain data. Instead of running your own indexer or database, query pre-indexed data through a REST API using standard SQL.

### Use Cases

| Use Case | Description |
|----------|-------------|
| **Trading Analytics** | Analyze trades, orders, fills, liquidations across time periods |
| **Market Research** | Query funding rates, open interest, volume patterns |
| **Position Tracking** | Monitor positions, PnL, and exposure across accounts |
| **Historical Analysis** | Time-series analysis with billions of rows of historical data |
| **Infrastructure Monitoring** | Track blocks, transactions, system actions |
| **Builder Analytics** | Analyze builder fills, fees, and transaction activity |

### Quick Setup

```bash
# Authentication via API key in header
curl -X POST 'https://api.quicknode.com/sql/rest/v1/query' \
  -H 'x-api-key: YOUR_API_KEY' \
  -H 'Content-Type: application/json' \
  -d '{
    "query": "SELECT * FROM hyperliquid_trades ORDER BY block_time DESC LIMIT 10",
    "clusterId": "hyperliquid-core-mainnet"
  }'
```

### Available Tables (Hyperliquid - 37 Tables)

**Trading:**
- `hyperliquid_trades` — Individual trades with buyer/seller details (888.2M rows)
- `hyperliquid_orders` — Order placements, cancellations, modifications (158.0B rows)
- `hyperliquid_fills` — Trade fills with execution details (2.2B rows)
- `hyperliquid_order_book_diffs` — Order book changes (71.8B rows)

**Markets:**
- `hyperliquid_perpetual_markets` — Perpetual market configurations (457 rows)
- `hyperliquid_spot_markets` — Spot market configurations (449 rows)
- `hyperliquid_perpetual_market_contexts` — Market snapshots with funding, OI, prices (223.7M rows)
- `hyperliquid_oracle_prices` — Oracle price feeds (31.2K rows)

**Positions & Funding:**
- `hyperliquid_funding` — Funding payments and rates (905.9M rows)
- `hyperliquid_liquidations_daily` — Daily liquidation statistics (23.6K rows)
- `hyperliquid_funding_summary_hourly` — Hourly funding summaries (991.3K rows)

**Infrastructure:**
- `hyperliquid_blocks` — Block-level data (249.2M rows)
- `hyperliquid_transactions` — Transaction data (137.1B rows)
- `hyperliquid_system_actions` — HyperVM/HyperCore bridging (798.2K rows)

**Metrics:**
- `hyperliquid_market_volume_hourly` — Hourly trading volume and OHLC (1.8M rows)
- `hyperliquid_metrics_dex_overview` — Daily DEX metrics by coin (106.1K rows)
- `hyperliquid_metrics_overview` — Platform-wide aggregate metrics (367 rows)

**Builder:**
- `hyperliquid_builder_fills` — Builder-specific fill events (153.6M rows)
- `hyperliquid_builder_transactions` — Builder transaction activity (109.9M rows)
- `hyperliquid_builder_labels` — Builder labels and metadata (295 rows)

**Accounts:**
- `hyperliquid_sub_accounts` — Sub-account relationships (2.7M rows)
- `hyperliquid_summaries` — Account summaries (1.1M rows)
- `hyperliquid_ledger_updates` — Ledger state changes (11.6M rows)
- `hyperliquid_asset_transfers` — Asset transfers (9.1M rows)

**Staking:**
- `hyperliquid_staking_events` — Staking, unstaking, rewards (209.4K rows)
- `hyperliquid_validator_rewards` — Validator rewards (7.6M rows)
- `hyperliquid_delegator_rewards` — Delegator rewards (42.5M rows)

**Other:**
- `hyperliquid_dex_trades` — DEX trades with execution details
- `hyperliquid_clearinghouse_states` — Clearinghouse snapshots (19.4M rows)
- `hyperliquid_spot_clearinghouse_states` — Spot clearinghouse snapshots (130.6M rows)
- `hyperliquid_display_names` — User display names (15.1K rows)
- `hyperliquid_agents` — Agent configurations (1.9M rows)
- `hyperliquid_bridge` — Bridge events between HyperEVM and HyperCore (393.8K rows)
- `hyperliquid_register_referral` — Referral registrations (32.9K rows)
- `hyperliquid_set_referrer` — Referrer assignments (533.4K rows)
- `hyperliquid_twap_statuses` — TWAP order tracking (758.9K rows)
- `hyperliquid_vault_equities` — Vault equity values (11.8M rows)

### Common Query Patterns

**Recent Trades:**
```sql
SELECT timestamp, coin, side, price, size, buyer_address, seller_address
FROM hyperliquid_trades
WHERE block_time > now() - INTERVAL 1 HOUR
ORDER BY block_number DESC
LIMIT 100
```

**Trading Volume by Coin (Last 7 Days):**
```sql
SELECT coin, sum(price * size) as total_volume
FROM hyperliquid_trades
WHERE block_time >= today() - INTERVAL 7 DAY
GROUP BY coin
ORDER BY total_volume DESC
```

**Liquidation Analysis:**
```sql
SELECT day, coin, liquidation_count, liquidated_volume
FROM hyperliquid_liquidations_daily
WHERE day >= today() - INTERVAL 30 DAY
ORDER BY liquidated_volume DESC
```

**Hourly Funding Rates:**
```sql
SELECT coin, hour, avg_funding_rate, total_funding
FROM hyperliquid_funding_summary_hourly
WHERE hour >= now() - INTERVAL 24 HOUR
ORDER BY hour DESC
```

### Query Optimization

- **Partitioning:** All tables partitioned monthly by `toYYYYMM(block_time)` — filter on time ranges for best performance
- **Sort Keys:** Each table has optimized sort keys (see schema reference) — filter on these columns first
- **LIMIT:** Use LIMIT for exploratory queries to reduce data scanned
- **Time Filters:** Always include time-based filters (e.g., `WHERE block_time > now() - INTERVAL 1 DAY`)

### SQL Syntax Support

- Standard SQL functions: `toDateTime()`, `round()`, `countIf()`, `sum()`, `avg()`
- Aggregations: `SUM`, `AVG`, `COUNT`, `MIN`, `MAX`, `GROUP BY`
- Window functions for time-series analysis
- Subqueries and CTEs (Common Table Expressions)
- JOINs across multiple tables
- ORDER BY and LIMIT for result control

### Response Format

```json
{
  "data": [
    { "timestamp": "2025-03-25 10:30:45", "coin": "BTC", "price": 50000.5 }
  ],
  "meta": [
    { "name": "timestamp", "type": "DateTime" },
    { "name": "coin", "type": "String" },
    { "name": "price", "type": "Float64" }
  ],
  "rows": 1,
  "statistics": {
    "elapsed": 0.156,
    "rows_read": 1000000,
    "bytes_read": 50000000
  }
}
```

### Pre-Built Queries

SQL Explorer includes 26 pre-built queries across 10 categories:
- Trading (4): Recent trades, volume by coin, trading pairs, largest trades
- Activity (3): Active traders, new traders, trading activity
- Fills (2): Recent fills, user fills
- Orders (2): Recent orders, order activity
- Funding (2): Funding rates, funding payments
- Infrastructure (3): Recent blocks, transactions, system actions
- Ledger (3): Ledger updates, deposits, withdrawals
- Markets (4): Perpetual markets, spot markets, market contexts, open interest
- Builders (1): Builder activity
- Staking (2): Staking events, validator rewards

See [references/sql-explorer.md](references/sql-explorer.md) for complete table schemas, all pre-built queries, and detailed examples.

## Quicknode SDK

Official JavaScript/TypeScript SDK for Quicknode services.

### Installation

```bash
npm install @quicknode/sdk
```

### Basic Usage

```typescript
import { Core } from '@quicknode/sdk';

const core = new Core({
  endpointUrl: process.env.QUICKNODE_RPC_URL!,
});

// Token API
const balances = await core.client.qn_getWalletTokenBalance({
  wallet: '0x...',
});

// NFT API
const nfts = await core.client.qn_fetchNFTs({
  wallet: '0x...',
  page: 1,
  perPage: 10,
});
```

See [references/sdk-reference.md](references/sdk-reference.md) for complete SDK documentation.

## Admin API

REST API for programmatic management of Quicknode endpoints, usage, rate limits, security, billing, and teams. Enables infrastructure-as-code workflows.

### Quick Reference

| Resource | Methods | Endpoint |
|----------|---------|----------|
| Chains | GET | `/v0/chains` |
| Endpoints | GET, POST, PATCH, DELETE | `/v0/endpoints` |
| Metrics | GET | `/v0/endpoints/{id}/metrics` |
| Rate Limits | GET, POST, PUT | `/v0/endpoints/{id}/method-rate-limits`, `/v0/endpoints/{id}/rate-limits` |
| Security | GET | `/v0/endpoints/{id}/security_options` |
| Usage | GET | `/v0/usage/rpc`, `by-endpoint`, `by-method`, `by-chain` |
| Billing | GET | `/v0/billing/invoices` |
| Teams | GET | `/v0/teams` |

### Authentication

All requests use the `x-api-key` header against `https://api.quicknode.com/v0/`.

```typescript
const QN_API_KEY = process.env.QUICKNODE_API_KEY!;

const res = await fetch('https://api.quicknode.com/v0/endpoints', {
  headers: { 'x-api-key': QN_API_KEY },
});
const endpoints = await res.json();
```

See [references/admin-api-reference.md](references/admin-api-reference.md) for full Admin API documentation including endpoint CRUD, usage monitoring, rate limit configuration, security options, billing, and teams.

## Key-Value Store (Beta)

Serverless storage for lists and key-value sets, primarily accessed from within Streams filter functions via the `qnLib` helper library. Also available via REST API.

### Stream Integration (qnLib)

**List operations** — manage lists of items (e.g., wallet addresses):
- `qnLib.qnUpsertList` — create or update a list
- `qnLib.qnAddListItem` — add item to a list
- `qnLib.qnRemoveListItem` — remove item from a list
- `qnLib.qnContainsListItems` — batch membership check
- `qnLib.qnDeleteList` — delete a list

**Set operations** — manage key-value pairs:
- `qnLib.qnAddSet` — create a key-value set
- `qnLib.qnGetSet` — retrieve value by key
- `qnLib.qnBulkSets` — bulk create/remove sets
- `qnLib.qnDeleteSet` — delete a set

Docs: https://www.quicknode.com/docs/key-value-store

## x402 (Pay-Per-Request RPC)

Pay-per-request RPC access via USDC micropayments on Base. No API key required — authenticate with Sign-In with Ethereum (SIWE), purchase credits, and access 140+ chain endpoints.

### Quick Setup

```typescript
import { wrapFetch } from "@x402/fetch";
import { createWalletClient, http } from "viem";
import { privateKeyToAccount } from "viem/accounts";
import { base } from "viem/chains";

const account = privateKeyToAccount(process.env.PRIVATE_KEY as `0x${string}`);
const walletClient = createWalletClient({
  account,
  chain: base,
  transport: http(),
});

// Wrap fetch to auto-handle 402 payments
const x402Fetch = wrapFetch(fetch, walletClient);

// Use like normal fetch — payments are handled automatically
const response = await x402Fetch("https://x402.quicknode.com/ethereum-mainnet", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({
    jsonrpc: "2.0",
    method: "eth_blockNumber",
    params: [],
    id: 1,
  }),
});
```

See [references/x402-reference.md](references/x402-reference.md) for complete x402 documentation including SIWE authentication, credit management, and the `@x402/fetch` wrapper.

## Common Patterns

### Multi-Chain dApp Setup

```typescript
import { Core } from '@quicknode/sdk';

const chains = {
  ethereum: new Core({
    endpointUrl: 'https://YOUR_ETH_ENDPOINT.quiknode.pro/KEY/'
  }),
  polygon: new Core({
    endpointUrl: 'https://YOUR_POLYGON_ENDPOINT.quiknode.pro/KEY/'
  }),
  arbitrum: new Core({
    endpointUrl: 'https://YOUR_ARB_ENDPOINT.quiknode.pro/KEY/'
  }),
};

// Use appropriate chain
const ethBalance = await chains.ethereum.client.getBalance({ address: '0x...' });
```

### Real-Time Transaction Monitoring

1. **Create Stream** filtering for your contract address
2. **Add Filter Function** to extract relevant events
3. **Configure Webhook** destination to your server
4. **Process Events** in your backend

### NFT Collection Indexing

1. **Use Streams** to capture mint/transfer events
2. **Store in Database** via PostgreSQL destination
3. **Query via Add-ons** for current ownership data
4. **Query via SDK** for custom API endpoints

### Real-Time Solana Monitoring with Yellowstone gRPC

1. **Connect via gRPC** on port 10000 with your auth token
2. **Subscribe to transactions** filtering by program or account
3. **Process updates** in real-time via the streaming interface
4. **Implement reconnection** with exponential backoff

### Hyperliquid Trading Data Pipeline

1. **Connect via gRPC** on port 10000 for lowest-latency data
2. **Subscribe to TRADES/ORDERS** streams with coin filters
3. **Process events** — handle ~12 blocks/sec throughput
4. **Use Info API** (`/info`) for account state and market metadata

### Historical Trading Analysis with SQL Explorer

1. **Query historical trades** — `SELECT * FROM hyperliquid_trades WHERE block_time > ...`
2. **Aggregate metrics** — Use `GROUP BY` for volume, counts, averages
3. **Join tables** — Combine trades, orders, funding for comprehensive analysis
4. **Export data** — Results as JSON for downstream processing

## Best Practices

### RPC
- Use WebSocket for subscriptions and real-time data
- Implement retry logic with exponential backoff
- Cache responses when data doesn't change frequently
- Use batch requests to reduce API calls

### Streams
- Start with narrow filters, expand as needed
- Test filter functions locally before deployment
- Streams will automatically retry on failures
- Monitor stream health via dashboard

### Security
- Store API keys in environment variables
- Enable IP allowlisting for production
- Use JWT authentication for sensitive operations
- Rotate API keys periodically

### gRPC
- Enable zstd compression to reduce bandwidth (up to 70% for Hyperliquid)
- Implement reconnection logic with exponential backoff — streams can drop
- Use narrow filters (specific accounts, coins, or programs) to minimize data volume
- Set appropriate commitment levels (Yellowstone: CONFIRMED for most use cases, FINALIZED for irreversibility)
- Send keepalive pings (every 10s for Yellowstone, every 30s for Hypercore) to maintain connections

### SQL Explorer
- Always filter by time ranges for partition pruning (`WHERE block_time > ...`)
- Use LIMIT on exploratory queries to reduce data scanned
- Leverage sort keys (check schema reference) for optimal query performance
- Start with pre-built queries and customize for your needs
- Monitor query statistics (rows_read, bytes_read) to optimize performance

## Documentation Links

### Quicknode Products
- **Main Docs**: https://www.quicknode.com/docs/
- **Streams**: https://www.quicknode.com/docs/streams
- **Webhooks**: https://www.quicknode.com/docs/webhooks
- **IPFS**: https://www.quicknode.com/docs/ipfs
- **SDK**: https://www.quicknode.com/docs/quicknode-sdk
- **Admin API**: https://www.quicknode.com/docs/admin-api
- **DAS API (Solana)**: https://www.quicknode.com/docs/solana/solana-das-api
- **Yellowstone gRPC**: https://www.quicknode.com/docs/solana/yellowstone-grpc/overview
- **Hyperliquid**: https://www.quicknode.com/docs/hyperliquid
- **Hyperliquid gRPC**: https://www.quicknode.com/docs/hyperliquid/grpc-api
- **SQL Explorer**: https://www.quicknode.com/docs/sql-explorer
- **SQL Explorer REST API**: https://www.quicknode.com/docs/sql-explorer/rest-api/overview
- **Hyperliquid Queries**: https://www.quicknode.com/docs/sql-explorer/rest-api/hyperliquid-queries
- **Key-Value Store**: https://www.quicknode.com/docs/key-value-store
- **x402**: https://x402.quicknode.com

### Chain-Specific Docs
- **Ethereum**: https://www.quicknode.com/docs/ethereum
- **Solana**: https://www.quicknode.com/docs/solana
- **Polygon**: https://www.quicknode.com/docs/polygon
- **Arbitrum**: https://www.quicknode.com/docs/arbitrum
- **Base**: https://www.quicknode.com/docs/base
- **Optimism**: https://www.quicknode.com/docs/optimism
- **Avalanche**: https://www.quicknode.com/docs/avalanche
- **BNB Smart Chain**: https://www.quicknode.com/docs/bnb-smart-chain
- **Hyperliquid**: https://www.quicknode.com/docs/hyperliquid

### LLM-Optimized Documentation
- **Platform Overview (llms.txt)**: https://www.quicknode.com/llms.txt — High-level index of all Quicknode products, chains, guides, and solutions
- **Docs Index (llms.txt)**: https://www.quicknode.com/docs/llms.txt — Per-chain and per-product documentation index (links to `https://www.quicknode.com/docs/{chain-or-product}/llms.txt`)
- **x402 (llms.txt)**: https://x402.quicknode.com/llms.txt

### Additional Resources
- **Quicknode Guides**: https://www.quicknode.com/guides
- **SDK Reference**: https://www.quicknode.com/docs/quicknode-sdk
- **Marketplace**: https://marketplace.quicknode.com/
- **Sample App Library**: https://www.quicknode.com/sample-app-library
- **Guide Examples Repo**: https://github.com/quiknode-labs/qn-guide-examples
