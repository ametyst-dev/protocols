# Competition

## Summary

The competitive landscape for agentic payments and agentic financial infrastructure is **fragmented and early**. Most players are exploratory and operate with different hypotheses (protocols, marketplaces, monetization layers, or banking R&D).

Consequences:
- No meaningful comparable KPIs yet
- Product boundaries are still fluid
- Competition is more about **thesis** than feature parity

## Key Competitors

### 1. Catena Labs - https://catenalabs.com/

**One-liner:** AI-native financial institution in R&D, exploring regulated banking redesign for agents.

- Exploring Agent Commerce Kits (ACKs) for AI identity, payments, policy-driven access, and guardrails for agent execution
- Has discussed potential future integrations with x402 and MCP, but these remain conceptual
- Founded by a former Circle executive; significant capital raised
- **Strengths:** Strong long-term vision; regulated-first mindset
- **Weaknesses:** No shipped product; no publicly communicated ICP or GTM strategy; pure R&D phase

### 2. Skyfire - https://skyfire.xyz/

**One-liner:** Protocol-centric platform for agent identity and payments via proprietary token standard.

- Designed a proprietary protocol for agent payments and identity with tokenized primitives (identity tokens, payment tokens)
- Enables sellers to recognize agent-originated payments and buyers to grant agents scoped access to funds
- Deep team experience from Ripple and crypto infrastructure
- **Strengths:** Active product with a clear technical thesis; two-sided (buyers and sellers); technology-first approach
- **Weaknesses:** Uses proprietary protocol rather than open payment primitives like x402; requires both sides to adopt the Skyfire standard; wallets assumed or ad-hoc; bank connectivity handled case-by-case; integrations often custom and consultative

### 3. Orthogonal - https://www.orthogonal.com/

**One-liner:** Agent-first API marketplace with a closed credit-based system.

- Users create an account, preload funds (typically via card), convert to internal credits, and connect the marketplace to agents through skills or MCP
- On the supply side, Orthogonal onboards API providers and acts as intermediary for pricing, access, and billing
- YC-backed; simple value prop ("one key gives you access to every API")
- **Strengths:** Pay-as-you-go model; simple marketplace UX
- **Weaknesses:** Closed ecosystem; credit-based abstraction (not real money); marketplace-centric, not a fintech or banking product; limited interoperability outside the marketplace; does not address general-purpose agentic payments

### 4. Nevermined - https://nevermined.ai/

**One-liner:** Monetization layer for pay-per-use access to AI services and APIs via facilitators.

- Provides facilitator servers that integrate with payment protocols (including x402) and abstract the complexity of processing payments, paywalls for APIs, and usage tracking/metering
- Seller-centric monetization + metering tooling
- **Strengths:** Integrates with open protocols (x402); focused monetization and usage tracking
- **Weaknesses:** Does not provide wallets for buyers; no demand-side acquisition or distribution/discovery layer; narrow scope (payments + tracking only); not a banking platform; no focus on agent spending UX

## How Ametyst Is Different

Ametyst is a **banking platform for agents**, designed to let them:
- **Spend autonomously with guardrails** (budgets, limits, policies)
- **Receive money programmatically**
- Operate within **compliant-ready, programmable execution accounts**

### Positioning Matrix

| Dimension | Catena Labs | Skyfire | Orthogonal | Nevermined | **Ametyst** |
|---|---|---|---|---|---|
| **Approach** | Regulated banking R&D | Proprietary protocol | Credit marketplace | Monetization layer | **Banking platform** |
| **Product status** | No shipped product | Active product | Active product | Active product | **MVP / demo** |
| **Payment standard** | TBD | Proprietary tokens | Internal credits | x402 integration | **x402 (open)** |
| **Wallet infrastructure** | TBD | Ad-hoc | N/A (credits) | None (buyer-side) | **Self-custodial** |
| **Two-sided** | Unknown | Yes | Yes (marketplace) | Seller-only | **Yes** |
| **Bank connectivity** | Planned | Case-by-case | Card-based | None | **Native (Iron)** |

## Strategic Note (Europe)

Ametyst is designed Europe-first, with European compliance assumptions (MiCA, GDPR, AI Act) as a structural moat. Many competitors are US-first and may not prioritize EU constraints early. In comparisons, highlight **market focus** and **compliance by design** as long-term differentiators.


## Other competitors or startup working on the topic 
 
startups working on the topic
- https://paysponge.com/
- https://docs.atxp.ai/agents
- https://www.natural.co/
- https://bankr.bot/
- https://www.ampersend.ai/
- https://www.starkbot.ai/
- https://www.daydreams.systems/

+ "incumbents players" that have dumped something
- https://www.moonpay.com/agents
- https://www.coinbase.com/it/developer-platform/discover/launches/agentic-wallets
- https://www.lobster.cash/
- Stripe with machine payments

