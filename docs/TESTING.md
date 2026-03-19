# AgentSecrets SDK — Testing Guide

> How to run unit tests, integration tests, and examples.
> **[← Back to README](../README.md)**

---

Unit tests — no proxy, no network, no CLI required:

```bash
npm install
node --test --experimental-strip-types tests/unit/proxy.test.ts
```

Typecheck everything (src + tests + examples):

```bash
npx tsc --project tsconfig.dev.json
```

Integration tests — run manually against a live proxy before opening a PR.

**Prerequisites:**

```bash
# Ensure the CLI is installed and initialized — see Prerequisites above
# or the CLI docs at https://github.com/The-17/agentsecrets

# If you don't have a project yet:
agentsecrets project create test

agentsecrets secrets set TEST_KEY=any-value       # value doesn't matter
agentsecrets workspace allowlist add httpbin.org  # used as a safe echo target
agentsecrets proxy start
```

No specific workspace or project name is required — the test uses whatever is currently active.

```bash
node --experimental-strip-types tests/integration/integration.test.ts
```

Individual sections skip gracefully if prerequisites aren't met.

---


## Running the Examples

End-to-end scripts that verify the SDK works against real APIs with a live proxy.

**Setup:**

Make sure you have the AgentSecrets CLI installed and initialized — see the [Prerequisites](#prerequisites) section above or the [CLI docs](https://github.com/The-17/agentsecrets).

```bash
# If you haven't already: agentsecrets init → workspace create → project create
agentsecrets secrets set STRIPE_KEY=sk_live_...
agentsecrets secrets set OPENAI_KEY=sk-proj-...
agentsecrets secrets set GITHUB_TOKEN=ghp_...
agentsecrets proxy start   # keep running in a separate terminal
```

**Run:**

```bash
node --experimental-strip-types examples/stripe.ts   # bearer + form-encoded POST
node --experimental-strip-types examples/openai.ts   # bearer + JSON body POST
node --experimental-strip-types examples/github.ts   # bearer + custom headers
```

All three should exit cleanly and print status 200.

---