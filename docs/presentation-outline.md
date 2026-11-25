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
- T11: FILSRV01 — 5 shares, IGDLA structure, Domain Users only
- **Live demo #1:** `ekonomi1` writes to `\\FILSRV01\Ekonomi`, denied on `\\FILSRV01\IT`

## 4. Certificates & Secure Directory (3 min) — Iaromir
- T9: CA01 → Enterprise Root CA, auto-enrollment templates + GPO, verified certs issued
- T10: LDAPS on port 636, tested with ldp.exe, LDAP signing enforced

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
- Findings: NTLM v2 by IP vs Kerberos by name (expected), **NTLMv1 observed from CA01** (flagged, not yet fixed)
- Honest note: Deny not applied — time constraint, not an oversight

## 8. Dropped scope (1 min) — Both
- T5/T8 (DC02/redundancy): teacher-approved
- T16-T18 (RADIUS/WiFi/EAP-TLS): design only, architecture shown but not built

## 9. Open issues & lessons learned (2 min) — Both
- T2/T3 gateway/VLAN mismatch — still being debugged with the network admin
- Clock drift on DC01 (8h, fixed 25 Nov) — explains odd timestamps in older evidence
- NTLMv1 finding on CA01
- What we'd do differently: catch the VLAN/IP plan mismatch earlier (lesson from the 10 Nov correction)

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
- [ ] Topology diagram (simple box diagram, can be done in PowerPoint/draw.io — want me to generate one as an artifact you can screenshot into slides?)
- [ ] Elis's screenshots (Event Viewer, GPMC, CLIENT01 test) dropped into evidence/sprint2 + sprint3
- [ ] Rehearsal — who presents which section out loud
