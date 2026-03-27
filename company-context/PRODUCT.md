# Ametyst Product Overview

Ametyst is a banking platform for the agentic economy that enables AI agents to spend money autonomously and allows digital services to receive money directly from agents.

## What is Ametyst?

Ametyst provides the financial infrastructure layer that sits between two emerging actors: agent owners (individuals and teams deploying AI agents) and service providers (SaaS companies and API providers). It solves a fundamental gap: AI agents cannot autonomously pay for and consume digital services. Current payment infrastructure is checkout-centric and human-in-the-loop — it breaks when autonomous software needs to transact at runtime.

### Job to be Done

**For agent owners:** Give my business a single control center to delegate financial authority to AI agents, monitor what they spend, and scale oversight across teams — the same way Brex gave companies control over employee spending, but for agents.

**For service providers (partners):** Let AI agents discover, consume, and pay for my services on a pay-per-use basis at runtime — without me building a proprietary payment system or wallet.

### Core Value Proposition

- **For agent owners:** Ametyst is the Brex for AI agents. As businesses deploy agents across teams and workflows, they need a way to delegate spending authority, enforce policies, and maintain visibility over what every agent does financially — without bottlenecking execution with manual approvals. Ametyst provides a financial control center where businesses can spin up wallets per agent, set granular guardrails (budgets, limits, time windows, allowed counterparties), monitor spending in real time, and audit agent activity. This makes human-in-the-loop scalable: instead of approving every transaction, humans define policies and the infrastructure enforces them — turning oversight from a bottleneck into a governance layer that scales with the number of agents deployed.

### Product Goal

Become the default banking and payment layer for agent-to-service and agent-to-agent transactions in Europe — the infrastructure through which agents spend and services get paid, capturing new transactional flows in the agentic economy.

## Core Features

### 1. Agent Wallets (Spend Side)

**What users can do:**
Connect a bank account, spin up virtual wallets for each agent, define spending policies (budgets, limits, time windows, allowed counterparties), and let agents spend autonomously within those guardrails. Tool-agnostic — works with any agent framework.

**Our differentiation:**
Self-custodial wallets — users control wallets directly via passkeys; Ametyst never holds custody of funds. Spending guardrails are enforced natively via smart contracts (not application-level rules), making them deterministic and tamper-proof.

**User experience / Flow:**
1. User connects bank account (fiat on-ramp via Iron)
2. Funds move to primary wallet (self-custodial, on Base blockchain, stablecoin-denominated)
3. User creates virtual wallets per agent with defined spending policies
4. Agent is connected to its wallet and can spend autonomously within guardrails

**Main use cases:**
- Agent accessing paid APIs during task execution
- Agent purchasing datasets or compute resources autonomously
- Budget-controlled multi-agent workflows with per-agent spending limits

### 2. Merchant Integration - Partners (Receive Side)

**What users can do:**
SaaS providers attach Ametyst SDKs to their service endpoints to make them payable by agents on a pay-per-use basis. No blockchain or payments expertise required. Services become discoverable and purchasable by any agent with an Ametyst wallet.

**Our differentiation:**
Uses the x402 open payment protocol — an HTTP-native protocol designed specifically for agent-to-service payments. No proprietary token standards; any agent with a compatible wallet can pay. Frictionless SDK integration for service providers.

**User experience / Flow:**
1. Service provider integrates Ametyst SDK into their backend endpoints
2. SDK handles pricing, payment verification, and settlement automatically
3. When an agent requests a service, payment happens inline at runtime (no checkout, no human approval)
4. Service provider receives payment in their Ametyst account

**Main use cases:**
- API provider monetizing usage by AI agents
- SaaS platform offering pay-per-use pricing for agent consumption
- Data provider selling access to datasets at runtime

### 3. Financial Programmability

**What users can do:**
Leverage smart contract-based wallets for conditional execution, time-based spending rules, automated reconciliation, and deterministic financial behavior. All financial logic is programmable and enforceable at the infrastructure level.

**Our differentiation:**
Native blockchain programmability via smart contracts — not application-level business logic. This enables features impossible with traditional payment rails: atomic conditional payments, automated multi-party settlement, real-time spending controls that cannot be bypassed.

**User experience / Flow:**
Built into the wallet and SDK layer — users set policies and the infrastructure enforces them. No separate configuration needed.

**Main use cases:**
- Conditional payments (pay only if service returns valid output)
- Time-boxed budgets (agent can spend X per day/week/month)
- Automated reconciliation across multi-agent workflows

## Technology Stack

- **Wallet infrastructure:** Self-custodial wallets via ZeroDev (account abstraction)
- **Bank connectivity:** Iron (fiat on/off-ramp, KYC handled by regulated partner)
- **Blockchain:** Base
- **Stablecoin:** Circle (USDC/EURC)
- **Payment protocol:** x402 (open, HTTP-native agent payment protocol)
- **Agent integration:** Skills, MCP (Model Context Protocol)

## Compliance Model

Ametyst is infrastructure, not a custodian. Wallets are self-custodial. Fiat ramps and KYC are handled by regulated third-party providers. Ametyst is structured as a technology provider and expects to play an active role in shaping compliance standards for agent-native finance in Europe.
