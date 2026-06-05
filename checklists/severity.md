# Severity & Scoring Guide

Use CVSS v3.1 for scoring; map to these bands for reporting consistency.

| Severity | CVSS | Meaning | Example |
|---|---|---|---|
| **Critical** | 9.0–10.0 | Full compromise / mass data exposure, trivially exploitable | Unauth RCE, SQLi dumping all data, auth bypass to admin |
| **High** | 7.0–8.9 | Significant impact, realistic exploit | Stored XSS in admin, IDOR exposing PII, SSRF to internal |
| **Medium** | 4.0–6.9 | Limited impact or needs conditions | Reflected XSS, CSRF on sensitive action, info disclosure |
| **Low** | 0.1–3.9 | Minor / defense-in-depth | Missing security headers, verbose errors, weak TLS ciphers |
| **Info** | 0.0 | No direct risk; hardening advice | Tech version disclosure, best-practice notes |

## Each finding must include
1. **Title** — clear and specific.
2. **Severity + CVSS vector** — e.g. `AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H`.
3. **Affected asset** — URL / host / file:line / endpoint.
4. **Description** — what the issue is.
5. **Reproduction steps** — exact, repeatable; include request/response or PoC.
6. **Impact** — what an attacker gains.
7. **Remediation** — concrete fix, ideally with code/config example.
8. **References** — OWASP, CWE, CVE links.

CVSS calculator: https://www.first.org/cvss/calculator/3.1
