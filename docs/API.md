# AgentSecrets SDK — API Reference

> Full reference for `@the-17/agentsecrets-sdk`.
> **[← Back to README](../README.md)**

---

### `new AgentSecrets(config?)`

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `port` | `number` | `8765` or `AGENTSECRETS_PORT` | Proxy port |
| `autoStart` | `boolean` | `true` | Auto-start proxy if not running |
| `workspace` | `string` | `AGENTSECRETS_WORKSPACE` | Active workspace |
| `project` | `string` | `AGENTSECRETS_PROJECT` | Active project |

---

### `client.call<T>(opts): Promise<AgentSecretsResponse<T>>`

`T` is an optional TypeScript generic — it describes the shape of the API response body. Providing it gives you type safety on `response.json()`. Omitting it makes `response.json()` return `unknown`.

```typescript
// With generic — full type safety
const res = await client.call<{ id: string }>({ url: "...", bearer: "KEY" });
res.json().id; // ✅ TypeScript knows .id exists

// Without generic — still works, json() returns unknown
const res = await client.call({ url: "...", bearer: "KEY" });
```

| Option | Type | Required | Description |
|--------|------|----------|-------------|
| `url` | `string` | ✅ | Target API endpoint |
| `bearer` | `string` | one auth | Secret key name for `Authorization: Bearer` |
| `basic` | `string` | one auth | Secret key name holding `user:pass` |
| `header` | `Record<string, string>` | one auth | `{ "Header-Name": "KEY_NAME" }` |
| `query` | `Record<string, string>` | one auth | `{ "param": "KEY_NAME" }` |
| `bodyField` | `Record<string, string>` | one auth | `{ "field": "KEY_NAME" }` |
| `formField` | `Record<string, string>` | one auth | `{ "field": "KEY_NAME" }` |
| `method` | `string` | — | HTTP method. Default: `GET` |
| `headers` | `Record<string, string>` | — | Non-auth request headers |
| `body` | `unknown` | — | Objects → JSON-serialised. Strings → verbatim |
| `agentId` | `string` | — | Agent identifier for audit log |
| `port` | `number` | — | Per-call proxy port override |
| `timeout` | `number` | `30000` | Per-call timeout in ms |

**`AgentSecretsResponse<T>` properties:**

| Property | Type | Description |
|----------|------|-------------|
| `statusCode` | `number` | HTTP status from target API |
| `headers` | `Record<string, string>` | Response headers |
| `body` | `Uint8Array` | Raw response bytes |
| `redacted` | `boolean` | Whether proxy redacted part of the response |
| `durationMs` | `number` | Round-trip duration in ms |
| `text` | `string` | Response body decoded as UTF-8 |
| `json()` | `T` | Response body parsed as JSON |

---

### `client.spawn(opts): Promise<SpawnResult>`

A thin wrapper around `agentsecrets call` — passes your command array directly as flags to the CLI. The CLI resolves credentials from the OS keychain and injects them into the request without ever exposing the values.

For most use cases, prefer `client.call()` which handles flag construction automatically. `spawn()` is useful in scripting contexts or when you need the raw CLI output.

```typescript
// Equivalent to: agentsecrets call --url https://api.stripe.com/v1/balance --bearer STRIPE_KEY
const result = await client.spawn({
  command: ["--url", "https://api.stripe.com/v1/balance", "--bearer", "STRIPE_KEY"],
});
console.log(result.stdout); // Stripe JSON response
```

| Option | Type | Required | Description |
|--------|------|----------|-------------|
| `command` | `string[]` | ✅ | Flags forwarded to `agentsecrets call` (e.g. `["--url", "...", "--bearer", "KEY"]`) |
| `capture` | `boolean` | — | Capture stdout/stderr. Default: `true` |
| `timeout` | `number` | — | Timeout in ms |

**Returns `SpawnResult`:** `{ exitCode: number, stdout: string, stderr: string }`

Exit code conventions: `0` = success, `1` = process error, `124` = timed out, `127` = `agentsecrets` binary not found.

---

### `client.withWorkspace<T>(name, fn): Promise<T>`

Switches to `name`, runs `fn()`, then restores the previous workspace. Restores even if `fn()` throws.

### `client.withProject<T>(name, fn): Promise<T>`

Same pattern for projects.

---

### `client.isProxyRunning(): Promise<boolean>`

Runs `agentsecrets proxy status` and returns `true` if the exit code is 0.

### `client.proxyStatus(): Promise<ProxyStatus>`

Returns `{ running: boolean, port: number, project?: string }`.

---

### Mock Client for Testing

```typescript
import { MockAgentSecrets } from "@the-17/agentsecrets-sdk/testing";
```

Drop-in replacement for `AgentSecrets` in unit tests. Records every `call()` and `spawn()` without touching the proxy or keychain.

```typescript
const mock = new MockAgentSecrets();
await mock.call({ url: "https://api.stripe.com/v1/balance", bearer: "STRIPE_KEY" });

assert(mock.calls.length === 1);
assert(mock.calls[0].bearer === "STRIPE_KEY"); // key name — never value
```

Constructor options: `defaultResponse` (custom `AgentSecretsResponse`), `defaultSpawnResult` (custom `SpawnResult`).

---

### Error Classes

All errors extend `AgentSecretsError`. Every error exposes `.message` (human-readable, includes the CLI command to fix the problem) and `.fixHint` (just the command, machine-readable).

| Class | When thrown | `.fixHint` |
|-------|-------------|------------|
| `AgentSecretsNotRunning` | Proxy not running | `agentsecrets proxy start` |
| `ProxyConnectionError` | Can't connect to proxy | `agentsecrets proxy start` |
| `SecretNotFound` | Key name not in keychain | `agentsecrets secrets set KEY=VALUE` |
| `DomainNotAllowed` | Target domain not allowlisted | `agentsecrets workspace allowlist add domain` |
| `UpstreamError` | Target API returned an error | — |
| `CLINotFound` | `agentsecrets` binary not on PATH | Install link |
| `CLIError` | CLI command returned non-zero | — |
| `SessionExpired` | Session token expired | `agentsecrets login` |
| `PermissionDenied` | Insufficient role for operation | — |
| `WorkspaceNotFound` | Workspace doesn't exist | `agentsecrets workspace list` |
| `ProjectNotFound` | Project doesn't exist | `agentsecrets project list` |
| `AllowlistModificationDenied` | Only admins can modify allowlist | — |

---

### TypeScript Types

All types exported from the main entry point. No `@types/` package needed.

```typescript
import type {
  CallOptions,
  SpawnOptions,
  SpawnResult,
  ProxyStatus,
} from "@the-17/agentsecrets-sdk";

import { AgentSecretsResponse } from "@the-17/agentsecrets-sdk";
```

> **Note:** `AgentSecretsResponse` is a class (not just a type) because it has `.text` and `.json()` methods. Import it as a value, not just as a type.

---