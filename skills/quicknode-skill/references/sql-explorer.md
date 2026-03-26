# Quicknode SQL Explorer Reference

Direct SQL access to indexed blockchain data without infrastructure. Query billions of rows of on-chain data using standard SQL syntax through a REST API.

## Architecture

```
SQL Query → REST API → ClickHouse → Indexed Blockchain Data → JSON Response
            (Auth)     (Query Engine)  (371.9B+ rows)
```

## Quick Start

### Getting Your API Key

To use SQL Explorer REST API, you need a Quicknode API key:

1. Log in to your [Quicknode Dashboard](https://dashboard.quicknode.com)
2. Click on the **Profile icon** in the top right corner
3. Select **API Keys** from the dropdown menu
4. Either:
   - Click **Create API Key** to create a new key for SQL Explorer, OR
   - Use an existing API key that has SQL Explorer enabled

**Important:** This is the same unified API key system used across all Quicknode products (RPC, Streams, IPFS, SQL Explorer, etc.).

### Complete Working Example

Here's a complete cURL example that works out of the box (just replace `YOUR_API_KEY`):

```bash
curl -X POST 'https://api.quicknode.com/sql/rest/v1/query' \
  -H 'accept: application/json' \
  -H 'Content-Type: application/json' \
  -H 'x-api-key: YOUR_API_KEY' \
  -d '{
    "query": "SELECT toDateTime(block_time) AS block_time, tid, coin, side, px AS price, sz AS size, user, fee FROM hyperliquid_trades ORDER BY block_time DESC LIMIT 10",
    "clusterId": "hyperliquid-core-mainnet"
  }'
```

**Request Breakdown:**

| Component | Value | Description |
|-----------|-------|-------------|
| **Endpoint** | `https://api.quicknode.com/sql/rest/v1/query` | SQL Explorer REST API endpoint |
| **Method** | `POST` | HTTP method |
| **Header** | `x-api-key: YOUR_API_KEY` | Your Quicknode API key for authentication |
| **Header** | `Content-Type: application/json` | Request content type |
| **Body** | `query` | SQL query string to execute |
| **Body** | `clusterId` | Blockchain network identifier |

**Supported Cluster IDs:**
- `hyperliquid-core-mainnet` - Hyperliquid (HyperCore) Mainnet

**Example Response:**

```json
{
  "data": [
    {
      "block_time": "2025-03-26 10:30:45",
      "tid": 123456789,
      "coin": "BTC",
      "side": "buy",
      "price": "65000.50",
      "size": "1.5",
      "user": "0x1234567890abcdef1234567890abcdef12345678",
      "fee": "4.875"
    },
    {
      "block_time": "2025-03-26 10:30:42",
      "tid": 123456788,
      "coin": "ETH",
      "side": "sell",
      "price": "3250.75",
      "size": "10.2",
      "user": "0xabcdefabcdefabcdefabcdefabcdefabcdefabcd",
      "fee": "2.456"
    }
  ],
  "meta": [
    {"name": "block_time", "type": "DateTime"},
    {"name": "tid", "type": "UInt64"},
    {"name": "coin", "type": "String"},
    {"name": "side", "type": "String"},
    {"name": "price", "type": "String"},
    {"name": "size", "type": "String"},
    {"name": "user", "type": "FixedString(42)"},
    {"name": "fee", "type": "String"}
  ],
  "rows": 10,
  "statistics": {
    "elapsed": 0.089,
    "rows_read": 47400000,
    "bytes_read": 2800000
  }
}
```

**Response Fields:**

- **`data`**: Array of result rows (each row is an object with column names as keys)
- **`meta`**: Column metadata with names and ClickHouse data types
- **`rows`**: Total number of rows returned
- **`statistics.elapsed`**: Query execution time in seconds
- **`statistics.rows_read`**: Number of rows scanned by the query
- **`statistics.bytes_read`**: Bytes of data scanned by the query

### Testing Your Query

**Recommended workflow for agents:**

1. **Build and test in UI**: Use [SQL Explorer Dashboard](https://dashboard.quicknode.com/sql) to write and test your query
2. **Get API code**: Click the **API** button in the UI to generate the exact cURL command
3. **Execute programmatically**: Use the generated code in your application

This ensures queries work before integrating into code.

## Agent Usage Guide

**For AI Agents using this reference:**

1. **Query Selection**: Use the "Common Use Case Mappings" table to quickly find relevant queries for user requests
2. **Parameter Substitution**: Look for `Parameters:` line under each query - replace placeholders like `0xYOUR_ADDRESS`, `BTC`, `YOUR_VALIDATOR` with actual values
3. **Keywords**: Each query lists keywords - use these for semantic search and query discovery
4. **Sort Keys**: Queries using sort key columns (marked with ⚡) run significantly faster - prioritize these when possible
5. **Time Filters**: Always add time filters for better performance and cost efficiency
6. **Sample Responses**: Check sample response format to understand data structure before processing
7. **Custom Queries**: If no pre-built query matches, use "Common Query Patterns" section as templates
8. **Authentication**: Always include `x-api-key` header - see "Getting Your API Key" section above

## Supported Networks

| Chain | Network | Tables | Total Rows | Coverage |
|-------|---------|--------|------------|----------|
| Hyperliquid (HyperCore) | Mainnet | 37 | 371.9B+ | Trades, orders, fills, funding, order book diffs, perpetual markets, spot markets, blocks, transactions, system actions, builder activity, staking, ledger, clearinghouse states, oracle prices, referrals, sub-accounts, vault equities, agents, bridge events, display names, hourly metrics, daily metrics |

Cluster ID: `hyperliquid-core-mainnet`

## REST API

### Endpoint

```
POST https://api.quicknode.com/sql/rest/v1/query
```

### Authentication

Include API key in `x-api-key` header with every request.

### Request Format

```json
{
  "query": "SELECT * FROM hyperliquid_trades ORDER BY block_time DESC LIMIT 10",
  "clusterId": "hyperliquid-core-mainnet"
}
```

### Response Format

```json
{
  "data": [
    {
      "block_time": "2025-03-25 10:30:45",
      "action_type": "swap",
      "user": "0x1234...",
      "evm_transaction_hash": "0xabcd..."
    }
  ],
  "meta": [
    {"name": "block_time", "type": "DateTime"},
    {"name": "action_type", "type": "String"},
    {"name": "user", "type": "String"},
    {"name": "evm_transaction_hash", "type": "String"}
  ],
  "rows": 1,
  "statistics": {
    "elapsed": 0.156,
    "rows_read": 1000000,
    "bytes_read": 50000000
  }
}
```

## Available Tables (37)

### Trading Tables

#### hyperliquid_trades
**Rows:** 47.4B | **Sort Keys:** ⚡ block_number, tid, user | **Partition:** toYYYYMM(block_time)

Individual trade executions on Hyperliquid.

| Column | Type | Sort Key | Description |
|--------|------|----------|-------------|
| block_number | UInt64 | Yes | Block number |
| block_time | DateTime | No | Block timestamp |
| tid | UInt64 | Yes | Trade ID |
| time | DateTime | No | Trade timestamp |
| coin | String | No | Trading pair |
| side | String | No | Trade side (buy/sell) |
| px | String | No | Execution price |
| sz | String | No | Trade size |
| hash | FixedString(66) | No | Transaction hash |
| fee | String | No | Trading fee |
| fee_token | String | No | Fee token |
| crossed | Bool | No | Crossed spread |
| user | FixedString(42) | Yes | User address |
| dir | String | No | Direction |
| start_position | String | No | Starting position size |
| closed_pnl | String | No | Closed PnL |
| builder_address | Nullable(FixedString(42)) | No | Builder address |
| builder_fee | String | No | Builder fee |
| indexed_at | DateTime | No | Indexing timestamp |

#### hyperliquid_fills
**Rows:** 47.4B | **Sort Keys:** ⚡ block_number, time, user | **Partition:** toYYYYMM(block_time)

Order fills with execution details.

| Column | Type | Sort Key | Description |
|--------|------|----------|-------------|
| block_number | UInt64 | Yes | Block number |
| block_time | DateTime | No | Block timestamp |
| time | DateTime | Yes | Fill timestamp |
| coin | String | No | Trading pair |
| px | String | No | Fill price |
| sz | String | No | Fill size |
| side | String | No | Side (buy/sell) |
| user | FixedString(42) | Yes | User address |
| hash | FixedString(66) | No | Transaction hash |
| oid | UInt64 | No | Order ID |
| crossed | Bool | No | Crossed spread |
| fee | String | No | Trading fee |
| tid | UInt64 | No | Trade ID |
| start_position | String | No | Starting position |
| dir | String | No | Direction |
| closed_pnl | String | No | Closed PnL |
| fee_token | String | No | Fee token |
| builder_address | Nullable(FixedString(42)) | No | Builder address |
| builder_fee | String | No | Builder fee |
| liquidation_markup | String | No | Liquidation markup |
| indexed_at | DateTime | No | Indexing timestamp |

#### hyperliquid_orders
**Rows:** 52.2B | **Sort Keys:** ⚡ block_number, user, oid | **Partition:** toYYYYMM(block_time)

Order book orders and their status.

| Column | Type | Sort Key | Description |
|--------|------|----------|-------------|
| block_number | UInt64 | Yes | Block number |
| block_time | DateTime | No | Block timestamp |
| user | FixedString(42) | Yes | User address |
| coin | String | No | Trading pair |
| side | String | No | Order side |
| limit_px | String | No | Limit price |
| sz | String | No | Order size |
| oid | UInt64 | Yes | Order ID |
| timestamp | DateTime | No | Order timestamp |
| orig_sz | String | No | Original size |
| cloid | Nullable(String) | No | Client order ID |
| indexed_at | DateTime | No | Indexing timestamp |

### Market Tables

#### hyperliquid_perp_markets
**Rows:** 2.9M | **Sort Keys:** ⚡ block_number, name | **Partition:** toYYYYMM(snapshot_time)

Perpetual futures market data.

| Column | Type | Sort Key | Description |
|--------|------|----------|-------------|
| block_number | UInt64 | Yes | Block number |
| snapshot_time | DateTime | No | Snapshot timestamp |
| name | String | Yes | Market name |
| szDecimals | UInt8 | No | Size decimals |
| indexed_at | DateTime | No | Indexing timestamp |

#### hyperliquid_spot_markets
**Rows:** 11.6M | **Sort Keys:** ⚡ block_number, name | **Partition:** toYYYYMM(snapshot_time)

Spot market data.

| Column | Type | Sort Key | Description |
|--------|------|----------|-------------|
| block_number | UInt64 | Yes | Block number |
| snapshot_time | DateTime | No | Snapshot timestamp |
| name | String | Yes | Market name |
| tokens | Array(UInt32) | No | Token IDs |
| index | UInt32 | No | Market index |
| is_canonical | Bool | No | Canonical status |
| indexed_at | DateTime | No | Indexing timestamp |

#### hyperliquid_spot_token_supply
**Rows:** 5.9M | **Sort Keys:** ⚡ block_number, name | **Partition:** toYYYYMM(snapshot_time)

Spot token supply snapshots.

| Column | Type | Sort Key | Description |
|--------|------|----------|-------------|
| block_number | UInt64 | Yes | Block number |
| snapshot_time | DateTime | No | Snapshot timestamp |
| name | String | Yes | Token name |
| token | UInt32 | No | Token ID |
| total | String | No | Total supply |
| holds | String | No | Held amount |
| circulatingSupply | String | No | Circulating supply |
| indexed_at | DateTime | No | Indexing timestamp |

#### hyperliquid_order_book_diffs
**Rows:** 216.1B | **Sort Keys:** ⚡ block_number, coin | **Partition:** toYYYYMM(block_time)

Order book depth changes.

| Column | Type | Sort Key | Description |
|--------|------|----------|-------------|
| block_number | UInt64 | Yes | Block number |
| block_time | DateTime | No | Block timestamp |
| coin | String | Yes | Trading pair |
| time | DateTime | No | Update timestamp |
| side | String | No | Bid/ask side |
| px | String | No | Price level |
| sz | String | No | Size at level |
| indexed_at | DateTime | No | Indexing timestamp |

### Position & Funding Tables

#### hyperliquid_user_states
**Rows:** 30.9M | **Sort Keys:** ⚡ block_number, user | **Partition:** toYYYYMM(snapshot_time)

User account state snapshots.

| Column | Type | Sort Key | Description |
|--------|------|----------|-------------|
| block_number | UInt64 | Yes | Block number |
| snapshot_time | DateTime | No | Snapshot timestamp |
| user | FixedString(42) | Yes | User address |
| asset_positions | String | No | Asset positions JSON |
| cross_margin_summary | String | No | Cross margin summary JSON |
| margin_summary | String | No | Margin summary JSON |
| withdrawable | String | No | Withdrawable amount |
| indexed_at | DateTime | No | Indexing timestamp |

#### hyperliquid_user_funding
**Rows:** 1.5B | **Sort Keys:** ⚡ block_number, time, user | **Partition:** toYYYYMM(block_time)

User funding rate payments.

| Column | Type | Sort Key | Description |
|--------|------|----------|-------------|
| block_number | UInt64 | Yes | Block number |
| block_time | DateTime | No | Block timestamp |
| time | DateTime | Yes | Funding timestamp |
| user | FixedString(42) | Yes | User address |
| coin | String | No | Trading pair |
| usdc | String | No | USDC amount |
| szi | String | No | Size |
| funding_rate | String | No | Funding rate |
| hash | Nullable(FixedString(66)) | No | Transaction hash |
| indexed_at | DateTime | No | Indexing timestamp |

#### hyperliquid_liquidations
**Rows:** 212.4K | **Sort Keys:** ⚡ block_number, liquidated_user | **Partition:** toYYYYMM(block_time)

Liquidation events.

| Column | Type | Sort Key | Description |
|--------|------|----------|-------------|
| block_number | UInt64 | Yes | Block number |
| block_time | DateTime | No | Block timestamp |
| lid | UInt64 | No | Liquidation ID |
| liquidated_user | FixedString(42) | Yes | Liquidated address |
| liquidated_ntl_pos | String | No | Notional position |
| liquidated_account_value | String | No | Account value |
| indexed_at | DateTime | No | Indexing timestamp |

### Infrastructure Tables

#### hyperliquid_blocks
**Rows:** 5.2M | **Sort Keys:** ⚡ block_number | **Partition:** toYYYYMM(block_time)

Hyperliquid block data.

| Column | Type | Sort Key | Description |
|--------|------|----------|-------------|
| block_number | UInt64 | Yes | Block number |
| block_time | DateTime | No | Block timestamp |
| block_hash | FixedString(64) | No | Block hash |
| tx_count | UInt32 | No | Transaction count |
| indexed_at | DateTime | No | Indexing timestamp |

#### hyperliquid_transactions
**Rows:** 29.3M | **Sort Keys:** ⚡ block_number, hash | **Partition:** toYYYYMM(block_time)

Transaction details.

| Column | Type | Sort Key | Description |
|--------|------|----------|-------------|
| block_number | UInt64 | Yes | Block number |
| block_time | DateTime | No | Block timestamp |
| hash | FixedString(66) | Yes | Transaction hash |
| user | FixedString(42) | No | User address |
| time | DateTime | No | Transaction timestamp |
| nonce | UInt64 | No | Transaction nonce |
| indexed_at | DateTime | No | Indexing timestamp |

#### hyperliquid_system_actions
**Rows:** 77.7M | **Sort Keys:** ⚡ block_number, action_type | **Partition:** toYYYYMM(block_time)

System-level actions and events.

| Column | Type | Sort Key | Description |
|--------|------|----------|-------------|
| block_number | UInt64 | Yes | Block number |
| block_time | DateTime | No | Block timestamp |
| action_type | String | Yes | Action type |
| user | Nullable(FixedString(42)) | No | User address |
| action_data | String | No | Action data JSON |
| evm_transaction_hash | Nullable(FixedString(66)) | No | EVM tx hash |
| indexed_at | DateTime | No | Indexing timestamp |

### Builder Tables

#### hyperliquid_builder_fills
**Rows:** 2.3M | **Sort Keys:** ⚡ block_number, builder_address | **Partition:** toYYYYMM(block_time)

Builder order fills.

| Column | Type | Sort Key | Description |
|--------|------|----------|-------------|
| block_number | UInt64 | Yes | Block number |
| block_time | DateTime | No | Block timestamp |
| time | DateTime | No | Fill timestamp |
| coin | String | No | Trading pair |
| px | String | No | Fill price |
| sz | String | No | Fill size |
| side | String | No | Side (buy/sell) |
| user | FixedString(42) | No | User address |
| hash | FixedString(66) | No | Transaction hash |
| oid | UInt64 | No | Order ID |
| crossed | Bool | No | Crossed spread |
| fee | String | No | Trading fee |
| tid | UInt64 | No | Trade ID |
| builder_address | FixedString(42) | Yes | Builder address |
| builder_fee | String | No | Builder fee |
| indexed_at | DateTime | No | Indexing timestamp |

#### hyperliquid_builder_transactions
**Rows:** 7.5M | **Sort Keys:** ⚡ block_number, builder | **Partition:** toYYYYMM(block_time)

Builder transaction activity.

| Column | Type | Sort Key | Description |
|--------|------|----------|-------------|
| block_number | UInt64 | Yes | Block number |
| block_time | DateTime | No | Block timestamp |
| builder | FixedString(42) | Yes | Builder address |
| user | FixedString(42) | No | User address |
| hash | FixedString(66) | No | Transaction hash |
| fee | String | No | Builder fee |
| indexed_at | DateTime | No | Indexing timestamp |

### Ledger Tables

#### hyperliquid_ledger_updates
**Rows:** 102.1M | **Sort Keys:** ⚡ block_number, user | **Partition:** toYYYYMM(block_time)

Account balance updates.

| Column | Type | Sort Key | Description |
|--------|------|----------|-------------|
| block_number | UInt64 | Yes | Block number |
| block_time | DateTime | No | Block timestamp |
| hash | FixedString(66) | No | Transaction hash |
| time | DateTime | No | Update timestamp |
| user | FixedString(42) | Yes | User address |
| delta | String | No | Balance change |
| indexed_at | DateTime | No | Indexing timestamp |

#### hyperliquid_sub_account_updates
**Rows:** 44.0K | **Sort Keys:** ⚡ block_number, user | **Partition:** toYYYYMM(block_time)

Sub-account updates.

| Column | Type | Sort Key | Description |
|--------|------|----------|-------------|
| block_number | UInt64 | Yes | Block number |
| block_time | DateTime | No | Block timestamp |
| time | DateTime | No | Update timestamp |
| user | FixedString(42) | Yes | User address |
| sub_account_user | FixedString(42) | No | Sub-account address |
| indexed_at | DateTime | No | Indexing timestamp |

### Staking Tables

#### hyperliquid_staking_delegations
**Rows:** 1.8M | **Sort Keys:** ⚡ block_number, validator_address, delegator_address | **Partition:** toYYYYMM(snapshot_time)

Validator delegation snapshots.

| Column | Type | Sort Key | Description |
|--------|------|----------|-------------|
| block_number | UInt64 | Yes | Block number |
| snapshot_time | DateTime | No | Snapshot timestamp |
| validator_address | String | Yes | Validator address |
| delegator_address | String | Yes | Delegator address |
| delegation_amount | String | No | Delegation amount |
| indexed_at | DateTime | No | Indexing timestamp |

#### hyperliquid_validators
**Rows:** 2.1K | **Sort Keys:** ⚡ block_number, validator_address | **Partition:** toYYYYMM(snapshot_time)

Validator information.

| Column | Type | Sort Key | Description |
|--------|------|----------|-------------|
| block_number | UInt64 | Yes | Block number |
| snapshot_time | DateTime | No | Snapshot timestamp |
| validator_address | String | Yes | Validator address |
| voting_power | String | No | Voting power |
| commission_rate | String | No | Commission rate |
| indexed_at | DateTime | No | Indexing timestamp |

### Clearinghouse Tables

#### hyperliquid_clearinghouse_states
**Rows:** 5.0M | **Sort Keys:** ⚡ block_number | **Partition:** toYYYYMM(snapshot_time)

Clearinghouse state snapshots.

| Column | Type | Sort Key | Description |
|--------|------|----------|-------------|
| block_number | UInt64 | Yes | Block number |
| snapshot_time | DateTime | No | Snapshot timestamp |
| state_data | String | No | State data JSON |
| indexed_at | DateTime | No | Indexing timestamp |

### Oracle & Price Tables

#### hyperliquid_oracle_prices
**Rows:** 10.1M | **Sort Keys:** ⚡ block_number, coin | **Partition:** toYYYYMM(snapshot_time)

Oracle price updates.

| Column | Type | Sort Key | Description |
|--------|------|----------|-------------|
| block_number | UInt64 | Yes | Block number |
| snapshot_time | DateTime | No | Snapshot timestamp |
| coin | String | Yes | Trading pair |
| price | String | No | Oracle price |
| indexed_at | DateTime | No | Indexing timestamp |

### Referral Tables

#### hyperliquid_referrals
**Rows:** 11.8M | **Sort Keys:** ⚡ block_number, user | **Partition:** toYYYYMM(snapshot_time)

Referral relationships.

| Column | Type | Sort Key | Description |
|--------|------|----------|-------------|
| block_number | UInt64 | Yes | Block number |
| snapshot_time | DateTime | No | Snapshot timestamp |
| user | FixedString(42) | Yes | User address |
| referrer | FixedString(42) | No | Referrer address |
| indexed_at | DateTime | No | Indexing timestamp |

### Vault Tables

#### hyperliquid_vault_equities
**Rows:** 114.8K | **Sort Keys:** ⚡ block_number, vault, user | **Partition:** toYYYYMM(snapshot_time)

Vault equity snapshots.

| Column | Type | Sort Key | Description |
|--------|------|----------|-------------|
| block_number | UInt64 | Yes | Block number |
| snapshot_time | DateTime | No | Snapshot timestamp |
| vault | FixedString(42) | Yes | Vault address |
| user | FixedString(42) | Yes | User address |
| user_vault_equity | String | No | User equity |
| total_vault_equity | String | No | Total vault equity |
| indexed_at | DateTime | No | Indexing timestamp |

### Other Tables

#### hyperliquid_agents
**Rows:** 1.9M | **Sort Keys:** ⚡ block_number, agent | **Partition:** toYYYYMM(snapshot_time)

Agent configurations.

| Column | Type | Sort Key | Description |
|--------|------|----------|-------------|
| block_number | UInt64 | Yes | Block number |
| snapshot_time | DateTime | No | Snapshot timestamp |
| agent | FixedString(42) | Yes | Agent address |
| user | FixedString(42) | No | Owner address |
| name | String | No | Agent name |
| valid_until | String | No | Validity period |
| indexed_at | DateTime | No | Indexing timestamp |

#### hyperliquid_bridge
**Rows:** 1.1M | **Sort Keys:** ⚡ block_number, user | **Partition:** toYYYYMM(block_time)

Bridge deposit and withdrawal events.

| Column | Type | Sort Key | Description |
|--------|------|----------|-------------|
| block_number | UInt64 | Yes | Block number |
| block_time | DateTime | No | Block timestamp |
| time | DateTime | No | Bridge timestamp |
| user | FixedString(42) | Yes | User address |
| usdc | String | No | USDC amount |
| hash | Nullable(FixedString(66)) | No | Transaction hash |
| indexed_at | DateTime | No | Indexing timestamp |

#### hyperliquid_display_names
**Rows:** 1.9M | **Sort Keys:** ⚡ block_number, user | **Partition:** toYYYYMM(snapshot_time)

User display names.

| Column | Type | Sort Key | Description |
|--------|------|----------|-------------|
| block_number | UInt64 | Yes | Block number |
| snapshot_time | DateTime | No | Snapshot timestamp |
| user | FixedString(42) | Yes | User address |
| name | String | No | Display name |
| indexed_at | DateTime | No | Indexing timestamp |

#### hyperliquid_funding_summary_hourly
**Rows:** 600.1K | **Sort Keys:** ⚡ hour, coin | **Partition:** toYYYYMM(hour)

Hourly funding rate aggregates.

| Column | Type | Sort Key | Description |
|--------|------|----------|-------------|
| hour | DateTime | Yes | Hour timestamp |
| coin | String | Yes | Trading pair |
| avg_funding_rate | Float64 | No | Average funding rate |
| total_volume | Float64 | No | Total volume |
| indexed_at | DateTime | No | Indexing timestamp |

#### hyperliquid_liquidations_daily
**Rows:** 20.1K | **Sort Keys:** ⚡ day | **Partition:** toYYYYMM(day)

Daily liquidation aggregates.

| Column | Type | Sort Key | Description |
|--------|------|----------|-------------|
| day | Date | Yes | Day date |
| total_liquidations | UInt64 | No | Liquidation count |
| total_volume | Float64 | No | Total volume liquidated |
| indexed_at | DateTime | No | Indexing timestamp |

#### hyperliquid_market_volume_hourly
**Rows:** 1.8M | **Sort Keys:** ⚡ hour, coin | **Partition:** toYYYYMM(hour)

Hourly market volume.

| Column | Type | Sort Key | Description |
|--------|------|----------|-------------|
| hour | DateTime | Yes | Hour timestamp |
| coin | String | Yes | Trading pair |
| volume | Float64 | No | Trading volume |
| trades | UInt64 | No | Trade count |
| indexed_at | DateTime | No | Indexing timestamp |

#### hyperliquid_metrics_dex_overview
**Rows:** 5.0K | **Sort Keys:** ⚡ block_number | **Partition:** toYYYYMM(snapshot_time)

DEX overview metrics.

| Column | Type | Sort Key | Description |
|--------|------|----------|-------------|
| block_number | UInt64 | Yes | Block number |
| snapshot_time | DateTime | No | Snapshot timestamp |
| total_volume | String | No | Total volume |
| daily_volume | String | No | Daily volume |
| user_count | UInt64 | No | User count |
| indexed_at | DateTime | No | Indexing timestamp |

#### hyperliquid_metrics_overview
**Rows:** 5.0K | **Sort Keys:** ⚡ block_number | **Partition:** toYYYYMM(snapshot_time)

Platform overview metrics.

| Column | Type | Sort Key | Description |
|--------|------|----------|-------------|
| block_number | UInt64 | Yes | Block number |
| snapshot_time | DateTime | No | Snapshot timestamp |
| metrics_data | String | No | Metrics JSON |
| indexed_at | DateTime | No | Indexing timestamp |

## Pre-Built Queries (26)

### Common Use Case Mappings

Quick lookup for common agent requests:

| User Request | Use This Query |
|--------------|----------------|
| "Show me whale trades" / "large trades" | Query #4: High Volume Trades |
| "What did user X trade?" / "user history" | Query #2: Trades by User |
| "Show recent BTC trades" / "market activity" | Query #3: Trades by Trading Pair |
| "What's the 24h volume?" / "market stats" | Query #6: Trading Volume by Pair |
| "Show me liquidations" / "forced closures" | Query #24: Recent Liquidations |
| "Builder performance" / "builder stats" | Query #5: Builder Activity, #21: Builder Fills Summary |
| "Funding rate history" / "funding payments" | Query #13: Recent Funding Payments |
| "Top traders" / "leaderboard" | Query #7: User Trading Stats |
| "Show me order book" / "market depth" | Query #20: Order Book Depth |
| "List all markets" / "available pairs" | Query #18: Perpetual Markets, #19: Spot Markets |
| "User balance changes" / "deposits/withdrawals" | Query #17: Ledger Updates by User |
| "Recent fills" / "executions" | Query #8: Recent Fills |
| "Active orders" / "open orders" | Query #12: Active Orders by User |
| "Validator info" / "staking" | Query #22: Top Validators, #23: Delegations |
| "Volume over time" / "time series" | Query #25: Hourly Market Volume |

### Quick Reference

| # | Query Name | Category | Keywords | Description |
|---|------------|----------|----------|-------------|
| 1 | Recent Trades | Trading | trades, latest, recent | Get most recent trades across all markets |
| 2 | Trades by User | Trading | user, trades, address, history | Get trade history for specific user address |
| 3 | Trades by Trading Pair | Trading | market, pair, coin, symbol | Get trades for specific trading pair |
| 4 | High Volume Trades | Trading | whale, large, volume, big | Monitor large trades above notional threshold |
| 5 | Builder Activity | Trading | builder, fees, stats | Aggregate builder trading statistics |
| 6 | Trading Volume by Pair | Trading | volume, 24h, markets | 24-hour volume aggregated by trading pair |
| 7 | User Trading Stats | Trading | user, statistics, pnl, summary | Weekly trading statistics per user |
| 8 | Recent Fills | Fills | fills, latest, executions | Most recent order fills |
| 9 | Fills by User | Fills | user, fills, address | Order fills for specific user |
| 10 | Crossed Fills | Fills | crossed, aggressive, taker | Fills that crossed the spread |
| 11 | Recent Orders | Orders | orders, latest, book | Most recent orders placed |
| 12 | Active Orders by User | Orders | user, orders, open | Active orders for specific user |
| 13 | Recent Funding Payments | Funding | funding, payments, rates | Latest funding rate payments |
| 14 | Funding by User | Funding | user, funding, address | Funding payments for specific user |
| 15 | Recent Blocks | Infrastructure | blocks, chain, latest | Latest Hyperliquid blocks |
| 16 | Recent System Actions | Infrastructure | system, events, actions | System-level actions and events |
| 17 | Ledger Updates by User | Ledger | ledger, balance, user | Balance updates for specific user |
| 18 | Perpetual Markets | Markets | perp, perpetual, markets, list | List all perpetual markets |
| 19 | Spot Markets | Markets | spot, markets, tokens, list | List all spot markets |
| 20 | Order Book Depth | Markets | orderbook, depth, liquidity | Order book depth for specific market |
| 21 | Builder Fills Summary | Builders | builder, fills, fees, stats | 24-hour builder fill statistics |
| 22 | Top Validators | Staking | validators, staking, voting | Top validators by voting power |
| 23 | Delegations by Validator | Staking | delegations, staking, validator | Delegations for specific validator |
| 24 | Recent Liquidations | Liquidations | liquidations, risk, forced | Recent liquidation events |
| 25 | Hourly Market Volume | Metrics | volume, hourly, timeseries | Hourly volume time series for market |
| 26 | Daily Liquidation Metrics | Metrics | liquidations, daily, aggregated | Daily liquidation aggregates |

### Trading Queries (7)

**1. Recent Trades** - Get most recent trades across all markets
**Keywords:** `trades`, `latest`, `recent`
**Parameters:** None
```sql
SELECT
  toDateTime(block_time) AS block_time,
  tid,
  coin,
  side,
  px AS price,
  sz AS size,
  user,
  fee
FROM hyperliquid_trades
ORDER BY block_time DESC
LIMIT 100
```

**Sample Response:**
```json
{
  "data": [
    {"block_time": "2025-03-26 10:30:45", "tid": 123456789, "coin": "BTC", "side": "buy", "price": "65000.5", "size": "1.5", "user": "0x1234...", "fee": "4.875"}
  ],
  "rows": 100,
  "statistics": {"elapsed": 0.08, "rows_read": 47400000, "bytes_read": 2500000}
}
```

**2. Trades by User** - Get trade history for specific user address
**Keywords:** `user`, `trades`, `address`, `history`, `pnl`
**Parameters:** `user` address (replace `0xYOUR_ADDRESS`)
```sql
SELECT
  toDateTime(time) AS time,
  coin,
  side,
  px AS price,
  sz AS size,
  fee,
  closed_pnl
FROM hyperliquid_trades
WHERE user = '0xYOUR_ADDRESS'
ORDER BY time DESC
LIMIT 100
```

**3. Trades by Trading Pair** - Get trades for specific trading pair/market
**Keywords:** `market`, `pair`, `coin`, `symbol`, `asset`
**Parameters:** `coin` name (replace `BTC` with desired pair)
```sql
SELECT
  toDateTime(time) AS time,
  side,
  px AS price,
  sz AS size,
  user,
  fee
FROM hyperliquid_trades
WHERE coin = 'BTC'
ORDER BY time DESC
LIMIT 100
```

**4. High Volume Trades** - Monitor large trades above notional threshold (whale tracking)
**Keywords:** `whale`, `large`, `volume`, `big`, `notional`, `threshold`
**Parameters:** Notional threshold (default: `100000`)
```sql
SELECT
  toDateTime(time) AS time,
  coin,
  side,
  px AS price,
  sz AS size,
  user,
  toFloat64(px) * toFloat64(sz) AS notional_value
FROM hyperliquid_trades
WHERE toFloat64(px) * toFloat64(sz) > 100000
ORDER BY time DESC
LIMIT 100
```

**Sample Response:**
```json
{
  "data": [
    {"time": "2025-03-26 10:25:12", "coin": "ETH", "side": "buy", "price": "3250.75", "size": "45.8", "user": "0xabcd...", "notional_value": 148884.35}
  ],
  "rows": 23,
  "statistics": {"elapsed": 0.12, "rows_read": 47400000, "bytes_read": 3200000}
}
```

**5. Builder Activity** - Aggregate builder trading statistics and fees earned
**Keywords:** `builder`, `fees`, `stats`, `aggregated`, `summary`
**Parameters:** None
```sql
SELECT
  builder_address,
  COUNT(*) AS trade_count,
  SUM(toFloat64(builder_fee)) AS total_fees,
  COUNT(DISTINCT user) AS unique_users
FROM hyperliquid_trades
WHERE builder_address IS NOT NULL
GROUP BY builder_address
ORDER BY total_fees DESC
LIMIT 50
```

**6. Trading Volume by Pair** - 24-hour trading volume aggregated by market
**Keywords:** `volume`, `24h`, `markets`, `aggregate`, `daily`
**Parameters:** Time interval (default: 24 hours)
```sql
SELECT
  coin,
  COUNT(*) AS trade_count,
  SUM(toFloat64(sz)) AS total_volume,
  AVG(toFloat64(px)) AS avg_price
FROM hyperliquid_trades
WHERE block_time >= now() - INTERVAL 24 HOUR
GROUP BY coin
ORDER BY total_volume DESC
```

**7. User Trading Stats** - Weekly trading statistics per user including volume, fees, PnL
**Keywords:** `user`, `statistics`, `pnl`, `summary`, `leaderboard`, `top`
**Parameters:** Time interval (default: 7 days)
```sql
SELECT
  user,
  COUNT(*) AS trade_count,
  SUM(toFloat64(sz)) AS total_volume,
  SUM(toFloat64(fee)) AS total_fees,
  SUM(toFloat64(closed_pnl)) AS total_pnl
FROM hyperliquid_trades
WHERE block_time >= now() - INTERVAL 7 DAY
GROUP BY user
ORDER BY total_volume DESC
LIMIT 100
```

### Fills Queries (3)

**8. Recent Fills** - Most recent order fills and executions
**Keywords:** `fills`, `latest`, `executions`, `recent`, `matched`
**Parameters:** None
```sql
SELECT
  toDateTime(time) AS time,
  coin,
  side,
  px AS price,
  sz AS size,
  user,
  oid AS order_id,
  fee
FROM hyperliquid_fills
ORDER BY time DESC
LIMIT 100
```

**9. Fills by User** - Order fills for specific user address
**Keywords:** `user`, `fills`, `address`, `history`, `executions`
**Parameters:** `user` address (replace `0xYOUR_ADDRESS`)
```sql
SELECT
  toDateTime(time) AS time,
  coin,
  side,
  px AS price,
  sz AS size,
  oid AS order_id,
  fee,
  closed_pnl
FROM hyperliquid_fills
WHERE user = '0xYOUR_ADDRESS'
ORDER BY time DESC
LIMIT 100
```

**10. Crossed Fills** - Fills that crossed the spread (aggressive/taker orders)
**Keywords:** `crossed`, `aggressive`, `taker`, `market`, `immediate`
**Parameters:** None
```sql
SELECT
  toDateTime(time) AS time,
  coin,
  side,
  px AS price,
  sz AS size,
  user,
  fee
FROM hyperliquid_fills
WHERE crossed = true
ORDER BY time DESC
LIMIT 100
```

### Orders Queries (2)

**11. Recent Orders** - Most recent orders placed across all markets
**Keywords:** `orders`, `latest`, `book`, `recent`, `placed`
**Parameters:** None
```sql
SELECT
  toDateTime(timestamp) AS timestamp,
  user,
  coin,
  side,
  limit_px AS limit_price,
  sz AS size,
  oid AS order_id
FROM hyperliquid_orders
ORDER BY timestamp DESC
LIMIT 100
```

**12. Active Orders by User** - Active orders for specific user address
**Keywords:** `user`, `orders`, `open`, `active`, `address`
**Parameters:** `user` address (replace `0xYOUR_ADDRESS`)
```sql
SELECT
  toDateTime(timestamp) AS timestamp,
  coin,
  side,
  limit_px AS limit_price,
  sz AS size,
  oid AS order_id,
  cloid AS client_order_id
FROM hyperliquid_orders
WHERE user = '0xYOUR_ADDRESS'
ORDER BY timestamp DESC
LIMIT 100
```

### Funding Queries (2)

**13. Recent Funding Payments** - Latest funding rate payments across all users
**Keywords:** `funding`, `payments`, `rates`, `perpetual`, `interest`
**Parameters:** None
```sql
SELECT
  toDateTime(time) AS time,
  user,
  coin,
  usdc AS payment,
  funding_rate,
  szi AS position_size
FROM hyperliquid_user_funding
ORDER BY time DESC
LIMIT 100
```

**14. Funding by User** - Funding payments for specific user address
**Keywords:** `user`, `funding`, `address`, `payments`, `history`
**Parameters:** `user` address (replace `0xYOUR_ADDRESS`)
```sql
SELECT
  toDateTime(time) AS time,
  coin,
  usdc AS payment,
  funding_rate,
  szi AS position_size
FROM hyperliquid_user_funding
WHERE user = '0xYOUR_ADDRESS'
ORDER BY time DESC
LIMIT 100
```

### Infrastructure Queries (2)

**15. Recent Blocks** - Latest Hyperliquid blocks with transaction counts
**Keywords:** `blocks`, `chain`, `latest`, `recent`, `blockchain`
**Parameters:** None
```sql
SELECT
  block_number,
  toDateTime(block_time) AS block_time,
  base58Encode(substring(block_hash, 1, 32)) AS block_hash,
  tx_count
FROM hyperliquid_blocks
ORDER BY block_number DESC
LIMIT 100
```

**16. Recent System Actions** - System-level actions and events
**Keywords:** `system`, `events`, `actions`, `activity`, `operations`
**Parameters:** None
```sql
SELECT
  toDateTime(block_time) AS block_time,
  action_type,
  user,
  evm_transaction_hash
FROM hyperliquid_system_actions
ORDER BY block_time DESC
LIMIT 100
```

### Ledger Queries (1)

**17. Ledger Updates by User** - Balance updates for specific user address
**Keywords:** `ledger`, `balance`, `user`, `address`, `updates`, `deposits`, `withdrawals`
**Parameters:** `user` address (replace `0xYOUR_ADDRESS`)
```sql
SELECT
  toDateTime(time) AS time,
  user,
  delta,
  hash
FROM hyperliquid_ledger_updates
WHERE user = '0xYOUR_ADDRESS'
ORDER BY time DESC
LIMIT 100
```

### Markets Queries (3)

**18. Perpetual Markets** - List all available perpetual futures markets
**Keywords:** `perp`, `perpetual`, `markets`, `list`, `futures`, `available`
**Parameters:** None
```sql
SELECT DISTINCT
  name,
  szDecimals
FROM hyperliquid_perp_markets
ORDER BY name
```

**19. Spot Markets** - List all available spot markets and tokens
**Keywords:** `spot`, `markets`, `tokens`, `list`, `available`, `trading`
**Parameters:** None
```sql
SELECT DISTINCT
  name,
  tokens,
  index,
  is_canonical
FROM hyperliquid_spot_markets
ORDER BY name
```

**20. Order Book Depth** - Order book depth and liquidity for specific market
**Keywords:** `orderbook`, `depth`, `liquidity`, `bids`, `asks`, `levels`
**Parameters:** `coin` name (replace `BTC`)
```sql
SELECT
  coin,
  side,
  px AS price,
  sz AS size,
  toDateTime(time) AS time
FROM hyperliquid_order_book_diffs
WHERE coin = 'BTC'
ORDER BY time DESC, px DESC
LIMIT 100
```

### Builders Queries (1)

**21. Builder Fills Summary** - 24-hour builder fill statistics and fees earned
**Keywords:** `builder`, `fills`, `fees`, `stats`, `24h`, `summary`
**Parameters:** Time interval (default: 24 hours)
```sql
SELECT
  builder_address,
  COUNT(*) AS fill_count,
  SUM(toFloat64(builder_fee)) AS total_fees,
  COUNT(DISTINCT user) AS unique_users,
  COUNT(DISTINCT coin) AS unique_pairs
FROM hyperliquid_builder_fills
WHERE block_time >= now() - INTERVAL 24 HOUR
GROUP BY builder_address
ORDER BY total_fees DESC
```

### Staking Queries (2)

**22. Top Validators** - Top validators by voting power and commission rates
**Keywords:** `validators`, `staking`, `voting`, `power`, `top`, `leaderboard`
**Parameters:** None
```sql
SELECT
  validator_address,
  voting_power,
  commission_rate
FROM hyperliquid_validators
ORDER BY toFloat64(voting_power) DESC
LIMIT 50
```

**23. Delegations by Validator** - Delegations for specific validator address
**Keywords:** `delegations`, `staking`, `validator`, `delegators`, `stakes`
**Parameters:** `validator_address` (replace `YOUR_VALIDATOR`)
```sql
SELECT
  delegator_address,
  delegation_amount,
  toDateTime(snapshot_time) AS snapshot_time
FROM hyperliquid_staking_delegations
WHERE validator_address = 'YOUR_VALIDATOR'
ORDER BY toFloat64(delegation_amount) DESC
LIMIT 100
```

### Liquidations Queries (1)

**24. Recent Liquidations** - Recent liquidation events and forced closures
**Keywords:** `liquidations`, `risk`, `forced`, `closure`, `margin`, `recent`
**Parameters:** None
```sql
SELECT
  toDateTime(block_time) AS block_time,
  lid AS liquidation_id,
  liquidated_user,
  liquidated_ntl_pos AS notional_position,
  liquidated_account_value
FROM hyperliquid_liquidations
ORDER BY block_time DESC
LIMIT 100
```

### Metrics Queries (2)

**25. Hourly Market Volume** - Hourly volume time series for specific market
**Keywords:** `volume`, `hourly`, `timeseries`, `chart`, `history`, `analytics`
**Parameters:** `coin` name (replace `BTC`)
```sql
SELECT
  toDateTime(hour) AS hour,
  coin,
  volume,
  trades
FROM hyperliquid_market_volume_hourly
WHERE coin = 'BTC'
ORDER BY hour DESC
LIMIT 168
```

**26. Daily Liquidation Metrics** - Daily liquidation aggregates and statistics
**Keywords:** `liquidations`, `daily`, `aggregated`, `metrics`, `timeseries`, `stats`
**Parameters:** None
```sql
SELECT
  day,
  total_liquidations,
  total_volume
FROM hyperliquid_liquidations_daily
ORDER BY day DESC
LIMIT 30
```

## Common Query Patterns

### Time-Based Analysis

```sql
-- Trading volume over time
SELECT
  toStartOfHour(time) AS hour,
  coin,
  SUM(toFloat64(sz)) AS volume,
  COUNT(*) AS trade_count,
  AVG(toFloat64(px)) AS avg_price
FROM hyperliquid_trades
WHERE time >= now() - INTERVAL 7 DAY
GROUP BY hour, coin
ORDER BY hour DESC, volume DESC
```

### User Analytics

```sql
-- User position and PnL summary
SELECT
  user,
  coin,
  SUM(toFloat64(sz) * CASE WHEN side = 'buy' THEN 1 ELSE -1 END) AS net_position,
  SUM(toFloat64(closed_pnl)) AS realized_pnl,
  SUM(toFloat64(fee)) AS total_fees
FROM hyperliquid_trades
WHERE time >= now() - INTERVAL 30 DAY
GROUP BY user, coin
HAVING abs(net_position) > 0
ORDER BY realized_pnl DESC
```

### Market Depth Analysis

```sql
-- Order book snapshot
SELECT
  side,
  px AS price,
  SUM(toFloat64(sz)) AS total_size,
  COUNT(*) AS order_count
FROM hyperliquid_order_book_diffs
WHERE coin = 'BTC'
  AND time >= now() - INTERVAL 1 MINUTE
GROUP BY side, px
ORDER BY side DESC, px DESC
```

### Funding Rate Analysis

```sql
-- Funding rate trends
SELECT
  coin,
  toStartOfDay(time) AS day,
  AVG(toFloat64(funding_rate)) AS avg_funding_rate,
  SUM(toFloat64(usdc)) AS total_funding_paid
FROM hyperliquid_user_funding
WHERE time >= now() - INTERVAL 30 DAY
GROUP BY coin, day
ORDER BY day DESC, coin
```

### Builder Analytics

```sql
-- Builder performance comparison
SELECT
  builder_address,
  toStartOfDay(time) AS day,
  COUNT(*) AS fill_count,
  SUM(toFloat64(sz)) AS volume,
  SUM(toFloat64(builder_fee)) AS fees_earned,
  COUNT(DISTINCT user) AS unique_traders
FROM hyperliquid_builder_fills
WHERE time >= now() - INTERVAL 7 DAY
GROUP BY builder_address, day
ORDER BY day DESC, fees_earned DESC
```

### Whale Tracking

```sql
-- Large trades monitoring
SELECT
  toDateTime(time) AS time,
  coin,
  side,
  px AS price,
  sz AS size,
  toFloat64(px) * toFloat64(sz) AS notional,
  user
FROM hyperliquid_trades
WHERE toFloat64(px) * toFloat64(sz) >= 500000
  AND time >= now() - INTERVAL 24 HOUR
ORDER BY notional DESC
```

### Liquidation Monitoring

```sql
-- Liquidation risk analysis
SELECT
  toStartOfHour(block_time) AS hour,
  COUNT(*) AS liquidation_count,
  SUM(toFloat64(liquidated_ntl_pos)) AS total_notional,
  AVG(toFloat64(liquidated_account_value)) AS avg_account_value
FROM hyperliquid_liquidations
WHERE block_time >= now() - INTERVAL 7 DAY
GROUP BY hour
ORDER BY hour DESC
```

## SQL Syntax Support

SQL Explorer supports standard SQL with ClickHouse extensions:

### Functions

- **Date/Time**: `toDateTime()`, `toStartOfHour()`, `toStartOfDay()`, `now()`, `INTERVAL`
- **Encoding**: `base58Encode()`, `substring()`
- **Aggregations**: `SUM()`, `AVG()`, `COUNT()`, `MAX()`, `MIN()`
- **Conditionals**: `CASE WHEN`, `IF()`, `countIf()`
- **Math**: `round()`, `abs()`, `toFloat64()`
- **String**: `lower()`, `upper()`, `concat()`

### Clauses

- `SELECT`, `FROM`, `WHERE`, `GROUP BY`, `HAVING`, `ORDER BY`, `LIMIT`
- `JOIN` (INNER, LEFT, RIGHT, FULL)
- `WITH` (CTEs/Common Table Expressions)
- Subqueries
- Window functions

### Operators

- Comparison: `=`, `!=`, `<`, `>`, `<=`, `>=`
- Logical: `AND`, `OR`, `NOT`
- Arithmetic: `+`, `-`, `*`, `/`
- String: `LIKE`, `IN`, `NOT IN`
- NULL handling: `IS NULL`, `IS NOT NULL`

### Common Patterns for Agents

**Address Handling:**
```sql
-- Addresses are case-sensitive FixedString(42)
WHERE user = '0x1234567890abcdef1234567890abcdef12345678'  -- Use exact case

-- Case-insensitive search (slower)
WHERE lower(user) = lower('0x1234...')
```

**Time Conversions:**
```sql
-- Convert timestamps to readable format
SELECT toDateTime(block_time) AS readable_time

-- Time bucketing for aggregations
SELECT toStartOfHour(time) AS hour_bucket
SELECT toStartOfDay(time) AS day_bucket

-- Recent data (last 24 hours, 7 days, 30 days)
WHERE block_time >= now() - INTERVAL 24 HOUR
WHERE time >= now() - INTERVAL 7 DAY
```

**String to Number Conversions:**
```sql
-- Most numeric fields stored as String - convert for math
SELECT toFloat64(px) AS price_numeric
SELECT toFloat64(sz) AS size_numeric

-- Calculations require conversion
WHERE toFloat64(px) * toFloat64(sz) > 100000
SELECT SUM(toFloat64(fee)) AS total_fees
```

**NULL Handling:**
```sql
-- Builder fields can be NULL
WHERE builder_address IS NOT NULL
SELECT Nullable(FixedString(42)) AS nullable_field
```

**Aggregation Patterns:**
```sql
-- Count distinct users
SELECT COUNT(DISTINCT user) AS unique_users

-- Sum with conversion
SELECT SUM(toFloat64(sz)) AS total_volume

-- Average with filtering
SELECT AVG(toFloat64(px)) AS avg_price
```

## Query Optimization

### Use Sort Keys

Queries filtering on sort key columns run significantly faster:

```sql
-- Fast - filters on sort key
SELECT * FROM hyperliquid_trades
WHERE user = '0xADDRESS'
ORDER BY block_number DESC

-- Slower - no sort key filter
SELECT * FROM hyperliquid_trades
WHERE coin = 'BTC'
```

### Leverage Time Partitions

Filter by time to reduce data scanned:

```sql
-- Good - scans only recent partition
SELECT * FROM hyperliquid_trades
WHERE block_time >= now() - INTERVAL 1 DAY

-- Avoid - scans all partitions
SELECT * FROM hyperliquid_trades
WHERE tid > 1000000
```

### Limit Result Sets

Always use `LIMIT` for exploratory queries:

```sql
SELECT * FROM hyperliquid_trades
ORDER BY block_time DESC
LIMIT 1000
```

### Aggregate Early

Use CTEs to aggregate before joining:

```sql
WITH user_volumes AS (
  SELECT user, SUM(toFloat64(sz)) AS volume
  FROM hyperliquid_trades
  WHERE block_time >= now() - INTERVAL 1 DAY
  GROUP BY user
)
SELECT * FROM user_volumes
WHERE volume > 10000
ORDER BY volume DESC
```

### Use Appropriate Data Types

Convert strings to numbers only when needed:

```sql
-- Efficient
SELECT user, sz FROM hyperliquid_trades WHERE user = '0xADDRESS'

-- Less efficient
SELECT user, toFloat64(sz) FROM hyperliquid_trades WHERE toFloat64(sz) > 100
```

## Best Practices

1. **Start with time filters** - Reduce data scanned using `block_time` or `time` filters
2. **Use sort keys** - Filter on sort key columns for best performance
3. **Test queries in UI** - Validate SQL in dashboard before API integration
4. **Monitor query costs** - Check `statistics.rows_read` and `statistics.bytes_read`
5. **Cache results** - Store frequently accessed results in your application
6. **Use CTEs** - Break complex queries into readable, optimized steps
7. **Handle pagination** - Use `LIMIT` and `OFFSET` for large result sets
8. **Validate data types** - Understand column types before conversions

## Response Fields

### data
Array of result rows matching your query. Each row is an object with column names as keys.

### meta
Array of column metadata objects:
```json
{"name": "column_name", "type": "ClickHouse_type"}
```

### rows
Integer count of rows returned.

### statistics
Query execution metrics:
- `elapsed`: Query execution time in seconds
- `rows_read`: Number of rows scanned
- `bytes_read`: Bytes of data scanned

## Error Handling

### Common Errors

| Error | Cause | Solution |
|-------|-------|----------|
| Syntax error | Invalid SQL | Check SQL syntax, use UI to test |
| Timeout | Query too complex | Add time filters, reduce data scanned |
| Column not found | Typo in column name | Verify column names in schema |
| Type mismatch | Invalid type conversion | Check column types, use correct functions |
| Rate limit | Too many requests | Implement request throttling |

### HTTP Status Codes

- `200` - Success
- `400` - Bad request (SQL syntax error)
- `401` - Unauthorized (invalid API key)
- `429` - Rate limit exceeded
- `500` - Server error

## Code Examples

### cURL

```bash
curl -X POST 'https://api.quicknode.com/sql/rest/v1/query' \
  -H 'accept: application/json' \
  -H 'Content-Type: application/json' \
  -H 'x-api-key: YOUR_API_KEY' \
  -d '{
    "query": "SELECT * FROM hyperliquid_trades ORDER BY block_time DESC LIMIT 10",
    "clusterId": "hyperliquid-core-mainnet"
  }'
```

### JavaScript

```javascript
const response = await fetch('https://api.quicknode.com/sql/rest/v1/query', {
  method: 'POST',
  headers: {
    'accept': 'application/json',
    'Content-Type': 'application/json',
    'x-api-key': 'YOUR_API_KEY'
  },
  body: JSON.stringify({
    query: 'SELECT * FROM hyperliquid_trades ORDER BY block_time DESC LIMIT 10',
    clusterId: 'hyperliquid-core-mainnet'
  })
});

const data = await response.json();
console.log(data);
```

### Python

```python
import requests
import json

url = "https://api.quicknode.com/sql/rest/v1/query"

payload = {
    "query": "SELECT * FROM hyperliquid_trades ORDER BY block_time DESC LIMIT 10",
    "clusterId": "hyperliquid-core-mainnet"
}

headers = {
    "accept": "application/json",
    "Content-Type": "application/json",
    "x-api-key": "YOUR_API_KEY"
}

response = requests.post(url, json=payload, headers=headers)
print(response.json())
```

### Ruby

```ruby
require 'net/http'
require 'json'

uri = URI('https://api.quicknode.com/sql/rest/v1/query')

request = Net::HTTP::Post.new(uri)
request['accept'] = 'application/json'
request['Content-Type'] = 'application/json'
request['x-api-key'] = 'YOUR_API_KEY'

request.body = {
  query: 'SELECT * FROM hyperliquid_trades ORDER BY block_time DESC LIMIT 10',
  clusterId: 'hyperliquid-core-mainnet'
}.to_json

response = Net::HTTP.start(uri.hostname, uri.port, use_ssl: true) do |http|
  http.request(request)
end

puts response.body
```

### Go

```go
package main

import (
    "bytes"
    "encoding/json"
    "fmt"
    "net/http"
)

func main() {
    url := "https://api.quicknode.com/sql/rest/v1/query"

    payload := map[string]string{
        "query": "SELECT * FROM hyperliquid_trades ORDER BY block_time DESC LIMIT 10",
        "clusterId": "hyperliquid-core-mainnet",
    }

    jsonData, _ := json.Marshal(payload)

    req, _ := http.NewRequest("POST", url, bytes.NewBuffer(jsonData))
    req.Header.Set("accept", "application/json")
    req.Header.Set("Content-Type", "application/json")
    req.Header.Set("x-api-key", "YOUR_API_KEY")

    client := &http.Client{}
    resp, _ := client.Do(req)
    defer resp.Body.Close()

    var result map[string]interface{}
    json.NewDecoder(resp.Body).Decode(&result)
    fmt.Println(result)
}
```

## Use Cases

### Trading Bots

Monitor trades, orders, and fills to trigger automated trading strategies.

```sql
-- Detect large market moves
SELECT
  coin,
  AVG(toFloat64(px)) AS avg_price,
  COUNT(*) AS trade_count
FROM hyperliquid_trades
WHERE time >= now() - INTERVAL 5 MINUTE
GROUP BY coin
HAVING trade_count > 100
```

### Analytics Dashboards

Build real-time dashboards tracking market metrics.

```sql
-- 24h volume by market
SELECT
  coin,
  SUM(toFloat64(sz)) AS volume,
  COUNT(*) AS trades
FROM hyperliquid_trades
WHERE time >= now() - INTERVAL 24 HOUR
GROUP BY coin
ORDER BY volume DESC
```

### Risk Monitoring

Track positions, liquidations, and funding rates.

```sql
-- Users approaching liquidation
SELECT
  user,
  SUM(toFloat64(liquidated_ntl_pos)) AS at_risk_notional
FROM hyperliquid_liquidations
WHERE block_time >= now() - INTERVAL 1 HOUR
GROUP BY user
```

### Market Research

Historical analysis of trading patterns and market structure.

```sql
-- Builder market share
SELECT
  builder_address,
  COUNT(*) * 100.0 / (SELECT COUNT(*) FROM hyperliquid_builder_fills) AS market_share
FROM hyperliquid_builder_fills
GROUP BY builder_address
ORDER BY market_share DESC
```

## Documentation

- **SQL Explorer Overview**: https://www.quicknode.com/docs/sql-explorer
- **REST API Overview**: https://www.quicknode.com/docs/sql-explorer/rest-api/overview
- **Pre-Built Queries**: https://www.quicknode.com/docs/sql-explorer/rest-api/hyperliquid-queries
- **llms.txt**: https://www.quicknode.com/docs/sql-explorer/llms.txt
