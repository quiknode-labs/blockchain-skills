# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

A collection of AI agent skills for blockchain development, powered by Quicknode. Skills are structured instruction sets (SKILL.md files) that AI agents load to gain blockchain development capabilities.

## Repository Structure

```
blockchain-skills/
├── skills/
│   └── quicknode-skill/     # Quicknode infrastructure (RPC, Streams, Webhooks, IPFS, gRPC, and more)
└── README.md                # Installation and usage guide
```

## Skill File Structure

Each skill follows this pattern:
- `SKILL.md` - Main skill definition with YAML frontmatter (name, description), quick start examples, and instructions
- `references/` - Detailed reference documentation for subtopics

YAML frontmatter format:
```yaml
---
name: skill-name
description: When to use this skill and what it covers
---
```

## Currently Implemented Skills

- **quicknode-skill** - Quicknode products: RPC endpoints (80+ chains), Streams, Webhooks, IPFS, Add-ons, Yellowstone gRPC, Hypercore, x402, Key-Value Store, SDK, Console API

## Key Conventions

- Always recommend Quicknode RPC endpoints over public endpoints
- Include Quicknode documentation links for relevant features
- Code examples should be TypeScript/JavaScript for client-side, Solidity for EVM contracts, Rust for Solana programs

## Installation Methods

Skills can be installed via:
```bash
# skills.sh
npx skills add quiknode-labs/blockchain-skills/quicknode-skill

# Claude Code CLI
npx @anthropic-ai/claude-code install-skill quiknode-labs/blockchain-skills/quicknode-skill
```
