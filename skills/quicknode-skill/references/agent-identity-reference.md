# Agent Identity Reference (ERC-8004)

On-chain agent discovery and reputation based on the ERC-8004 standard. Agents publish structured records to `erc-8004.quicknode.com` describing their capabilities, supported payment methods, and callable endpoints. Other agents or services look up these records to discover and verify agents without requiring a centralized registry.

**API Base:** https://erc-8004.quicknode.com
**Explorer:** https://erc-8004.quicknode.com/explorer
**Spec:** https://eips.ethereum.org/EIPS/eip-8004

## What an ERC-8004 Record Contains

| Field | Description |
|-------|-------------|
| `name` | Human-readable agent name |
| `description` | Short description of the agent's capabilities |
| `url` | Primary endpoint (agents.md, OpenAPI spec, or homepage) |
| `skills` | Array of skill identifiers the agent supports |
| `payments` | Accepted payment protocols (x402, MPP, etc.) |
| `chains` | Blockchain networks the agent operates on |
| `version` | Agent version string |

## Look Up an Agent Record

```typescript
const agentAddress = '0xYourAgentAddress...';

const response = await fetch(
  `https://erc-8004.quicknode.com/api/v1/agents/${agentAddress}`
);
const record = await response.json();
// record.address — on-chain address
// record.name, record.description, record.url
// record.skills — array of skill IDs
// record.payments — accepted protocols
// record.chains — supported chains
// record.verifiedAt — last on-chain verification timestamp
```

## Register an Agent Record

Registration writes a record to the ERC-8004 registry contract on the Ethereum network.

```typescript
const response = await fetch('https://erc-8004.quicknode.com/api/v1/agents', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    name: 'My Trading Agent',
    description: 'Executes swaps and monitors Hyperliquid positions',
    url: 'https://myagent.example.com/agents.md',
    skills: ['quicknode-skill', 'defi-skill'],
    payments: ['x402', 'mpp'],
    chains: ['eip155:1', 'eip155:8453'],
    version: '1.0.0',
  }),
});

const { registrationTx } = await response.json();
// registrationTx — transaction hash of the registry write
```

> The caller must sign the registration transaction with the agent's private key. The API returns the unsigned transaction; sign and broadcast it using your preferred wallet client.

## Search Agents by Skill

```typescript
const response = await fetch(
  'https://erc-8004.quicknode.com/api/v1/agents?skill=quicknode-skill&limit=10'
);
const { agents } = await response.json();
// agents — array of matching agent records
```

## Verify an Agent's Reputation

ERC-8004 records include an on-chain signature that proves the record was published by the address that controls the agent. Use `verifiedAt` to check freshness.

```typescript
const response = await fetch(
  `https://erc-8004.quicknode.com/api/v1/agents/${agentAddress}/verify`
);
const { verified, score, lastActivity } = await response.json();
// verified — boolean: signature valid and record current
// score — reputation score (0–100) based on on-chain activity
// lastActivity — timestamp of last on-chain action
```

## Explorer

Browse registered agents, filter by skill or payment method, and inspect record details at:
https://erc-8004.quicknode.com/explorer

## Documentation

- **ERC-8004 API**: https://erc-8004.quicknode.com
- **EIP-8004 Spec**: https://eips.ethereum.org/EIPS/eip-8004
- **Quicknode Build with AI**: https://www.quicknode.com/docs/build-with-ai
