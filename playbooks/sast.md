# Source Code Review / SAST Playbook

Point this at another project's codebase to find vulnerabilities by reading the code. This is
the most natural fit for Claude Code — it can read the whole repo. Use `/pentest-sast` to have
Claude drive it, or run the automated scanners below.

## 1. Orient
- [ ] Identify language(s), framework(s), and entry points (routes, handlers, CLI, jobs).
- [ ] Map the architecture: where does untrusted input enter, where does it reach a sink?
- [ ] Find the dependency manifests (`package.json`, `requirements.txt`, `pom.xml`, `go.mod`,
      `Gemfile`, `composer.json`, `*.csproj`).

## 2. Automated first pass
- [ ] **Secrets:** `gitleaks detect` / `trufflehog` — committed keys, tokens, passwords.
- [ ] **Dependencies (SCA):** `osv-scanner`, `npm audit`, `pip-audit`, `trivy fs .`.
- [ ] **SAST engine:** `semgrep --config auto .` (great signal-to-noise across languages).
- [ ] **IaC/containers:** `trivy config .`, `checkov` for Terraform/K8s/Dockerfiles.
  - Script: `./scripts/sast/sast-scan.ps1 -Target <path-to-repo>`

## 3. Manual review — follow the taint
Trace untrusted input → sink. Focus on these sink categories:

| Vuln class | What to grep / look for |
|---|---|
| SQL injection | string-concatenated queries, raw `execute(`, missing parameterization |
| Command injection | `exec`, `system`, `child_process`, `subprocess` with shell/interpolation |
| Path traversal | file paths built from user input, `../` not sanitized |
| Deserialization | `pickle`, `yaml.load`, `ObjectInputStream`, `unserialize`, BinaryFormatter |
| SSRF | HTTP clients fed user-controlled URLs |
| XSS | template output without escaping, `dangerouslySetInnerHTML`, `innerHTML` |
| Auth/authz | missing checks on routes, IDOR, role checks done client-side only |
| Crypto | hardcoded keys, weak algos (MD5/SHA1/DES/ECB), `Math.random` for secrets |
| SSTI | user input into template strings |
| Secrets | hardcoded creds, API keys, private keys, connection strings |
| Open redirect | redirects built from request params |
| Mass assignment | binding request body straight to models |

## 4. Auth & access control review
- [ ] Where is authn enforced? Is every sensitive route covered (middleware vs per-handler)?
- [ ] Authz checks — object ownership verified server-side? Any IDOR patterns?
- [ ] Session/token handling, password storage (bcrypt/argon2 vs plain/MD5).

## 5. Config & secrets
- [ ] `.env`, config files, CI files for committed secrets and debug flags in prod.
- [ ] Insecure defaults (debug=true, permissive CORS, disabled TLS verification).

## 6. Dependencies
- [ ] Known-vulnerable versions (from SCA step) actually reachable in the code path?
- [ ] Outdated/unmaintained critical libs.

## How Claude should review (when running `/pentest-sast`)
1. Read the dependency manifests and the directory tree first to understand the app.
2. Run `semgrep`/`gitleaks`/`osv-scanner` if available; triage their output (drop false positives).
3. Read the highest-risk files (auth, input handlers, query builders, file/exec sinks).
4. For each candidate, trace the data flow and confirm it's reachable & exploitable.
5. Write one finding file per confirmed issue with file:line, snippet, impact, fix.

## Tooling
| Goal | Tool |
|---|---|
| SAST | `semgrep` |
| Secrets | `gitleaks`, `trufflehog` |
| SCA | `osv-scanner`, `trivy fs`, `npm audit`, `pip-audit` |
| IaC | `trivy config`, `checkov` |
