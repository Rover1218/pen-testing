# API Pen-Test Playbook (REST / GraphQL)

Aligned with the **OWASP API Security Top 10**. Assumes you have the base URL and, ideally,
two test accounts at different privilege levels. Stay in scope; observe before exploiting.

## 1. Discover the API
- [ ] Find the spec — `/swagger.json`, `/openapi.json`, `/v2/api-docs`, GraphQL introspection.
- [ ] Enumerate versions (`/v1`, `/v2`, deprecated paths often less protected).
- [ ] Capture real traffic from the client app to learn undocumented endpoints.
- [ ] Import the spec into Postman/Burp to drive systematic testing.

## 2. OWASP API Top 10 coverage

### API1 — Broken Object Level Authorization (BOLA/IDOR)
- [ ] For every object ID, try accessing another account's object. Numeric, UUID, and
      encoded IDs all count. This is the #1 API bug.

### API2 — Broken Authentication
- [ ] Weak/!verified JWT (`alg:none`, weak secret, no exp), API keys in URLs, no rotation.
- [ ] Token accepted after logout; refresh-token abuse; auth on every endpoint?

### API3 — Broken Object Property Level Authorization
- [ ] **Excessive data exposure** — endpoint returns more fields than the UI shows (PII, hashes).
- [ ] **Mass assignment** — send extra fields (`role`, `isAdmin`, `verified`) and see if accepted.

### API4 — Unrestricted Resource Consumption
- [ ] Missing rate limits, no pagination caps, large/expensive queries, costly operations.

### API5 — Broken Function Level Authorization
- [ ] Call admin/privileged functions as a low-priv user; guess admin routes.

### API6 — Unrestricted Access to Sensitive Business Flows
- [ ] Automatable abuse of flows (signup, purchase, invite) lacking bot protection.

### API7 — Server-Side Request Forgery
- [ ] Any endpoint taking a URL/host — probe internal services & cloud metadata.

### API8 — Security Misconfiguration
- [ ] CORS, verbose errors/stack traces, missing security headers, unneeded HTTP methods.

### API9 — Improper Inventory Management
- [ ] Shadow/old API versions, debug/staging endpoints, undocumented hosts.

### API10 — Unsafe Consumption of 3rd-party APIs
- [ ] Trust boundaries with upstream APIs, injection via integrated services.

## GraphQL specifics
- [ ] Introspection enabled? (`__schema`) — map all types/queries/mutations.
- [ ] Query depth / complexity limits (DoS via nested queries), batching abuse, aliasing.
- [ ] Field-level authorization gaps; injection through resolver arguments.
- [ ] Tools: `graphw00f`, `clairvoyance`, `inql`, GraphQL Voyager.

## Tooling
| Goal | Tool |
|---|---|
| Drive endpoints | Postman, Burp, `httpie`/`curl` |
| Fuzz params/paths | `ffuf`, Burp Intruder |
| JWT | `jwt_tool`, jwt.io, `hashcat` |
| GraphQL | `graphw00f`, `inql`, `clairvoyance` |
| Templates | `nuclei` (exposures, misconfig) |
