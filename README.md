# Universal Pen-Testing Toolkit

A reusable, structured methodology + automation toolkit for performing **authorized**
penetration testing against any target — **web apps, APIs, networks/hosts, source code (SAST),
and mobile/Flutter apps**.

Point it at a target, and it creates a self-contained *engagement*, runs the relevant scans,
and produces a clean severity-ranked report. The same toolkit is reused for every target.

> ⚠️ **Authorized testing only.** See [`AUTHORIZATION.md`](AUTHORIZATION.md). Only test systems
> you own or have **written permission** to test. Unauthorized testing is illegal in most
> jurisdictions (CFAA, Computer Misuse Act, and equivalents).

---

## 📂 What's in this repository (public)

This repo contains the **methodology and templates** — the reusable knowledge that drives the
testing. It's designed so anyone can follow the same process.

```
.
├── playbooks/            ← step-by-step testing methodology per target type
│   ├── web.md            ·  OWASP-aligned web app testing
│   ├── api.md            ·  REST/GraphQL (OWASP API Top 10)
│   ├── network.md        ·  host/service enumeration
│   ├── sast.md           ·  source code review
│   └── mobile.md         ·  Android/Flutter (+ traffic interception)
├── checklists/
│   ├── methodology.md    ·  end-to-end engagement phases
│   └── severity.md       ·  CVSS scoring + finding format
├── reports/_template.md  ← final report template
├── engagements/_template/← per-target workspace skeleton (scope, notes, findings)
├── AUTHORIZATION.md      ← rules-of-engagement / scope template
└── COMMANDS.md           ← command reference (for the local automation layer)
```

## 🔒 What's kept local (not published here)

The **automation layer is intentionally private** and excluded from this repo via
[`.gitignore`](.gitignore). If you're browsing this repo and don't see scripts — that's by design.
Kept local only:

| Component | Why it's local |
|---|---|
| `scan.ps1` (the launcher) + `scripts/*.ps1` | the actual scanners/automation |
| `.claude/` (Claude Code skills + settings) | AI-assisted testing helpers |
| `engagements/<target>/` (real engagement data) | sensitive: scopes, findings, loot |
| `wordlists/` | third-party wordlists |

So this public repo = **the playbook**. The automation that executes it stays on the operator's
machine.

---

## 🧭 The approach

Everything funnels toward the **same model**: discover the surface → probe it → confirm the
finding → record it. Each target type applies that loop differently:

| Target | What it checks |
|---|---|
| **Web / API** | security headers, exposed files, auth on endpoints, injection, CORS, TLS |
| **Network** | open ports + exposed services (databases, RDP, SMB…) — *owned servers only* |
| **Source (SAST)** | secrets, injection sinks, missing auth, vulnerable dependencies (CVEs) |
| **Mobile (APK)** | hardcoded secrets, API endpoints in the binary, manifest, pinning |

### The 3-layer coverage model
No single tool is complete. Real coverage = three layers:

1. **Self-contained scanners** — instant, zero-setup first pass (the bulk of common bugs).
2. **Tool-backed scanners** (nmap, nuclei, gitleaks, trivy, semgrep…) — deeper, more authoritative.
3. **Manual review + triage** — the human/AI layer for logic bugs, authenticated testing, and
   separating real findings from false positives. **This is the part automation can't do.**

> Automation *finds* candidates; a human *verifies* them. A scan is a list of leads, not a verdict.

---

## 🚀 Workflow (operator, with the local automation)

```
1. Start an engagement   → an isolated workspace (scope + notes + findings) is created
2. Run the relevant scan → web / hosted / source / mobile / network
3. Read the report       → severity-ranked Markdown in engagements/<target>/scans/
4. Triage + manual pass  → confirm real findings, do authenticated/logic testing
5. Compile the report    → from reports/_template.md
```

Operators: see [`COMMANDS.md`](COMMANDS.md) for the exact commands and the launcher menu.

## ✅ Principles baked in

- **Scope guard** — scans refuse to run against any target not in the engagement's scope file.
- **Authorized only** — every engagement requires a completed `AUTHORIZATION.md`.
- **Non-destructive by default** — read/observe over exploit; intrusive actions are gated.
- **Everything logged** — timestamped, reproducible, auditable output per engagement.
- **Managed-hosting aware** — network scanning is for servers you control (a VPS), **not**
  managed hosts (Vercel/Netlify/S3) where the infrastructure isn't yours.

## 📖 Start here

Read the methodology in [`playbooks/`](playbooks/), the engagement phases in
[`checklists/methodology.md`](checklists/methodology.md), and the scoring guide in
[`checklists/severity.md`](checklists/severity.md).
