# x402 Reference

x402 enables pay-per-request RPC access via USDC micropayments. No API key required — authenticate with Sign-In with Ethereum (SIWE), purchase credits with USDC on Base, and access 140+ blockchain RPC endpoints.

## Overview

| Property | Value |
|----------|-------|
| **Protocol** | HTTP 402 Payment Required |
| **Payment Method** | USDC on Base |
| **Authentication** | SIWE (Sign-In with Ethereum) |
| **Chains** | 140+ (same as Quicknode RPC network) |
| **Base URL** | `https://x402.quicknode.com` |
| **Use Cases** | Keyless RPC access, AI agents, pay-as-you-go, ephemeral wallets |

## How It Works

1. **Authenticate** — Sign a SIWE message with your Ethereum wallet to get a session token
2. **Purchase credits** — Send USDC on Base to buy credits (or use the testnet faucet for free credits)
3. **Make RPC requests** — Use credits to call any supported chain's RPC endpoint

## Key Endpoints

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/auth` | POST | Authenticate via SIWE, returns session token |
| `/credits` | GET | Check credit balance |
| `/drip` | POST | Testnet faucet — get free credits for testing |
| `/:network` | POST | JSON-RPC request to a specific chain (e.g., `/ethereum-mainnet`) |
| `/:network/ws` | WebSocket | WebSocket RPC connection to a specific chain |

## Credit Pricing

| Environment | Credits | Cost |
|-------------|---------|------|
| **Testnet** | 100 credits | $0.01 USDC (or free via `/drip`) |
| **Mainnet** | 1,000,000 credits | $10 USDC |

Credits are consumed per RPC request. Different methods cost different amounts of credits based on computational complexity.

## Setup

### Install Dependencies

```bash
npm install @x402/fetch @x402/evm viem siwe
```

- **`@x402/fetch`** — Drop-in `fetch` wrapper that automatically handles HTTP 402 payment responses. Use this for the simplest integration.
- **`@x402/evm`** — EVM payment scheme implementation. Only needed if you are building a custom payment flow or server-side payment verification.

### Authentication

```typescript
import { createWalletClient, http } from "viem";
import { privateKeyToAccount } from "viem/accounts";
import { base } from "viem/chains";
import { SiweMessage } from "siwe";

const X402_BASE_URL = "https://x402.quicknode.com";

// Create wallet client (use your preferred wallet method)
const account = privateKeyToAccount(process.env.PRIVATE_KEY as `0x${string}`);
const walletClient = createWalletClient({
  account,
  chain: base,
  transport: http(),
});

// Build and sign SIWE message
const siweMessage = new SiweMessage({
  domain: "x402.quicknode.com",
  address: account.address,
  statement: "Sign in to x402",
  uri: X402_BASE_URL,
  version: "1",
  chainId: 8453, // Base
  nonce: crypto.randomUUID(),
  issuedAt: new Date().toISOString(),
});

const message = siweMessage.prepareMessage();
const signature = await walletClient.signMessage({ message });

// Authenticate
const authResponse = await fetch(`${X402_BASE_URL}/auth`, {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ message, signature }),
});
const { token } = await authResponse.json();
```

### Making RPC Requests

```typescript
// Use the session token to make RPC requests
const rpcResponse = await fetch(`${X402_BASE_URL}/ethereum-mainnet`, {
  method: "POST",
  headers: {
    "Content-Type": "application/json",
    Authorization: `Bearer ${token}`,
  },
  body: JSON.stringify({
    jsonrpc: "2.0",
    method: "eth_blockNumber",
    params: [],
    id: 1,
  }),
});
const result = await rpcResponse.json();
console.log("Latest block:", result.result);
```

### Check Credit Balance

```typescript
const creditsResponse = await fetch(`${X402_BASE_URL}/credits`, {
  headers: { Authorization: `Bearer ${token}` },
});
const { credits } = await creditsResponse.json();
console.log("Remaining credits:", credits);
```

### Get Free Testnet Credits

```typescript
// Drip free testnet credits for testing
const dripResponse = await fetch(`${X402_BASE_URL}/drip`, {
  method: "POST",
  headers: { Authorization: `Bearer ${token}` },
});
const dripResult = await dripResponse.json();
console.log("Credits received:", dripResult);
```

## Using with @x402/fetch

The `@x402/fetch` package provides a drop-in `fetch` wrapper that handles 402 payment responses automatically.

```typescript
import { wrapFetch } from "@x402/fetch";
import { createWalletClient, http } from "viem";
import { privateKeyToAccount } from "viem/accounts";
import { base } from "viem/chains";

const account = privateKeyToAccount(process.env.PRIVATE_KEY as `0x${string}`);
const walletClient = createWalletClient({
  account,
  chain: base,
  transport: http(),
});

// Wrap fetch to auto-handle 402 payments
const x402Fetch = wrapFetch(fetch, walletClient);

// Use like normal fetch — payments are handled automatically
const response = await x402Fetch("https://x402.quicknode.com/ethereum-mainnet", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({
    jsonrpc: "2.0",
    method: "eth_getBalance",
    params: ["0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045", "latest"],
    id: 1,
  }),
});
const result = await response.json();
```

## Best Practices

1. **Use testnet credits first** — Call `/drip` to get free credits for development and testing before purchasing mainnet credits.
2. **Reuse session tokens** — SIWE authentication returns a session token. Reuse it across requests instead of re-authenticating each time.
3. **Monitor credit balance** — Check `/credits` periodically to avoid unexpected failures when credits run out.
4. **Ideal for AI agents** — x402 is well-suited for AI agents that need RPC access without managing API keys or infrastructure.
5. **Use `@x402/fetch` for simplicity** — The wrapper handles 402 payment flows automatically, reducing boilerplate.
6. **WebSocket support** — Use `/:network/ws` for subscription-based workloads to reduce per-request overhead.

## Documentation

- **x402 Platform**: https://x402.quicknode.com
- **x402 Documentation (llms.txt)**: https://x402.quicknode.com/llms.txt
