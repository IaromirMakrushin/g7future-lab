# Presentation outline — Group 7 (T21)

**Slot:** 26 Nov 2026, 12:00 (~20 min incl. demo) · **Report due same day, 17:00**

## 1. Intro (1 min) — Both
- Group 7, scenario: 5-department company (Ekonomi, Lager, IT, Sälj, VD), scope confirmed with teacher 5 Nov
- Scope note: 2-person group, DC02/redundancy (T5/T8) dropped with teacher approval; WiFi/RADIUS (T16-T18) dropped 24 Nov due to time — presented as design only

## 2. Architecture overview (2 min) — Iaromir
- Topology diagram: DC01, FILSRV01, CA01, CLIENT01 on `10.7.1.0/24`
- Domain `g7future.local`, single DC
- **Honest disclosure:** gateway/Fortigate routing issue (T2/T3) — VLAN tagging mismatch (951 vs 1070), still open, escalated to network admin, not blocking internal domain function

## 3. Identity & Access (3 min) — Iaromir
- T4: DC01, DNS, dcdiag clean
- T6: 5 department OUs/users/groups
- T7: CLIENT01 domain-joined, login verified as `ekonomi1`
- T11: FILSRV01 — 5 shares. Chain: user → `G_<avd>` → `DL_<avd>_RW` → share/NTFS (Domain Users has no permissions)
- **Live demo #1:** `ekonomi1` writes to `\\FILSRV01\Ekonomi`, denied on `\\FILSRV01\IT`

## 4. Certificates & Secure Directory (3 min) — Iaromir
- T9: CA01 → Enterprise Root CA verified (`certutil -ca.cert`). Templates created and permissioned, but auto-enrollment of our own templates is **not verified** — the GPO isn't linked in GPMC and DC01's certs come from built-in templates, not ours
- T10: Port 636 + ldp.exe SSL bind succeeded, but tested locally on DC01, not from a client. LDAP signing set in Default Domain Policy; whether it's enforced on DC01 is **not verified** (Default Domain Controllers Policy may override with None) — check `LDAPServerIntegrity`

## 5. Hardening & Policy (3 min) — Elis
- T12: password policy (12/14 chars), PSO for admins, lockout-until-admin-unlock by design
- T13: SMBv1 disabled, SMB signing required (DC01/FILSRV01/CA01)
- T15: admin account count — only built-in Administrator, documented

## 6. Auditing (2 min) — Elis
- T14: group membership changes (4728/4732/4756), file access/delete (4663/4660), denied access (5145)
- **Lesson learned:** GPO edit briefly reset DC01's audit policy for 15m43s — caught via event 4719, fixed same day, process improved
- **Live demo #2 or #3:** a group change showing up as 4728/4732, or NTLM vs Kerberos by IP/hostname

## 7. NTLM (2 min) — Elis
- T19: audit-only mode, GPO `G7-T19-NTLM-Audit`
- Findings: NTLM v2 by IP vs Kerberos by name (expected), **anonymous NTLM session observed from CA01 at 17:14** (not NTLMv1 — flagged, not yet fixed)
- Honest note: Deny not applied — time constraint, not an oversight

## 8. Dropped scope (1 min) — Both
- T5/T8 (DC02/redundancy): teacher-approved
- T16-T18 (RADIUS/WiFi/EAP-TLS): design only, architecture shown but not built

## 9. Open issues & lessons learned (2 min) — Both
- T2 fixed and verified 25 Nov 19:40 (stale ESXi active-port binding); T3 (firewall rules/DHCP) still not tested
- T9/T10 corrected to "partly verified" after a second pass found the evidence didn't support "verified"
- Clock drift on DC01 (8h, fixed 25 Nov) — explains odd timestamps in older evidence; CLIENT01 separately runs on Pacific time zone
- Anonymous NTLM session finding on CA01
- What we'd do differently: check actual ESXi port state, not just the configured setting; verify against a remote client, not localhost

## 10. Q&A

---

## Slide-by-slide (suggested, ~12-14 slides)
1. Title + group
2. Scenario + scope (what's in, what's dropped, why)
3. Architecture/topology diagram
4. AD foundation (T4, T6, T7)
5. File server + IGDLA (T11) + demo screenshot
6. Certificate Authority (T9)
7. LDAPS (T10)
8. Password policy + PSO (T12)
9. SMB hardening (T13)
10. Admin accounts (T15)
11. Auditing + the audit-policy incident (T14)
12. NTLM findings (T19)
13. Dropped scope: RADIUS/WiFi design (T16-T18)
14. Open issues + lessons learned + Q&A

## Still needed before tomorrow
- [ ] Elis's screenshots (Event Viewer, GPMC, CLIENT01 test) dropped into evidence/sprint2 + sprint3
- [ ] Rehearsal — who presents which section out loud
