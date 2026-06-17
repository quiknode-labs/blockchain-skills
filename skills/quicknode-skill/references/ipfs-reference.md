# Quicknode IPFS Reference

Quicknode provides IPFS (InterPlanetary File System) storage for decentralized file hosting, ideal for NFT metadata, images, and other blockchain-related assets.

## Overview

| Feature | Description |
|---------|-------------|
| **Storage** | Persistent IPFS pinning |
| **Gateway** | Dedicated IPFS gateway for fast retrieval |
| **API** | REST API for uploads and management |
| **Formats** | Any file type (images, JSON, video, etc.) |

## Upload Files

### Single File Upload

```javascript
const FormData = require('form-data');
const fs = require('fs');

async function uploadFile(filePath) {
  const form = new FormData();
  form.append('file', fs.createReadStream(filePath));

  const response = await fetch('https://api.quicknode.com/ipfs/rest/v1/s3/put-object', {
    method: 'POST',
    headers: {
      'x-api-key': process.env.QUICKNODE_API_KEY
    },
    body: form
  });

  const result = await response.json();
  console.log(`CID: ${result.pin.cid}`);
  console.log(`Gateway URL: https://<your-gateway-name>.quicknode-ipfs.com/ipfs/${result.pin.cid}`);

  return result;
}

await uploadFile('./image.png');
```

### Upload JSON Data

```javascript
async function uploadJSON(data) {
  const response = await fetch('https://api.quicknode.com/ipfs/rest/v1/s3/put-object', {
    method: 'POST',
    headers: {
      'x-api-key': process.env.QUICKNODE_API_KEY,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify(data)
  });

  const result = await response.json();
  return result.pin.cid;
}

const metadata = {
  name: "My NFT",
  description: "An awesome NFT",
  image: "ipfs://QmImageCid..."
};

const cid = await uploadJSON(metadata);
```

### Upload from Buffer

```javascript
async function uploadBuffer(buffer, filename) {
  const form = new FormData();
  form.append('file', buffer, { filename });

  const response = await fetch('https://api.quicknode.com/ipfs/rest/v1/s3/put-object', {
    method: 'POST',
    headers: {
      'x-api-key': process.env.QUICKNODE_API_KEY
    },
    body: form
  });

  return await response.json();
}
```

## Directory Upload

Upload entire directories while preserving structure.

```javascript
const FormData = require('form-data');
const fs = require('fs');
const path = require('path');

async function uploadDirectory(dirPath) {
  const form = new FormData();
  const files = fs.readdirSync(dirPath);

  for (const file of files) {
    const filePath = path.join(dirPath, file);
    form.append('file', fs.createReadStream(filePath), {
      filepath: file // Preserves filename in IPFS
    });
  }

  const response = await fetch('https://api.quicknode.com/ipfs/rest/v1/s3/put-object', {
    method: 'POST',
    headers: {
      'x-api-key': process.env.QUICKNODE_API_KEY
    },
    body: form
  });

  return await response.json();
}

// Upload NFT collection assets
await uploadDirectory('./nft-images/');
```

## Pin by CID

Pin existing IPFS content to Quicknode's infrastructure.

```javascript
async function pinByCID(cid, name) {
  const response = await fetch('https://api.quicknode.com/ipfs/rest/v1/pinning/pinByHash', {
    method: 'POST',
    headers: {
      'x-api-key': process.env.QUICKNODE_API_KEY,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      cid: cid,
      name: name || `Pinned: ${cid}`
    })
  });

  return await response.json();
}

// Pin existing content
await pinByCID('QmExistingCID...', 'My Pinned Content');
```

## Retrieve Content

### Via Gateway

```javascript
const GATEWAY_URL = 'https://<your-gateway-name>.quicknode-ipfs.com';

// Fetch JSON metadata
async function fetchMetadata(cid) {
  const response = await fetch(`${GATEWAY_URL}/ipfs/${cid}`);
  return await response.json();
}

// Fetch image as buffer
async function fetchImage(cid) {
  const response = await fetch(`${GATEWAY_URL}/ipfs/${cid}`);
  return await response.buffer();
}
```

### Gateway URL Formats

```
# Direct CID access
https://<your-gateway-name>.quicknode-ipfs.com/ipfs/{cid}

# Path within directory
https://<your-gateway-name>.quicknode-ipfs.com/ipfs/{directoryCid}/image.png

# IPNS resolution
https://<your-gateway-name>.quicknode-ipfs.com/ipns/{name}
```

## List Pinned Content

```javascript
async function listPins(pageNumber = 1, perPage = 100) {
  const response = await fetch(
    `https://api.quicknode.com/ipfs/rest/v1/pinning/pins?pageNumber=${pageNumber}&perPage=${perPage}`,
    {
      headers: {
        'x-api-key': process.env.QUICKNODE_API_KEY
      }
    }
  );

  return await response.json();
}

const pins = await listPins();
console.log(`Total pinned: ${pins.totalItems}`);
pins.data.forEach(pin => {
  console.log(`${pin.cid} - ${pin.name}`);
});
```

## Unpin Content

```javascript
async function unpin(requestId) {
  const response = await fetch(
    `https://api.quicknode.com/ipfs/rest/v1/pinning/pins/${requestId}`,
    {
      method: 'DELETE',
      headers: {
        'x-api-key': process.env.QUICKNODE_API_KEY
      }
    }
  );

  return response.status === 200;
}

// Use the requestId from the pin response, not the CID
await unpin('12345678-abcd-...');
```

## NFT Metadata Examples

### Standard NFT Metadata

```javascript
const nftMetadata = {
  name: "Cool Cat #1234",
  description: "A very cool cat from the Cool Cats collection",
  image: "ipfs://QmImageCID...",
  external_url: "https://coolcats.com/cat/1234",
  attributes: [
    {
      trait_type: "Background",
      value: "Blue"
    },
    {
      trait_type: "Fur",
      value: "Orange"
    },
    {
      trait_type: "Eyes",
      value: "Laser"
    },
    {
      trait_type: "Rarity Score",
      value: 85,
      display_type: "number"
    }
  ]
};

const metadataCID = await uploadJSON(nftMetadata);
console.log(`Token URI: ipfs://${metadataCID}`);
```

### Collection-Level Metadata

```javascript
const collectionMetadata = {
  name: "Cool Cats",
  description: "10,000 randomly generated Cool Cats",
  image: "ipfs://QmCollectionImageCID...",
  external_link: "https://coolcats.com",
  seller_fee_basis_points: 500, // 5%
  fee_recipient: "0xYourAddress..."
};

await uploadJSON(collectionMetadata);
```

### Batch Upload NFT Collection

```javascript
async function uploadNFTCollection(basePath, count) {
  const results = [];

  for (let i = 1; i <= count; i++) {
    // Upload image
    const imagePath = `${basePath}/images/${i}.png`;
    const imageResult = await uploadFile(imagePath);
    const imageCID = imageResult.pin.cid;

    // Create and upload metadata
    const metadata = {
      name: `My NFT #${i}`,
      description: `NFT number ${i} from my collection`,
      image: `ipfs://${imageCID}`,
      attributes: [
        { trait_type: "Number", value: i }
      ]
    };

    const metadataCID = await uploadJSON(metadata);

    results.push({
      tokenId: i,
      imageCID,
      metadataCID,
      tokenURI: `ipfs://${metadataCID}`
    });

    console.log(`Uploaded NFT #${i}`);
  }

  return results;
}
```

## Dedicated Gateway Configuration

### Custom Domain

1. Navigate to Quicknode Dashboard → IPFS → Gateways
2. Create dedicated gateway
3. Configure custom domain (optional)
4. Set up DNS CNAME record

### Gateway Features

| Feature | Description |
|---------|-------------|
| **Caching** | Edge caching for fast delivery |
| **Rate Limits** | Configurable per gateway |
| **Analytics** | Request metrics and bandwidth |
| **Access Control** | IP allowlisting, authentication |

## Storage Plans

> **Note:** Storage plans and limits may change. Verify current pricing and limits at https://www.quicknode.com/pricing or in the QuickNode dashboard.

| Plan | Storage | Bandwidth | Pins |
|------|---------|-----------|------|
| Free | 1 GB | 5 GB/mo | 500 |
| Starter | 10 GB | 50 GB/mo | 5,000 |
| Growth | 100 GB | 500 GB/mo | 50,000 |
| Business | Custom | Custom | Custom |

## Best Practices

1. **Use CIDs in contracts** - Store `ipfs://CID` format in smart contracts
2. **Upload images first** - Get image CID before creating metadata
3. **Verify uploads** - Always verify content is accessible via gateway
4. **Batch operations** - Upload multiple files concurrently when possible
5. **Pin important content** - Ensure critical content is pinned to multiple services
6. **Use directory uploads** - For collections, upload as directory for cleaner structure

## Error Handling

```javascript
async function safeUpload(filePath) {
  try {
    const result = await uploadFile(filePath);

    // Verify upload
    const verifyResponse = await fetch(
      `https://<your-gateway-name>.quicknode-ipfs.com/ipfs/${result.pin.cid}`,
      { method: 'HEAD' }
    );

    if (!verifyResponse.ok) {
      throw new Error('Upload verification failed');
    }

    return result;
  } catch (error) {
    console.error('Upload failed:', error.message);
    throw error;
  }
}
```

## Common Issues

| Issue | Cause | Solution |
|-------|-------|----------|
| Slow uploads | Large files | Compress before upload |
| 404 on gateway | Not yet propagated | Wait or use direct pin |
| Rate limited | Too many requests | Implement backoff |
| Content not found | Unpinned | Re-pin content |

## Documentation

- **IPFS Overview**: https://www.quicknode.com/docs/ipfs
- **IPFS Overview (llms.txt)**: https://www.quicknode.com/docs/ipfs/llms.txt
- **Guides**: https://www.quicknode.com/guides/tags/ipfs
