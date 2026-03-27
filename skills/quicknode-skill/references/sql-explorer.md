# Quicknode SQL Explorer Reference

Direct SQL access to indexed blockchain data without infrastructure. Query billions of rows of on-chain data using standard SQL syntax.

## Access Methods

SQL Explorer can be accessed in two ways:

### 1. Dashboard UI (Interactive)
Web-based SQL editor with syntax highlighting, schema browser, pre-built queries, and visualization tools. Access at [https://dashboard.quicknode.com/sql](https://dashboard.quicknode.com/sql)

### 2. REST API (Programmatic)
Execute SQL queries via HTTP POST for automation and integration. This is the recommended approach for AI agents and programmatic access.

```
POST https://api.quicknode.com/sql/rest/v1/query
```

**For AI Agents:** This reference focuses on REST API usage. Use the Dashboard UI only for obtaining your API key.

## Architecture

```
SQL Query → REST API → ClickHouse → Indexed Blockchain Data → JSON Response
            (Auth)     (Query Engine)  
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

Here's a complete cURL example that works out of the box (just replace `<your-api-key>`):

```bash
curl -X POST 'https://api.quicknode.com/sql/rest/v1/query' \
  -H 'accept: application/json' \
  -H 'Content-Type: application/json' \
  -H 'x-api-key: <your-api-key>' \
  -d '{
  "query": "SELECT timestamp, coin, side, price, size, price * size AS notional_usd, buyer_address, seller_address, buyer_fee, seller_fee, fee_token FROM hyperliquid_trades WHERE block_time > now() - INTERVAL 1 HOUR ORDER BY block_number DESC, trade_id DESC LIMIT 3",
  "clusterId": "hyperliquid-core-mainnet"
}'
```

**Request Breakdown:**

| Component | Value | Description |
|-----------|-------|-------------|
| **Endpoint** | `https://api.quicknode.com/sql/rest/v1/query` | SQL Explorer REST API endpoint |
| **Method** | `POST` | HTTP method |
| **Header** | `x-api-key: <your-api-key>` | Your Quicknode API key for authentication |
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
      "timestamp": "2026-03-25 10:45:23.000",
      "coin": "BTC",
      "side": "B",
      "price": "95432.500000000000000000",
      "size": "0.150000000000000000",
      "notional_usd": "14314.875000000000000000000000000000000000",
      "buyer_address": "0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb",
      "seller_address": "0x8dF3aad3a84da6b69A4DA8aeC3eA40d9091B2Ac",
      "buyer_fee": "7.156437500000000000",
      "seller_fee": "7.156437500000000000",
      "fee_token": "USDC"
    },
    {
      "timestamp": "2026-03-25 10:45:18.000",
      "coin": "ETH",
      "side": "A",
      "price": "3245.750000000000000000",
      "size": "2.500000000000000000",
      "notional_usd": "8114.375000000000000000000000000000000000",
      "buyer_address": "0x1a2B3c4D5e6F7a8b9C0d1E2f3A4b5C6d7E8f9A0",
      "seller_address": "0x9f8e7D6c5B4a3F2e1D0c9B8a7F6e5D4c3B2a1F0",
      "buyer_fee": "4.057187500000000000",
      "seller_fee": "4.057187500000000000",
      "fee_token": "USDC"
    },
    {
      "timestamp": "2026-03-25 10:45:12.000",
      "coin": "SOL",
      "side": "B",
      "price": "142.350000000000000000",
      "size": "50.000000000000000000",
      "notional_usd": "7117.500000000000000000000000000000000000",
      "buyer_address": "0x5a6B7c8D9e0F1a2b3C4d5E6f7A8b9C0d1E2f3A4",
      "seller_address": "0x4f3E2d1C0b9A8f7E6d5C4b3A2f1E0d9C8b7A6f5",
      "buyer_fee": "3.558750000000000000",
      "seller_fee": "3.558750000000000000",
      "fee_token": "USDC"
    }
  ],
  "meta": [
    {"name": "timestamp", "type": "DateTime64(3, 'UTC')"},
    {"name": "coin", "type": "LowCardinality(String)"},
    {"name": "side", "type": "Enum8('B' = 1, 'A' = 2)"},
    {"name": "price", "type": "Decimal(38, 18)"},
    {"name": "size", "type": "Decimal(38, 18)"},
    {"name": "notional_usd", "type": "Decimal(38, 36)"},
    {"name": "buyer_address", "type": "FixedString(42)"},
    {"name": "seller_address", "type": "FixedString(42)"},
    {"name": "buyer_fee", "type": "Decimal(38, 18)"},
    {"name": "seller_fee", "type": "Decimal(38, 18)"},
    {"name": "fee_token", "type": "LowCardinality(String)"}
  ],
  "rows": 3,
  "statistics": {
    "elapsed": 0.042,
    "rows_read": 15847,
    "bytes_read": 2456789
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

**For AI Agents:**

Execute queries directly via the REST API. Start with simple queries and add complexity as needed:

```bash
curl -X POST 'https://api.quicknode.com/sql/rest/v1/query' \
  -H 'x-api-key: <your-api-key>' \
  -H 'Content-Type: application/json' \
  -d '{"query": "...", "clusterId": "hyperliquid-core-mainnet"}'
```

Check the `statistics` field in the response to monitor query performance (elapsed time, rows_read, bytes_read).

## Agent Usage Guide

**For AI Agents using this reference:**

1. **Query Selection**: Use the "Common Use Case Mappings" table to quickly find relevant queries for user requests
2. **Parameter Substitution**: Look for `Parameters:` line under each query - replace placeholders like `<user-address>`, `<coin-name>`, `<validator-address>` with actual values
3. **Keywords**: Each query lists keywords - use these for semantic search and query discovery
4. **Sort Keys**: Queries using sort key columns (marked with ⚡) run significantly faster - prioritize these when possible
5. **Time Filters**: Always add time filters for better performance and cost efficiency
6. **Sample Responses**: Check sample response format to understand data structure before processing
7. **Custom Queries**: If no pre-built query matches, use "Common Query Patterns" section as templates
8. **Authentication**: Always include `x-api-key` header - see "Getting Your API Key" section above

## Supported Networks

| Chain | Network | Cluster ID | Tables | Coverage |
|-------|---------|------------|--------|----------|
| Hyperliquid (HyperCore) | Mainnet | `hyperliquid-core-mainnet` | 37 | Trades, orders, fills, funding, order book diffs, perpetual markets, spot markets, blocks, transactions, system actions, builder activity, staking, ledger, clearinghouse states, oracle prices, referrals, sub-accounts, vault equities, agents, bridge events, display names, hourly metrics, daily metrics |

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
    "meta": [
        {
            "name": "block_number",
            "type": "UInt64"
        },
        {
            "name": "block_time",
            "type": "DateTime64(6, 'UTC')"
        }
    ],
    "data": [
        {
            "block_number": 936231661,
            "block_time": "2026-03-26 19:45:01.625244",
            "trade_id": 496316863559640,
            "coin": "cash:HOOD",
            "timestamp": "2026-03-26 19:45:01.625",
            "transaction_hash": "0x0fd9a48c5a19906511530437cdc2ed0205c00071f51caf37b3a24fdf191d6a4f",
            "price": 70.495,
            "size": 0.709,
            "side": "A",
            "buyer_address": "0xcee1cc9b396bde5944482f64f3e18be7b8d5df73",
            "buyer_order_id": 362256178610,
            "buyer_fee": -0.000149,
            "buyer_closed_pnl": 0,
            "buyer_start_position": 305.417,
            "buyer_crossed": 0,
            "buyer_dir": "Open Long",
            "buyer_twap_id": null,
            "buyer_builder_address": null,
            "buyer_builder_fee": null,
            "seller_address": "0x399965e15d4e61ec3529cc98b7f7ebb93b733336",
            "seller_order_id": 362256236744,
            "seller_fee": 0.001679,
            "seller_closed_pnl": -0.005672,
            "seller_start_position": 35.543,
            "seller_crossed": 1,
            "seller_dir": "Close Long",
            "seller_twap_id": null,
            "seller_builder_address": "0x4950994884602d1b6c6d96e4fe30f58205c39395",
            "seller_builder_fee": null,
            "fee_token": "USDT0",
            "total_builder_fee": null,
            "liquidated_user": null,
            "liquidation_mark_price": null,
            "liquidation_method": null,
            "indexed_at": "2026-03-26 19:45:02.580"
        }
    ],
    "rows": 10,
    "rows_before_limit_at_least": 892148168,
    "statistics": {
        "elapsed": 1.269332554,
        "rows_read": 892148586,
        "bytes_read": 14274604630
    }
}
```

## Available Tables

### Trading Tables

#### hyperliquid_trades
**Sort Keys:** ⚡ block_number, trade_id | **Partition:** toYYYYMM(block_time)

Individual trade executions on Hyperliquid.

| Column | Type | Sort Key | Description |
|--------|------|----------|-------------|
| block_number | UInt64 | Yes | Block Number |
| block_time | DateTime | No | Block Time |
| trade_id | UInt64 | Yes | Trade Id |
| coin | String | No | Coin |
| timestamp | DateTime | No | Timestamp |
| transaction_hash | FixedString | No | Transaction Hash |
| price | Decimal | No | Price |
| size | Decimal | No | Size |
| side | Enum | No | Side |
| buyer_address | FixedString | No | Buyer Address |
| buyer_order_id | UInt64 | No | Buyer Order Id |
| buyer_fee | Decimal | No | Buyer Fee |
| buyer_closed_pnl | Decimal | No | Buyer Closed Pnl |
| buyer_start_position | Decimal | No | Buyer Start Position |
| buyer_crossed | UInt64 | No | Buyer Crossed |
| buyer_dir | String | No | Buyer Dir |
| buyer_twap_id | UInt64 | No | Buyer Twap Id |
| buyer_builder_address | FixedString | No | Buyer Builder Address |
| buyer_builder_fee | Decimal | No | Buyer Builder Fee |
| seller_address | FixedString | No | Seller Address |
| seller_order_id | UInt64 | No | Seller Order Id |
| seller_fee | Decimal | No | Seller Fee |
| seller_closed_pnl | Decimal | No | Seller Closed Pnl |
| seller_start_position | Decimal | No | Seller Start Position |
| seller_crossed | UInt64 | No | Seller Crossed |
| seller_dir | String | No | Seller Dir |
| seller_twap_id | UInt64 | No | Seller Twap Id |
| seller_builder_address | FixedString | No | Seller Builder Address |
| seller_builder_fee | Decimal | No | Seller Builder Fee |
| fee_token | String | No | Fee Token |
| total_builder_fee | Decimal | No | Total Builder Fee |
| liquidated_user | FixedString | No | Liquidated User |
| liquidation_mark_price | Decimal | No | Liquidation Mark Price |
| liquidation_method | String | No | Liquidation Method |
| unique_id | String | No | Unique Id |
| indexed_at | DateTime | No | Indexed At |

#### hyperliquid_fills
**Sort Keys:** ⚡ block_number, tid, user | **Partition:** toYYYYMM(block_time)

Order fills with execution details.

| Column | Type | Sort Key | Description |
|--------|------|----------|-------------|
| block_number | UInt64 | Yes | Block Number |
| block_time | DateTime | No | Block Time |
| user | FixedString | Yes | User |
| tid | UInt64 | Yes | Tid |
| oid | UInt64 | No | Oid |
| coin | String | No | Coin |
| side | Enum | No | Side |
| price | Decimal | No | Price |
| size | Decimal | No | Size |
| time | DateTime | No | Time |
| dir | String | No | Dir |
| crossed | UInt64 | No | Crossed |
| fee | Decimal | No | Fee |
| fee_token | String | No | Fee Token |
| closed_pnl | Decimal | No | Closed Pnl |
| start_position | Decimal | No | Start Position |
| hash | FixedString | No | Hash |
| twap_id | UInt64 | No | Twap Id |
| builder | FixedString | No | Builder |
| builder_fee | Decimal | No | Builder Fee |
| cloid | String | No | Cloid |
| is_liquidation | UInt64 | No | Is Liquidation |
| liquidated_user | FixedString | No | Liquidated User |
| liquidation_mark_price | Decimal | No | Liquidation Mark Price |
| liquidation_method | String | No | Liquidation Method |
| indexed_at | DateTime | No | Indexed At |

#### hyperliquid_orders
**Sort Keys:** ⚡ block_number, oid, status_time | **Partition:** toYYYYMM(block_time)

Order placements, cancellations, and modifications.

| Column | Type | Sort Key | Description |
|--------|------|----------|-------------|
| block_number | UInt64 | Yes | Block Number |
| block_time | DateTime | No | Block Time |
| oid | UInt64 | Yes | Oid |
| user | FixedString | No | User |
| coin | String | No | Coin |
| side | Enum | No | Side |
| status | String | No | Status |
| order_type | String | No | Order Type |
| limit_price | Decimal | No | Limit Price |
| size | Decimal | No | Size |
| orig_size | Decimal | No | Orig Size |
| tif | String | No | Tif |
| is_trigger | UInt64 | No | Is Trigger |
| trigger_price | Decimal | No | Trigger Price |
| trigger_condition | String | No | Trigger Condition |
| reduce_only | UInt64 | No | Reduce Only |
| is_position_tpsl | UInt64 | No | Is Position Tpsl |
| children | String | No | Children |
| cloid | String | No | Cloid |
| hash | FixedString | No | Hash |
| builder | String | No | Builder |
| order_timestamp | DateTime | No | Order Timestamp |
| status_time | DateTime | Yes | Status Time |
| unique_id | String | No | Unique Id |
| indexed_at | DateTime | No | Indexed At |

#### hyperliquid_order_book_diffs
**Sort Keys:** ⚡ coin, side, block_number, oid | **Partition:** toYYYYMM(block_time)

Order book changes for reconstruction.

| Column | Type | Sort Key | Description |
|--------|------|----------|-------------|
| block_number | UInt64 | Yes | Block Number |
| block_time | DateTime | No | Block Time |
| user | FixedString | No | User |
| oid | UInt64 | Yes | Oid |
| coin | String | Yes | Coin |
| side | Enum | Yes | Side |
| price | Decimal | No | Price |
| diff_type | String | No | Diff Type |
| size | Decimal | No | Size |
| orig_size | Decimal | No | Orig Size |
| new_size | Decimal | No | New Size |
| indexed_at | DateTime | No | Indexed At |

#### hyperliquid_dex_trades
**Sort Keys:** ⚡ PARTITION BY: (not specified)

DEX trades with full buyer/seller details.

| Column | Type | Sort Key | Description |
|--------|------|----------|-------------|
| market_type | String | No | Market Type |
| price | Decimal | No | Price |
| size | Decimal | No | Size |
| usd_amount | Decimal | No | Usd Amount |
| side | Enum | No | Side |
| buyer_address | FixedString | No | Buyer Address |
| seller_address | FixedString | No | Seller Address |
| trade_id | UInt64 | No | Trade Id |
| timestamp | DateTime | No | Timestamp |
| transaction_hash | FixedString | No | Transaction Hash |
| block_number | UInt64 | No | Block Number |
| block_time | DateTime | No | Block Time |
| buyer_order_id | UInt64 | No | Buyer Order Id |
| seller_order_id | UInt64 | No | Seller Order Id |
| buyer_fee | Decimal | No | Buyer Fee |
| seller_fee | Decimal | No | Seller Fee |
| fee_token | String | No | Fee Token |
| buyer_closed_pnl | Decimal | No | Buyer Closed Pnl |
| seller_closed_pnl | Decimal | No | Seller Closed Pnl |
| buyer_start_position | Decimal | No | Buyer Start Position |
| seller_start_position | Decimal | No | Seller Start Position |
| buyer_crossed | UInt64 | No | Buyer Crossed |
| seller_crossed | UInt64 | No | Seller Crossed |
| buyer_dir | String | No | Buyer Dir |
| seller_dir | String | No | Seller Dir |
| buyer_twap_id | UInt64 | No | Buyer Twap Id |
| seller_twap_id | UInt64 | No | Seller Twap Id |
| buyer_builder_address | FixedString | No | Buyer Builder Address |
| seller_builder_address | FixedString | No | Seller Builder Address |
| buyer_builder_fee | Decimal | No | Buyer Builder Fee |
| seller_builder_fee | Decimal | No | Seller Builder Fee |
| total_builder_fee | Decimal | No | Total Builder Fee |
| liquidated_user | FixedString | No | Liquidated User |
| liquidation_mark_price | Decimal | No | Liquidation Mark Price |
| liquidation_method | String | No | Liquidation Method |
| unique_id | String | No | Unique Id |
| indexed_at | DateTime | No | Indexed At |

### Market Tables

#### hyperliquid_perpetual_markets
**Sort Keys:** ⚡ coin

Perpetual market configurations.

| Column | Type | Sort Key | Description |
|--------|------|----------|-------------|
| coin | String | Yes | Coin |
| max_leverage | UInt64 | No | Max Leverage |
| sz_decimals | UInt64 | No | Sz Decimals |
| margin_table_id | UInt64 | No | Margin Table Id |
| only_isolated | UInt64 | No | Only Isolated |
| updated_at | DateTime | No | Updated At |

#### hyperliquid_spot_markets
**Sort Keys:** ⚡ token_index

Spot market configurations.

| Column | Type | Sort Key | Description |
|--------|------|----------|-------------|
| token | String | No | Token |
| token_id | String | No | Token Id |
| sz_decimals | UInt64 | No | Sz Decimals |
| wei_decimals | UInt64 | No | Wei Decimals |
| is_canonical | UInt64 | No | Is Canonical |
| evm_contract | String | No | Evm Contract |
| full_name | String | No | Full Name |
| deployer_trading_fee_share | String | No | Deployer Trading Fee Share |
| pair_index | UInt64 | No | Pair Index |
| pair_name | String | No | Pair Name |
| updated_at | DateTime | No | Updated At |

#### hyperliquid_perpetual_market_contexts
**Sort Keys:** ⚡ coin, polled_at | **Partition:** toYYYYMM(polled_at)

Perpetual market snapshots with funding, OI, prices.

| Column | Type | Sort Key | Description |
|--------|------|----------|-------------|
| polled_at | DateTime | Yes | Polled At |
| coin | String | Yes | Coin |
| funding | Decimal | No | Funding |
| open_interest | Decimal | No | Open Interest |
| day_ntl_vlm | Decimal | No | Day Ntl Vlm |
| day_base_vlm | Decimal | No | Day Base Vlm |
| premium | Decimal | No | Premium |
| oracle_px | Decimal | No | Oracle Px |
| mark_px | Decimal | No | Mark Px |
| mid_px | Decimal | No | Mid Px |
| prev_day_px | Decimal | No | Prev Day Px |
| impact_bid | Decimal | No | Impact Bid |
| impact_ask | Decimal | No | Impact Ask |
| indexed_at | DateTime | No | Indexed At |

#### hyperliquid_market_volume_hourly
**Sort Keys:** ⚡ coin, hour | **Partition:** toYYYYMM(hour)

Hourly trading volume and OHLC data.

| Column | Type | Sort Key | Description |
|--------|------|----------|-------------|
| coin | String | Yes | Coin |
| hour | DateTime | Yes | Hour |
| volume | Decimal | No | Volume |
| trade_count | UInt64 | No | Trade Count |
| high | Decimal | No | High |
| low | Decimal | No | Low |
| open | Decimal | No | Open |
| close | Decimal | No | Close |

### Funding & Liquidation Tables

#### hyperliquid_funding
**Sort Keys:** ⚡ block_number, user, coin, time | **Partition:** toYYYYMM(block_time)

Funding payments and rates by address and coin.

| Column | Type | Sort Key | Description |
|--------|------|----------|-------------|
| block_number | UInt64 | Yes | Block Number |
| block_time | DateTime | No | Block Time |
| time | DateTime | Yes | Time |
| hash | FixedString | No | Hash |
| user | FixedString | Yes | User |
| coin | String | Yes | Coin |
| funding_amount | Decimal | No | Funding Amount |
| funding_rate | Decimal | No | Funding Rate |
| szi | Decimal | No | Szi |
| indexed_at | DateTime | No | Indexed At |

#### hyperliquid_funding_summary_hourly
**Sort Keys:** ⚡ coin, hour | **Partition:** toYYYYMM(hour)

Hourly aggregated funding rate summaries.

| Column | Type | Sort Key | Description |
|--------|------|----------|-------------|
| coin | String | Yes | Coin |
| hour | DateTime | Yes | Hour |
| avg_funding_rate | Float64 | No | Avg Funding Rate |
| total_funding | Decimal | No | Total Funding |
| unique_users | UInt64 | No | Unique Users |

#### hyperliquid_liquidations_daily
**Sort Keys:** ⚡ day, coin | **Partition:** toYYYYMM(day)

Daily aggregated liquidation statistics.

| Column | Type | Sort Key | Description |
|--------|------|----------|-------------|
| day | DateTime | Yes | Day |
| coin | String | Yes | Coin |
| liquidation_count | UInt64 | No | Liquidation Count |
| liquidated_volume | Decimal | No | Liquidated Volume |
| unique_liquidated_users | UInt64 | No | Unique Liquidated Users |

### Infrastructure Tables

#### hyperliquid_blocks
**Sort Keys:** ⚡ block_number | **Partition:** toYYYYMM(block_time)

Hyperliquid block data.

| Column | Type | Sort Key | Description |
|--------|------|----------|-------------|
| block_number | UInt64 | Yes | Block Number |
| block_time | DateTime | No | Block Time |
| fills_count | UInt64 | No | Fills Count |
| orders_count | UInt64 | No | Orders Count |
| misc_events_count | UInt64 | No | Misc Events Count |
| book_diffs_count | UInt64 | No | Book Diffs Count |
| twap_statuses_count | UInt64 | No | Twap Statuses Count |
| writer_actions_count | UInt64 | No | Writer Actions Count |
| indexed_at | DateTime | No | Indexed At |

#### hyperliquid_transactions
**Sort Keys:** ⚡ round, tx_hash, nonce | **Partition:** toYYYYMM(block_time)

Transaction data with hashes and status.

| Column | Type | Sort Key | Description |
|--------|------|----------|-------------|
| round | UInt64 | Yes | Round |
| block_time | DateTime | No | Block Time |
| proposer | String | No | Proposer |
| tx_hash | String | Yes | Tx Hash |
| user | String | No | User |
| action_type | String | No | Action Type |
| action | JSON | No | Action |
| error | String | No | Error |
| is_success | UInt64 | No | Is Success |
| nonce | UInt64 | Yes | Nonce |
| vault_address | String | No | Vault Address |
| indexed_at | DateTime | No | Indexed At |

#### hyperliquid_system_actions
**Sort Keys:** ⚡ block_number, evm_tx_hash, nonce | **Partition:** toYYYYMM(block_time)

System-level actions including HyperVM/HyperCore bridging.

| Column | Type | Sort Key | Description |
|--------|------|----------|-------------|
| block_number | UInt64 | Yes | Block Number |
| block_time | DateTime | No | Block Time |
| user | FixedString | No | User |
| nonce | UInt64 | Yes | Nonce |
| evm_tx_hash | FixedString | Yes | Evm Tx Hash |
| action_type | String | No | Action Type |
| action | String | No | Action |
| destination | FixedString | No | Destination |
| token | UInt64 | No | Token |
| wei | UInt64 | No | Wei |
| from_sub_account | UInt64 | No | From Sub Account |
| source_dex_or_spot | UInt64 | No | Source Dex Or Spot |
| destination_dex_or_spot | UInt64 | No | Destination Dex Or Spot |
| validator | FixedString | No | Validator |
| is_undelegate | UInt64 | No | Is Undelegate |
| signature_chain_id | String | No | Signature Chain Id |
| hyperliquid_chain | String | No | Hyperliquid Chain |
| ntl | Decimal | No | Ntl |
| to_perp | UInt64 | No | To Perp |
| order_grouping | String | No | Order Grouping |
| agent_address | FixedString | No | Agent Address |
| agent_name | String | No | Agent Name |
| is_mint | UInt64 | No | Is Mint |
| indexed_at | DateTime | No | Indexed At |

### Builder Tables

#### hyperliquid_builder_fills
**Sort Keys:** ⚡ builder_address, timestamp, tid | **Partition:** toYYYYMM(block_time)

Builder-specific fill events.

| Column | Type | Sort Key | Description |
|--------|------|----------|-------------|
| block_number | UInt64 | No | Block Number |
| block_time | DateTime | No | Block Time |
| timestamp | DateTime | Yes | Timestamp |
| created_at | DateTime | No | Created At |
| updated_at | DateTime | No | Updated At |
| builder_address | FixedString | Yes | Builder Address |
| builder_fee | Decimal | No | Builder Fee |
| user | FixedString | No | User |
| closed_pnl | Decimal | No | Closed Pnl |
| coin | String | No | Coin |
| crossed | UInt64 | No | Crossed |
| price | Decimal | No | Price |
| side | Enum | No | Side |
| size | Decimal | No | Size |
| twap_id | UInt64 | No | Twap Id |
| dir | String | No | Dir |
| fee | Decimal | No | Fee |
| start_position | Decimal | No | Start Position |
| cloid | String | No | Cloid |
| is_liquidation | UInt64 | No | Is Liquidation |
| liquidated_user | FixedString | No | Liquidated User |
| liquidation_mark_price | Decimal | No | Liquidation Mark Price |
| liquidation_method | String | No | Liquidation Method |
| tid | UInt64 | Yes | Tid |
| oid | UInt64 | No | Oid |
| hash | FixedString | No | Hash |
| indexed_at | DateTime | No | Indexed At |

#### hyperliquid_builder_transactions
**Sort Keys:** ⚡ block_number, hash, user | **Partition:** toYYYYMM(block_time)

Builder transaction activity.

| Column | Type | Sort Key | Description |
|--------|------|----------|-------------|
| block_number | UInt64 | Yes | Block Number |
| block_time | DateTime | No | Block Time |
| hash | String | Yes | Hash |
| user | String | Yes | User |
| builder | String | No | Builder |
| builder_fee | Decimal | No | Builder Fee |
| action_type | String | No | Action Type |
| coin | String | No | Coin |
| is_success | UInt64 | No | Is Success |
| indexed_at | DateTime | No | Indexed At |

#### hyperliquid_builder_labels
**Sort Keys:** ⚡ builder_address

Builder labels and metadata.

| Column | Type | Sort Key | Description |
|--------|------|----------|-------------|
| builder_name | String | No | Builder Name |
| builder_category | String | No | Builder Category |
| indexed_at | DateTime | No | Indexed At |

### Ledger Tables

#### hyperliquid_ledger_updates
**Sort Keys:** ⚡ block_number, user, time, hash | **Partition:** toYYYYMM(block_time)

Ledger state changes and account updates.

| Column | Type | Sort Key | Description |
|--------|------|----------|-------------|
| block_number | UInt64 | Yes | Block Number |
| block_time | DateTime | No | Block Time |
| time | DateTime | Yes | Time |
| hash | FixedString | Yes | Hash |
| delta_type | String | No | Delta Type |
| user | FixedString | Yes | User |
| usdc_amount | Decimal | No | Usdc Amount |
| token | String | No | Token |
| amount | Decimal | No | Amount |
| destination | FixedString | No | Destination |
| fee | Decimal | No | Fee |
| fee_token | String | No | Fee Token |
| native_token_fee | Decimal | No | Native Token Fee |
| nonce | UInt64 | No | Nonce |
| usdc_value | Decimal | No | Usdc Value |
| source_dex | String | No | Source Dex |
| destination_dex | String | No | Destination Dex |
| account_value | Decimal | No | Account Value |
| leverage_type | String | No | Leverage Type |
| liquidated_ntl_pos | Decimal | No | Liquidated Ntl Pos |
| liquidated_positions | String | No | Liquidated Positions |
| vault | FixedString | No | Vault |
| requested_usd | Decimal | No | Requested Usd |
| net_withdrawn_usd | Decimal | No | Net Withdrawn Usd |
| basis | Decimal | No | Basis |
| closing_cost | Decimal | No | Closing Cost |
| commission | Decimal | No | Commission |
| to_perp | UInt64 | No | To Perp |
| is_deposit | UInt64 | No | Is Deposit |
| operation | String | No | Operation |
| interest_amount | Decimal | No | Interest Amount |
| dex | String | No | Dex |
| secondary_user | FixedString | No | Secondary User |
| indexed_at | DateTime | No | Indexed At |

#### hyperliquid_asset_transfers
**Sort Keys:** ⚡ block_number, tx_hash, user | **Partition:** toYYYYMM(block_time)

Asset transfers between addresses.

| Column | Type | Sort Key | Description |
|--------|------|----------|-------------|
| block_number | UInt64 | Yes | Block Number |
| block_time | DateTime | No | Block Time |
| time | DateTime | No | Time |
| tx_hash | FixedString | Yes | Tx Hash |
| transfer_type | String | No | Transfer Type |
| user | FixedString | Yes | User |
| destination | FixedString | No | Destination |
| token | String | No | Token |
| amount | Decimal | No | Amount |
| usdc_amount | Decimal | No | Usdc Amount |
| fee | Decimal | No | Fee |
| nonce | UInt64 | No | Nonce |
| indexed_at | DateTime | No | Indexed At |

### Staking Tables

#### hyperliquid_staking_events
**Sort Keys:** ⚡ block_number, hash | **Partition:** toYYYYMM(block_time)

Staking, unstaking, and reward events.

| Column | Type | Sort Key | Description |
|--------|------|----------|-------------|
| block_number | UInt64 | Yes | Block Number |
| block_time | DateTime | No | Block Time |
| time | DateTime | No | Time |
| hash | FixedString | Yes | Hash |
| event_type | Enum | No | Event Type |
| user | FixedString | No | User |
| amount | Decimal | No | Amount |
| is_finalized | UInt64 | No | Is Finalized |
| validator | FixedString | No | Validator |
| is_undelegate | UInt64 | No | Is Undelegate |
| indexed_at | DateTime | No | Indexed At |

#### hyperliquid_validator_rewards
**Sort Keys:** ⚡ block_number, validator, time | **Partition:** toYYYYMM(block_time)

Validator reward distributions.

| Column | Type | Sort Key | Description |
|--------|------|----------|-------------|
| block_number | UInt64 | Yes | Block Number |
| block_time | DateTime | No | Block Time |
| time | DateTime | Yes | Time |
| hash | FixedString | No | Hash |
| validator | FixedString | Yes | Validator |
| reward | Decimal | No | Reward |
| indexed_at | DateTime | No | Indexed At |

#### hyperliquid_delegator_rewards
**Sort Keys:** ⚡ block_number, validator, delegator | **Partition:** toYYYYMM(snapshot_time)

Delegator reward distributions.

| Column | Type | Sort Key | Description |
|--------|------|----------|-------------|
| block_number | UInt64 | Yes | Block Number |
| snapshot_time | DateTime | No | Snapshot Time |
| validator | FixedString | Yes | Validator |
| delegator | FixedString | Yes | Delegator |
| c | UInt64 | No | C |
| m | UInt64 | No | M |
| reward_wei | UInt64 | No | Reward Wei |
| reward | Decimal | No | Reward |
| validator_total_delegated | UInt64 | No | Validator Total Delegated |
| commission_bps | UInt64 | No | Commission Bps |
| indexed_at | DateTime | No | Indexed At |

### Clearinghouse Tables

#### hyperliquid_clearinghouse_states
**Sort Keys:** ⚡ block_number, clearinghouse, user, asset_idx | **Partition:** toYYYYMM(snapshot_time)

Clearinghouse state snapshots.

| Column | Type | Sort Key | Description |
|--------|------|----------|-------------|
| block_number | UInt64 | Yes | Block Number |
| snapshot_time | DateTime | No | Snapshot Time |
| clearinghouse | UInt64 | Yes | Clearinghouse |
| user | FixedString | Yes | User |
| asset_idx | UInt64 | Yes | Asset Idx |
| coin | String | No | Coin |
| size | Int64 | No | Size |
| entry_notional | UInt64 | No | Entry Notional |
| margin | UInt64 | No | Margin |
| funding_alltime | Int64 | No | Funding Alltime |
| funding_since_open | Int64 | No | Funding Since Open |
| funding_since_change | Int64 | No | Funding Since Change |
| usdc_balance | Int64 | No | Usdc Balance |
| indexed_at | DateTime | No | Indexed At |

#### hyperliquid_spot_clearinghouse_states
**Sort Keys:** ⚡ block_number, user, token_idx | **Partition:** toYYYYMM(snapshot_time)

Spot clearinghouse state snapshots.

| Column | Type | Sort Key | Description |
|--------|------|----------|-------------|
| block_number | UInt64 | Yes | Block Number |
| snapshot_time | DateTime | No | Snapshot Time |
| user | FixedString | Yes | User |
| token_idx | UInt64 | Yes | Token Idx |
| token | String | No | Token |
| total | Int64 | No | Total |
| escrowed | Int64 | No | Escrowed |
| indexed_at | DateTime | No | Indexed At |

### Other Tables

#### hyperliquid_oracle_prices
**Sort Keys:** ⚡ block_number, clearinghouse, asset_idx | **Partition:** toYYYYMM(snapshot_time)

Oracle price feeds.

| Column | Type | Sort Key | Description |
|--------|------|----------|-------------|
| block_number | UInt64 | Yes | Block Number |
| snapshot_time | DateTime | No | Snapshot Time |
| clearinghouse | UInt64 | Yes | Clearinghouse |
| asset_idx | UInt64 | Yes | Asset Idx |
| coin | String | No | Coin |
| mark_px | String | No | Mark Px |
| daily_px | String | No | Daily Px |
| indexed_at | DateTime | No | Indexed At |

#### hyperliquid_agents
**Sort Keys:** ⚡ block_number, agent | **Partition:** toYYYYMM(snapshot_time)

Agent configurations.

| Column | Type | Sort Key | Description |
|--------|------|----------|-------------|
| block_number | UInt64 | Yes | Block Number |
| snapshot_time | DateTime | No | Snapshot Time |
| agent | FixedString | Yes | Agent |
| user | FixedString | No | User |
| name | String | No | Name |
| valid_until | String | No | Valid Until |
| indexed_at | DateTime | No | Indexed At |

#### hyperliquid_bridge
**Sort Keys:** ⚡ block_number, bridge_type, user, nonce, eth_block_number, tx_hash | **Partition:** toYYYYMM(snapshot_time)

Bridge deposit and withdrawal events.

| Column | Type | Sort Key | Description |
|--------|------|----------|-------------|
| block_number | UInt64 | Yes | Block Number |
| snapshot_time | DateTime | No | Snapshot Time |
| bridge_type | String | Yes | Bridge Type |
| user | FixedString | Yes | User |
| amount_wei | UInt64 | No | Amount Wei |
| eth_block_number | UInt64 | Yes | Eth Block Number |
| tx_hash | String | Yes | Tx Hash |
| nonce | UInt64 | Yes | Nonce |
| event_time | String | No | Event Time |
| indexed_at | DateTime | No | Indexed At |

#### hyperliquid_display_names
**Sort Keys:** ⚡ block_number, user | **Partition:** toYYYYMM(snapshot_time)

User display names and identifiers.

| Column | Type | Sort Key | Description |
|--------|------|----------|-------------|
| block_number | UInt64 | Yes | Block Number |
| snapshot_time | DateTime | No | Snapshot Time |
| user | FixedString | Yes | User |
| display_name | String | No | Display Name |
| indexed_at | DateTime | No | Indexed At |

#### hyperliquid_register_referral
**Sort Keys:** ⚡ referral_code, user

Referral registration events.

| Column | Type | Sort Key | Description |
|--------|------|----------|-------------|
| block_time | DateTime | No | Block Time |
| tx_hash | String | No | Tx Hash |
| user | String | Yes | User |
| referral_code | String | Yes | Referral Code |
| is_success | UInt64 | No | Is Success |
| error | String | No | Error |
| indexed_at | DateTime | No | Indexed At |

#### hyperliquid_set_referrer
**Sort Keys:** ⚡ referral_code, user

Referrer assignment events.

| Column | Type | Sort Key | Description |
|--------|------|----------|-------------|
| block_time | DateTime | No | Block Time |
| tx_hash | String | No | Tx Hash |
| user | String | Yes | User |
| referral_code | String | Yes | Referral Code |
| is_success | UInt64 | No | Is Success |
| error | String | No | Error |
| indexed_at | DateTime | No | Indexed At |

#### hyperliquid_sub_accounts
**Sort Keys:** ⚡ block_number, sub_account | **Partition:** toYYYYMM(snapshot_time)

Sub-account relationships and metadata.

| Column | Type | Sort Key | Description |
|--------|------|----------|-------------|
| block_number | UInt64 | Yes | Block Number |
| snapshot_time | DateTime | No | Snapshot Time |
| sub_account | FixedString | Yes | Sub Account |
| master_account | FixedString | No | Master Account |
| name | String | No | Name |
| indexed_at | DateTime | No | Indexed At |

#### hyperliquid_summaries
**Sort Keys:** ⚡ user, polled_at | **Partition:** toYYYYMM(polled_at)

Aggregated account summaries.

| Column | Type | Sort Key | Description |
|--------|------|----------|-------------|
| polled_at | DateTime | Yes | Polled At |
| user | String | Yes | User |
| cross_maintenance_margin_used | Decimal | No | Cross Maintenance Margin Used |
| cross_margin_account_value | Decimal | No | Cross Margin Account Value |
| cross_margin_total_margin_used | Decimal | No | Cross Margin Total Margin Used |
| cross_margin_total_natl_pos | Decimal | No | Cross Margin Total Natl Pos |
| cross_margin_total_raw_usd | Decimal | No | Cross Margin Total Raw Usd |
| margin_account_value | Decimal | No | Margin Account Value |
| margin_total_margin_used | Decimal | No | Margin Total Margin Used |
| margin_total_natl_pos | Decimal | No | Margin Total Natl Pos |
| margin_total_raw_usd | Decimal | No | Margin Total Raw Usd |
| withdrawable | Decimal | No | Withdrawable |
| indexed_at | DateTime | No | Indexed At |

#### hyperliquid_twap_statuses
**Sort Keys:** ⚡ block_number, twap_id, time | **Partition:** toYYYYMM(block_time)

TWAP order status and execution tracking.

| Column | Type | Sort Key | Description |
|--------|------|----------|-------------|
| block_number | UInt64 | Yes | Block Number |
| block_time | DateTime | No | Block Time |
| time | DateTime | Yes | Time |
| twap_id | UInt64 | Yes | Twap Id |
| status | String | No | Status |
| user | FixedString | No | User |
| coin | String | No | Coin |
| side | Enum | No | Side |
| size | Decimal | No | Size |
| executed_size | Decimal | No | Executed Size |
| executed_ntl | Decimal | No | Executed Ntl |
| minutes | UInt64 | No | Minutes |
| randomize | UInt64 | No | Randomize |
| reduce_only | UInt64 | No | Reduce Only |
| timestamp | DateTime | No | Timestamp |
| indexed_at | DateTime | No | Indexed At |

#### hyperliquid_vault_equities
**Sort Keys:** ⚡ block_number, vault_address, depositor | **Partition:** toYYYYMM(snapshot_time)

Vault equity values and performance.

| Column | Type | Sort Key | Description |
|--------|------|----------|-------------|
| block_number | UInt64 | Yes | Block Number |
| snapshot_time | DateTime | No | Snapshot Time |
| vault_address | FixedString | Yes | Vault Address |
| vault_name | String | No | Vault Name |
| leader | FixedString | No | Leader |
| depositor | FixedString | Yes | Depositor |
| ownership_fraction | Float64 | No | Ownership Fraction |
| net_deposits | Int64 | No | Net Deposits |
| leader_commission | Float64 | No | Leader Commission |
| indexed_at | DateTime | No | Indexed At |

#### hyperliquid_metrics_dex_overview
**Sort Keys:** ⚡ day, coin | **Partition:** toYYYYMM(day)

Daily DEX metrics by coin.

| Column | Type | Sort Key | Description |
|--------|------|----------|-------------|
| day | DateTime | Yes | Day |
| coin | String | Yes | Coin |
| volume_usd | Decimal | No | Volume Usd |
| fill_count | UInt64 | No | Fill Count |
| unique_traders | UInt64 | No | Unique Traders |
| fees | Decimal | No | Fees |
| liquidations | UInt64 | No | Liquidations |
| high_price | Decimal | No | High Price |
| low_price | Decimal | No | Low Price |

#### hyperliquid_metrics_overview
**Sort Keys:** ⚡ day | **Partition:** toYYYYMM(day)

Platform-wide aggregate metrics.

| Column | Type | Sort Key | Description |
|--------|------|----------|-------------|
| day | DateTime | Yes | Day |
| total_volume_usd | Decimal | No | Total Volume Usd |
| total_fills | UInt64 | No | Total Fills |
| active_traders | UInt64 | No | Active Traders |
| total_fees | Decimal | No | Total Fees |
| liquidation_count | UInt64 | No | Liquidation Count |
| liquidation_volume_usd | Decimal | No | Liquidation Volume Usd |
| coins_traded | UInt64 | No | Coins Traded |
| total_builder_fees | Decimal | No | Total Builder Fees |
| builder_fill_count | UInt64 | No | Builder Fill Count |


## Pre-Built Queries

### Quick Reference

| # | Query Name | Category | Keywords | Description |
|---|------------|----------|----------|-------------|
| 1 | Recent Trades | Trading | trades, latest, recent | Get most recent trades across all markets |
| 2 | Volume by Coin (24h) | Trading | volume, coin, 24h, statistics | Top coins by trading volume with price stats |
| 3 | Whale Trades (24h) | Trading | whale, large, trades, notional | Large trades over $100K |
| 4 | DEX Trades (Enriched View) | Trading | dex, trades, market, type | Trades with market type classification |
| 5 | Hourly Volume | Activity | hourly, volume, timeseries | Trading volume aggregated by hour |
| 6 | Daily Volume | Activity | daily, volume, traders | Daily trading volume with unique traders |
| 7 | Top Traders by Volume (7d) | Activity | traders, volume, leaderboard | Most active traders by volume |
| 8 | Recent Fills | Fills | fills, latest, executions | Most recent order fills |
| 9 | Recent Liquidations | Fills | liquidations, risk, forced | Recent liquidation events |
| 10 | Recent Orders | Orders | orders, latest, book | Most recent orders placed |
| 11 | Order Type Distribution (1h) | Orders | orders, distribution, types | Distribution of order types and status |
| 12 | Latest Funding Payments | Funding | funding, payments, grouped | Recent funding payments grouped by coin |
| 13 | Funding Rate History by Coin | Funding | funding, rate, history, trend | Historical funding rates over time |
| 14 | Block Activity | Infrastructure | blocks, activity, transactions | Block data with transaction counts |
| 15 | Recent Transactions | Infrastructure | transactions, recent, actions | Latest transactions with action types |
| 16 | Action Type Breakdown (24h) | Infrastructure | actions, breakdown, success | Transaction success rates by action type |
| 17 | Recent Asset Transfers | Ledger | transfers, assets, movements | Latest asset transfers between addresses |
| 18 | Transfer Volume by Type (7d) | Ledger | transfers, volume, type | Asset transfer volume by type |
| 19 | Recent Ledger Updates | Ledger | ledger, updates, changes | Latest ledger state changes |
| 20 | Bridge Deposits & Withdrawals | Ledger | bridge, deposits, withdrawals | Bridge activity between chains |
| 21 | Perpetual Markets | Markets | perp, perpetual, markets, list | List all perpetual markets |
| 22 | Spot Markets | Markets | spot, markets, tokens, list | List all spot markets |
| 23 | Market Context (Latest) | Markets | market, context, funding, prices | Latest market context data |
| 24 | Oracle Prices (All DEXes) | Markets | oracle, prices, dex | Latest oracle prices for all DEXes |
| 25 | Largest Open Positions (All DEXes) | Portfolio | positions, open, largest | Biggest open positions across users |
| 26 | Spot Balances for Address | Portfolio | spot, balances, tokens, user | Spot token balances for user |
| 27 | Vault Depositor Equity | Portfolio | vault, equity, depositor | Vault holdings and performance |
| 28 | Sub-Account Mappings | Portfolio | sub, account, mappings | Sub-account relationships |
| 29 | Bot/Agent Lookups | Portfolio | bot, agent, automated | Active bots and agents |
| 30 | Display Name Lookups | Portfolio | display, name, identifier | User display names |
| 31 | Full Portfolio View | Portfolio | portfolio, full, snapshot | Complete portfolio across all assets |
| 32 | Delegator Rewards by Address | Staking | delegator, rewards, staking | Staking rewards for a delegator |
| 33 | Validator Commission History | Staking | validator, commission, earnings | Validator commission and rewards history |
| 34 | Builder Activity (24h) | Builders | builder, activity, transactions | Builder transaction activity |
| 35 | Builder Fill Volume (24h) | Builders | builder, fill, volume | Builder fill volume and fees |
| 36 | Funding Rate Summary (Hourly) | Analytics | funding, rate, summary, hourly | Hourly funding rate statistics |
| 37 | Daily Liquidation Stats | Analytics | liquidations, daily, stats | Daily liquidation statistics by coin |
| 38 | Hourly OHLCV | Analytics | ohlcv, hourly, candles | Hourly OHLCV data |
| 39 | Daily Per-Coin Metrics | Analytics | daily, metrics, coin | Daily metrics by coin |
| 40 | Platform Daily Overview | Analytics | platform, overview, daily | Daily platform-wide statistics |


### Trading

#### **1. Recent Trades**

Last 100 trades with buyer/seller details.

**Keywords:** `trades`, `latest`, `recent`, `buyer`, `seller`, `fees`

```sql
SELECT
    timestamp,
    coin,
    side,
    price,
    size,
    price * size AS notional_usd,
    buyer_address,
    seller_address,
    buyer_fee,
    seller_fee,
    fee_token
FROM hyperliquid_trades
WHERE block_time > now() - INTERVAL 1 HOUR
ORDER BY block_number DESC, trade_id DESC
LIMIT 100
```

#### **2. Volume by Coin (24h)**

Top coins by trading volume in the last 24 hours.

**Keywords:** `volume`, `coin`, `24h`, `statistics`, `high`, `low`, `avg`

```sql
SELECT
    coin,
    count() AS trade_count,
    sum(price * size) AS volume_usd,
    min(price) AS low,
    max(price) AS high,
    avg(price) AS avg_price
FROM hyperliquid_trades
WHERE timestamp > now() - INTERVAL 24 HOUR
GROUP BY coin
ORDER BY volume_usd DESC
```

#### **3. Whale Trades (24h)**

Trades over $100K in the last 24 hours.

**Keywords:** `whale`, `large`, `trades`, `notional`, `high-value`

```sql
SELECT
    timestamp,
    coin,
    side,
    price,
    size,
    price * size AS notional_usd,
    buyer_address,
    seller_address
FROM hyperliquid_trades
WHERE timestamp > now() - INTERVAL 24 HOUR
    AND price * size > 100000
ORDER BY notional_usd DESC
```

#### **4. DEX Trades (Enriched View)**

Trades with market type (perpetual vs spot) from the enriched view.

**Keywords:** `dex`, `trades`, `market`, `type`, `perpetual`, `spot`

```sql
SELECT
    timestamp,
    coin,
    market_type,
    side,
    price,
    size,
    usd_amount,
    buyer_address,
    seller_address
FROM hyperliquid_dex_trades
WHERE block_time > now() - INTERVAL 1 HOUR
ORDER BY block_number DESC, trade_id DESC
LIMIT 100
```

---

### Activity

#### **5. Hourly Volume**

Trading volume aggregated by hour for the last 7 days.

**Keywords:** `hourly`, `volume`, `timeseries`, `trends`

```sql
SELECT
    toStartOfHour(timestamp) AS hour,
    count() AS trades,
    sum(price * size) AS volume_usd
FROM hyperliquid_trades
WHERE timestamp > now() - INTERVAL 7 DAY
GROUP BY hour
ORDER BY hour DESC
```

#### **6. Daily Volume**

Daily trading volume for the last 30 days.

**Keywords:** `daily`, `volume`, `traders`, `unique`, `users`

```sql
SELECT
    toStartOfDay(timestamp) AS day,
    count() AS trades,
    sum(price * size) AS volume_usd,
    uniqExact(buyer_address) + uniqExact(seller_address) AS unique_traders
FROM hyperliquid_trades
WHERE timestamp > now() - INTERVAL 30 DAY
GROUP BY day
ORDER BY day DESC
```

#### **7. Top Traders by Volume (7d)**

Most active traders by volume in the last 7 days.

**Keywords:** `traders`, `volume`, `leaderboard`, `top`, `active`

```sql
SELECT
    buyer_address AS trader,
    count() AS trade_count,
    sum(price * size) AS total_volume,
    uniqExact(coin) AS coins_traded
FROM hyperliquid_trades
WHERE timestamp > now() - INTERVAL 7 DAY
GROUP BY trader
ORDER BY total_volume DESC
```

---

### Fills

#### **8. Recent Fills**

Most recent order fills with fee and PnL information.

**Keywords:** `fills`, `latest`, `executions`, `pnl`, `fees`

```sql
SELECT
    time,
    coin,
    side,
    price,
    size,
    fee,
    fee_token,
    closed_pnl,
    dir,
    user,
    hash
FROM hyperliquid_fills
WHERE block_time > now() - INTERVAL 1 HOUR
ORDER BY block_number DESC, tid DESC
LIMIT 100
```

#### **9. Recent Liquidations**

Recent liquidation events with liquidated user details.

**Keywords:** `liquidations`, `risk`, `forced`, `closures`, `mark-price`

```sql
SELECT
    time,
    coin,
    side,
    price,
    size,
    price * size AS notional,
    liquidated_user,
    liquidation_mark_price,
    liquidation_method,
    user
FROM hyperliquid_fills
WHERE block_time > now() - INTERVAL 24 HOUR
    AND is_liquidation = 1
ORDER BY block_number DESC, tid DESC
LIMIT 100
```

---

### Orders

#### **10. Recent Orders**

Most recent orders placed on the platform.

**Keywords:** `orders`, `latest`, `book`, `limit`, `market`

```sql
SELECT
    status_time,
    coin,
    side,
    order_type,
    limit_price,
    size,
    orig_size,
    status,
    tif,
    user
FROM hyperliquid_orders
WHERE block_time > now() - INTERVAL 1 HOUR
ORDER BY block_number DESC, oid DESC
LIMIT 100
```

#### **11. Order Type Distribution (1h)**

Distribution of order types and status in the last hour.

**Keywords:** `orders`, `distribution`, `types`, `status`, `statistics`

```sql
SELECT
    order_type,
    side,
    status,
    count() AS order_count,
    uniqExact(user) AS unique_users
FROM hyperliquid_orders
WHERE block_time > now() - INTERVAL 1 HOUR
GROUP BY order_type, side, status
ORDER BY order_count DESC
```

---

### Funding

#### **12. Latest Funding Payments**

Recent funding rate payments grouped by coin and rate.

**Keywords:** `funding`, `payments`, `rates`, `position`, `fees`

```sql
SELECT
    coin,
    funding_rate,
    count() AS payments,
    sum(funding_amount) AS total_funding,
    avg(szi) AS avg_position_size
FROM hyperliquid_funding
WHERE time > now() - INTERVAL 8 HOUR
GROUP BY coin, funding_rate
ORDER BY abs(total_funding) DESC
```

#### **13. Funding Rate History by Coin**

Historical funding rates over time for specific coins.

**Keywords:** `funding`, `rate`, `history`, `trend`, `timeseries`

```sql
SELECT
    toStartOfHour(time) AS hour,
    coin,
    avg(funding_rate) AS avg_funding_rate,
    count() AS payment_count,
    sum(funding_amount) AS total_funding
FROM hyperliquid_funding
WHERE time > now() - INTERVAL 7 DAY
    AND coin IN ('BTC', 'ETH', 'SOL')
GROUP BY hour, coin
ORDER BY hour DESC, coin
```

---

### Infrastructure

#### **14. Block Activity**

Block data with transaction and event counts.

**Keywords:** `blocks`, `activity`, `transactions`, `events`, `fills`

```sql
SELECT
    block_number,
    block_time,
    fills_count,
    orders_count,
    misc_events_count,
    book_diffs_count,
    twap_statuses_count,
    writer_actions_count
FROM hyperliquid_blocks
ORDER BY block_number DESC
LIMIT 100
```

#### **15. Recent Transactions**

Latest transactions with action types and success status.

**Keywords:** `transactions`, `recent`, `actions`, `success`, `errors`

```sql
SELECT
    block_time,
    round,
    tx_hash,
    user,
    action_type,
    is_success,
    error
FROM hyperliquid_transactions
ORDER BY round DESC
LIMIT 100
```

#### **16. Action Type Breakdown (24h)**

Transaction success rates by action type in the last 24 hours.

**Keywords:** `actions`, `breakdown`, `success`, `rates`, `statistics`

```sql
SELECT
    action_type,
    count() AS tx_count,
    countIf(is_success = 1) AS success_count,
    countIf(is_success = 0) AS failed_count,
    round(countIf(is_success = 1) / count() * 100, 2) AS success_rate
FROM hyperliquid_transactions
WHERE block_time > now() - INTERVAL 24 HOUR
GROUP BY action_type
ORDER BY tx_count DESC
```

---

### Ledger

#### **17. Recent Asset Transfers**

Latest asset transfers between addresses.

**Keywords:** `transfers`, `assets`, `movements`, `deposits`, `withdrawals`

```sql
SELECT
    time,
    transfer_type,
    user,
    destination,
    token,
    amount,
    usdc_amount,
    fee,
    tx_hash
FROM hyperliquid_asset_transfers
ORDER BY block_number DESC
LIMIT 100
```

#### **18. Transfer Volume by Type (7d)**

Asset transfer volume by type in the last 7 days.

**Keywords:** `transfers`, `volume`, `type`, `aggregated`, `statistics`

```sql
SELECT
    transfer_type,
    count() AS transfer_count,
    sum(usdc_amount) AS total_usdc,
    uniqExact(user) AS unique_users
FROM hyperliquid_asset_transfers
WHERE time > now() - INTERVAL 7 DAY
GROUP BY transfer_type
ORDER BY total_usdc DESC
```

#### **19. Recent Ledger Updates**

Latest ledger state changes including balance updates.

**Keywords:** `ledger`, `updates`, `changes`, `balances`, `deltas`

```sql
SELECT
    time,
    delta_type,
    user,
    usdc_amount,
    token,
    amount,
    destination,
    fee,
    hash
FROM hyperliquid_ledger_updates
ORDER BY block_number DESC
LIMIT 100
```

#### **20. Bridge Deposits & Withdrawals**

Bridge activity between chains at latest snapshot.

**Keywords:** `bridge`, `deposits`, `withdrawals`, `cross-chain`, `eth`

```sql
SELECT
    bridge_type,
    user,
    amount_wei,
    tx_hash,
    eth_block_number,
    event_time,
    snapshot_time
FROM hyperliquid_bridge
WHERE block_number = (SELECT max(block_number) FROM hyperliquid_bridge)
ORDER BY amount_wei DESC
```

---

### Markets

#### **21. Perpetual Markets**

List all perpetual markets with leverage and decimals.

**Keywords:** `perp`, `perpetual`, `markets`, `list`, `leverage`

```sql
SELECT
    coin,
    market_index,
    sz_decimals,
    max_leverage,
    only_isolated
FROM hyperliquid_perpetual_markets
ORDER BY coin
```

#### **22. Spot Markets**

List all spot markets with token details.

**Keywords:** `spot`, `markets`, `tokens`, `list`, `canonical`

```sql
SELECT
    token_index,
    token,
    token_id,
    full_name,
    sz_decimals,
    wei_decimals,
    is_canonical,
    evm_contract
FROM hyperliquid_spot_markets
ORDER BY token_index
```

#### **23. Market Context (Latest)**

Latest market context data including funding, OI, and prices.

**Keywords:** `market`, `context`, `funding`, `prices`, `open-interest`

```sql
SELECT
    coin,
    funding,
    open_interest,
    mark_px,
    oracle_px,
    mid_px,
    premium,
    day_ntl_vlm,
    prev_day_px
FROM hyperliquid_perpetual_market_contexts
WHERE polled_at > now() - INTERVAL 5 MINUTE
ORDER BY day_ntl_vlm DESC
```

#### **24. Oracle Prices (All DEXes)**

Mark and daily prices for all assets at the latest snapshot.

**Keywords:** `oracle`, `prices`, `dex`, `mark`, `daily`

```sql
SELECT
    clearinghouse,
    coin,
    mark_px,
    daily_px,
    snapshot_time
FROM hyperliquid_oracle_prices
WHERE block_number = (SELECT max(block_number) FROM hyperliquid_oracle_prices)
ORDER BY clearinghouse, asset_idx
```

### Portfolio & Positions

#### **25. Largest Open Positions (All DEXes)**

Top positions by notional value across all users.

**Keywords:** `positions`, `open`, `largest`, `notional`, `leverage`

```sql
SELECT
    user,
    coin,
    size,
    entry_px,
    position_value,
    unrealized_pnl,
    return_on_equity,
    leverage_type,
    leverage_value,
    liquidation_price
FROM hyperliquid_perpetual_positions
WHERE polled_at > now() - INTERVAL 10 MINUTE
ORDER BY abs(position_value) DESC
LIMIT 100
```

#### **26. Spot Balances for Address**

Spot token balances for a specific user address.

**Keywords:** `spot`, `balances`, `tokens`, `user`, `holdings`

```sql
-- Replace address below
SELECT
    token,
    token_idx,
    total,
    escrowed,
    snapshot_time
FROM hyperliquid_spot_clearinghouse_states
WHERE user = lower('0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb')
    AND block_number = (SELECT max(block_number) FROM hyperliquid_spot_clearinghouse_states)
    AND total != 0
ORDER BY abs(total) DESC
```

#### **27. Vault Depositor Equity**

Vault holdings and performance for depositors.

**Keywords:** `vault`, `equity`, `depositor`, `ownership`, `commission`

```sql
SELECT
    vault_name,
    depositor,
    ownership_fraction,
    net_deposits,
    leader,
    leader_commission,
    snapshot_time
FROM hyperliquid_vault_equities
WHERE block_number = (SELECT max(block_number) FROM hyperliquid_vault_equities)
ORDER BY ownership_fraction DESC
```

#### **28. Sub-Account Mappings**

Sub-account relationships for a specific master account.

**Keywords:** `sub`, `account`, `mappings`, `master`, `relationships`

```sql
-- Replace address to find sub-accounts for a specific master
SELECT
    sub_account,
    master_account,
    name,
    snapshot_time
FROM hyperliquid_sub_accounts
WHERE master_account = lower('0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb')
    AND block_number = (SELECT max(block_number) FROM hyperliquid_sub_accounts)
ORDER BY name
```

#### **29. Bot/Agent Lookups**

Active bots and agents for a specific user.

**Keywords:** `bot`, `agent`, `automated`, `trading`, `permissions`

```sql
-- Replace address to find agents for a specific user
SELECT
    agent,
    user,
    name,
    valid_until,
    snapshot_time
FROM hyperliquid_agents
WHERE user = lower('0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb')
    AND block_number = (SELECT max(block_number) FROM hyperliquid_agents)
ORDER BY name
```

#### **30. Display Name Lookups**

User display names for address resolution.

**Keywords:** `display`, `name`, `identifier`, `username`, `alias`

```sql
SELECT
    user,
    display_name,
    snapshot_time
FROM hyperliquid_display_names
WHERE block_number = (SELECT max(block_number) FROM hyperliquid_display_names)
ORDER BY display_name
```

#### **31. Full Portfolio View**

Complete portfolio across all assets for a specific address.

**Keywords:** `portfolio`, `full`, `snapshot`, `complete`, `holdings`

```sql
-- Replace address below for full portfolio snapshot
SELECT
    'perps' AS type,
    coin AS asset,
    size AS amount,
    entry_notional AS extra
FROM hyperliquid_clearinghouse_states
WHERE user = lower('0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb')
    AND block_number = (SELECT max(block_number) FROM hyperliquid_clearinghouse_states)
    AND size != 0
UNION ALL
SELECT
    'spot',
    token,
    total,
    escrowed
FROM hyperliquid_spot_clearinghouse_states
WHERE user = lower('0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb')
    AND block_number = (SELECT max(block_number) FROM hyperliquid_spot_clearinghouse_states)
    AND total != 0
UNION ALL
SELECT
    'vault',
    vault_name,
    toInt64(ownership_fraction * 1000000),
    net_deposits
FROM hyperliquid_vault_equities
WHERE depositor = lower('0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb')
    AND block_number = (SELECT max(block_number) FROM hyperliquid_vault_equities)
UNION ALL
SELECT
    'delegation',
    validator,
    toInt64(reward),
    toInt64(commission_bps)
FROM hyperliquid_delegator_rewards
WHERE delegator = lower('0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb')
    AND block_number = (SELECT max(block_number) FROM hyperliquid_delegator_rewards)
```

### Staking & Rewards

#### **32. Delegator Rewards by Address**

Staking rewards for a specific delegator address.

**Keywords:** `delegator`, `rewards`, `staking`, `delegation`, `validator`

```sql
-- Replace address below
SELECT
    delegator,
    validator,
    reward,
    commission_bps,
    snapshot_time
FROM hyperliquid_delegator_rewards
WHERE delegator = lower('0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb')
    AND block_number = (SELECT max(block_number) FROM hyperliquid_delegator_rewards)
ORDER BY reward DESC
```

#### **33. Validator Commission History**

Validator commission and rewards history over time.

**Keywords:** `validator`, `commission`, `earnings`, `rewards`, `history`

```sql
SELECT
    time,
    validator,
    reward,
    block_number
FROM hyperliquid_validator_rewards
ORDER BY block_number DESC
LIMIT 100
```

### Builders

#### **34. Builder Activity (24h)**

Builder transaction activity in the last 24 hours.

**Keywords:** `builder`, `activity`, `transactions`, `fees`, `users`

```sql
SELECT
    b.builder,
    l.builder_name,
    count() AS tx_count,
    sum(b.builder_fee) AS total_fees,
    uniqExact(b.user) AS unique_users
FROM hyperliquid_builder_transactions b
LEFT JOIN hyperliquid_builder_labels l ON b.builder = l.builder_address
WHERE b.block_time > now() - INTERVAL 24 HOUR
GROUP BY b.builder, l.builder_name
ORDER BY total_fees DESC
```

#### **35. Builder Fill Volume (24h)**

Builder fill volume and fees in the last 24 hours.

**Keywords:** `builder`, `fill`, `volume`, `fees`, `statistics`

```sql
SELECT
    builder_address,
    count() AS fills,
    sum(price * size) AS volume_usd,
    sum(builder_fee) AS total_builder_fees,
    uniqExact(user) AS unique_users
FROM hyperliquid_builder_fills
WHERE block_time > now() - INTERVAL 24 HOUR
GROUP BY builder_address
ORDER BY volume_usd DESC
```

### Analytics

#### **36. Funding Rate Summary (Hourly)**

Hourly funding rate statistics from aggregated view.

**Keywords:** `funding`, `rate`, `summary`, `hourly`, `aggregated`

```sql
SELECT
    coin,
    hour,
    avg_funding_rate,
    total_funding,
    unique_users
FROM hyperliquid_funding_summary_hourly
WHERE hour > now() - INTERVAL 24 HOUR
ORDER BY hour DESC, abs(avg_funding_rate) DESC
```

#### **37. Daily Liquidation Stats**

Daily liquidation statistics by coin.

**Keywords:** `liquidations`, `daily`, `stats`, `volume`, `users`

```sql
SELECT
    day,
    coin,
    liquidation_count,
    liquidated_volume,
    unique_liquidated_users
FROM hyperliquid_liquidations_daily
ORDER BY day DESC, liquidated_volume DESC
```

#### **38. Hourly OHLCV**

Hourly OHLCV data for a specific coin.

**Keywords:** `ohlcv`, `hourly`, `candles`, `price`, `volume`

```sql
SELECT
    coin,
    hour,
    volume,
    trade_count,
    high,
    low,
    open,
    close
FROM hyperliquid_market_volume_hourly
WHERE hour > now() - INTERVAL 24 HOUR
    AND coin = 'BTC'
ORDER BY hour DESC
```

#### **39. Daily Per-Coin Metrics**

Daily metrics by coin from aggregated view.

**Keywords:** `daily`, `metrics`, `coin`, `volume`, `fees`, `liquidations`

```sql
SELECT
    day,
    coin,
    volume_usd,
    fill_count,
    unique_traders,
    fees,
    liquidations,
    high_price,
    low_price
FROM hyperliquid_metrics_dex_overview
ORDER BY day DESC, volume_usd DESC
```

#### **40. Platform Daily Overview**

Daily platform-wide statistics and metrics.

**Keywords:** `platform`, `overview`, `daily`, `total`, `aggregate`

```sql
SELECT
    day,
    total_volume_usd,
    total_fills,
    active_traders,
    total_fees,
    liquidation_count,
    liquidation_volume_usd,
    coins_traded,
    total_builder_fees,
    builder_fill_count
FROM hyperliquid_metrics_overview
ORDER BY day DESC
```

## Common Query Patterns

### Time-Based Analysis

```sql
-- Trading volume over time
SELECT
  toStartOfHour(time) AS hour,
  coin,
  SUM(toFloat64(size)) AS volume,
  COUNT(*) AS trade_count,
  AVG(toFloat64(price)) AS avg_price
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
  SUM(toFloat64(size) * CASE WHEN side = 'buy' THEN 1 ELSE -1 END) AS net_position,
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
  price AS price,
  SUM(toFloat64(size)) AS total_size,
  COUNT(*) AS order_count
FROM hyperliquid_order_book_diffs
WHERE coin = 'BTC'
  AND time >= now() - INTERVAL 1 MINUTE
GROUP BY side, price
ORDER BY side DESC, price DESC
```

### Funding Rate Analysis

```sql
-- Funding rate trends
SELECT
  coin,
  toStartOfDay(time) AS day,
  AVG(toFloat64(funding_rate)) AS avg_funding_rate,
  SUM(toFloat64(usdc)) AS total_funding_paid
FROM hyperliquid_funding
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
  SUM(toFloat64(size)) AS volume,
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
  price AS price,
  size AS size,
  toFloat64(price) * toFloat64(size) AS notional,
  user
FROM hyperliquid_trades
WHERE toFloat64(price) * toFloat64(size) >= 500000
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
FROM hyperliquid_liquidations_daily
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
SELECT toFloat64(price) AS price_numeric
SELECT toFloat64(size) AS size_numeric

-- Calculations require conversion
WHERE toFloat64(price) * toFloat64(size) > 100000
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
SELECT SUM(toFloat64(size)) AS total_volume

-- Average with filtering
SELECT AVG(toFloat64(price)) AS avg_price
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
WHERE trade_id > 1000000
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
  SELECT user, SUM(toFloat64(size)) AS volume
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
SELECT user, size FROM hyperliquid_trades WHERE user = '0xADDRESS'

-- Less efficient
SELECT user, toFloat64(size) FROM hyperliquid_trades WHERE toFloat64(size) > 100
```

## Best Practices

1. **Start with time filters** - Reduce data scanned using `block_time` or `time` filters
2. **Use sort keys** - Filter on sort key columns for best performance
3. **Test incrementally** - Start with simple queries, add complexity gradually while monitoring performance
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
| Syntax error | Invalid SQL | Verify SQL syntax, check schema reference for correct column names and types |
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
  -H 'x-api-key: <your-api-key>' \
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
    'x-api-key': '<your-api-key>'
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
    "x-api-key": "<your-api-key>"
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
request['x-api-key'] = '<your-api-key>'

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
    req.Header.Set("x-api-key", "<your-api-key>")

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
  AVG(toFloat64(price)) AS avg_price,
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
  SUM(toFloat64(size)) AS volume,
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
FROM hyperliquid_liquidations_daily
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
