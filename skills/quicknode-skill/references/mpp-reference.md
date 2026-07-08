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
| `/sql/rest/v1/clusters` | GET | List Quicknode SQL Explorer clusters (public, 1-hour CDN cached) |
| `/sql/rest/v1/schema` | GET | List SQL Explorer schemas across clusters (public, 1-hour CDN cached) |
| `/sql/rest/v1/schema/:clusterId` | GET | Inspect one SQL Explorer cluster schema (public, 1-hour CDN cached) |
| `/session/sql/rest/*` | ALL | SQL Explorer REST pass-through via MPP session; variable per-query billing |
| `/session/v1/text2sql` | POST | Natural-language to SQL generator via MPP session; variable per-token billing |
| `/networks` | GET | List all supported network slugs (public, rate limited) |
| `/llms.txt` | GET | Machine-readable documentation |

Replace `:network` with slugs like `tempo-mainnet`, `ethereum-mainnet`, `solana-mainnet`, etc.

## Payment Networks

| Network | Chain ID | Token | Charge | Session |
|---------|----------|-------|:---:|:---:|
| Tempo Testnet | 42431 | PathUSD | Yes | Yes |
| Tempo Mainnet | 4217 | PathUSD, USDC.e | Yes | Yes |
| Solana Mainnet | — | USDC | Yes | No |

The payment network is independent of the chain you query. For example, you can pay with PathUSD on Tempo and query Ethereum, Solana, or any other supported chain.

## Testnet Monthly Cap

Each wallet gets a **single pool of 1,000,000 API credits per month** across Quicknode's agentic payment surfaces, including x402 and MPP. The pool resets on the 1st of each month and covers both RPC and SQL Explorer calls. Mainnet wallets are uncapped.

| Testnet Pool | Value |
|-----------------|-------|
| Credits per wallet | 1,000,000/month |
| Reset | 1st of each month |
| Scope | RPC + SQL Explorer + text2sql |
| Shared across | Charge, Session |

When the testnet cap is exhausted, the server returns HTTP 403 with `{ "error": "monthly_limit_reached" }` and no payment is taken. Switch to a mainnet-funded wallet or wait for the reset.

## Rate Limits

| Route | Limit |
|-------|-------|
| `/networks` | 10 requests per 10 seconds per IP |
| `/:network` | 1,000 requests per 10 seconds per IP:network pair |
| `/session/:network` | 1,000 requests per 10 seconds per IP:session:network pair |

Back off and retry on HTTP 429 `rate_limit_exceeded`.

## SQL Explorer

MPP exposes Quicknode SQL Explorer through session-gated REST pass-through routes at `https://mpp.quicknode.com/session/sql/rest/*`. Paths mirror the upstream `https://api.quicknode.com/sql/rest/*` routes; for example, `POST /session/sql/rest/v1/query` executes a SQL Explorer query through an open MPP session.

Free discovery endpoints require no payment credential and are cached for 1 hour:

- `GET /sql/rest/v1/clusters`
- `GET /sql/rest/v1/schema`
- `GET /sql/rest/v1/schema/:clusterId`

Paid SQL endpoints require an MPP session voucher. Billing is variable: the upstream response's `credits` field is charged against the open session channel at $0.00001 per SQL credit. The client tops up vouchers automatically when a query exceeds the currently signed amount.

## text2sql

`POST /session/v1/text2sql` is a Tempo-session-gated natural-language to SQL generator. It shares the same SQL session voucher store as `/session/sql/rest/*`, so one open session can fund both SQL queries and text2sql calls.

Supported modes:

| Mode | Required fields | Description |
|------|-----------------|-------------|
| Generate | `prompt` | Generate fresh SQL from natural language |
| Edit | `prompt`, `sql` | Modify supplied SQL according to the prompt |
| Fix | `sql`, `validation_errors` | Repair SQL using validation errors |

Limits: `prompt` <= 4000 characters, `sql` <= 10000 characters, and up to 10 validation errors at 2000 characters each. `cluster_id` defaults to the server-configured SQL cluster when omitted. Billing is variable: `(input + output) * $0.00001`, charged against the session after the response.

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
npm install @solana/mpp mppx @solana/kit
```

```typescript
import { Mppx } from 'mppx/client'
import { solana } from '@solana/mpp/client'
import { createKeyPairSignerFromBytes } from '@solana/kit'
import { readFileSync } from 'fs'
import { homedir } from 'os'

const keypairFile = readFileSync(
  process.env.SOLANA_KEYPAIR_PATH ?? `${homedir()}/.config/solana/id.json`,
  'utf-8'
)
const signer = await createKeyPairSignerFromBytes(
  new Uint8Array(JSON.parse(keypairFile))
)

Mppx.create({ methods: [solana.charge({ signer })] })

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
| 403 | `monthly_limit_reached` | Monthly testnet credit cap exceeded; no payment taken; switch to mainnet or wait for reset |
| 404 | `unsupported_network` | Network slug not found |
| 429 | `rate_limit_exceeded` | Route-specific limit exceeded; see [Rate Limits](#rate-limits) |
| 503 | `mpp_not_configured` | MPP unavailable in this environment |

## Best Practices

1. **Use session intents for high volume** — 100x cheaper than charge at scale ($0.00001 vs $0.001 per request).
2. **Use the polyfill for simplicity** — `Mppx.create()` patches `globalThis.fetch` so all fetch calls handle 402 automatically.
3. **Parse receipts** — Use `Receipt.fromResponse()` to verify payment status and get transaction references.
4. **Multi-service agents** — MPP works across any MPP-enabled service, so one wallet and protocol handles Quicknode RPC, LLM providers, and other APIs.

## NPM Packages

- **mppx** — Official TypeScript SDK (client, server, middleware, CLI)
- **@solana/mpp** — Solana payment method (client, server)
- **viem** — Ethereum/Tempo utilities (peer dependency)
- **@solana/kit** — Solana SDK (peer dependency)

## Documentation

- **MPP Platform**: https://mpp.quicknode.com
- **MPP Documentation (llms.txt)**: https://mpp.quicknode.com/llms.txt
- **MPP Docs**: https://mpp.dev
- **MPP Spec (IETF)**: https://datatracker.ietf.org/doc/draft-ryan-httpauth-payment/
- **Tempo Docs**: https://docs.tempo.xyz
