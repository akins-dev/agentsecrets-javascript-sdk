# AgentSecrets SDK — Testing Guide

> How to run unit tests, integration tests, and examples.
> **[← Back to README](../README.md)**

---

## Unit Tests

No proxy, no network, no CLI required. Run these first.

```bash
npm install
npm test
# or directly:
node --experimental-strip-types tests/unit/proxy.test.ts
```

86 assertions covering all auth styles, all error classes, URL validation, header sanitisation, `AgentSecretsResponse`, and zero-knowledge structural guarantees.

---

## Integration Tests

Run manually against a live proxy before opening a PR.

### Prerequisites

```bash
# Ensure the CLI is installed and initialized
# Full setup guide: https://agentsecrets.theseventeen.co/docs

# If you don't have a project yet:
agentsecrets project create test

agentsecrets secrets set TEST_KEY=any-value       # value doesn't matter
agentsecrets workspace allowlist add httpbin.org  # used as a safe echo target
agentsecrets proxy start
```

No specific workspace or project name required — the test uses whatever is currently active.

### Run

```bash
npm run test:integration
# or directly:
node --experimental-strip-types tests/integration/integration.test.ts
```

Individual sections skip gracefully if prerequisites aren't met — you'll see `○ skipped` lines for anything that needs the proxy or a live secret.

### What it tests

| Section | Needs proxy? |
|---------|-------------|
| Imports & version | No |
| Client construction | No |
| Error quality | No |
| Response model | No |
| Header injection safety | No |
| Mock client | No |
| Live proxy calls (httpbin.org) | Yes |

---

## Examples

End-to-end scripts that verify the SDK works against real APIs.

### Prerequisites

```bash
# Ensure the CLI is installed and initialized
# Full setup guide: https://agentsecrets.theseventeen.co/docs

# If you don't have a project yet:
agentsecrets project create my-app

agentsecrets secrets set STRIPE_KEY=sk_live_...
agentsecrets secrets set OPENAI_KEY=sk-proj-...
agentsecrets secrets set GITHUB_TOKEN=ghp_...
agentsecrets proxy start   # keep running in a separate terminal
```

### Run

```bash
npm run examples:stripe   # bearer + form-encoded POST → Stripe API
npm run examples:openai   # bearer + JSON body POST → OpenAI API
npm run examples:github   # bearer + custom headers → GitHub API

# or directly:
node --experimental-strip-types examples/stripe.ts
node --experimental-strip-types examples/openai.ts
node --experimental-strip-types examples/github.ts
```

All three should exit cleanly and print `Status: 200`.

---

## Typechecking

```bash
npx tsc --project tsconfig.dev.json
```

Typechecks `src/`, `tests/`, and `examples/` together using `allowImportingTsExtensions`.
