# Commands Cheatsheet

Every command you need, copy-paste ready. Run all of these from the project root:
`cd "c:\Users\Anindya\Desktop\pen testing"`

> **Every** scan script writes a Markdown report to
> `engagements/<name>/scans/<phase>-<timestamp>/report-*.md`. The `*-audit` scripts are
> self-contained (no tools needed); the others go deeper but need tools installed.

---

## 🚀 Shortcut: the launcher (instead of typing each command)

`scan.ps1` runs the right scripts for you and prints a combined summary. Use the **menu** or a **one-liner**:
```powershell
./scan.ps1                                                          # interactive menu (pick 1-6)
./scan.ps1 -Mode local  -Url http://localhost:3000 -Source "<APP>" -Engagement myapp
./scan.ps1 -Mode hosted -Url https://yoursite.com -Engagement site-live
./scan.ps1 -Mode code   -Source "<APP>" -Engagement myapp
./scan.ps1 -Mode mobile -Apk "C:\app.apk" -Engagement myapp
./scan.ps1 -Mode dashboard                                         # build + open the HTML dashboard
```
- **local** = web-audit + code-audit + sast-scan (the 3-in-1 for a running app + its source)
- **hosted** = recon + web-audit + web-enum + TLS (skips network scan for managed hosts)
- **dashboard** = aggregates every engagement's latest reports into `dashboard.html` (open in a browser)

## ⭐ What to run, when (the full list)

`<APP>` = `"C:\Users\Anindya\Desktop\Rover\projects\advanced portfolio"`

| Category | Script | Time | When to run |
|---|---|---|---|
| **lib** | `lib/common.ps1` | — | ❌ Never run directly (shared engine) |
| **mobile** | `mobile/flutter-scan.ps1 -Apk "<apk>"` | ~10s | When you have a built `.apk` |
| **network** 🟢 | `network/net-audit.ps1 -Engagement <e>` | seconds | **Default** quick port check |
| **network** 🔵 | `network/portscan.ps1 -Engagement <e>` | ~2-3 min | Deep nmap (version detection) |
| **web** 🟢 | `web/web-audit.ps1 -Url <url> -Source <APP> -Engagement <e>` | seconds | **Default** (app must be running) |
| **web** 🔵 | `web/web-enum.ps1 -Engagement <e> -Url <url>` | ~1-2 min | Deep nuclei CVE templates |
| **sast** | `sast/code-audit.ps1 -Source <APP> -Engagement <e>` | seconds | App-logic bugs |
| **sast** | `sast/sast-scan.ps1 -Target <APP> -Engagement <e>` | ~1 min | Deps (CVEs) + secrets |
| **recon** | `recon/recon.ps1 -Engagement <e>` | seconds | Public domains only (not localhost) |

🟢 = quick default · 🔵 = deep, run occasionally. For **network** and **web** pick ONE per run
(quick by default). For **sast**, run BOTH (`code-audit` + `sast-scan` check different things).

**Everyday localhost routine** (start app, then run these 3 — under a minute):
```powershell
./scripts/web/web-audit.ps1   -Url http://localhost:3000 -Source "<APP>" -Engagement portfolio
./scripts/sast/code-audit.ps1 -Source "<APP>" -Engagement portfolio
./scripts/sast/sast-scan.ps1  -Target "<APP>" -Engagement portfolio
```

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

## 3. The tool-backed scripts (need tools installed; deeper — also write a report now)
```powershell
./scripts/recon/recon.ps1      -Engagement myapp     # subdomains, DNS, fingerprint (subfinder/whatweb)
./scripts/network/portscan.ps1 -Engagement myapp     # full nmap scan (-sV -sC)  [nmap installed]
./scripts/web/web-enum.ps1     -Engagement myapp -Url http://localhost:3000   # nuclei/ffuf  [nuclei installed]
./scripts/sast/sast-scan.ps1   -Target "C:\path\to\repo" -Engagement myapp    # gitleaks + trivy
```
Installed & working: **nmap, nuclei, ffuf, sslscan, subfinder, httpx, gitleaks, trivy**.
Wordlist for ffuf: `wordlists/common.txt`. (semgrep removed — broken on Windows; use
`code-audit.ps1` or `/pentest-sast` for code-pattern analysis instead.)

## 3b. Testing a DEPLOYED / hosted site (not localhost)

Requirements: (1) **you own it or have written permission**; (2) **know your hosting** —
on managed hosts (Vercel/Netlify/Cloudflare) **skip nmap/net-audit** (not your servers);
(3) **be gentle** (no DoS, mind rate limits/WAFs).

```powershell
./scripts/new-engagement.ps1 -Name site-live -Scope "yourdomain.com"
# fill engagements/site-live/AUTHORIZATION.md, then:
./scripts/recon/recon.ps1   -Engagement site-live                                  # subdomains + tech
./scripts/web/web-audit.ps1 -Url https://yourdomain.com -Engagement site-live      # https now
./scripts/web/web-enum.ps1  -Url https://yourdomain.com -Engagement site-live -Wordlist wordlists\common.txt   # nuclei + ffuf
sslscan yourdomain.com                                                             # TLS / ciphers
# net-audit/portscan ONLY if you own the server (not managed hosting)
```
What's extra vs localhost: **TLS check (sslscan)**, **subdomain recon (subfinder/httpx)**, and a
**wordlist for ffuf content discovery**. Business-logic bugs still need a manual `/pentest-web` pass.

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
| `/pentest-mobile`  | Analyze a Flutter/Android APK (secrets, endpoints, manifest) |
| `/pentest-report`  | Compile findings into a polished report |

> Tip: automation (scripts) = fast first pass. Claude skills / manual review = catch the logic bugs
> and false positives. Use both for real coverage. See `playbooks/` for the full methodology.

---

## Three-layer coverage (how to not miss things)
1. **Self-contained `*-audit` script** - instant, no setup. Your first pass.
2. **Tool-backed script** (`sast-scan` with gitleaks/trivy, `portscan` with nmap, `web-enum` with nuclei) - deeper.
3. **Claude skill / manual review** - logic bugs, IDOR, business-logic flaws no scanner finds.
