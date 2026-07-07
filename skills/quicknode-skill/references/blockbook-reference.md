# Blockbook Reference

Wallet-centric blockchain data via the Blockbook add-on on your Quicknode endpoint. Provides address balances, UTXO sets, transaction history, and balance history — available via JSON-RPC or REST. Particularly useful for UTXO-based chains (Bitcoin, Dogecoin, Litecoin, etc.) where standard RPC methods lack wallet-level indexing.

**Marketing:** https://www.quicknode.com/blockbook
**Docs:** https://www.quicknode.com/docs/bitcoin/blockbook/overview

## Prerequisites

Activate the **Blockbook** add-on for each chain you need in the Quicknode dashboard. Each chain is a separate add-on. Blockbook methods are available on your existing endpoint URL once active.

## Supported Chains

| Chain | Add-on |
|-------|--------|
| Bitcoin (BTC) | BTC Blockbook |
| Ethereum (ETH) | ETH Blockbook |
| Bitcoin Cash (BCH) | BCH Blockbook |
| Dogecoin (DOGE) | Dogecoin Blockbook |
| Litecoin (LTC) | Litecoin Blockbook |
| Zcash (ZEC) | Zcash Blockbook |

## Methods Overview

| Method | Description |
|--------|-------------|
| `bb_getAddress` | Address balance, total received/sent, and transaction list |
| `bb_getXPUB` | XPUB-level balance and transaction history |
| `bb_getUTXOs` | Unspent transaction outputs (UTXOs) for an address or XPUB |
| `bb_getBalanceHistory` | Historical balance for an address over time |
| `bb_getTx` | Transaction details by txid |
| `bb_getTxSpecific` | Backend-native transaction details by txid |
| `bb_getBlock` | Block data including transactions |
| `bb_getBlockHash` | Block hash by block height |
| `bb_getEstimateFee` | Fee estimation for a target confirmation window |
| `bb_getTickers` / `bb_getTickersList` | Fiat ticker data |
| `bb_sendTransaction` | Broadcast a signed raw transaction |

Both JSON-RPC and Blockbook REST interfaces are available on the same endpoint. The REST base path is `/addon/3/api/v2/` on a Bitcoin endpoint with the add-on enabled.

## Get Address Balance and History

```typescript
const response = await fetch(process.env.QUICKNODE_RPC_URL!, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    jsonrpc: '2.0',
    id: 1,
    method: 'bb_getAddress',
    params: [
      'bc1qxy2kgdygjrsqtzq2n0yrf2493p83kkfjhx0wlh',
      {
        details: 'txs', // basic | txids | txs
        size: 10,
        page: 1,
      },
    ],
  }),
});

const { result } = await response.json();
// result.address — the queried address
// result.balance — confirmed balance in satoshis
// result.unconfirmedBalance — unconfirmed balance in satoshis
// result.totalReceived — total received in satoshis
// result.totalSent — total sent in satoshis
// result.txs — total transaction count
// result.transactions — array of transaction objects (when details='txs')
```

## Get UTXOs

```typescript
const response = await fetch(process.env.QUICKNODE_RPC_URL!, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    jsonrpc: '2.0',
    id: 1,
    method: 'bb_getUTXOs',
    params: [
      'bc1qxy2kgdygjrsqtzq2n0yrf2493p83kkfjhx0wlh',
      { confirmed: true }, // true = confirmed UTXOs only
    ],
  }),
});

const { result } = await response.json();
// result — array of UTXO objects:
// [{ txid, vout, value, height, confirmations, coinbase? }]
```

**Example UTXO:**
```json
{
  "txid": "7f3a1c2b...",
  "vout": 0,
  "value": "500000",
  "height": 820000,
  "confirmations": 6
}
```

## Get Balance History

```typescript
const response = await fetch(process.env.QUICKNODE_RPC_URL!, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    jsonrpc: '2.0',
    id: 1,
    method: 'bb_getBalanceHistory',
    params: [
      'bc1qxy2kgdygjrsqtzq2n0yrf2493p83kkfjhx0wlh',
      { groupBy: 86400 }, // seconds per bucket (86400 = daily)
    ],
  }),
});

const { result } = await response.json();
// result — array of { time, txs, received, sent, sentToSelf, rates }
```

## REST Access

```bash
curl -X GET \
  "https://YOUR_QUICKNODE_ENDPOINT.com/addon/3/api/v2/address/bc1p72h09wplu60qdxyr8q3ftgdhga7jxnjhdz08qs4u9we9q3lzmqmqa4yzj6?details=txids"
```

## Troubleshooting

| Error | Cause | Fix |
|-------|-------|-----|
| Method not found | Add-on not enabled | Activate Blockbook for this chain on your endpoint in the dashboard |
| Invalid address | Incorrect address format | Verify address format matches the chain (e.g., bech32 for BTC) |
| Empty result | Address has no activity | Confirm the address is on the correct network |

## Documentation

- **Blockbook Overview**: https://www.quicknode.com/docs/bitcoin/blockbook/overview
- **Marketing**: https://www.quicknode.com/blockbook
