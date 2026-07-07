# Agent Subscriptions Reference

Programmatic Quicknode account creation for autonomous agents. A single authenticated POST returns a `QN_*` full platform API key — no dashboard, email confirmation, or human operator required.

**Docs:** https://www.quicknode.com/docs/build-with-ai/agent-subscriptions

## Endpoints

| Method | Path | Auth | Rate Limit | Purpose |
|--------|------|------|------------|---------|
| POST | `/api/v1/agent/subscriptions` | Payment header (x402 or MPP) | 20/min, 5/hour by IP + email | Create account + subscription |
| POST | `/api/v1/agent/top_up` | API key + payment header | 30/min by API key | Add credits to existing subscription |
| GET | `/api/v1/agent/balance` | API key | 60/min by API key | Read current credit balance |

All requests target `https://www.quicknode.com`.

## Discover Plans (Send Without Payment First)

Send the request without a payment header. The server returns HTTP 402 with the available plans, prices, and payment details in the body.

```bash
curl -X POST https://www.quicknode.com/api/v1/agent/subscriptions \
  -H "Content-Type: application/json" \
  -d '{}'
```

Parse the 402 body to get `plan_name` options, per-plan prices, accepted `paymentNetwork` values, asset contract addresses, and the `payTo` recipient address. Then retry with a payment header and the chosen plan.

## Create a Subscription (x402)

```typescript
import { createQuicknodeX402Client } from '@quicknode/x402'

const client = await createQuicknodeX402Client({
  baseUrl: 'https://www.quicknode.com',
  network: 'eip155:8453',  // Base Mainnet
  evmPrivateKey: process.env.PRIVATE_KEY as `0x${string}`,
})

const res = await client.fetch(
  'https://www.quicknode.com/api/v1/agent/subscriptions',
  {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      plan_name: 'b6_build',          // b6_build | b6_accelerate | b6_scale | b6_business
      interval: 'monthly',            // monthly | yearly
      email: 'agent@example.com',
      password: process.env.ACCOUNT_PASSWORD,
      password_confirmation: process.env.ACCOUNT_PASSWORD,
      full_name: 'Autonomous Agent',
      name: 'Agent Account',
      billing_address: {
        line1: '123 Main St',
        city: 'New York',
        postal_code: '10001',
        country: 'US',
      },
    }),
  },
)

const { api_key } = await res.json()
// api_key → "QN_..." — store this; it's the QUICKNODE_API_KEY for all subsequent calls
```

## Create a Subscription (MPP)

```typescript
import { Mppx, tempo } from 'mppx/client'
import { privateKeyToAccount } from 'viem/accounts'

const account = privateKeyToAccount(process.env.PRIVATE_KEY as `0x${string}`)
Mppx.create({ methods: [tempo({ account })] })

const res = await fetch('https://www.quicknode.com/api/v1/agent/subscriptions', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    plan_name: 'b6_build',
    interval: 'monthly',
    email: 'agent@example.com',
    password: process.env.ACCOUNT_PASSWORD,
    password_confirmation: process.env.ACCOUNT_PASSWORD,
    full_name: 'Autonomous Agent',
    name: 'Agent Account',
    billing_address: {
      line1: '123 Main St',
      city: 'New York',
      postal_code: '10001',
      country: 'US',
    },
  }),
})

const { api_key } = await res.json()
```

## Top Up Credits

```typescript
const res = await client.fetch(
  'https://www.quicknode.com/api/v1/agent/top_up',
  {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'x-api-key': process.env.QUICKNODE_API_KEY!,
    },
    body: JSON.stringify({
      amount: 10.0,  // top-up amount in USD
    }),
  },
)
const { balance_cents } = await res.json()
```

## Check Credit Balance

```typescript
const res = await fetch('https://www.quicknode.com/api/v1/agent/balance', {
  headers: { 'x-api-key': process.env.QUICKNODE_API_KEY! },
})
const { balance_cents } = await res.json()
// balance_cents — current balance in cents
```

## Plan IDs

| Plan ID | Dashboard Name |
|---------|---------------|
| `b6_build` | Build |
| `b6_accelerate` | Accelerate |
| `b6_scale` | Scale |
| `b6_business` | Business |

See https://www.quicknode.com/pricing for current plan pricing and credit allocations.

## Guard Rails

- **Real funds** — Subscriptions settle in real stablecoins on a mainnet payment network. Always confirm plan, interval, and payment network with the user before sending.
- **Duplicate email** — Re-using an existing email returns an error. Retry with the same `email` and `password` to resume; never create a duplicate account.
- **Payment retry safety** — If validation fails after a payment header is generated, retry with the same signed payment header byte-for-byte. Do not sign a fresh payment for the same attempted operation unless the server requires a new challenge.
- **No free trials** — All subscriptions are production-grade from the first request.
- **Synchronous** — Account and subscription are created in a single request. No async job to poll.
- **Password requirements** — 8–64 characters, at least one lowercase, one uppercase, one number, one special character. Never fabricate credentials.
- **Name requirements** — `full_name` and account `name` must each be 4–40 characters. The password must not equal the email.

## How It Pairs with x402 / MPP

- Use **x402 or MPP** for short-lived, stateless RPC access. No account needed.
- Use **Agent Subscriptions** when the agent needs the full platform: Streams, Webhooks, Key-Value Store, multiple endpoints, security rules, or billing control.
- The returned `api_key` is identical to `QUICKNODE_API_KEY` used in all other Quicknode API calls.

## Documentation

- **Agent Subscriptions**: https://www.quicknode.com/docs/build-with-ai/agent-subscriptions
- **Build with AI Overview**: https://www.quicknode.com/docs/build-with-ai
- **x402 Reference**: [x402-reference.md](x402-reference.md)
- **MPP Reference**: [mpp-reference.md](mpp-reference.md)
