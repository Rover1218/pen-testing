# Universal Pen-Testing Project

A reusable, structured toolkit + Claude-guided playbooks for performing **authorized**
penetration testing against any target — web apps, APIs, networks/hosts, source code (SAST),
and mobile/thick clients.

You point it at a new target, create an *engagement folder*, and follow the playbooks
(either driving the scripts yourself or having Claude Code walk you through them).
Everything from one engagement is self-contained, so the same project is reused for every target.

> ⚠️ **Authorized testing only.** Read [`AUTHORIZATION.md`](AUTHORIZATION.md) before touching
> any target. Only test systems you own or have **written permission** to test. Unauthorized
> testing is illegal in most jurisdictions.

> ℹ️ **Note on this repository.** The PowerShell automation scripts (`scripts/`, `tools/`) and the
> Claude Code skills (`.claude/`) are kept **local-only** and are intentionally not published here.
> This repo contains the methodology playbooks, checklists, and templates. Commands like
> `./scripts/recon/recon.ps1` referenced below run from the local copy of the toolkit.

---

## How it's organized

```
pen testing/
├── README.md              ← you are here
├── AUTHORIZATION.md       ← scope + rules-of-engagement template (fill out per engagement)
├── playbooks/             ← methodology: step-by-step testing guides per target type
│   ├── web.md
│   ├── api.md
│   ├── network.md
│   ├── sast.md
│   └── mobile.md
├── checklists/            ← coverage checklists (OWASP, recon, reporting)
├── scripts/               ← PowerShell automation wrapping real tools
│   ├── lib/common.ps1     ← shared helpers (logging, engagement paths, scope guard)
│   ├── new-engagement.ps1 ← scaffold a fresh engagement folder
│   ├── recon/recon.ps1
│   ├── network/portscan.ps1
│   ├── web/web-enum.ps1
│   ├── sast/sast-scan.ps1
│   └── mobile/flutter-scan.ps1  ← decompile APK, find secrets/endpoints (Flutter-aware)
├── tools/                 ← install + verify the external CLI tools
│   ├── install.ps1
│   └── check-tools.ps1
├── reports/_template.md   ← findings report template
├── engagements/           ← one folder per target (git-ignored; holds real scope data)
│   └── _template/
└── .claude/               ← Claude Code skills + permission settings
    ├── settings.json
    └── skills/
```

## Quickstart

### 1. One-time setup — install the tooling
```powershell
# from the project root, in PowerShell
./tools/install.ps1        # installs nmap, nuclei, ffuf, etc. via winget/scoop/choco
./tools/check-tools.ps1    # verifies what's available and prints versions
```

### 2. Start an engagement
```powershell
./scripts/new-engagement.ps1 -Name acme-web -Scope "app.acme.com,api.acme.com"
```
This creates `engagements/acme-web/` with a pre-filled scope file, notes, loot, and report
skeleton. **Fill out `engagements/acme-web/AUTHORIZATION.md` before running anything.**

### 3. Run / follow the playbook
Either drive the scripts directly:
```powershell
./scripts/recon/recon.ps1     -Engagement acme-web
./scripts/network/portscan.ps1 -Engagement acme-web
./scripts/web/web-enum.ps1     -Engagement acme-web
```
…or open Claude Code in this folder and say e.g. **`/pentest-web`** to be guided through the
web methodology against the engagement target, with Claude running and interpreting the tools.

### 4. Report
Findings accumulate in `engagements/<name>/findings/`. Generate the final report from
[`reports/_template.md`](reports/_template.md) (or ask Claude: *"compile the report for acme-web"*).

## The Claude skills

When you open this project in Claude Code, these guided playbooks are available:

| Command | What it does |
|---|---|
| `/pentest-recon`   | OSINT + asset discovery for the engagement target |
| `/pentest-web`     | Walk the web-app methodology (OWASP-aligned) |
| `/pentest-network` | Host/service enumeration and analysis |
| `/pentest-sast`    | Point Claude at another codebase to find vulns by reading the code |
| `/pentest-report`  | Compile findings into the report template |

## Principles baked in

- **Scope guard.** Scripts read the engagement's scope file and refuse to run against
  out-of-scope hosts. Don't bypass it.
- **Everything logged.** Each script timestamps its output under the engagement so the
  test is reproducible and auditable.
- **Non-destructive by default.** Playbooks favor read/observe over exploit; anything
  intrusive is called out and gated behind explicit confirmation.

See [`playbooks/`](playbooks/) for the actual methodology.
