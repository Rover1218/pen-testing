# Authorization & Rules of Engagement (Master Template)

> Copy this file into each engagement folder (`engagements/<name>/AUTHORIZATION.md`) and fill
> it out **before** running any tool against the target. A script or playbook should not be
> run until this is complete for the engagement.

## ⚠️ Legal notice

Penetration testing without explicit, written authorization from the system owner is illegal
in most jurisdictions (e.g., the Computer Fraud and Abuse Act in the US, the Computer Misuse
Act in the UK, and equivalents elsewhere). This project is for **authorized** testing only:
systems you own, lab/CTF environments, or engagements where you hold signed permission.

If you cannot tick every box in the checklist below, **stop**.

## Engagement details

| Field | Value |
|---|---|
| Engagement name | |
| Client / system owner | |
| Authorizing contact (name, role, email) | |
| Date authorization granted | |
| Test window (start → end) | |
| Authorization reference (contract / ticket / email) | |

## Scope

**In scope** (hosts, IP ranges, domains, apps, repos):
```
# one per line, e.g.
# app.example.com
# 203.0.113.0/24
# https://github.com/example/repo
```

**Explicitly OUT of scope:**
```
# production databases, third-party SaaS, employee endpoints, etc.
```

## Rules of engagement

- [ ] Written authorization obtained and on file (reference above).
- [ ] Scope confirmed in writing with the authorizing contact.
- [ ] Testing window agreed; testing only happens inside it.
- [ ] Allowed activity level agreed (recon-only / non-destructive / full exploit).
- [ ] Denial-of-service / load testing is **excluded** unless explicitly authorized here: ____
- [ ] Social engineering / phishing is **excluded** unless explicitly authorized here: ____
- [ ] Data handling agreed — no exfiltration of real PII beyond proof-of-access screenshots.
- [ ] Emergency contact + stop procedure agreed (who to call, how to halt).

## Allowed activity level (pick one)

- [ ] **Recon only** — passive + light enumeration, no exploitation.
- [ ] **Non-destructive** — enumeration + safe vuln verification, no data modification or DoS.
- [ ] **Full** — exploitation permitted within scope, still no DoS unless ticked above.

## Stop conditions

Halt immediately and notify the emergency contact if you observe: production outage,
evidence of a pre-existing compromise, exposure of sensitive third-party data, or anything
outside the agreed scope.

---

**Tester:** ________________   **Signature/confirmation:** ________   **Date:** ________
