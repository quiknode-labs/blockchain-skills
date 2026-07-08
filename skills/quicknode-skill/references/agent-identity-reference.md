# Agent Identity Reference (ERC-8004)

ERC-8004 Explorer indexes ERC-8004 agent registrations, feedback, validations, and reputation across supported EVM networks. The public REST API is read-only and paywalled per request with x402. Use it to search indexed agents and reputation data; use the on-chain ERC-8004 registry contracts to register or update agents.

**Base URL:** `https://erc-8004.quicknode.com`
**REST prefix:** `/v1`
**Spec:** https://eips.ethereum.org/EIPS/eip-8004
**API docs:** https://erc-8004.quicknode.com/docs/api/v1

## Verified Routing

```bash
curl -i "https://erc-8004.quicknode.com/v1/agents?network=base-mainnet&page=1&per_page=3"
```

Without a valid x402 payment header, REST endpoints return HTTP 402 `Payment Required` with an `accepts` array describing the required payment.

## x402 Payment

Every REST API call requires an x402 micropayment.

| Item | Value |
|------|-------|
| Production payment network | Base mainnet |
| Staging payment network | Base Sepolia |
| Currency | USDC |
| Advertised unauthenticated cost | `maxAmountRequired: "1000"` USDC base units ($0.001) |
| Payment header on retry | `X-PAYMENT` |

Always parse the 402 response and use its `accepts[0]` payment details instead of hardcoding recipient, token, or amount. The docs state the amount is subject to change and is advertised in each 402 response.

## Rate Limits

| Scope | Limit |
|-------|-------|
| Global per source IP | 300 requests/min |
| `/v1/agents` per source IP | 60 requests/min |

Requests beyond the cap return HTTP 429 with `Retry-After`. Clients behind the same NAT share the source-IP bucket.

## REST Endpoints

All endpoints below are x402-paywalled unless otherwise noted.

| Method | Path | Purpose |
|--------|------|---------|
| GET | `/v1/agents` | List/search indexed agents |
| GET | `/v1/agents/:agent_id` | Fetch one indexed agent |
| GET | `/v1/agents/:agent_id/feedback` | Fetch feedback for one agent |
| GET | `/v1/agents/:agent_id/validations` | Fetch validations for one agent |
| GET | `/v1/agents/:agent_id/validations/summary` | Fetch aggregate validation counts and tag breakdowns |
| GET | `/v1/agents/:agent_id/reputation` | Fetch live reputation calculation |
| GET | `/v1/feedback` | Fetch global feedback feed |
| GET | `/v1/validators/:validator_address/requests` | Fetch validation requests handled by a validator |

The Explorer UI, documentation pages, and static discovery files are not the paid REST API. The REST API catalog is also available as a Postman collection at `https://erc-8004.quicknode.com/docs/collections/postman.json`.

## List Agents

```bash
curl -i "https://erc-8004.quicknode.com/v1/agents?network=base-mainnet&page=1&per_page=20"
```

Query parameters confirmed by the Postman collection:

| Parameter | Notes |
|-----------|-------|
| `network` | Network slug, for example `base-mainnet`, `ethereum-mainnet`, `bnb-mainnet`, `avalanche-mainnet`, `mantle-mainnet` |
| `owner` | Filter by owner address |
| `include_testnets` | Include testnet networks when supported |
| `page`, `per_page` | Pagination |

## Get an Agent

```bash
curl -i "https://erc-8004.quicknode.com/v1/agents/42?network=base-mainnet"
```

Use `network` when the same `agent_id` may exist on multiple chains.

## Agent Feedback

```bash
curl -i "https://erc-8004.quicknode.com/v1/agents/42/feedback?network=base-mainnet&per_page=20"
```

Query parameters:

| Parameter | Notes |
|-----------|-------|
| `tag1`, `tag2` | Filter by feedback tags |
| `since`, `until` | ISO 8601 time bounds |
| `include_revoked` | Include revoked feedback rows; default is false |
| `network` | Required when the `agent_id` is ambiguous |
| `page`, `per_page` | Pagination |

## Agent Validations

```bash
curl -i "https://erc-8004.quicknode.com/v1/agents/42/validations?network=base-mainnet"
```

Query parameters:

| Parameter | Notes |
|-----------|-------|
| `tag` | Filter by validation tag |
| `has_response` | Filter validations with or without response payload |
| `since`, `until` | ISO 8601 time bounds |
| `network` | Required when ambiguous |
| `page`, `per_page` | Pagination |

## Validation Summary

```bash
curl -i "https://erc-8004.quicknode.com/v1/agents/42/validations/summary?network=base-mainnet"
```

Supported filters include `network`, `tag`, `validators`, and `include_testnets`.

## Reputation

```bash
curl -i "https://erc-8004.quicknode.com/v1/agents/42/reputation?network=base-mainnet"
```

The agent show route may include cached reputation data. Call `/v1/agents/:agent_id/reputation` directly when the freshest live reputation calculation is needed.

## Global Feedback

```bash
curl -i "https://erc-8004.quicknode.com/v1/feedback?network=base-mainnet&page=1&per_page=20"
```

Common filters include `network`, `include_revoked`, `page`, and `per_page`.

## Validator Requests

```bash
curl -i "https://erc-8004.quicknode.com/v1/validators/0x0000000000000000000000000000000000000000/requests?network=base-mainnet"
```

Common filters include `network`, `tag`, and `has_response`.

## Register or Update an Agent

The REST API does not expose a verified `POST /v1/agents` registration endpoint. Register agents on-chain by calling the ERC-8004 Identity Registry contract. The registration transaction mints an ERC-721 agent token and emits:

```text
Registered(uint256 indexed agentId, string agentURI, address indexed owner)
```

The Explorer indexes that event. After confirmation, the agent page appears at:

```text
https://erc-8004.quicknode.com/agents/<network-slug>/<tokenId>
```

Use the official registration tutorial and contracts page for current contract addresses, ABIs, metadata schema, and viem examples:

- https://erc-8004.quicknode.com/tutorials/register-agent
- https://erc-8004.quicknode.com/docs/contracts

## Documentation

- **REST API v1**: https://erc-8004.quicknode.com/docs/api/v1
- **Agents API**: https://erc-8004.quicknode.com/docs/api/v1/agents
- **x402 payment flow**: https://erc-8004.quicknode.com/docs/api/v1/x402
- **Postman collection**: https://erc-8004.quicknode.com/docs/collections/postman.json
- **Contracts and ABIs**: https://erc-8004.quicknode.com/docs/contracts
- **EIP-8004 Spec**: https://eips.ethereum.org/EIPS/eip-8004
