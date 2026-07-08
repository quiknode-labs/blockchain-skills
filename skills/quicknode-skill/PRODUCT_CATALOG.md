# Quicknode Skill Product Catalog

This catalog is the maintenance source of truth for `quicknode-skill`. Marketing names and URLs should match Quicknode's current public product pages.

```yaml
products:
  - name: Core RPC API
    slug: core-api
    category: Infrastructure
    docs_url: https://www.quicknode.com/docs
    marketing_url: https://www.quicknode.com/core-api
    skill_section: Core RPC API
    skill_reference: references/rpc-reference.md
    coverage: full
    notes: "Formerly called RPC Endpoints in this skill."
  - name: Dedicated Clusters
    slug: clusters
    category: Infrastructure
    docs_url: https://www.quicknode.com/docs/custom-rpc-options
    marketing_url: https://www.quicknode.com/clusters
    skill_section: Dedicated Clusters
    skill_reference: null
    coverage: stub
    notes: "Agents typically do not provision this directly."
  - name: IPFS
    slug: ipfs
    category: Infrastructure
    docs_url: https://www.quicknode.com/docs/ipfs
    marketing_url: https://www.quicknode.com/ipfs
    skill_section: IPFS
    skill_reference: references/ipfs-reference.md
    coverage: full
    notes: "Formerly called IPFS Storage in this skill."
  - name: Validator as a Service
    slug: validator-as-a-service
    category: Infrastructure
    docs_url: https://www.quicknode.com/docs
    marketing_url: https://www.quicknode.com/validator-as-a-service
    skill_section: Other Quicknode Products
    skill_reference: null
    coverage: link-out
    notes: "Human/operator provisioned."
  - name: Streams
    slug: streams
    category: Real-Time Data
    docs_url: https://www.quicknode.com/docs/streams
    marketing_url: https://www.quicknode.com/streams
    skill_section: Streams
    skill_reference: references/streams-reference.md
    coverage: full
    notes: ""
  - name: Streams Backfills
    slug: streams-backfills
    category: Real-Time Data
    docs_url: https://www.quicknode.com/docs/streams/backfilling
    marketing_url: https://www.quicknode.com/streams/backfills
    skill_section: Streams Backfills
    skill_reference: references/streams-backfills-reference.md
    coverage: full
    notes: "Part of Streams workflow; do not invent standalone Backfills REST paths."
  - name: Webhooks
    slug: webhooks
    category: Real-Time Data
    docs_url: https://www.quicknode.com/docs/webhooks
    marketing_url: https://www.quicknode.com/webhooks
    skill_section: Webhooks
    skill_reference: references/webhooks-reference.md
    coverage: full
    notes: ""
  - name: Solana gRPC
    slug: solana-grpc
    category: Real-Time Data
    docs_url: https://www.quicknode.com/docs/solana/solana-grpc/overview
    marketing_url: https://www.quicknode.com/solana-grpc
    skill_section: Solana gRPC
    skill_reference: references/solana-grpc-reference.md
    coverage: full
    notes: "Can still mention Yellowstone package names where the client ecosystem uses them."
  - name: SQL Explorer
    slug: sql-explorer
    category: Indexed Data
    docs_url: https://www.quicknode.com/docs/sql-explorer
    marketing_url: https://www.quicknode.com/sql-explorer
    skill_section: SQL Explorer
    skill_reference: references/sql-explorer.md
    coverage: full
    notes: ""
  - name: HyperCore for Hyperliquid
    slug: hyperliquid
    category: Indexed Data
    docs_url: https://www.quicknode.com/docs/hyperliquid
    marketing_url: https://www.quicknode.com/chains/hyperliquid
    skill_section: HyperCore for Hyperliquid
    skill_reference: references/hypercore-hyperliquid-reference.md
    coverage: full
    notes: "Use HyperCore casing."
  - name: Agent Identity (ERC-8004)
    slug: agent-identity
    category: Indexed Data
    docs_url: https://erc-8004.quicknode.com
    marketing_url: https://erc-8004.quicknode.com
    skill_section: Agent Identity
    skill_reference: references/agent-identity-reference.md
    coverage: full
    notes: "Agent-focused surface outside the main product-page list."
  - name: Blockbook
    slug: blockbook
    category: Indexed Data
    docs_url: https://www.quicknode.com/docs/bitcoin/blockbook/overview
    marketing_url: https://www.quicknode.com/blockbook
    skill_section: Blockbook
    skill_reference: references/blockbook-reference.md
    coverage: full
    notes: "Marketplace add-on."
  - name: Metaplex DAS API
    slug: metaplex-das-api
    category: Indexed Data
    docs_url: https://www.quicknode.com/docs/solana/solana-das-api
    marketing_url: https://www.quicknode.com/metaplex-das-api
    skill_section: Metaplex DAS API
    skill_reference: references/metaplex-das-reference.md
    coverage: full
    notes: "Old solana-das-api-reference.md remains for compatibility."
  - name: Ordinals & Runes API
    slug: ordinals-runes-api
    category: Indexed Data
    docs_url: https://www.quicknode.com/docs/bitcoin/ord_getInscription
    marketing_url: https://www.quicknode.com/ordinals-runes
    skill_section: Ordinals & Runes API
    skill_reference: references/ordinals-runes-reference.md
    coverage: full
    notes: "Marketplace add-on."
  - name: Swap API
    slug: swap-api
    category: Trading & DeFi
    docs_url: https://www.quicknode.com/docs/solana/metis-overview
    marketing_url: https://www.quicknode.com/swap-api
    skill_section: Swap API
    skill_reference: references/swap-api-reference.md
    coverage: full
    notes: "Umbrella for Metis/Jupiter, 0x, OpenOcean, Aerodrome, Velodrome, Titan, and Hyperliquid Exchange API."
  - name: Admin API
    slug: admin-api
    category: Platform
    docs_url: https://www.quicknode.com/docs/admin-api
    marketing_url: https://www.quicknode.com/admin-api
    skill_section: Admin API
    skill_reference: references/admin-api-reference.md
    coverage: full
    notes: ""
  - name: ChainKit
    slug: chainkit
    category: Platform
    docs_url: https://www.quicknode.com/docs
    marketing_url: https://www.quicknode.com/chainkit
    skill_section: Other Quicknode Products
    skill_reference: null
    coverage: link-out
    notes: "Agents typically do not invoke this directly."
  - name: Key-Value Store
    slug: key-value-store
    category: Platform
    docs_url: https://www.quicknode.com/docs/key-value-store
    marketing_url: https://www.quicknode.com/docs/key-value-store
    skill_section: Key-Value Store
    skill_reference: references/kv-reference.md
    coverage: full
    notes: "Docs/product surface."
  - name: Quicknode SDK
    slug: quicknode-sdk
    category: Platform
    docs_url: https://www.quicknode.com/docs/quicknode-sdk
    marketing_url: https://www.quicknode.com/sdk
    skill_section: Quicknode SDK
    skill_reference: references/sdk-reference.md
    coverage: full
    notes: "Product API SDK, not a chain RPC wrapper."
  - name: x402
    slug: x402
    category: Agent Surface
    docs_url: https://www.quicknode.com/docs/build-with-ai
    marketing_url: https://www.quicknode.com/agents
    skill_section: x402
    skill_reference: references/x402-reference.md
    coverage: full
    notes: ""
  - name: MPP
    slug: mpp
    category: Agent Surface
    docs_url: https://www.quicknode.com/docs/build-with-ai
    marketing_url: https://www.quicknode.com/agents
    skill_section: MPP
    skill_reference: references/mpp-reference.md
    coverage: full
    notes: ""
  - name: Agent Subscriptions
    slug: agent-subscriptions
    category: Agent Surface
    docs_url: https://www.quicknode.com/docs/build-with-ai/agent-subscriptions
    marketing_url: https://www.quicknode.com/agents
    skill_section: Agent Subscriptions
    skill_reference: references/agent-subscriptions-reference.md
    coverage: full
    notes: ""
  - name: Quicknode CLI
    slug: quicknode-cli
    category: Agent Surface
    docs_url: https://www.quicknode.com/docs/quicknode-cli
    marketing_url: https://www.quicknode.com/cli
    skill_section: Quicknode CLI
    skill_reference: references/cli-reference.md
    coverage: full
    notes: "Use current singular command groups: endpoint, stream, webhook."
  - name: Quicknode MCP
    slug: quicknode-mcp
    category: Agent Surface
    docs_url: https://www.quicknode.com/docs/build-with-ai/quicknode-mcp
    marketing_url: https://www.quicknode.com/agents
    skill_section: Quicknode MCP
    skill_reference: references/mcp-reference.md
    coverage: full
    notes: "Available through native Claude/OpenAI surfaces and generic MCP clients."
  - name: Solana Validator
    slug: solana-validator
    category: Trading & DeFi
    docs_url: https://www.quicknode.com/docs
    marketing_url: https://www.quicknode.com/chains/solana/validator
    skill_section: Other Quicknode Products
    skill_reference: null
    coverage: link-out
    notes: "Human/operator provisioned."
  - name: Monad Validator
    slug: monad-validator
    category: Trading & DeFi
    docs_url: https://www.quicknode.com/docs
    marketing_url: https://www.quicknode.com/chains/monad/validator
    skill_section: Other Quicknode Products
    skill_reference: null
    coverage: link-out
    notes: "Human/operator provisioned."
```

## Future Sync Runbook

1. Compare Quicknode's current public product-page catalog with this file.
2. For each new or renamed page, copy the canonical product name and public URL.
3. Choose coverage:
   - `full` when an AI agent is likely to invoke the product directly.
   - `stub` when the agent should know the product exists but usually should not configure it directly.
   - `link-out` for staking, enterprise, or human-operated products.
   - `none` only with a note explaining why the product is intentionally excluded.
4. Update this catalog first, then propagate the change to `SKILL.md`, `README.md`, and the relevant reference file.
5. Keep product-specific references separate when a product has enough direct agent usage to justify its own file.
6. Verify CLI and SDK examples against the current docs before release:
   - https://www.quicknode.com/docs/quicknode-cli
   - https://www.quicknode.com/docs/quicknode-sdk
7. Bump `.claude-plugin/marketplace.json`:
   - minor version for new products or substantial restructuring
   - patch version for renames, wording fixes, and stale-example cleanup
8. Run local checks for feedback markers, stale CLI examples, and invented Backfills paths. Avoid copying marker strings into docs outside this runbook.

```bash
rg -n "<feedback-marker-pattern>" .
rg -n "<stale-cli-or-backfills-pattern>" skills/quicknode-skill README.md
```
