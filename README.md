# AgentSecrets — JavaScript/TypeScript SDK

> Zero-Knowledge Secrets for AI Agents

[![npm](https://img.shields.io/npm/v/@the-17/agentsecrets-sdk)](https://www.npmjs.com/package/@the-17/agentsecrets-sdk)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Node 18+](https://img.shields.io/badge/node-18+-339933?logo=node.js&logoColor=white)](https://nodejs.org/)

**Website:** [agentsecrets.theseventeen.co](https://agentsecrets.theseventeen.co) &nbsp;|&nbsp;
**Docs:** [agentsecrets.theseventeen.co/docs](https://agentsecrets.theseventeen.co/docs) &nbsp;|&nbsp;
**Blog:** [engineering.theseventeen.co](https://engineering.theseventeen.co) &nbsp;|&nbsp;
**CLI:** [github.com/The-17/agentsecrets](https://github.com/The-17/agentsecrets) &nbsp;|&nbsp;
**Built by:** [theseventeen.co](https://theseventeen.co) &nbsp;|&nbsp;
**Security:** [hello@theseventeen.co](mailto:hello@theseventeen.co)

---

## What This Is

A TypeScript SDK that lets your code make authenticated API calls without ever holding a credential value. You pass a key name. The proxy resolves the real value from your OS keychain, injects it into the outbound request, and returns only the API response. The value never enters your process.

No `.env` files. No `process.env`. No `vault.get()`. Just: make the call.

---

## Prerequisites

You need the AgentSecrets CLI installed and a proxy running before using this SDK.
Full setup guide: [agentsecrets.theseventeen.co/docs](https://agentsecrets.theseventeen.co/docs)

```bash
# 1. Install the CLI (pick one)
pip install agentsecrets
npm install -g @the-17/agentsecrets
curl -sSL https://get.agentsecrets.com | sh

# 2. Create your account (first time only)
agentsecrets init

# 3. Create a workspace if you don't have one, or skip if one already exists
agentsecrets workspace create my-workspace

# 4. Create a project
agentsecrets project create my-app

# 5. Add your secrets (the one time values enter your terminal)
agentsecrets secrets set STRIPE_KEY=sk_live_...
agentsecrets secrets set OPENAI_KEY=sk-proj-...
agentsecrets secrets set GITHUB_TOKEN=ghp_...

# 6. Start the proxy (keep this running in a separate terminal)
agentsecrets proxy start
# → Proxy running at localhost:8765
```

---

## Install

```bash
npm install @the-17/agentsecrets-sdk
```

---

## Quick Start

```typescript
import { AgentSecrets } from "@the-17/agentsecrets-sdk";

const client = new AgentSecrets();

const response = await client.call<{ available: Array<{ amount: number; currency: string }> }>({
  url: "https://api.stripe.com/v1/balance",
  bearer: "STRIPE_KEY",   // key name — proxy resolves the real value
});

console.log(response.statusCode);       // 200
console.log(response.json().available); // [{ amount: 10000, currency: "usd" }]
// The real value of STRIPE_KEY never entered this process.
```

---

## Examples

See [docs/EXAMPLES.md](./docs/EXAMPLES.md) for a complete walkthrough of every feature — all auth styles, error handling, mock client, workspace switching, and more.

---

## Documentation

| | |
|---|---|
| [Tutorial](./TUTORIAL.md) | Step-by-step: from zero to your first zero-knowledge API call |
| [Examples](./docs/EXAMPLES.md) | Every feature in one place — all auth styles, error handling, mock client |
| [API Reference](./docs/API.md) | All methods, options, response shape, error classes, TypeScript types |
| [Proxy Protocol](./docs/PROXY-PROTOCOL.md) | How `client.call()` communicates with the proxy |
| [Testing Guide](./docs/TESTING.md) | Running unit tests, integration tests, and examples |

---

## Further Reading

- [AgentSecrets CLI docs](https://agentsecrets.theseventeen.co/docs)
- [Engineering Blog](https://engineering.theseventeen.co)
- [GitHub — The-17/agentsecrets](https://github.com/The-17/agentsecrets)
- [The Seventeen](https://theseventeen.co)

---

MIT License — Built by [The Seventeen](https://theseventeen.co)

---

**The agent operates it. The agent never sees it.**