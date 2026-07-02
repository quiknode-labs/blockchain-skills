# Quicknode Marketplace Add-ons Reference

Quicknode Marketplace add-ons extend a Quicknode endpoint with specialized APIs or infrastructure behavior. Enable add-ons from the endpoint's **Add-ons** tab in the Quicknode dashboard.

This file intentionally stays concise. Product-specific references live in their own files when an agent is likely to invoke them directly:

- [Metaplex DAS API](metaplex-das-reference.md)
- [Swap API](swap-api-reference.md)
- [Solana gRPC](solana-grpc-reference.md)
- [Blockbook](blockbook-reference.md)
- [Ordinals & Runes API](ordinals-runes-reference.md)

## Solana Add-ons

### Priority Fee API

Use `qn_estimatePriorityFees` to estimate Solana priority fees from recent blocks and, optionally, account-specific activity.

**Docs:** https://www.quicknode.com/docs/solana/qn_estimatePriorityFees

```typescript
import { createSolanaRpc } from "@solana/kit";

const rpc = createSolanaRpc(process.env.QUICKNODE_RPC_URL!);

const response = await rpc.request("qn_estimatePriorityFees", {
  last_n_blocks: 100,
  account: "YourAccountPubkey...",
}).send();

console.log(response.result.per_compute_unit);
```

### Jito Bundles

Use Jito bundle methods for MEV-aware Solana transaction submission when the add-on is enabled.

**Docs:** https://www.quicknode.com/docs/solana/sendBundle

```typescript
const result = await rpc.request("sendBundle", {
  transactions: [
    "Base64EncodedTx1...",
    "Base64EncodedTx2...",
  ],
}).send();

const status = await rpc.request("getBundleStatuses", {
  bundleIds: [result.bundleId],
}).send();
```

## Transaction Delivery Add-ons

### Single Flight RPC

Single Flight RPC deduplicates identical in-flight RPC requests so high-concurrency applications do not repeatedly send the same expensive request while the first one is still resolving.

**Marketplace:** https://marketplace.quicknode.com/add-on/single-flight-rpc

Use this when an application has request bursts for identical reads, for example many users loading the same token, NFT, or block state at once.

### Multi-region Transaction Broadcast

Multi-region Transaction Broadcast improves transaction propagation by broadcasting signed transactions across multiple regions.

**Marketplace:** https://marketplace.quicknode.com/add-on/multi-region-transaction-broadcast

Use this for latency-sensitive transaction submission. For EVM endpoints, prefer Quicknode custom broadcast methods documented in the chain/RPC reference when available.

## Data and Risk Add-ons

### Scorechain Risk Assessment API

Scorechain Risk Assessment API provides wallet or transaction risk data for compliance, monitoring, and fraud analysis workflows.

**Marketplace:** https://marketplace.quicknode.com/add-on/risk-assessment-api

Use it when the user explicitly needs risk scoring or compliance context. Do not substitute it for standard wallet balance or transaction-history APIs.

### Block Timestamp Lookup

Block Timestamp Lookup maps timestamps to nearby block heights and can help agents translate human time windows into block ranges.

**Marketplace:** https://marketplace.quicknode.com/add-on/block-timestamp-lookup

For common EVM and Bitcoin timestamp-to-block workflows, also check Core RPC custom methods such as `qn_getBlockFromTimestamp` and `qn_getBlocksInTimestampRange`.

### Multi-chain Stablecoin Balance API

Multi-chain Stablecoin Balance API returns stablecoin balances across supported chains for portfolio, treasury, and payments workflows.

**Marketplace:** https://marketplace.quicknode.com/add-on/multi-chain-stablecoin-balance-api

Use it when the desired output is specifically stablecoin exposure across chains.

### Covalent GoldRush Wallet/Data APIs

Covalent GoldRush APIs provide wallet and token/NFT portfolio data across many chains.

**Marketplace:** https://marketplace.quicknode.com/add-on/covalent-wallet-api

Use it for multi-chain wallet summaries, portfolio views, token balances, or transaction histories when the endpoint has the Covalent add-on enabled.

## Documentation

- **Marketplace**: https://marketplace.quicknode.com/
- **Solana Priority Fees**: https://www.quicknode.com/docs/solana/qn_estimatePriorityFees
- **Jito Bundles**: https://www.quicknode.com/docs/solana/sendBundle
- **Core RPC API Reference**: https://www.quicknode.com/docs
