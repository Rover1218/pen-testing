# Network / Host Pen-Test Playbook

For testing hosts, IP ranges, and exposed services. Stay strictly within the scope file —
never scan IPs you aren't authorized for. No DoS unless explicitly authorized.

## 1. Host discovery
- [ ] Confirm live hosts (in scope only): `nmap -sn <range>` or ping sweep.
- [ ] Don't expand scope — resolve every host against the engagement scope file first.
  - Script: `./scripts/network/portscan.ps1 -Engagement <name>`

## 2. Port & service enumeration
- [ ] Fast full-port sweep, then service/version detection on open ports:
  ```
  nmap -p- --min-rate 1000 <host>            # find open ports
  nmap -sV -sC -p <ports> -oA scan <host>    # versions + default scripts
  ```
- [ ] UDP top ports if relevant: `nmap -sU --top-ports 50 <host>`.
- [ ] OS fingerprint (`-O`), record everything to the engagement `scans/` dir.

## 3. Per-service enumeration
| Port | Service | Check |
|---|---|---|
| 21 | FTP | anonymous login, version CVEs |
| 22 | SSH | version, auth methods, weak algos |
| 23 | Telnet | exposure (should not exist) |
| 25/465/587 | SMTP | open relay, user enum (VRFY) |
| 53 | DNS | zone transfer (`dig axfr`), version |
| 80/443/8080 | HTTP(S) | hand to [`web.md`](web.md) |
| 88/389/636/445 | AD/LDAP/SMB | `enum4linux-ng`, `ldapsearch`, null sessions, shares |
| 110/143 | POP/IMAP | cleartext auth |
| 161 | SNMP | default community strings (`snmpwalk public`) |
| 1433/3306/5432/27017/6379 | DBs | exposure, default creds, no-auth (Redis/Mongo) |
| 3389 | RDP | exposure, NLA, BlueKeep-era CVEs |
| 5985/5986 | WinRM | exposure, auth |

## 4. Vulnerability identification
- [ ] Map service versions to known CVEs (vulners, searchsploit, vendor advisories).
- [ ] `nuclei` against discovered HTTP services.
- [ ] Default credentials on management interfaces / databases.
- [ ] Validate findings — confirm the version really is vulnerable before reporting.

## 5. (Authorized exploitation only)
Only if the engagement authorization level is **Full**:
- [ ] Verify exploitability safely; avoid anything that risks availability.
- [ ] Document exact steps; capture proof-of-access (whoami/hostname), not bulk data.

## Active Directory (if in scope)
- [ ] Unauthenticated: SMB null sessions, `enum4linux-ng`, AS-REP roasting candidates.
- [ ] With creds: BloodHound collection, Kerberoasting, ACL analysis, share hunting.

## Tooling
| Goal | Tool |
|---|---|
| Discovery/scan | `nmap`, `masscan`, `rustscan` |
| SMB/AD | `enum4linux-ng`, `crackmapexec`/`nxc`, `smbclient`, BloodHound |
| Vuln templates | `nuclei` |
| Exploit lookup | `searchsploit`, vulners |
| SNMP | `snmpwalk`, `onesixtyone` |
