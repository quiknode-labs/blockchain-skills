# Metaplex DAS API Reference

> Formerly documented in this skill as `solana-das-api-reference.md` — links to that file still resolve; this file is the canonical reference going forward.

The Metaplex Digital Asset Standard (DAS) API is a comprehensive service for querying Solana digital assets efficiently. Available as the **Metaplex Digital Asset Standard (DAS) API** add-on in the Quicknode Marketplace.

**Marketing:** https://www.quicknode.com/metaplex-das-api
**Docs:** https://www.quicknode.com/docs/solana/solana-das-api

## Prerequisites

Enable the **Metaplex Digital Asset Standard (DAS) API** add-on on your Quicknode Solana endpoint via the [Marketplace](https://marketplace.quicknode.com/).

## Supported Asset Types

- **Standard NFTs** — traditional Solana NFTs
- **Compressed NFTs (cNFTs)** — Merkle tree-based, cost-efficient NFTs
- **Fungible Tokens** — SPL tokens
- **MPL Core Assets** — single-account design NFTs
- **Token 2022 Assets** — tokens using the Token Extensions program

## Methods Overview

| Method | Description |
|--------|-------------|
| `getAsset` | Get metadata for a single asset |
| `getAssets` | Get metadata for multiple assets in one call |
| `getAssetProof` | Get Merkle proof for a compressed asset |
| `getAssetProofs` | Get Merkle proofs for multiple compressed assets |
| `getAssetsByAuthority` | List assets controlled by an authority |
| `getAssetsByCreator` | List assets by creator address |
| `getAssetsByGroup` | List assets by group (e.g., collection) |
| `getAssetsByOwner` | List assets owned by a wallet |
| `getAssetSignatures` | Get transaction signatures for compressed assets |
| `getTokenAccounts` | List token accounts by mint or owner |
| `getNftEditions` | Get edition details of a master NFT |
| `searchAssets` | Search assets with flexible filters |

## getAsset

Retrieve metadata for a single asset by its mint address.

```javascript
const response = await fetch(process.env.QUICKNODE_RPC_URL, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    jsonrpc: '2.0',
    id: 1,
    method: 'getAsset',
    params: {
      id: '9ARngHhVaCtH5JFieRdSS5Y8cdZk2TMF4tfGSWFB9iSK',
      options: {
        showFungible: true,
        showCollectionMetadata: true
      }
    }
  })
});
const { result } = await response.json();
// result.content — metadata (name, description, image, attributes)
// result.ownership — owner, delegate, frozen status
// result.compression — tree, leaf, proof info (if compressed)
// result.royalty — royalty model, basis points, creators
// result.creators — array of creator addresses with verified status
// result.supply — edition/print supply info
```

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | string | Yes | The asset mint address |
| `options.showFungible` | boolean | No | Include fungible token info |
| `options.showCollectionMetadata` | boolean | No | Include collection metadata |
| `options.showUnverifiedCollections` | boolean | No | Include unverified collections |

## getAssets

Fetch metadata for multiple assets in a single request.

```javascript
const response = await fetch(process.env.QUICKNODE_RPC_URL, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    jsonrpc: '2.0',
    id: 1,
    method: 'getAssets',
    params: {
      ids: [
        '9ARngHhVaCtH5JFieRdSS5Y8cdZk2TMF4tfGSWFB9iSK',
        'BwJHge5FmE5RBkmWPoKzCWwxZFXsnqCMKHiiibXPJias'
      ]
    }
  })
});
const { result } = await response.json();
// result.items — array of asset metadata objects
```

## getAssetProof

Get the Merkle proof for a compressed asset. Required for transferring or modifying compressed NFTs on-chain.

```javascript
const response = await fetch(process.env.QUICKNODE_RPC_URL, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    jsonrpc: '2.0',
    id: 1,
    method: 'getAssetProof',
    params: {
      id: 'D85MZkvir9yQZFDHt8U2ZmS7D3LXKdiSjvw2MBdscJJa'
    }
  })
});
const { result } = await response.json();
// result.root — Merkle tree root hash
// result.proof — array of proof nodes
// result.node_index — index in the tree
// result.leaf — leaf hash
// result.tree_id — Merkle tree address
```

## getAssetsByOwner

List all digital assets owned by a wallet address.

```javascript
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
      sortBy: { sortBy: 'recent_action', sortDirection: 'desc' },
      options: {
        showFungible: true,
        showCollectionMetadata: true
      }
    }
  })
});
const { result } = await response.json();
// result.total — total assets owned
// result.items — array of asset metadata objects
// result.cursor — use in next request for pagination
```

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `ownerAddress` | string | Yes | Wallet address |
| `limit` | integer | No | Max results per page |
| `page` | integer | No | Page number (page-based pagination) |
| `cursor` | string | No | Cursor from previous response (cursor-based pagination) |
| `before` / `after` | string | No | Range-based pagination |
| `sortBy` | object | No | `{ sortBy: "created" \| "recent_action" \| "id" \| "none", sortDirection: "asc" \| "desc" }` |
| `options.showFungible` | boolean | No | Include fungible tokens |
| `options.showCollectionMetadata` | boolean | No | Include collection metadata |

## getAssetsByGroup

List assets by group identifier (e.g., collection address).

```javascript
const response = await fetch(process.env.QUICKNODE_RPC_URL, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    jsonrpc: '2.0',
    id: 1,
    method: 'getAssetsByGroup',
    params: {
      groupKey: 'collection',
      groupValue: 'CollectionMintAddress...',
      limit: 10
    }
  })
});
const { result } = await response.json();
// result.total, result.items, result.cursor
```

## searchAssets

Search for assets using flexible filter criteria. The most powerful query method in the DAS API.

```javascript
const response = await fetch(process.env.QUICKNODE_RPC_URL, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    jsonrpc: '2.0',
    id: 1,
    method: 'searchAssets',
    params: {
      ownerAddress: 'WalletPubkey...',
      compressed: true,
      limit: 10,
      sortBy: { sortBy: 'recent_action', sortDirection: 'desc' }
    }
  })
});
const { result } = await response.json();
// result.total, result.items, result.cursor
```

**Key Search Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `ownerAddress` | string | Filter by asset owner |
| `creatorAddress` | string | Filter by creator |
| `grouping` | array | Filter by group (e.g., `[["collection", "address"]]`) |
| `compressed` | boolean | Filter by compression status |
| `frozen` | boolean | Filter by frozen status |
| `burnt` | boolean | Filter by burn status |
| `interface` | string | Filter by asset interface type |

## Pagination

Three modes available:

```javascript
// Page-based
{ page: 1, limit: 100 }

// Cursor-based (recommended for large datasets)
{ cursor: 'previousResponse.result.cursor', limit: 100 }

// Range-based
{ before: 'assetId', after: 'assetId', limit: 100 }
```

## Common Use Cases

### Get all compressed NFTs owned by a wallet

```javascript
const response = await fetch(process.env.QUICKNODE_RPC_URL, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    jsonrpc: '2.0',
    id: 1,
    method: 'searchAssets',
    params: {
      ownerAddress: 'WalletPubkey...',
      compressed: true,
      limit: 100
    }
  })
});
```

### Transfer a compressed NFT (get proof first)

```javascript
// 1. Get the asset proof
const { result: proof } = await fetch(process.env.QUICKNODE_RPC_URL, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    jsonrpc: '2.0', id: 1, method: 'getAssetProof',
    params: { id: 'CompressedNftMint...' }
  })
}).then(r => r.json());

// 2. Use proof.root, proof.proof, proof.node_index with
//    @metaplex-foundation/mpl-bubblegum to construct the transfer instruction
```

## Documentation

- **DAS API Overview**: https://www.quicknode.com/docs/solana/solana-das-api
- **getAsset**: https://www.quicknode.com/docs/solana/getAsset
- **getAssetsByOwner**: https://www.quicknode.com/docs/solana/getAssetsByOwner
- **searchAssets**: https://www.quicknode.com/docs/solana/searchAssets
- **getTokenAccounts**: https://www.quicknode.com/docs/solana/getTokenAccounts
- **Marketing**: https://www.quicknode.com/metaplex-das-api
