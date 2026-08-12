# Swap API Reference

Quicknode Swap API is a marketing umbrella for swap-provider add-ons available through Quicknode endpoints. It gives teams one Quicknode account, API key, support path, and billing layer across providers.

**Marketing:** https://www.quicknode.com/swap-api

## Providers

| Provider | Chain(s) | Primary Use |
|----------|----------|-------------|
| Metis (Jupiter) | Solana | Jupiter swap quotes and transaction building on Quicknode infrastructure |
| 0x Swap API | EVM chains | Aggregated EVM liquidity and Permit2 quote flow |
| OpenOcean V4 | EVM, Solana, Sui, NEAR, and others | Multi-chain swap routing |
| Aerodrome Swap API | Base | Base liquidity routing, pool data, and swap transaction building |
| Velodrome Swap API | Optimism | Optimism liquidity routing, pool data, and swap transaction building |
| Titan Meta Aggregator | Solana | Solana meta-aggregation through REST and WebSocket APIs |
| Hyperliquid Exchange API | Hyperliquid | Hyperliquid exchange trading workflows |

Enable the required provider add-on in the Quicknode dashboard before calling provider endpoints.

## Solana - Metis (Jupiter)

Set `QUICKNODE_METIS_URL` to the Metis endpoint URL from the add-on page, for example `https://jupiter-swap-api.quiknode.pro/YOUR_TOKEN`.

**Docs:** https://www.quicknode.com/docs/solana/metis-overview

### Get a Quote

```typescript
const quoteUrl = new URL(`${process.env.QUICKNODE_METIS_URL}/quote`);
quoteUrl.searchParams.set("inputMint", "So11111111111111111111111111111111111111112");
quoteUrl.searchParams.set("outputMint", "EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v");
quoteUrl.searchParams.set("amount", "1000000000");
quoteUrl.searchParams.set("slippageBps", "50");

const quote = await fetch(quoteUrl).then((res) => res.json());
console.log(quote.outAmount, quote.routePlan);
```

### Build a Swap Transaction

```typescript
import { createJupiterApiClient } from "@jup-ag/api";

const jupiterApi = createJupiterApiClient({
  basePath: process.env.QUICKNODE_METIS_URL!,
});

const swap = await jupiterApi.swapPost({
  swapRequest: {
    quoteResponse: quote,
    userPublicKey: "YourPubkey...",
  },
});

// swap.swapTransaction is a base64 serialized transaction ready for signing.
```

## Hyperliquid Exchange API

Hyperliquid Exchange API is for Hyperliquid exchange workflows, separate from HyperCore data streaming and SQL Explorer analytics.

**Hyperliquid Docs:** https://www.quicknode.com/docs/hyperliquid
**Build Market Order:** https://www.quicknode.com/docs/hyperliquid/exchange-api/build-market-order
**Marketing:** https://www.quicknode.com/chains/hyperliquid

Use HyperCore gRPC/JSON-RPC/WebSocket/Info API references for market data. Use the Exchange API only when the user explicitly needs trading/exchange actions and has the required credentials and risk confirmation.

```typescript
const url = `${process.env.QUICKNODE_HYPERLIQUID_ENDPOINT}/hypercore/exchange`;
const post = (body) =>
  fetch(url, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(body),
  }).then((r) => r.json());

// 1. Build a spot market buy of HYPE (no signature)
const built = await post({
  action: { type: 'order', orders: [
    { asset: 'HYPE', side: 'buy', size: '5', tif: 'market' },
  ] },
});

// 2. Sign the EIP-712 hash locally (keys never leave your machine)
const signature = await wallet.signTypedData(built.typedData);

// 3. Send the signed order
const { exchangeResponse } = await post({
  action: built.action, nonce: built.nonce, signature,
});
```

## EVM - 0x Swap API

Use the 0x add-on for EVM aggregated liquidity. The v2 quote endpoint uses Permit2.

**Add-on page:** https://www.quicknode.com/add-ons/0x-swap-api
**Provider docs:** https://0x.org/docs/api

```typescript
const params = new URLSearchParams({
  chainId: "1",
  sellToken: "0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48",
  buyToken: "0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2",
  sellAmount: "100000000",
  taker: "0xYourAddress...",
});

const quote = await fetch(`${process.env.QUICKNODE_RPC_URL}/addon/1117/swap/permit2/quote?${params}`)
  .then((res) => res.json());

console.log(quote.transaction);
```

## EVM and Multi-chain - OpenOcean

OpenOcean provides multi-chain routing. Use the provider docs for the exact chain slug and quote/swap path for the enabled add-on.

**Add-on page:** https://www.quicknode.com/add-ons/openocean-v4-swap-api
**Provider docs:** https://docs.openocean.finance/

```typescript
const chain = "eth";
const params = new URLSearchParams({
  inTokenAddress: "0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48",
  outTokenAddress: "0xEeeeeEeeeEeEeeEeEeEeeEEEeeeeEeeeeeeeEEeE",
  amount: "100",
  slippage: "1",
  account: "0xYourAddress...",
});

const quote = await fetch(`${process.env.QUICKNODE_RPC_URL}/addon/807/v4/${chain}/quote?${params}`)
  .then((res) => res.json());
```

## Base - Aerodrome Swap API

Aerodrome is the primary liquidity hub on Base. The add-on is used for routing, quote, swap transaction construction, pool analytics, and price feeds.

**Add-on page:** https://www.quicknode.com/add-ons/aerodrome-swap-api

```typescript
const params = new URLSearchParams({
  quotetarget: "base",
  from_token: "0x940181a94A35A4569E4529A3CDfB74e38FD98631",
  to_token: "0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913",
  amount: "1",
});

const quote = await fetch(`${process.env.QUICKNODE_RPC_URL}/addon/1051/v1/quote?${params}`)
  .then((res) => res.json());

const swap = await fetch(`${process.env.QUICKNODE_RPC_URL}/addon/1051/v1/swap/build?target=aero`, {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({
    from_token: "0x940181a94A35A4569E4529A3CDfB74e38FD98631",
    to_token: "0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913",
    amount: 1.0,
    wallet_address: "0xYourAddress...",
    slippage: 0.01,
  }),
}).then((res) => res.json());
```

## Optimism - Velodrome Swap API

Velodrome is the primary liquidity hub on Optimism. The add-on is used for routing, quote, swap transaction construction, pool analytics, and price feeds.

**Add-on page:** https://www.quicknode.com/add-ons/velodrome-swap-api

```typescript
const params = new URLSearchParams({
  target: "velo",
  limit: "50",
  symbols: "VELO,USDC",
});

const prices = await fetch(`${process.env.QUICKNODE_RPC_URL}/addon/1050/v1/prices?${params}`)
  .then((res) => res.json());

const swap = await fetch(`${process.env.QUICKNODE_RPC_URL}/addon/1050/v1/swap/build?target=velo`, {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({
    from_token: "0xYourInputToken...",
    to_token: "0xYourOutputToken...",
    amount: 1.0,
    wallet_address: "0xYourAddress...",
    slippage: 0.01,
  }),
}).then((res) => res.json());
```

## Solana - Titan Meta Aggregator

Titan Meta Aggregator provides Solana swap aggregation through REST and WebSocket APIs.

**Add-on page:** https://www.quicknode.com/add-ons/titan-swap

```typescript
import { decode } from '@msgpack/msgpack';

const res = await fetch(
  `${process.env.QUICKNODE_ENDPOINT}/addon/1147/api/v1/quote/price?` +
  new URLSearchParams({
    inputMint:  'So11111111111111111111111111111111111111112', // SOL
    outputMint: 'EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v', // USDC
    amount:     '1000000000', // lamports
  }),
  { headers: { Accept: 'application/vnd.msgpack' } }
);
const { amountIn, amountOut } = decode(
  new Uint8Array(await res.arrayBuffer())
) as { amountIn: number; amountOut: number };
```

## Safety Defaults

- Always get explicit user confirmation before constructing or submitting a swap transaction.
- Never request or store private keys.
- Quote first; show route, expected output, price impact, slippage, and estimated fees before building a transaction.
- Re-fetch quotes immediately before signing. Swap quotes can expire quickly.
- Verify token mint/contract addresses from trusted sources.

## Troubleshooting

| Error | Cause | Fix |
|-------|-------|-----|
| `403 Forbidden` | Provider add-on not enabled | Enable the add-on on the endpoint |
| No route | Pair has insufficient liquidity | Try a different token, amount, or provider |
| Slippage exceeded | Price moved after quote | Re-quote or adjust slippage |
| Invalid token | Wrong address format or chain | Verify token mint/contract and chain |

## Documentation

- **Swap API Marketing**: https://www.quicknode.com/swap-api
- **Metis Overview**: https://www.quicknode.com/docs/solana/metis-overview
- **0x Swap API Add-on**: https://www.quicknode.com/add-ons/0x-swap-api
- **OpenOcean V4 Add-on**: https://www.quicknode.com/add-ons/openocean-v4-swap-api
- **Aerodrome Swap API Add-on**: https://www.quicknode.com/add-ons/aerodrome-swap-api
- **Velodrome Swap API Add-on**: https://www.quicknode.com/add-ons/velodrome-swap-api
- **Titan Swap Add-on**: https://www.quicknode.com/add-ons/titan-swap
- **Titan API Specification**: https://titan-exchange.gitbook.io/titan-exchange/
- **Hyperliquid**: https://www.quicknode.com/docs/hyperliquid
- **Hyperliquid Build Market Order**: https://www.quicknode.com/docs/hyperliquid/exchange-api/build-market-order
- **Add-ons Catalog**: https://www.quicknode.com/add-ons
