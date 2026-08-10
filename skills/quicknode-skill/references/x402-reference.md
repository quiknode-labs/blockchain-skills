# x402 Reference

x402 enables wallet-based RPC access via stablecoin payments. No API key required. Three payment models: pay-per-request (no auth), nanopayment (batched via Circle Gateway), and credit drawdown (SIWX auth + credit bundle).

## Overview

| Property | Value |
|----------|-------|
| **Protocol** | HTTP 402 Payment Required |
| **Payment Models** | Pay-per-request, Nanopayment, Credit Drawdown |
| **Authentication** | None (pay-per-request, nanopayment) or SIWX + JWT (credit drawdown) |
| **Supported Protocols** | JSON-RPC, REST, gRPC-Web, WebSocket (varies by payment model) |
| **Payment Networks** | Varies by payment model. See [Payment Networks](#payment-networks-caip-2) |
| **Chains** | All Quicknode-supported networks |
| **Base URL** | `https://x402.quicknode.com` |
| **Use Cases** | Keyless RPC access, AI agents, pay-as-you-go, ephemeral wallets |

## Payment Models

| Model | Auth | Cost | Protocols | Best For |
|-------|------|------|-----------|----------|
| **Pay-per-request** | None | $0.001/request | JSON-RPC, REST | Simple integrations, low volume |
| **Nanopayment** | None | $0.0001/request | JSON-RPC, REST | High-volume testing (EVM testnets only for payment chain) |
| **Credit Drawdown** | SIWX + JWT | Testnet: $1/100,000 credits. Mainnet: $10/1,000,000 credits | JSON-RPC, REST, gRPC-Web, WebSocket | Sustained usage, streaming protocols |

The payment chain does not need to match the chain you query. For example, you can pay with Base Sepolia USDC and query Ethereum Mainnet.

### Credit Drawdown Flow

1. **Authenticate** via SIWX to get a JWT (1 hour expiry)
2. **Make a request** that returns HTTP 402 with payment requirements
3. **Pay** with a supported stablecoin to receive a credit bundle
4. **Consume credits** per successful response; when depleted, step 2 repeats automatically

### Pay-per-request Flow

1. **Make a request** (no auth needed); server returns HTTP 402
2. **Pay** with a payment signature included in each request (handled automatically by the client)

## Key Endpoints

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/auth` | POST | Exchange SIWE/SIWX message + signature for JWT (public, no auth). Optional `type: "siwx"` field enables multi-chain auth |
| `/credits` | GET | Check credit balance (JWT required) |
| `/drip` | POST | Testnet USDC faucet (Base Sepolia only, requires JWT) |
| `/networks` | GET | List all supported network slugs (public, rate limited) |
| `/:network` | POST | JSON-RPC/REST proxy to Quicknode. Anonymous requests return 402 with payment requirements and extensions |
| `/:network/ws` | GET | WebSocket JSON-RPC proxy (JWT required, credits must exist) |
| `/sql/rest/v1/clusters` | GET | List Quicknode SQL Explorer clusters (public, 1-hour CDN cached) |
| `/sql/rest/v1/schema` | GET | List SQL Explorer schemas across clusters (public, 1-hour CDN cached) |
| `/sql/rest/v1/schema/:clusterId` | GET | Inspect one SQL Explorer cluster schema (public, 1-hour CDN cached) |
| `/sql/rest/v1/query` | POST | Paid SQL Explorer query endpoint; SIWX drawdown only, variable per-query credit cost |
| `/sql/rest/*` | ALL | Paid SQL Explorer REST pass-through; paths mirror `https://api.quicknode.com/sql/rest/*` |
| `/discovery/resources` | GET | Bazaar-compatible catalog of supported networks and payment requirements |
| `/openapi.json` | GET | OpenAPI 3.1 spec with x402 payment annotations |

## Testnet Monthly Cap

Each wallet gets a **single pool of 1,000,000 API credits per month** across Quicknode's agentic payment surfaces, including x402 and MPP. The pool resets on the 1st of each month and covers both RPC and SQL Explorer calls. RPC requests consume 1 API credit each; SQL Explorer consumes the variable number of credits the query uses. Mainnet wallets are uncapped.

| Testnet Pool | Value |
|-----------------|-------|
| Credits per wallet | 1,000,000/month |
| Reset | 1st of each month |
| Scope | RPC + SQL Explorer |
| Shared across | Pay-per-request, Nanopayment, Credit Drawdown |

When the testnet cap is exhausted, requests return HTTP 403 `monthly_limit_reached` until reset. Switch to a mainnet-funded wallet for uninterrupted access.

## SQL Explorer

Quicknode SQL Explorer is exposed as a REST pass-through at `https://x402.quicknode.com/sql/rest/*`; every path mirrors the upstream `https://api.quicknode.com/sql/rest/*` route.

Free discovery endpoints require no auth and are cached for 1 hour:

- `GET /sql/rest/v1/clusters`
- `GET /sql/rest/v1/schema`
- `GET /sql/rest/v1/schema/:clusterId`

Paid SQL endpoints, such as `POST /sql/rest/v1/query`, require SIWX-authenticated credit drawdown. SQL cost is variable: the response's `credits` value is debited 1:1 from the caller's drawdown balance after the query runs. Because cost is only known after execution, per-request `PAYMENT-SIGNATURE` is not accepted for `/sql/rest/*`; authenticate with SIWX and pre-fund credits instead. An unauthenticated request, or a request that only presents `PAYMENT-SIGNATURE`, returns HTTP 402 with the SIWX drawdown extension and `use-drawdown` guidance.

## Rate Limits

| Route | Limit |
|-------|-------|
| `/auth` | 10 requests per 10 seconds per IP |
| `/credits` | 50 requests per 10 seconds per account |
| `/drip` | 5 requests per 60 seconds per account |
| `/networks` | 10 requests per 10 seconds per IP |
| `/:network` | 1,000 requests per 10 seconds per network:account pair |

Back off and retry on HTTP 429 `rate_limit_exceeded`. Error responses and JSON-RPC errors are not metered.

## Setup

### Install Dependencies

```bash
npm install @quicknode/x402
```

The `@quicknode/x402` package handles payment negotiation for all three models. For credit drawdown, it also manages SIWX authentication and JWT sessions.

### Create a Client and Make RPC Calls

```typescript
import { createQuicknodeX402Client } from '@quicknode/x402';

const X402_BASE_URL = 'https://x402.quicknode.com';

// Credit drawdown: preAuth authenticates (SIWX + JWT) upfront for faster payment flow
const client = await createQuicknodeX402Client({
  baseUrl: X402_BASE_URL,
  network: 'eip155:84532', // Base Sepolia (payment network)
  evmPrivateKey: process.env.PRIVATE_KEY as `0x${string}`,
  paymentModel: 'credit-drawdown',
  preAuth: true,
});

// Make RPC calls (payment is automatic on 402)
const response = await client.fetch(`${X402_BASE_URL}/ethereum-mainnet`, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    jsonrpc: '2.0',
    id: 1,
    method: 'eth_blockNumber',
    params: [],
  }),
});

const { result } = await response.json();
console.log('Block number:', BigInt(result));
```

### Client Configuration

| Parameter | Description |
|-----------|-------------|
| `baseUrl` | The x402 gateway URL (`https://x402.quicknode.com`) |
| `network` | CAIP-2 chain identifier for the payment network |
| `evmPrivateKey` | Hex-encoded private key for EVM chains |
| `svmPrivateKey` | Base58-encoded secret key for Solana |
| `paymentModel` | `'pay-per-request'`, `'nanopayment'`, or `'credit-drawdown'` (default) |
| `preAuth` | Credit drawdown only. When `true`, pre-authenticates (SIWX + JWT) before the first request |

### Solana Client

```typescript
import { createQuicknodeX402Client } from '@quicknode/x402';

const client = await createQuicknodeX402Client({
  baseUrl: 'https://x402.quicknode.com',
  network: 'solana:EtWTRABZaYq6iMfeYKouRu166VU2xqa1', // Solana Devnet
  svmPrivateKey: '<YOUR_BASE58_SECRET_KEY>',
  preAuth: true,
});

// Query any chain (payment network doesn't need to match)
const response = await client.fetch('https://x402.quicknode.com/solana-devnet', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ jsonrpc: '2.0', id: 1, method: 'getSlot', params: [] }),
});
```

### Pay-per-request Client (No Auth)

```typescript
import { createQuicknodeX402Client } from '@quicknode/x402';

const client = await createQuicknodeX402Client({
  baseUrl: 'https://x402.quicknode.com',
  network: 'eip155:84532',
  evmPrivateKey: process.env.PRIVATE_KEY as `0x${string}`,
  paymentModel: 'pay-per-request',
});

const response = await client.fetch('https://x402.quicknode.com/base-mainnet', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ jsonrpc: '2.0', id: 1, method: 'eth_blockNumber', params: [] }),
});
```

### Nanopayment Client (Circle Gateway)

Nanopayment requires a one-time USDC deposit into the Circle Gateway Wallet contract. After depositing, the Gateway API waits for a chain-specific number of block confirmations before updating your unified balance. See [Circle's supported blockchains reference](https://developers.circle.com/gateway/references/supported-blockchains#required-block-confirmations) for confirmation times per chain.

```typescript
import { createQuicknodeX402Client } from "@quicknode/x402";

const client = await createQuicknodeX402Client({
  baseUrl: "https://x402.quicknode.com",
  network: "eip155:84532", // Base Sepolia (must be a nanopayment-eligible EVM testnet)
  evmPrivateKey: process.env.PRIVATE_KEY as `0x${string}`,
  paymentModel: "nanopayment",
});

// Check Gateway Wallet balance and deposit if needed
if (client.gatewayClient) {
  const balances = await client.gatewayClient.getBalances();
  console.log("Gateway available:", balances.gateway.formattedAvailable);

  // 1 USDC = 1_000_000 base units (6 decimals)
  if (balances.gateway.available < 1_000_000n) {
    console.log("Depositing 1 USDC...");
    const deposit = await client.gatewayClient.deposit("1");
    console.log(`Deposit tx: ${deposit.depositTxHash}`);
  }
}

const response = await client.fetch("https://x402.quicknode.com/base-mainnet", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ jsonrpc: "2.0", id: 1, method: "eth_blockNumber", params: [] }),
});

const data = await response.json();
console.log("Block:", data.result);
```

### Check Credit Balance (Credit Drawdown Only)

```typescript
const creditsResponse = await fetch('https://x402.quicknode.com/credits', {
  headers: { Authorization: `Bearer ${client.getToken()}` },
});
const { credits } = await creditsResponse.json();
console.log('Remaining credits:', credits);
```

### Get Free Testnet Credits (Base Sepolia Only)

```typescript
const dripResponse = await fetch('https://x402.quicknode.com/drip', {
  method: 'POST',
  headers: { Authorization: `Bearer ${client.getToken()}` },
});
const dripResult = await dripResponse.json();
console.log('Credits received:', dripResult);
```

## Payment Networks (CAIP-2)

| Network | CAIP-2 ID | Token | Pay-per-request | Credit Drawdown | Nanopayment |
|---------|-----------|-------|:---:|:---:|:---:|
| Base Sepolia | `eip155:84532` | USDC | Yes | Yes | Yes |
| Base Mainnet | `eip155:8453` | USDC | Yes | Yes | No |
| Polygon Amoy | `eip155:80002` | USDC | Yes | Yes | Yes |
| Polygon Mainnet | `eip155:137` | USDC | Yes | Yes | No |
| XLayer Testnet | `eip155:1952` | USDG | Yes | Yes | No |
| XLayer Mainnet | `eip155:196` | USDG | Yes | Yes | No |
| Arc Testnet | `eip155:5042002` | USDC | No | No | Yes |
| Solana Devnet | `solana:EtWTRABZaYq6iMfeYKouRu166VU2xqa1` | USDC | Yes | Yes | No |
| Solana Mainnet | `solana:5eykt4UsFv8P8NJdTREpY1vzqKqZKvdp` | USDC | Yes | Yes | No |

## Best Practices

1. **Start with pay-per-request** for simplicity (no auth needed), switch to credit drawdown for sustained usage or streaming protocols.
2. **Use testnet first.** Call `/drip` for Base Sepolia, or pre-fund other supported testnets with the required stablecoin before using mainnet.
3. **Use `preAuth: true`** with credit drawdown to speed up the first payment flow.
4. **Reuse session tokens.** The client caches JWTs automatically (1 hour expiry).
5. **gRPC-Web and WebSocket require credit drawdown.** Pay-per-request and nanopayment support JSON-RPC and REST only.
6. **Multi-protocol support.** The same client works with JSON-RPC, REST, gRPC-Web (`client.createGrpcTransport()`), and WebSocket (`client.createWebSocket()`).
7. **Discover dynamically.** Use `/networks`, `/discovery/resources`, or `/openapi.json` instead of hardcoding the supported network list.

## Using x402 From The CLI Or SDK

The Quicknode CLI and SDK wrap this gateway, so an agent can pay for RPC calls without writing payment code:

- CLI (v0.6.0+): `qn wallet generate` then `qn rpc call --x402`, or prepaid credits via `qn rpc x402 buy-credits` and `--x402-drawdown`. See [cli-reference.md](cli-reference.md#paid-rpc-x402-and-mpp).
- SDK (Node 3.8.0+, other languages 0.8.0+): set `rpc.payment` with `scheme: "x402"`, or use `gatewayAuthenticate`/`gatewayBuyCredits`/`gatewayDrawdownCall`. See [sdk-reference.md](sdk-reference.md#rpc-micropayments).

## Documentation

- **x402 Platform**: https://x402.quicknode.com
- **x402 Documentation (llms.txt)**: https://x402.quicknode.com/llms.txt
- **@quicknode/x402 Package**: https://github.com/quiknode-labs/quicknode-x402
- **Examples**: https://github.com/quiknode-labs/qn-x402-examples
