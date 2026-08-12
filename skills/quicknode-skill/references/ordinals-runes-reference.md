# Ordinals & Runes API Reference

Bitcoin inscription and Runes protocol data via the Quicknode Ordinals & Runes API add-on. 21 methods covering inscriptions, satoshi metadata, Runes, and Bitcoin outputs — available through JSON-RPC and ORD-compatible REST on your Quicknode Bitcoin endpoint. No custom indexer required.

**Marketing:** https://www.quicknode.com/ordinals-runes
**Docs:** https://www.quicknode.com/docs/bitcoin/ord_getInscription

## Prerequisites

Activate the **Ordinals & Runes API** add-on on your Quicknode Bitcoin endpoint. Methods become available on your existing endpoint URL once active.

## Methods Overview

| Method | Description |
|--------|-------------|
| `ord_getInscription` | Metadata for a single inscription by ID |
| `ord_getInscriptions` | Paginated list of inscriptions |
| `ord_getInscriptionsByBlock` | All inscriptions in a specific block |
| `ord_getChildren` | Child inscriptions of a parent inscription |
| `ord_getSat` | Satoshi rarity, percentile, and linked inscriptions |
| `ord_getOutput` | Bitcoin output and the inscriptions or Runes it contains |
| `ord_getCollections` | Paginated collection listings |
| `ord_getRune` | Rune etching details and live supply by ID or name |
| `ord_getRunes` | Paginated Rune listings |
| `ord_getStatus` | Ordinals indexer status |
| `ord_getTx` | Transaction details |
| `ord_getBlockInfo` | Recursive block info by height |
| `ord_getBlockHash` | Block hash by height |
| `ord_getContent` | Raw inscription content |
| `ord_getCurrentBlockHash` | Latest indexed block hash |
| `ord_getCurrentBlockHeight` | Latest indexed block height |
| `ord_getCurrentBlockTime` | Latest indexed block timestamp |
| `ord_getMetadata` | CBOR-decoded inscription metadata |
| `ord_getInscriptionRecursive` | Recursive inscription details |
| `ord_getSatAtIndex` | Inscription ID at an index on a sat |
| `ord_getSatRecursive` | Paginated inscription IDs on a sat |

Both JSON-RPC and REST interfaces are available on the same endpoint.

## Get Inscription Details

```typescript
const response = await fetch(process.env.QUICKNODE_RPC_URL!, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    jsonrpc: '2.0',
    id: 1,
    method: 'ord_getInscription',
    params: ['6fb976ab49dcec017f1e201e84395983204ae1a7c2abf7ced0a85d692e442799i0'],
  }),
});

const { result } = await response.json();
// result.id — inscription ID (txid + index)
// result.number — ordinal inscription number
// result.sat — sat number the inscription is bonded to
// result.satpoint — current location (txid:vout:offset)
// result.address — current holder address
// result.content_type — MIME type of the inscription content
// result.content_length — byte length
// result.height — block height of the inscription transaction
// result.fee — fee paid at inscription
// result.charms, result.children, result.parents — relationship and classification arrays
// result.previous, result.next — neighboring inscription IDs
// result.rune — associated Rune, if any
// result.value — output value in sats
// result.timestamp — Unix timestamp of genesis block
```

## Get Satoshi Data

```typescript
const response = await fetch(process.env.QUICKNODE_RPC_URL!, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    jsonrpc: '2.0',
    id: 1,
    method: 'ord_getSat',
    params: [1452891330221420],
  }),
});

const { result } = await response.json();
// result.number — sat number
// result.rarity — rarity classification (e.g., "uncommon", "rare")
// result.percentile — sat's position in total supply
// result.inscriptions — array of inscription IDs on this sat
// result.block — block in which this sat was mined
// result.satpoint, result.timestamp, result.charms — current sat metadata
```

## Get Bitcoin Output

```typescript
const response = await fetch(process.env.QUICKNODE_RPC_URL!, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    jsonrpc: '2.0',
    id: 1,
    method: 'ord_getOutput',
    params: ['7f3a1c2b4d5e6f7a8b9c0d1e2f3a4b5c6d7e8f9a0b1c2d3e4f5a6b7c8d9e0f1a:0'],
  }),
});

const { result } = await response.json();
// result.value — satoshi value of the output
// result.address — output address
// result.inscriptions — inscriptions held in this output
// result.runes — Runes and their amounts in this output
// result.indexed — whether this output is indexed
// result.sat_ranges — sat ranges in the output
// result.script_pubkey — script pubkey
// result.spent — whether the output has been spent
// result.transaction — transaction ID
```

## Get Rune Details

```typescript
const response = await fetch(process.env.QUICKNODE_RPC_URL!, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    jsonrpc: '2.0',
    id: 1,
    method: 'ord_getRune',
    params: ['840000:1'],
  }),
});

const { result } = await response.json();
// result.id — rune ID (block:tx)
// result.mintable — whether the Rune can still be minted
// result.parent — parent inscription, if any
// result.entry.spaced_rune — rune name with spacers
// result.entry.symbol — single character symbol
// result.entry.burned — burned amount
// result.entry.divisibility — decimal places
// result.entry.etching — etching transaction ID
// result.entry.mints — number of mints
// result.entry.premine — premine amount
// result.entry.terms — mint terms (cap, amount, height, offset)
```

## Troubleshooting

| Error | Cause | Fix |
|-------|-------|-----|
| Method not found | Add-on not enabled | Activate Ordinals & Runes API on your Bitcoin endpoint |
| Unknown inscription | Inscription ID does not exist | Verify the full inscription ID including the `i0` suffix |
| Rune not found | Name or ID incorrect | Use bullet separators in the name (e.g., `UNCOMMON•GOODS`) |

## Documentation

- **ord_getInscription Docs**: https://www.quicknode.com/docs/bitcoin/ord_getInscription
- **Build with Ordinals and Runes API Guide**: https://www.quicknode.com/guides/bitcoin/build-with-ordinals-and-runes-api
- **Marketing**: https://www.quicknode.com/ordinals-runes
