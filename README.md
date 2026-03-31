# @proco/sdk

**Financial infrastructure for AI agents.** Wallets · payment policies · programmable spending · agent-to-agent settlement · treasury.

[![npm version](https://img.shields.io/npm/v/@proco/sdk.svg)](https://www.npmjs.com/package/@proco/sdk)
[![License: MIT](https://img.shields.io/badge/License-MIT-black.svg)](https://opensource.org/licenses/MIT)
[![x402 compatible](https://img.shields.io/badge/x402-native-green.svg)](https://procohq.com/x402)

---

## What is Proco?

AI agents are making decisions that cost real money — paying for APIs, buying compute, settling agent-to-agent tasks, and managing budgets across long-running workflows. Today none of that has financial infrastructure behind it.

Proco is the payment layer built purely for agents: programmable wallets, policy enforcement, x402-native payment execution, and full audit trails — without any of the overhead designed for humans.

## Installation

```bash
npm install @proco/sdk
# or
pnpm add @proco/sdk
```

## Quick start

```typescript
import { Proco } from '@proco/sdk'

const proco = new Proco({
  apiKey: process.env.PROCO_API_KEY
})

// Create a wallet for your agent
const wallet = await proco.wallets.create({
  agentId: 'research-agent-01',
  policies: {
    dailyCap: 50_00,           // $50.00 per day
    vendors:  ['openai.com', 'perplexity.ai', 'serper.dev'],
    currency: 'USDC'
  }
})

// Execute a payment — x402 handled automatically
const tx = await proco.payments.create({
  wallet: wallet.id,
  amount: 2_50,                // $2.50
  vendor: 'api.perplexity.ai',
  memo:   'market research query'
})

console.log(tx.status)        // 'settled'
console.log(tx.settlementMs)  // ~28000 (p99)
```

## Core concepts

### Agent wallets

Each agent (or agent role) gets its own wallet with its own policy set. Wallets are non-custodial: Proco never holds funds — it enforces policies and executes payments on behalf of the agent.

```typescript
const wallet = await proco.wallets.create({
  agentId: 'procurement-agent',
  policies: {
    dailyCap:    100_00,        // $100 / day hard limit
    perTx:       25_00,         // $25 / transaction max
    categories:  ['data', 'compute', 'research'],
    vendors:     ['approved-vendor.com'],
    allowUnknown: false         // reject unlisted vendors
  }
})
```

### Payment policies

Policies are evaluated in real time before every payment. If a payment would breach any policy rule, it is rejected before execution — no partial spends, no retroactive clawbacks.

```typescript
// Update policies at any time
await proco.wallets.updatePolicy(wallet.id, {
  dailyCap: 200_00,            // increase limit
  addVendors: ['new-api.com']
})
```

### x402 payments

x402 is the open HTTP standard for machine-to-machine payments. When an API returns HTTP 402, Proco intercepts the payment challenge, validates it against the agent's policy, and executes the payment — returning the settled request back to your agent.

```typescript
// Fetch any x402-protected endpoint — payment is automatic
const response = await proco.fetch('https://api.perplexity.ai/search', {
  wallet: wallet.id,
  body:   JSON.stringify({ query: 'latest AI news' })
})

const data = await response.json()
```

### Agent-to-agent settlement

When one agent commissions work from another, Proco handles the settlement atomically.

```typescript
const invoice = await proco.invoices.create({
  from: 'worker-agent-07',
  to:   'orchestrator-agent-01',
  amount: 5_00,
  description: 'PDF extraction task #4821'
})

await proco.invoices.settle(invoice.id, {
  wallet: orchestratorWallet.id
})
```

## Framework integrations

### LangChain

```typescript
import { ProcoPaymentTool } from '@proco/sdk/langchain'

const tools = [
  new ProcoPaymentTool({ wallet: wallet.id, proco })
]

const agent = await createOpenAIFunctionsAgent({ llm, tools, prompt })
```

## Environments

| Environment | Base URL | Notes |
|-------------|----------|-------|
| Sandbox     | `https://sandbox.api.procohq.com` | Free testnet USDC, no real money |
| Production  | `https://api.procohq.com`         | Real USDC on Base                |

```typescript
const proco = new Proco({
  apiKey: process.env.PROCO_API_KEY,
  env: 'sandbox'
})
```

## Contributing

PRs and issues are welcome. Please open an issue before submitting a large change.

1. Fork the repo
2. Create a feature branch: `git checkout -b feat/my-feature`
3. Commit your changes
4. Open a pull request

## License

MIT — see [LICENSE](./LICENSE)

---

Built by [Proco](https://procohq.com) · [x.com/procohq](https://x.com/procohq) · [LinkedIn](https://linkedin.com/company/procohq)
