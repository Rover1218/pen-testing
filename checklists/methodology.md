# Engagement Methodology Checklist

A high-level phase checklist that applies to any engagement. Use the per-target playbooks in
[`../playbooks/`](../playbooks/) for the detailed steps.

## Phase 0 — Authorize & scope
- [ ] `AUTHORIZATION.md` filled out and signed.
- [ ] Scope file (`scope.txt`) lists every in-scope target; out-of-scope noted.
- [ ] Test window and activity level confirmed.

## Phase 1 — Recon
- [ ] Passive OSINT (DNS, certs, subdomains, leaked creds, public repos).
- [ ] Asset discovery — enumerate the attack surface within scope.
- [ ] Tech fingerprinting.

## Phase 2 — Enumeration & scanning
- [ ] Port/service enumeration (network targets).
- [ ] Content/endpoint discovery (web/API targets).
- [ ] Automated vuln scanning (nuclei, semgrep, etc.) — triage output.

## Phase 3 — Vulnerability analysis
- [ ] Map findings to known CVEs / OWASP categories.
- [ ] Confirm each candidate is real (remove false positives).

## Phase 4 — Exploitation (only if authorized)
- [ ] Verify exploitability safely, no DoS.
- [ ] Capture minimal proof-of-impact (not bulk data exfil).

## Phase 5 — Post-exploitation (only if authorized)
- [ ] Document access gained; assess blast radius; no persistence beyond test window.

## Phase 6 — Reporting
- [ ] One finding file per issue with severity, evidence, impact, remediation.
- [ ] Compile final report from `reports/_template.md`.
- [ ] Executive summary + technical detail + remediation roadmap.

## Phase 7 — Cleanup
- [ ] Remove any artifacts/accounts/files created during testing.
- [ ] Confirm no lingering access; notify client of completion.
