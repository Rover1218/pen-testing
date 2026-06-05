# Commands Cheatsheet

Every command you need, copy-paste ready. Run all of these from the project root:
`cd "c:\Users\Anindya\Desktop\pen testing"`

> All the `*-audit` scripts are **self-contained** (pure PowerShell, no tools to install) and
> **write a Markdown report** automatically. Reports land in
> `engagements/<name>/scans/<phase>-<timestamp>/report-*.md`.

---

## 0. One-time setup (optional, only for the deeper tools)
```powershell
./tools/install.ps1        # installs nmap, semgrep, gitleaks, etc. (NOT needed for *-audit scripts)
./tools/check-tools.ps1    # shows which tools are installed
```

## 1. Start an engagement (once per target)
```powershell
./scripts/new-engagement.ps1 -Name myapp -Scope "localhost,127.0.0.1"
# then fill in engagements/myapp/AUTHORIZATION.md (for your own apps: just note "self-owned")
```

## 2. The self-contained auditors (turn on the app → run → read the report)

### Web app / API  (app must be running)
```powershell
./scripts/web/web-audit.ps1 -Url http://localhost:3000 -Source "C:\path\to\app" -Engagement myapp
```
- Live: security headers, exposed files (`.env`, `.git`), secrets in HTML, framework disclosure
- Static (`-Source`): unauthenticated write endpoints, mass assignment, `dangerouslySetInnerHTML`, token-in-URL
- Drop `-Source` for live-only.

### Source code (any language)
```powershell
./scripts/sast/code-audit.ps1 -Source "C:\path\to\repo" -Engagement myapp
```
- Secrets, eval/exec, SQL injection, unsafe deserialization, SSRF, disabled TLS verify, weak crypto,
  committed `.env`, missing-auth API routes.

### Network / hosts  (scope-guarded)
```powershell
./scripts/network/net-audit.ps1 -Engagement myapp
./scripts/network/net-audit.ps1 -Target 127.0.0.1 -Ports 80,443,3000,3306,6379
```
- TCP port scan + flags risky exposed services (databases, RDP, Telnet, Redis, Mongo...).

### Mobile (Flutter / Android APK)
```powershell
./scripts/mobile/flutter-scan.ps1 -Apk "C:\path\to\app.apk" -Engagement myapp
```
- Decompiles, pulls API endpoints + hardcoded secrets from `libapp.so`, checks manifest, writes a report.

## 3. The tool-backed scripts (need ./tools/install.ps1 first; raw output, no report)
```powershell
./scripts/recon/recon.ps1      -Engagement myapp     # subdomains, DNS, fingerprint (subfinder/whatweb)
./scripts/network/portscan.ps1 -Engagement myapp     # full nmap scan (-sV -sC)
./scripts/web/web-enum.ps1     -Engagement myapp -Url http://localhost:3000   # ffuf/nuclei
./scripts/sast/sast-scan.ps1   -Target "C:\path\to\repo" -Engagement myapp    # semgrep/gitleaks/osv-scanner
```

## 4. Read the report
The path is printed at the end of every audit. Open it with:
```powershell
code "engagements\myapp\scans\<folder>\report-<timestamp>.md"
```

---

## Scripts vs. Claude skills - what's the difference?

You have **two ways** to run any test. They do the same testing; the difference is who drives.

| | **Run the script yourself** (this file) | **Use a Claude skill** (`/pentest-web`, etc.) |
|---|---|---|
| Who runs it | You type the command | You ask Claude in this folder; Claude runs the scripts for you |
| Output | A Markdown report | Claude runs the same script **and explains** the findings, triages false positives, and does manual logic testing (IDOR, business logic) the scripts can't |
| Best for | Fast, repeatable self-service checks | Going deeper, understanding results, finding logic bugs |
| Setup | Just PowerShell | Open this project in Claude Code |

**They are not separate tools** - the skills literally call the same `scripts/*.ps1` you run by hand.
So your workflow can be either:

- **Solo:** `./scripts/web/web-audit.ps1 ...` → open the report. Done.
- **Guided:** open this folder in Claude Code → type `/pentest-web` → Claude runs the audit, reads
  the report with you, verifies findings, and hunts the logic bugs automation misses.

The Claude skills live in `.claude/skills/` (kept local, not on GitHub):
| Command | Purpose |
|---|---|
| `/pentest-recon`   | OSINT / asset discovery |
| `/pentest-web`     | Web + API methodology (drives web-audit + manual tests) |
| `/pentest-network` | Host/service enumeration |
| `/pentest-sast`    | Read a codebase and confirm real vulns |
| `/pentest-report`  | Compile findings into a polished report |

> Tip: automation (scripts) = fast first pass. Claude skills / manual review = catch the logic bugs
> and false positives. Use both for real coverage. See `playbooks/` for the full methodology.

---

## Three-layer coverage (how to not miss things)
1. **Self-contained `*-audit` script** - instant, no setup. Your first pass.
2. **Tool-backed script** (`sast-scan` with semgrep, `portscan` with nmap) - deeper, after `install.ps1`.
3. **Claude skill / manual review** - logic bugs, IDOR, business-logic flaws no scanner finds.
