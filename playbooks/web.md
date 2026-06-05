# Web Application Pen-Test Playbook

OWASP-aligned methodology for testing a web app. Work top-to-bottom; record findings in
`engagements/<name>/findings/` as you go. Stay inside the scope file. Prefer observe over
exploit unless the engagement's authorization level allows more.

## 0. Pre-flight
- [ ] Engagement `AUTHORIZATION.md` complete and target in scope.
- [ ] Note the base URL(s), tech stack guesses, and any provided test accounts.

## 1. Recon & mapping
- [ ] Passive: `whois`, DNS, certificate transparency (crt.sh), subdomains.
- [ ] Identify stack — headers, cookies, `whatweb`, Wappalyzer, error pages, favicon hash.
- [ ] Spider the app (Burp/ZAP) + content discovery: `ffuf` / `feroxbuster` for dirs & files.
- [ ] Find `robots.txt`, `sitemap.xml`, `.git/`, backups, `/api`, swagger/openapi docs.
- [ ] Map every input: params, headers, cookies, file uploads, websockets.
  - Script: `./scripts/web/web-enum.ps1 -Engagement <name>`

## 2. Configuration & deployment
- [ ] TLS config (`testssl.sh` / `sslscan`), HSTS, weak ciphers, cert validity.
- [ ] Security headers (CSP, X-Frame-Options, X-Content-Type-Options, Referrer-Policy).
- [ ] Default/admin interfaces, debug endpoints, verbose errors, exposed `.env`/config.
- [ ] CORS misconfig (reflected origin, `*` with credentials).
- [ ] HTTP methods (OPTIONS/PUT/DELETE/TRACE), clickjacking.

## 3. Authentication
- [ ] Username enumeration (login, register, reset differing responses/timing).
- [ ] Weak password policy; credential stuffing / brute-force protection (lockout, rate limit).
- [ ] Default creds; password reset flow (token entropy, reuse, host-header poisoning).
- [ ] MFA bypass; "remember me" tokens; SSO/OAuth flow flaws.

## 4. Session management
- [ ] Cookie flags (`HttpOnly`, `Secure`, `SameSite`), token entropy & predictability.
- [ ] Session fixation, session not invalidated on logout / password change.
- [ ] JWT issues — `alg:none`, weak secret (try `hashcat`), missing signature verification.

## 5. Authorization (often the highest-impact bugs)
- [ ] **IDOR / BOLA** — swap IDs between two accounts; access other users' objects.
- [ ] **Vertical privesc** — call admin functions as a normal user.
- [ ] **Horizontal privesc** — access peer-user data.
- [ ] Forced browsing to privileged pages; mass-assignment of role/permission fields.

## 6. Input validation / injection
- [ ] **SQLi** — error-based, boolean/time blind; confirm carefully with `sqlmap` (authorized).
- [ ] **XSS** — reflected, stored, DOM; test each output context.
- [ ] **Command / template injection** (SSTI), **XXE**, **LDAP/NoSQL injection**.
- [ ] **SSRF** — any URL/host parameter; check cloud metadata (169.254.169.254).
- [ ] **Path traversal / LFI/RFI**, **open redirect**, **CRLF / header injection**.
- [ ] **File upload** — type/extension bypass, content-type, path, stored XSS, webshell.

## 7. Business logic
- [ ] Workflow bypass (skip steps), negative quantities/prices, race conditions.
- [ ] Replay / idempotency issues, coupon/discount abuse, rate-limit on costly actions.

## 8. Client-side
- [ ] DOM XSS sinks, `postMessage` handlers, sensitive data in localStorage/JS.
- [ ] Third-party scripts / SRI, prototype pollution, exposed API keys in JS bundles.

## 9. API surface
- See [`api.md`](api.md) for REST/GraphQL specifics.

## Tooling cheat sheet
| Goal | Tool |
|---|---|
| Intercept proxy | Burp Suite / OWASP ZAP |
| Content discovery | `ffuf`, `feroxbuster`, `gobuster` |
| Vuln templates | `nuclei` |
| Stack fingerprint | `whatweb`, `nuclei -t technologies` |
| TLS | `testssl.sh`, `sslscan` |
| SQLi (authorized) | `sqlmap` |
| Subdomains | `subfinder`, `amass`, `crt.sh` |

## Logging findings
For each issue capture: title, severity (CVSS), affected URL/param, reproduction steps,
request/response evidence, impact, remediation. Drop one Markdown file per finding in
`engagements/<name>/findings/`.
