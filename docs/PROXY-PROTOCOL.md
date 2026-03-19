# AgentSecrets — Proxy Protocol

> How `client.call()` communicates with the local AgentSecrets proxy.
> **[← Back to README](../README.md)**

---

Every `client.call()` sends an HTTP request to `localhost:8765/proxy` with `X-AS-*` headers. The proxy reads them, resolves values from the OS keychain, injects them into the forwarded request, and returns only the API response.

| Header | Auth Style | Format |
|--------|-----------|--------|
| `X-AS-Target-URL` | All | Target endpoint URL |
| `X-AS-Method` | All | HTTP method |
| `X-AS-Inject-Bearer` | `bearer` | Secret key name |
| `X-AS-Inject-Basic` | `basic` | Secret key name |
| `X-AS-Inject-Header-{Name}` | `header` | Secret key name (header name is the suffix) |
| `X-AS-Inject-Query-{param}` | `query` | Secret key name (param name is the suffix) |
| `X-AS-Inject-Body-{path}` | `bodyField` | Secret key name (field path is the suffix) |
| `X-AS-Inject-Form-{key}` | `formField` | Secret key name (field name is the suffix) |
| `X-AS-Agent-ID` | All (optional) | Agent identifier for audit log |

The proxy error codes: `403` = domain not in allowlist, `502` = all engine errors (including secret not found — distinguished by inspecting the error message body).

---
