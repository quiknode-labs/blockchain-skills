# Blockchain Skills

> A collection of AI agent skills for blockchain development, powered by [Quicknode](https://www.quicknode.com/)


## Introduction

This repository contains a curated set of skills that enable AI agents to interact with blockchain networks, smart contracts, and decentralized applications. Each skill provides specialized knowledge and capabilities for specific blockchain ecosystems and protocols.

## What are Skills?

Skills are structured instruction sets that AI agents can use to perform blockchain-related tasks. They provide:

- **Context** - Domain-specific knowledge about protocols, APIs, and best practices
- **Capabilities** - Step-by-step guidance for common operations
- **Integration** - Direct access to Quicknode's RPC endpoints and APIs

When an AI agent loads a skill, it gains the ability to help you build, deploy, and interact with blockchain applications.

## Available Skills

| Skill | Description |
|-------|-------------|
| `quicknode-skill` | Quicknode blockchain infrastructure across five product categories: **Infrastructure** (Core RPC API, IPFS, Dedicated Clusters), **Real-Time Data** (Streams, Streams Backfills, Webhooks, Solana gRPC), **Indexed Data** (SQL Explorer, HyperCore for Hyperliquid, Metaplex DAS API, Blockbook, Ordinals & Runes API, Agent Identity ERC-8004), **Trading & DeFi** (Swap API), and **Agent Surface** (x402, MPP, Agent Subscriptions, Quicknode CLI, Quicknode MCP) |

## Installation

### Using [skills.sh](https://skills.sh/)

```bash
npx skills add quiknode-labs/blockchain-skills
```

## Usage

Once installed, you can use natural language prompts to interact with Quicknode's infrastructure:

- "Set up a stream to monitor new blocks on Ethereum"
- "Create a filter for large USDC transfers"
- "Get the SOL balance for this wallet address"
- "Check the ETH balance across multiple wallets"
- "Query recent trades on Hyperliquid for BTC"
- "Show me whale trades above $500k notional"
- "Get liquidation data for the last 24 hours"
- "Quote a swap from USDC to ETH on Base"
- "Get wallet data and UTXOs for this BTC address"
- "List recent Solana NFT mints using Metaplex DAS"
- "Backfill last week of Ethereum blocks into my Stream"
- "Pay an RPC call with my wallet using x402 or MPP"
- "Create a Quicknode account with my wallet and provision an endpoint"
- "Provision a Stream using the Quicknode CLI"
- "Connect Claude to my Quicknode workspace via MCP"

## Quicknode Integration

This repository leverages Quicknode's infrastructure for reliable blockchain access:

### Infrastructure

Connect to 80+ blockchain networks with low-latency RPC endpoints:
- **Core RPC API** — Ethereum, Polygon, Arbitrum, Optimism, Base, Solana, Bitcoin, TON, and many more
- **IPFS** — Decentralized file storage for NFT metadata and assets
- **Dedicated Clusters** — Private node clusters for high-throughput production workloads

### Real-Time Data

Real-time and historical blockchain data delivery:
- **Streams** — Block, transaction, log, and receipt pipelines with custom JavaScript filters
- **Streams Backfills** — On-demand historical replay into existing Streams
- **Webhooks** — Address and event-based notifications over HTTP
- **Solana gRPC** — Geyser plugin streaming for accounts, transactions, and slots

### Indexed Data

Query and discover indexed blockchain data:
- **SQL Explorer** — Direct SQL access to indexed Hyperliquid (HyperCore) data at scale
- **HyperCore for Hyperliquid** — gRPC, JSON-RPC, WebSocket, and Info API for Hyperliquid L1
- **Metaplex DAS API** — Solana NFT, compressed NFT, and token queries
- **Blockbook** — Wallet-centric address balances, UTXOs, and transaction history
- **Ordinals & Runes API** — Bitcoin inscription and Runes protocol data
- **Agent Identity (ERC-8004)** — On-chain agent discovery and reputation records

### Trading & DeFi

- **Swap API** — Multi-DEX aggregated token swaps: Solana (Metis/Jupiter), EVM (0x, OpenOcean, Aerodrome, Velodrome, Titan)

### Agent Surface

Stablecoin-paid access and tooling for autonomous agents:
- **x402** and **MPP** — Free tier of 1,000,000 API credits per wallet per month (covers RPC + SQL Explorer), with pay-per-request billing after the free tier resets on the 1st of each month
- **Agent Subscriptions** — Create a Quicknode account and receive a full platform API key in a single payment
- **Quicknode CLI** — `qn` CLI for managing endpoints, Streams, Webhooks, KV, SQL, usage, billing, and teams from the terminal
- **Quicknode MCP** — Native Claude/OpenAI connector plus generic MCP server; manage Quicknode from Claude, ChatGPT, Codex, Cursor, or any MCP-capable AI tool
- **Quicknode SDK** — Product API SDK for Admin API, Streams, Webhooks, KV, and SQL Explorer

### Other Quicknode Products

Agents typically do not invoke these directly: Validator as a Service, Solana Validator, Monad Validator, ChainKit, Dedicated Clusters.

Get your Quicknode endpoint at [Quicknode.com](https://www.quicknode.com/)

## Skill Structure

Each skill follows a consistent file organization:

```
skill-name/
├── SKILL.md              # Main skill definition
├── PRODUCT_CATALOG.md    # Canonical product map and sync runbook
└── references/           # API and protocol references
    └── api.md
```

## Contributing

We welcome contributions! To add a new skill:

1. Fork this repository
2. Create a new directory for your skill
3. Add a `SKILL.md` with the skill definition
4. Include examples and documentation
5. Submit a pull request

### Skill Guidelines

- Focus on a specific chain, protocol, or use case
- Include practical examples with expected outputs
- Reference official documentation and APIs
- Test with real Quicknode endpoints

## License

MIT License

---

Built with Quicknode - [Get Started](https://www.quicknode.com/)
