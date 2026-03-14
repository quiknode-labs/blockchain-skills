# x402 Reference

x402 enables pay-per-request RPC access via stablecoin micropayments. No API key required — authenticate with Sign-In with X (SIWX), purchase credits with USDC or USDG, and access 140+ blockchain RPC endpoints.

## Overview

| Property | Value |
|----------|-------|
| **Protocol** | HTTP 402 Payment Required |
| **Payment Methods** | USDC on Base, Polygon, Solana; USDG on XLayer |
| **Authentication** | SIWX (Sign-In with X) — EVM + Solana |
| **Chains** | 140+ (same as Quicknode RPC network) |
| **Base URL** | `https://x402.quicknode.com` |
| **Use Cases** | Keyless RPC access, AI agents, pay-as-you-go, ephemeral wallets |

## How It Works

1. **Authenticate** — Sign a SIWX message with your wallet (EVM or Solana) to get a session token
2. **Purchase credits** — Pay with a supported stablecoin to buy credits (or use the Base Sepolia faucet for free credits)
3. **Make RPC requests** — Use credits to call any supported chain's RPC endpoint via JSON-RPC, REST, gRPC-Web, or WebSocket

## Key Endpoints

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/auth` | POST | Authenticate via SIWX, returns session token |
| `/credits` | GET | Check credit balance |
| `/drip` | POST | Testnet faucet — free credits (Base Sepolia only) |
| `/:network` | POST | JSON-RPC request to a specific chain (e.g., `/ethereum-mainnet`) |
| `/:network/ws` | WebSocket | WebSocket RPC connection to a specific chain |
| `/discovery/resources` | GET | Bazaar-compatible catalog of all supported networks |

## Credit Pricing

| Environment | Payment Network | Token | Credits | Cost |
|-------------|----------------|-------|---------|------|
| **Testnet** | Base Sepolia | USDC | 100 | $0.01 |
| **Testnet** | Polygon Amoy | USDC | 100 | $0.01 |
| **Testnet** | XLayer Testnet | USDG | 100 | $0.01 |
| **Testnet** | Solana Devnet | USDC | 100 | $0.01 |
| **Mainnet** | Base | USDC | 1,000,000 | $10 |
| **Mainnet** | Polygon | USDC | 1,000,000 | $10 |
| **Mainnet** | XLayer | USDG | 1,000,000 | $10 |
| **Mainnet** | Solana | USDC | 1,000,000 | $10 |

Credits are consumed per successful response. The payment network does not need to match the chain you are querying.

## Setup

### Install Dependencies

```bash
npm install @quicknode/x402
```

The `@quicknode/x402` package handles SIWX authentication, JWT management, x402 payment negotiation, gRPC-Web transport, and WebSocket connections automatically.

### Create a Client and Make RPC Calls

```typescript
import { createQuicknodeX402Client } from '@quicknode/x402';

const X402_BASE_URL = 'https://x402.quicknode.com';

// Create client — preAuth authenticates (SIWX + JWT) upfront for faster payment flow
const client = await createQuicknodeX402Client({
  baseUrl: X402_BASE_URL,
  network: 'eip155:84532', // Base Sepolia (payment network)
  evmPrivateKey: process.env.PRIVATE_KEY as `0x${string}`,
  preAuth: true,
});

// Make RPC calls — payment is automatic on 402
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
| `preAuth` | When `true`, pre-authenticates via SIWX before the first request |

### Solana Client

```typescript
import { createQuicknodeX402Client } from '@quicknode/x402';

const client = await createQuicknodeX402Client({
  baseUrl: 'https://x402.quicknode.com',
  network: 'solana:EtWTRABZaYq6iMfeYKouRu166VU2xqa1', // Solana Devnet
  svmPrivateKey: '<your-base58-secret-key>',
  preAuth: true,
});

// Query any chain — payment network doesn't need to match
const response = await client.fetch('https://x402.quicknode.com/solana-devnet', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ jsonrpc: '2.0', id: 1, method: 'getSlot', params: [] }),
});
```

### Check Credit Balance

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

| Network | CAIP-2 ID | Token |
|---------|-----------|-------|
| Base Sepolia | `eip155:84532` | USDC |
| Base Mainnet | `eip155:8453` | USDC |
| Polygon Amoy | `eip155:80002` | USDC |
| Polygon Mainnet | `eip155:137` | USDC |
| XLayer Testnet | `eip155:1952` | USDG |
| XLayer Mainnet | `eip155:196` | USDG |
| Solana Devnet | `solana:EtWTRABZaYq6iMfeYKouRu166VU2xqa1` | USDC |
| Solana Mainnet | `solana:5eykt4UsFv8P8NJdTREpY1vzqKqZKvdp` | USDC |

## Best Practices

1. **Use testnet credits first** — Call `/drip` (Base Sepolia) or use a faucet to get free credits before purchasing mainnet credits.
2. **Reuse session tokens** — The client caches JWTs automatically. Tokens are valid for 1 hour.
3. **Monitor credit balance** — Use `client.getToken()` with the `/credits` endpoint to check remaining credits.
4. **Ideal for AI agents** — x402 is well-suited for AI agents that need RPC access without managing API keys or infrastructure.
5. **Use `preAuth: true`** — Pre-authentication speeds up the first payment flow.
6. **Multi-protocol support** — The same client works with JSON-RPC, REST, gRPC-Web (`client.createGrpcTransport()`), and WebSocket (`client.createWebSocket()`).

## Documentation

- **x402 Platform**: https://x402.quicknode.com
- **x402 Documentation (llms.txt)**: https://x402.quicknode.com/llms.txt
- **@quicknode/x402 Package**: https://github.com/quiknode-labs/quicknode-x402
- **Examples**: https://github.com/quiknode-labs/qn-x402-examples
