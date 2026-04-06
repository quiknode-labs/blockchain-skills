# MPP Reference

MPP (Machine Payments Protocol) enables pay-per-request RPC access via stablecoin micropayments using IETF Payment Authentication headers. No API key required.

## Overview

| Property | Value |
|----------|-------|
| **Protocol** | IETF Payment Authentication (WWW-Authenticate / Authorization / Payment-Receipt) |
| **Payment Methods** | Varies by network. See [Payment Networks](#payment-networks) |
| **Authentication** | None required (payment headers handle auth) |
| **Chains** | 140+ (same as Quicknode RPC network) |
| **Base URL** | `https://mpp.quicknode.com` |
| **Use Cases** | AI agents, pay-as-you-go, simple integrations, high-volume sessions |

## How It Works

1. **Send request** — POST to `/:network/*` (charge) or `/session/:network/*` (session)
2. **Receive challenge** — Server returns 402 with `WWW-Authenticate: Payment` header
3. **Sign payment** — Client signs token transfer and retries with `Authorization: Payment` header
4. **Receive response** — Server returns result with `Payment-Receipt` header

## Intent Types

| Intent | Cost | Endpoint | Best For |
|--------|------|----------|----------|
| **Charge** | $0.001/request (1,000 atomic units) | `POST /:network/*` | Simple integrations, low volume |
| **Session** | $0.00001/request ($10/1M requests) | `POST /session/:network/*` | High volume, agents, metered usage |

### Charge

One on-chain transaction per request. No session state, no escrow. The `mppx` SDK handles the 402 challenge and payment signing automatically.

### Session

Payment channels for high-frequency access:
1. Client deposits funds into escrow contract (~500ms initial setup)
2. Each request sends a cumulative EIP-712 signed voucher (off-chain)
3. Server verifies with `ecrecover` — no RPC or DB lookup
4. Settlement is batched when the server closes the channel
5. Unused deposit is refunded on channel close

## Key Endpoints

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/:network/*` | POST | Charge-based JSON-RPC and REST proxy |
| `/session/:network/*` | POST | Session-based JSON-RPC and REST proxy |
| `/llms.txt` | GET | Machine-readable documentation |

Replace `:network` with slugs like `tempo-mainnet`, `ethereum-mainnet`, `solana-mainnet`, etc.

## Payment Networks

| Network | Chain ID | Token | Charge | Session |
|---------|----------|-------|:---:|:---:|
| Tempo Testnet | 42431 | PathUSD | Yes | Yes |
| Tempo Mainnet | 4217 | PathUSD, USDC.e | Yes | Yes |
| Solana Mainnet | — | USDC | Yes | No |

The payment network is independent of the chain you query. For example, you can pay with PathUSD on Tempo and query Ethereum, Solana, or any other supported chain.

## Testnet Caps

Testnet caps are limited to 10,000 requests per intent per wallet (charge and session counted separately). After reaching a cap, switch to mainnet for production usage.

## Rate Limits

1,000 requests per 10 seconds per IP:network combination.

## Setup

### TypeScript (Tempo — Charge)

```bash
npm install mppx viem
```

```typescript
import { Mppx, tempo } from 'mppx/client'
import { privateKeyToAccount } from 'viem/accounts'

const account = privateKeyToAccount(process.env.PRIVATE_KEY as `0x${string}`)

// Polyfills globalThis.fetch — handles 402 challenges automatically
Mppx.create({
  methods: [tempo({ account })],
})

// Charge intent ($0.001/req) — payment is transparent
const response = await fetch('https://mpp.quicknode.com/tempo-mainnet', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    jsonrpc: '2.0',
    id: 1,
    method: 'eth_blockNumber',
    params: [],
  }),
})

const { result } = await response.json()
console.log('Block number:', BigInt(result))
```

### TypeScript (Tempo — Session)

```typescript
import { Mppx, tempo } from 'mppx/client'
import { privateKeyToAccount } from 'viem/accounts'

const account = privateKeyToAccount(process.env.PRIVATE_KEY as `0x${string}`)

// Session-only — payment channels for 100x cheaper requests
Mppx.create({
  methods: [tempo.session({ account })],
})

// First request opens channel on-chain (~500ms),
// subsequent requests use off-chain vouchers (microseconds)
const response = await fetch('https://mpp.quicknode.com/session/tempo-mainnet', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    jsonrpc: '2.0',
    id: 1,
    method: 'eth_blockNumber',
    params: [],
  }),
})

console.log(await response.json())
```

### Solana

```bash
npm install solana-mpp mppx @solana/kit @solana/spl-token
```

```typescript
import { Mppx, solana } from 'solana-mpp/client'
import { createKeyPairFromBytes } from '@solana/kit'
import { readFileSync } from 'fs'
import { homedir } from 'os'

const keypairFile = readFileSync(
  process.env.SOLANA_KEYPAIR_PATH ?? `${homedir()}/.config/solana/id.json`,
  'utf-8'
)
const keypair = await createKeyPairFromBytes(
  new Uint8Array(JSON.parse(keypairFile))
)

Mppx.create({ methods: [solana({ wallet: keypair })] })

const response = await fetch('https://mpp.quicknode.com/solana-mainnet', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    jsonrpc: '2.0',
    id: 1,
    method: 'getSlot',
    params: [],
  }),
})

const { result } = await response.json()
console.log('Slot:', result)
```

### Manual Payment Handling (No Polyfill)

```typescript
const mppx = Mppx.create({
  polyfill: false,
  methods: [tempo()],
})

// Step 1: Initial request gets 402
const response = await fetch('https://mpp.quicknode.com/tempo-mainnet', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ jsonrpc: '2.0', id: 1, method: 'eth_blockNumber', params: [] }),
})

if (response.status === 402) {
  // Step 2: Create credential from 402 challenge
  const credential = await mppx.createCredential(response, {
    account: privateKeyToAccount('0xYOUR_PRIVATE_KEY'),
  })

  // Step 3: Retry with Authorization header
  const paidResponse = await fetch('https://mpp.quicknode.com/tempo-mainnet', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      Authorization: credential,
    },
    body: JSON.stringify({ jsonrpc: '2.0', id: 1, method: 'eth_blockNumber', params: [] }),
  })
}
```

### CLI

```bash
npm install -g mppx

mppx account create
mppx -X POST -H 'Content-Type: application/json' \
  -d '{"jsonrpc":"2.0","id":1,"method":"eth_blockNumber","params":[]}' \
  https://mpp.quicknode.com/tempo-mainnet
```

Environment variables: `MPPX_PRIVATE_KEY`, `MPPX_ACCOUNT`

## Payment Receipts

```typescript
import { Receipt } from 'mppx'

const receipt = Receipt.fromResponse(response)
console.log(receipt.status)    // "success"
console.log(receipt.reference) // tx hash (charge) or channelId (session)
```

## Error Responses

| Status | Code | Meaning |
|--------|------|---------|
| 402 | (challenge) | Payment required; see `WWW-Authenticate: Payment` header |
| 403 | `lifetime_limit_reached` | Testnet lifetime request cap exceeded; switch to mainnet |
| 404 | `unsupported_network` | Network slug not found |
| 429 | `rate_limit_exceeded` | 1,000 req/10s limit exceeded per IP:network |
| 503 | `mpp_not_configured` | MPP unavailable in this environment |

## Best Practices

1. **Use session intents for high volume** — 100x cheaper than charge at scale ($0.00001 vs $0.001 per request).
2. **Use the polyfill for simplicity** — `Mppx.create()` patches `globalThis.fetch` so all fetch calls handle 402 automatically.
3. **Parse receipts** — Use `Receipt.fromResponse()` to verify payment status and get transaction references.
4. **Multi-service agents** — MPP works across any MPP-enabled service, so one wallet and protocol handles Quicknode RPC, LLM providers, and other APIs.

## NPM Packages

- **mppx** — Official TypeScript SDK (client, server, middleware, CLI)
- **solana-mpp** — Solana payment method (client, server)
- **viem** — Ethereum/Tempo utilities (peer dependency)
- **@solana/web3.js** — Solana web3 (peer dependency)

## Documentation

- **MPP Platform**: https://mpp.quicknode.com
- **MPP Documentation (llms.txt)**: https://mpp.quicknode.com/llms.txt
- **MPP Docs**: https://mpp.dev
- **MPP Spec (IETF)**: https://datatracker.ietf.org/doc/draft-ryan-httpauth-payment/
- **Tempo Docs**: https://docs.tempo.xyz
