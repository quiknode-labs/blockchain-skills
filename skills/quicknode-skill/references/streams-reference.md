# Quicknode Streams Reference

Streams provide real-time & historical blockchain data pipelines that filter, transform, and deliver data to various destinations.

## Stream Architecture

```
Blockchain → Quicknode → Filter Function → Transform → Destination
              Node                                      (Webhook/S3/DB)
```

## Stream Types

When creating a stream via the API, specify the `dataset` parameter:

| Stream Type | `dataset` Value |
|-------------|----------------|
| Block | `block` |
| Block with Receipts | `block_with_receipts` |
| Transaction | `transaction` |
| Logs | `log` |
| Receipt | `receipt` |

### Block Streams

Receive full block data including all transactions.

```javascript
function main(stream) {
  const block = stream.data;

  return {
    blockNumber: block.number,
    timestamp: block.timestamp,
    transactionCount: block.transactions.length,
    gasUsed: block.gasUsed,
    baseFeePerGas: block.baseFeePerGas
  };
}
```

### Transaction Streams

Receive individual transaction data.

```javascript
function main(stream) {
  const tx = stream.data;

  // Filter high-value transactions (> 10 ETH)
  const value = BigInt(tx.value);
  const threshold = BigInt('10000000000000000000'); // 10 ETH

  if (value > threshold) {
    return {
      hash: tx.hash,
      from: tx.from,
      to: tx.to,
      value: tx.value,
      gasPrice: tx.gasPrice
    };
  }

  return null; // Filter out
}
```

### Logs Streams

Receive contract event logs.

```javascript
function main(stream) {
  const TRANSFER_TOPIC = '0xddf252ad1be2c89b69c2b068fc378daa952ba7f163c4a11628f55a4df523b3ef';

  const transfers = stream.data.filter(log =>
    log.topics[0] === TRANSFER_TOPIC
  );

  return transfers.map(log => ({
    contract: log.address,
    from: '0x' + log.topics[1].slice(26),
    to: '0x' + log.topics[2].slice(26),
    value: log.data,
    blockNumber: log.blockNumber,
    transactionHash: log.transactionHash
  }));
}
```

### Receipt Streams

Receive transaction receipts with execution results.

```javascript
function main(stream) {
  const receipt = stream.data;

  return {
    transactionHash: receipt.transactionHash,
    status: receipt.status === '0x1' ? 'success' : 'failed',
    gasUsed: receipt.gasUsed,
    effectiveGasPrice: receipt.effectiveGasPrice,
    logsCount: receipt.logs.length
  };
}
```

## Filter Functions

### Function Signature

```javascript
function main(stream) {
  // stream.data - Blockchain data (block, tx, logs, receipt)
  // stream.metadata - Stream metadata (network, streamId, etc.)

  // Return data to send to destination
  // Return null to filter out
  return processedData;
}
```

### Available Utilities

```javascript
function main(stream) {
  // BigInt for large numbers
  const value = BigInt(stream.data.value);

  // Hex conversions
  const decimal = parseInt(stream.data.gasUsed, 16);

  // String operations
  const address = stream.data.to.toLowerCase();

  return { value: value.toString(), decimal, address };
}
```

### Complex Filter Example

```javascript
function main(stream) {
  // Monitor multiple DEX contracts for swap events
  const DEX_CONTRACTS = [
    '0x7a250d5630B4cF539739dF2C5dAcb4c659F2488D', // Uniswap V2
    '0xE592427A0AEce92De3Edee1F18E0157C05861564', // Uniswap V3
    '0xd9e1cE17f2641f24aE83637ab66a2cca9C378B9F', // SushiSwap
  ].map(a => a.toLowerCase());

  const SWAP_TOPICS = [
    '0xd78ad95fa46c994b6551d0da85fc275fe613ce37657fb8d5e3d130840159d822', // V2 Swap
    '0xc42079f94a6350d7e6235f29174924f928cc2ac818eb64fed8004e115fbcca67', // V3 Swap
  ];

  const swaps = stream.data.filter(log =>
    DEX_CONTRACTS.includes(log.address.toLowerCase()) &&
    SWAP_TOPICS.includes(log.topics[0])
  );

  if (swaps.length === 0) return null;

  return swaps.map(log => ({
    dex: log.address,
    txHash: log.transactionHash,
    blockNumber: log.blockNumber,
    topic: log.topics[0],
    data: log.data
  }));
}
```

## Destinations

### Webhook

Send data to HTTP endpoints.

**Configuration:**
```json
{
  "type": "webhook",
  "url": "https://your-server.com/webhook",
  "headers": {
    "Authorization": "Bearer <your-token>",
    "Content-Type": "application/json"
  },
  "retryPolicy": {
    "maxRetries": 3,
    "backoffMs": 1000
  }
}
```

**Payload Format:**
```json
{
  "streamId": "stream_abc123",
  "network": "ethereum-mainnet",
  "dataset": "block",
  "data": { /* your filtered/transformed data */ },
  "metadata": {
    "blockNumber": 18000000,
    "timestamp": 1693526400
  }
}
```

### Amazon S3

Store data in S3 buckets.

**Configuration:**
```json
{
  "type": "s3",
  "bucket": "your-bucket-name",
  "region": "us-east-1",
  "prefix": "blockchain-data/",
  "format": "json",
  "compression": "gzip",
  "credentials": {
    "accessKeyId": "YOUR_ACCESS_KEY",
    "secretAccessKey": "YOUR_SECRET_KEY"
  }
}
```

### PostgreSQL

Insert data directly into PostgreSQL.

**Configuration:**
```json
{
  "type": "postgresql",
  "connectionString": "postgresql://user:pass@host:5432/db",
  "table": "blockchain_events",
  "schema": {
    "tx_hash": "TEXT",
    "block_number": "BIGINT",
    "from_address": "TEXT",
    "to_address": "TEXT",
    "value": "NUMERIC",
    "timestamp": "TIMESTAMP"
  }
}
```

### Snowflake

Stream to Snowflake data warehouse.

**Configuration:**
```json
{
  "type": "snowflake",
  "account": "your-account",
  "warehouse": "COMPUTE_WH",
  "database": "BLOCKCHAIN_DATA",
  "schema": "PUBLIC",
  "table": "EVENTS",
  "credentials": {
    "username": "YOUR_USERNAME",
    "password": "YOUR_PASSWORD"
  }
}
```

### Key-Value Store Integration

Streams filter functions can use the `qnLib` helper to persist state across invocations via the Key-Value Store. This enables use cases like tracking seen addresses, maintaining watchlists, or accumulating counters.

> **Important:** All `qnLib` methods are asynchronous and return Promises. Declare your filter function as `async function main(stream)` and use `await` on all `qnLib` calls.

```javascript
async function main(stream) {
  const tx = stream.data;

  // Check if sender is on our watchlist
  const isWatched = await qnLib.qnContainsListItems('watchlist', [tx.from]);

  if (isWatched.includes(tx.from)) {
    // Store the transaction hash in a set for later retrieval
    await qnLib.qnAddSet('watched_txs', tx.hash, JSON.stringify({
      from: tx.from,
      to: tx.to,
      value: tx.value,
      block: tx.blockNumber
    }));

    return {
      type: 'watchlist_hit',
      hash: tx.hash,
      from: tx.from,
      to: tx.to,
      value: tx.value
    };
  }

  return null;
}
```

See the [Key-Value Store docs](https://www.quicknode.com/docs/key-value-store) for full `qnLib` method reference.

## EVM Stream Examples

### Monitor Specific Contract

```javascript
function main(stream) {
  const TARGET_CONTRACT = '0xdAC17F958D2ee523a2206206994597C13D831ec7'; // USDT

  const logs = stream.data.filter(log =>
    log.address.toLowerCase() === TARGET_CONTRACT.toLowerCase()
  );

  return logs.length > 0 ? { events: logs } : null;
}
```

### Track Whale Transactions

```javascript
function main(stream) {
  const WHALE_THRESHOLD = BigInt('1000000000000000000000'); // 1000 ETH

  const tx = stream.data;
  const value = BigInt(tx.value || '0');

  if (value >= WHALE_THRESHOLD) {
    return {
      type: 'whale_transaction',
      hash: tx.hash,
      from: tx.from,
      to: tx.to,
      valueEth: (Number(value) / 1e18).toFixed(2)
    };
  }

  return null;
}
```

### NFT Transfer Tracking

```javascript
function main(stream) {
  // ERC-721 Transfer event
  const TRANSFER_TOPIC = '0xddf252ad1be2c89b69c2b068fc378daa952ba7f163c4a11628f55a4df523b3ef';

  const nftTransfers = stream.data.filter(log =>
    log.topics[0] === TRANSFER_TOPIC &&
    log.topics.length === 4 // ERC-721 has tokenId in topics[3]
  );

  return nftTransfers.map(log => ({
    contract: log.address,
    from: '0x' + log.topics[1].slice(26),
    to: '0x' + log.topics[2].slice(26),
    tokenId: BigInt(log.topics[3]).toString(),
    txHash: log.transactionHash
  }));
}
```

## Solana Stream Examples

### Monitor Program Logs

```javascript
function main(stream) {
  const PROGRAM_ID = 'TokenkegQfeZyiNwAJbNbGKPFXCWuBvf9Ss623VQ5DA'; // SPL Token

  const programLogs = stream.data.filter(log =>
    log.programId === PROGRAM_ID
  );

  return programLogs.length > 0 ? { logs: programLogs } : null;
}
```

### Track SOL Transfers

```javascript
function main(stream) {
  const THRESHOLD = 1000000000000; // 1000 SOL in lamports

  const tx = stream.data;

  const transfers = tx.meta?.preBalances?.map((pre, i) => ({
    account: tx.transaction.message.accountKeys[i],
    change: tx.meta.postBalances[i] - pre
  })).filter(t => Math.abs(t.change) >= THRESHOLD);

  return transfers?.length > 0 ? { transfers } : null;
}
```

## Stream Management API

### Create Stream

```bash
curl -X POST https://api.quicknode.com/streams/rest/v1/streams \
  -H "Authorization: Bearer $QUICKNODE_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "My Stream",
    "network": "ethereum-mainnet",
    "dataset": "receipts",
    "filterFunction": "function main(stream) { return stream.data; }",
    "destination": {
      "type": "webhook",
      "url": "https://your-server.com/webhook"
    }
  }'
```

### List Streams

```bash
curl https://api.quicknode.com/streams/rest/v1/streams \
  -H "Authorization: Bearer $QUICKNODE_API_KEY"
```

### Update Stream

```bash
curl -X PATCH https://api.quicknode.com/streams/rest/v1/streams/{streamId} \
  -H "Authorization: Bearer $QUICKNODE_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "status": "paused"
  }'
```

### Delete Stream

```bash
curl -X DELETE https://api.quicknode.com/streams/rest/v1/streams/{streamId} \
  -H "Authorization: Bearer $QUICKNODE_API_KEY"
```

## Best Practices

1. **Start narrow** - Begin with specific filters, expand as needed
2. **Test locally** - Validate filter functions before deployment
3. **Handle errors** - Implement proper error handling in destinations
4. **Monitor health** - Check stream status regularly in dashboard
5. **Use batching** - Batch webhook deliveries for high-volume streams
6. **Idempotency** - Design consumers to handle duplicate deliveries

## Troubleshooting

| Issue | Cause | Solution |
|-------|-------|----------|
| No data received | Filter too restrictive | Broaden filter conditions |
| Webhook failures | Endpoint unavailable | Check server health, increase timeout |
| Data lag | Processing backlog | Optimize filter function |
| Missing events | Incorrect topics | Verify event signatures |

## Documentation

- **Streams Overview**: https://www.quicknode.com/docs/streams
- **Streams Overview (llms.txt)**: https://www.quicknode.com/docs/streams/llms.txt
- **Filter Functions**: https://www.quicknode.com/docs/streams/filters
- **Destinations**: https://www.quicknode.com/docs/streams/destinations
- **API Reference**: https://www.quicknode.com/docs/streams/rest-api/getting-started
- **Guides**: https://www.quicknode.com/guides/tags/streams
