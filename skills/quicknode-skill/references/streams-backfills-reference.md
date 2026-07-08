# Streams Backfills Reference

Streams Backfills use Quicknode Streams to retrieve historical blockchain data over a selected block or slot range. They are part of the Streams workflow: choose a chain, dataset, range, optional filter, batching/compression settings, and a destination.

**Product:** https://www.quicknode.com/streams/backfills
**Docs:** https://www.quicknode.com/docs/streams/backfilling

## When to Use Backfills

- Populate a database or index with historical chain data.
- Recover data after a destination outage or missed ingestion window.
- Reprocess historical blocks after changing a filter function.
- Run analytics, compliance, or audit jobs over a bounded historical range.

## How It Works

1. Select the chain and network supported by Streams.
2. Pick the dataset, such as EVM blocks, logs, receipts, traces, Solana blocks, Bitcoin blocks, or XRPL ledgers.
3. Define the start block/slot and end block/slot, or configure the Stream to continue into live delivery.
4. Add a filter function when only a subset of the data should be delivered.
5. Configure batching and compression for high-volume historical delivery.
6. Choose a destination such as Webhook, S3, PostgreSQL, Azure Storage, or another Streams destination.
7. Test the destination and filter, then start the Stream.

## What an Agent Should Ask For

| Field | Required | Notes |
|-------|----------|-------|
| Chain/network | Yes | Example: `ethereum-mainnet`, `base-mainnet`, `solana-mainnet` |
| Dataset | Yes | Block, transactions, logs, receipts, traces, or chain-specific dataset |
| Start block/slot | Yes | Historical lower bound |
| End block/slot | Yes | Historical upper bound, or continue to live delivery |
| Destination | Yes | Webhook, S3, PostgreSQL, Azure Storage, etc. |
| Filter function | Optional | Use for server-side filtering/transformation |
| Batching/compression | Recommended | Important for high-volume backfills |

## Dataset Notes

### EVM Chains

Common datasets include Block, Block with Receipts, Transactions, Logs, Receipts, Traces (`debug_trace`), Traces (`trace_block`), and composite block/receipt/trace datasets. Availability can vary by chain and plan.

Useful helper: `decodeEVMReceipts` can decode EVM receipts with contract ABIs inside a Streams filter.

### Solana

Streams supports Solana historical slot ranges on paid plans. Common datasets include Block and Programs + Logs. Very large Solana backfills should usually be scoped to a specific time window, account, program, or filter.

### Bitcoin

Bitcoin and Bitcoin Cash backfills use UTXO-oriented block data, including Blockbook-backed block payloads.

## Cost and Performance

- Backfills consume API credits based on blocks processed and dataset multipliers.
- Filtering reduces delivered payloads, but credits are still based on processed blocks.
- Use the Streams API Credits Calculator before large backfills.
- Increase batch size for historical ingestion when the destination can handle larger payloads.
- Enable gzip compression for large payloads.
- Use Elastic Batch if the Stream should transition from historical catch-up into low-latency live delivery.

## Automation Guidance

Use the Quicknode Streams REST API, SDK, or CLI to create and manage Streams programmatically. The current public docs describe Backfills as a Streams configuration/workflow, not as a separate Backfills REST resource. Agents should create or duplicate a Stream with the desired historical range rather than calling invented `/backfills` endpoints.

CLI example:

```bash
qn stream create \
  --name ethereum-usdc-backfill \
  --network ethereum-mainnet \
  --dataset logs \
  --start 17811625 \
  --end 17821625 \
  --region usa-east \
  --webhook https://example.com/streams
```

## Documentation

- **Streams Backfills**: https://www.quicknode.com/docs/streams/backfilling
- **Backfill Templates**: https://www.quicknode.com/streams/backfills
- **Streams Overview**: https://www.quicknode.com/docs/streams
- **Streams Data Sources**: https://www.quicknode.com/docs/streams/data-sources
- **Streams Destinations**: https://www.quicknode.com/docs/streams/destinations
- **Streams Filters**: https://www.quicknode.com/docs/streams/filters
- **Streams Billing**: https://www.quicknode.com/docs/streams/billing
