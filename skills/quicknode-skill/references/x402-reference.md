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
| **Credit Drawdown** | SIWX + JWT | Testnet: $1/1,000 credits. Mainnet: $10/1,000,000 credits | JSON-RPC, REST, gRPC-Web, WebSocket | Sustained usage, streaming protocols |

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
| `/auth` | POST | Authenticate via SIWX, returns JWT (credit drawdown only) |
| `/credits` | GET | Check credit balance (credit drawdown only) |
| `/drip` | POST | Testnet USDC faucet (Base Sepolia only, requires JWT) |
| `/:network` | POST | JSON-RPC request to a specific chain (e.g., `/ethereum-mainnet`) |
| `/:network/ws` | WebSocket | WebSocket RPC connection to a specific chain |
| `/discovery/resources` | GET | Bazaar-compatible catalog of all supported networks |

## Testnet Caps

| Model | Testnet Lifetime Cap |
|-------|---------------------|
| Pay-per-request | 1,000 requests |
| Nanopayment | 10,000 requests |
| Credit Drawdown | 1,000 credits |

Caps are tracked independently per model. After reaching a cap, switch to the corresponding mainnet.

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
  svmPrivateKey: '<your-base58-secret-key>',
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
2. **Use testnet first.** Call `/drip` (Base Sepolia) or the [Circle faucet](https://faucet.circle.com/) to get testnet USDC before using mainnet.
3. **Use `preAuth: true`** with credit drawdown to speed up the first payment flow.
4. **Reuse session tokens.** The client caches JWTs automatically (1 hour expiry).
5. **gRPC-Web and WebSocket require credit drawdown.** Pay-per-request and nanopayment support JSON-RPC and REST only.
6. **Multi-protocol support.** The same client works with JSON-RPC, REST, gRPC-Web (`client.createGrpcTransport()`), and WebSocket (`client.createWebSocket()`).

## Documentation

- **x402 Platform**: https://x402.quicknode.com
- **x402 Documentation (llms.txt)**: https://x402.quicknode.com/llms.txt
- **@quicknode/x402 Package**: https://github.com/quiknode-labs/quicknode-x402
- **Examples**: https://github.com/quiknode-labs/qn-x402-examples
