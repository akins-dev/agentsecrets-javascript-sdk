# AgentSecrets SDK — Working Examples

> Complete examples covering every feature of `@the-17/agentsecrets-sdk`.
> **[← Back to README](../README.md)**

---

## Understanding TypeScript Generics in This SDK

Throughout this SDK you'll see calls like:

```typescript
const response = await client.call<{ login: string; public_repos: number }>({
  url: "https://api.github.com/user",
  bearer: "GITHUB_TOKEN",
});
```

The `<{ login: string; public_repos: number }>` part is a **TypeScript generic type parameter** — it tells the SDK what shape you expect the API to return. It has no effect at runtime. Its purpose is purely to give you type safety on `response.json()`:

```typescript
// Without the generic — TypeScript doesn't know the shape
const response = await client.call({ url: "...", bearer: "KEY" });
const data = response.json(); // type: unknown — no autocomplete, no type checking

// With the generic — TypeScript knows exactly what's in the response
const response = await client.call<{ login: string; public_repos: number }>({
  url: "...",
  bearer: "KEY",
});
const data = response.json(); // type: { login: string; public_repos: number }
data.login;        // ✅ TypeScript knows this exists
data.public_repos; // ✅ TypeScript knows this exists
data.nonexistent;  // ❌ TypeScript error — doesn't exist on the type
```

You can always omit it if you don't need type safety on the response:

```typescript
// This is valid — response.json() returns unknown
const response = await client.call({ url: "...", bearer: "KEY" });
```

---


---


---

## Full Working Example

This is the complete picture — every feature of the SDK in one flow.

```typescript
import {
  AgentSecrets,
  AgentSecretsNotRunning,
  SecretNotFound,
  DomainNotAllowed,
} from "@the-17/agentsecrets-sdk";
import { MockAgentSecrets } from "@the-17/agentsecrets-sdk/testing";

// ── 1. Instantiate ────────────────────────────────────────────────────────────

const client = new AgentSecrets();
// Options:
// const client = new AgentSecrets({
//   port: 9000,         // custom proxy port (default: 8765 or AGENTSECRETS_PORT)
//   autoStart: false,   // don't auto-start proxy if not running (default: true)
//   workspace: "prod",  // active workspace (default: AGENTSECRETS_WORKSPACE)
//   project: "my-app",  // active project   (default: AGENTSECRETS_PROJECT)
// });


// ── 2. Health check ───────────────────────────────────────────────────────────

const running = await client.isProxyRunning(); // true | false
const status  = await client.proxyStatus();
// → { running: true, port: 8765, project: "my-app" }

if (!running) {
  console.error("Run: agentsecrets proxy start");
  process.exit(1);
}


// ── 3. Bearer token call ──────────────────────────────────────────────────────
// Used by: Stripe, OpenAI, GitHub, and most REST APIs.
//
// The generic <{ available: ... }> tells TypeScript what shape to expect from
// response.json(). It has no effect at runtime — omit it if you don't need it.

const balance = await client.call<{ available: Array<{ amount: number; currency: string }> }>({
  url: "https://api.stripe.com/v1/balance",
  bearer: "STRIPE_KEY",   // key name — proxy resolves the real value
});

console.log(balance.statusCode);          // 200
console.log(balance.json().available);    // [{ amount: 10000, currency: "usd" }]
console.log(balance.text);               // raw response string
console.log(balance.durationMs);         // 212
console.log(balance.redacted);           // false


// ── 4. POST with JSON body ────────────────────────────────────────────────────
// Objects are JSON-serialised automatically. Content-Type is set to
// application/json unless you override it.

const completion = await client.call<{ choices: Array<{ message: { content: string } }> }>({
  url: "https://api.openai.com/v1/chat/completions",
  method: "POST",
  bearer: "OPENAI_KEY",
  body: {
    model: "gpt-4o",
    messages: [{ role: "user", content: "Say hello." }],
    max_tokens: 20,
  },
});
console.log(completion.json().choices[0]?.message.content); // "Hello!"


// ── 5. POST with form-encoded body (Stripe, OAuth) ────────────────────────────
// String bodies are forwarded verbatim — not JSON-serialised.
// Set Content-Type manually when using non-JSON formats.

const customer = await client.call<{ id: string; email: string }>({
  url: "https://api.stripe.com/v1/customers",
  method: "POST",
  bearer: "STRIPE_KEY",
  headers: { "Content-Type": "application/x-www-form-urlencoded" },
  body: "email=user@example.com&description=SDK+test", // string — sent verbatim
});
console.log(customer.json().id); // "cus_..."


// ── 6. Custom header injection (SendGrid, AWS API Gateway) ────────────────────
// header is a Record<string, string> — key is the header name, value is the
// secret key name. The proxy injects: X-Api-Key: <resolved value>

const email = await client.call({
  url: "https://api.sendgrid.com/v3/mail/send",
  method: "POST",
  header: { "X-Api-Key": "SENDGRID_KEY" },
  body: {
    personalizations: [{ to: [{ email: "user@example.com" }] }],
    from: { email: "you@yourdomain.com" },
    subject: "Hello",
    content: [{ type: "text/plain", value: "Hello." }],
  },
});


// ── 7. Query parameter injection (Google Maps, weather APIs) ─────────────────
// query is a Record<string, string> — key is the param name, value is the
// secret key name. The proxy injects: ?key=<resolved value>

const geocode = await client.call({
  url: "https://maps.googleapis.com/maps/api/geocode/json?address=New+York",
  query: { key: "GMAP_KEY" },
});


// ── 8. Basic auth injection (Jira, legacy REST) ───────────────────────────────
// basic is a string — the secret key name holding "username:password".
// The proxy injects: Authorization: Basic <base64(username:password)>

const issue = await client.call({
  url: "https://yourorg.atlassian.net/rest/api/2/issue/PROJ-1",
  basic: "JIRA_CREDS",
});


// ── 9. Body field injection ───────────────────────────────────────────────────
// bodyField is a Record<string, string> — key is the JSON field path,
// value is the secret key name. The proxy injects the value into the body.

const dbResult = await client.call({
  url: "https://api.example.com/v1/query",
  method: "POST",
  bodyField: { api_key: "MY_KEY" },
  body: { query: "SELECT *" },
});


// ── 10. Form field injection ──────────────────────────────────────────────────
// Like bodyField but for form-encoded bodies.

const formResult = await client.call({
  url: "https://api.example.com/v1/auth",
  method: "POST",
  formField: { token: "SERVICE_TOKEN" },
});


// ── 11. Custom request headers ────────────────────────────────────────────────
// headers are forwarded as-is alongside the injected credential headers.
// Use for non-auth headers: Accept, API version, content negotiation, etc.

const github = await client.call<{ login: string; public_repos: number }>({
  url: "https://api.github.com/user",
  bearer: "GITHUB_TOKEN",
  headers: {
    "Accept": "application/vnd.github+json",
    "X-GitHub-Api-Version": "2022-11-28",
  },
});


// ── 12. Agent ID for audit logging ────────────────────────────────────────────
// Attach an identifier to a call so it appears in the proxy audit log.
// Useful when multiple agents share the same proxy.

const auditedCall = await client.call({
  url: "https://api.stripe.com/v1/balance",
  bearer: "STRIPE_KEY",
  agentId: "billing-agent-v2",
});


// ── 13. Per-call timeout ──────────────────────────────────────────────────────

const slowCall = await client.call({
  url: "https://api.example.com/slow-endpoint",
  bearer: "MY_KEY",
  timeout: 60_000, // 60 seconds for this call only
});


// ── 14. One-shot CLI call via spawn() ────────────────────────────────────────
//
// spawn() wraps `agentsecrets call` directly, passing your command array as
// CLI flags. The CLI resolves credentials from the OS keychain without ever
// exposing the values. Prefer client.call() for most use cases — spawn() is
// for scripting contexts or when you need the raw CLI output.

// Equivalent to: agentsecrets call --url https://api.stripe.com/v1/balance --bearer STRIPE_KEY
const curlResult = await client.spawn({
  command: ["--url", "https://api.stripe.com/v1/balance", "--bearer", "STRIPE_KEY"],
});
console.log(curlResult.exitCode); // 0
console.log(curlResult.stdout);   // Stripe JSON response

// POST with body via CLI
const postResult = await client.spawn({
  command: [
    "--url", "https://api.openai.com/v1/chat/completions",
    "--method", "POST",
    "--bearer", "OPENAI_KEY",
    "--body", JSON.stringify({ model: "gpt-4o-mini", messages: [{ role: "user", content: "Hi" }] }),
  ],
});


// ── 15. Temporary workspace/project switch ────────────────────────────────────
// withWorkspace() and withProject() switch context, run your function,
// then restore the previous context automatically.

await client.withWorkspace("production", async () => {
  const prodBalance = await client.call({
    url: "https://api.stripe.com/v1/balance",
    bearer: "STRIPE_KEY_PROD",
  });
  console.log("Prod balance:", prodBalance.json());
});
// Workspace restored to previous after the block exits


// ── 16. Error handling ────────────────────────────────────────────────────────

try {
  await client.call({ url: "https://api.stripe.com/v1/balance", bearer: "STRIPE_KEY" });
} catch (err) {
  if (err instanceof AgentSecretsNotRunning) {
    // err.port    → 8765
    // err.fixHint → "agentsecrets proxy start"
    console.error(err.message);

  } else if (err instanceof SecretNotFound) {
    // err.key     → "STRIPE_KEY"
    // err.fixHint → "agentsecrets secrets set STRIPE_KEY=VALUE"
    console.error(err.message);

  } else if (err instanceof DomainNotAllowed) {
    // err.domain  → "api.stripe.com"
    // err.fixHint → "agentsecrets workspace allowlist add api.stripe.com"
    console.error(err.message);
  }
}


// ── 17. Mock client for testing ───────────────────────────────────────────────
// MockAgentSecrets is a drop-in replacement for AgentSecrets in unit tests.
// It records every call() and spawn() without touching the proxy or keychain.

const mock = new MockAgentSecrets();
await mock.call({ url: "https://api.stripe.com/v1/balance", bearer: "STRIPE_KEY" });

console.log(mock.calls.length);    // 1
console.log(mock.calls[0]?.url);   // "https://api.stripe.com/v1/balance"
console.log(mock.calls[0]?.bearer); // "STRIPE_KEY" — key name only, never value

// Custom mock response:
// const mock = new MockAgentSecrets({
//   defaultResponse: new AgentSecretsResponse({
//     statusCode: 200,
//     headers: {},
//     body: new TextEncoder().encode('{"balance": 100}'),
//   }),
// });
```

---


---