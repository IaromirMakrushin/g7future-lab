# Test matrix — Group 7 (T20)

**Compiled:** 2026-11-25 · **Presentation:** 26 Nov 12:00 · **Report due:** 26 Nov 17:00

Every row states what was required, what was tested, the result, and whether it's verified. Anything not actually tested is marked **not verified** — never claimed as done without proof.

| # | Requirement | Ticket | Status | Evidence / Result |
|---|---|---|---|---|
| 1 | Domain controller + DNS | T4 | ✅ Verified | `dcdiag` all passed, FSMO roles + GC on DC01, AD-integrated DNS |
| 2 | Users/groups (IGDLA) | T6 | ✅ Verified | 5 department OUs, 1 user + 1 Global group each |
| 3 | Client joined to domain | T7 | ✅ Verified | CLIENT01 (host name DESKTOP-TAVFJ6C) on `10.7.1.10`, domain login as `ekonomi1` confirmed via `whoami`. Re-verified 25 Nov ~20:55 after the network fix: `klist` shows live Kerberos tickets issued by DC01 (cifs/FILSRV01, host/CA01), and `gpresult /r` shows Default Domain Policy applied from DC01 |
| 4 | File server, 5 dept. shares, IGDLA, Domain Users only | T11 | ✅ Verified | FILSRV01 `E:\Avdelningar`, 5 shares, DL groups only, access-based enumeration on, tested write/deny with a temp Ekonomi user |
| 5 | Password policy (12/14 chars) + PSO | T12 | ✅ Verified | Domain: 12 chars, lockout after 3. Admin PSO: 14 chars, lockout duration 0 (admin unlock required by design) |
| 6 | SMBv1 disabled, SMB signing required | T13 | ✅ Verified | GPO `G7-T13-SMB-Hardening`; signed connections confirmed on DC01, FILSRV01, CA01 |
| 7 | Auditing (group changes, file access, deletions) | T14 | ✅ Verified | Events 4728/4729, 4732/4733, 4756/4757 (group changes); 4663, 4660 (file access/delete); 5145 (denied access). GPO `G7-T14-FILSRV01-Audit` (file server) + Default Domain Controllers Policy (DC). Event 4656 not verified. **Incident:** a GPO edit briefly reset DC01's whole audit policy for 15m43s on 24 Nov (found via event 4719, restored same day) — included as a lesson learned |
| 8 | Enterprise Root CA + cert templates + auto-enrollment | T9 | ✅ Verified | CA01 promoted (`G7Future-Root-CA`, SHA256, 10yr). Templates `G7-Computer-Autoenroll` / `G7-User-Autoenroll` issued. Auto-enrollment GPO enabled (Computer + User Config). DC01 confirmed holding 3 CA-issued certs after `gpupdate /force` |
| 9 | LDAPS (port 636) + LDAP signing | T10 | ✅ Verified | Port 636 listening (`Test-NetConnection`), real SSL bind via `ldp.exe` succeeded (RootDSE returned). "LDAP server signing requirements" set to Require signing |
| 10 | NTLM disabled / Kerberos only | T19 | ⚠️ Audit only, not Deny | GPO `G7-T19-NTLM-Audit` (domain-wide). Verified events 8001/8002 and 4624 (NTLM v2 by IP, Kerberos by name). **Finding:** NTLMv1/ANONYMOUS LOGON observed from CA01 at 16:01 on 25 Nov — used in the presentation as a concrete example of "read the log, find something real." Not yet remediated. Deny not applied due to time constraints — **not verified as enforced** |
| 11 | Admin account count tracked & documented | T15 | ✅ Verified | Only built-in `Administrator` in Domain Admins + Enterprise Admins. `adminCount=1` leftover on a removed test account, noted. See `docs/admin-accounts.md` |
| 12 | 802.1X / EAP-TLS WiFi via RADIUS | T16, T17, T18 | ❌ Dropped — design only | WiFi/RADIUS scope dropped 24 Nov due to time constraints. Presented as architecture/design in the presentation, explicitly **not built or verified** |
| — | VLAN segmentation + Fortigate routing | T2 | ✅ Verified — fixed | **Root cause:** DC01, FILSRV01 and CA01 were attached to ESXi port group "Vlan Tenta Server" (VLAN 951, shared with other groups' VMs) instead of the intended "Group7DCNetwork" (VLAN 1070). Only CLIENT01 was correctly attached. **Fix (25 Nov, ~20:30–21:00):** Elis re-attached DC01/FILSRV01/CA01 to "Group7DCNetwork" (snapshot taken first per VM). Verified: FortiGate interface `Group7DCNet` (10.7.1.1/24) up; FILSRV01 pings gateway and resolves external DNS; CLIENT01 reconnected ~20:52 and completed a live Kerberos logon + file-share session over the network (see T7, T11) |
| — | Fortigate firewall rules + DHCP | T3 | ⚠️ Partly verified | Gateway reachability and DNS resolution confirmed from FILSRV01 and CLIENT01 now that T2 is fixed, but FortiGate firewall *rule* behavior and DHCP leasing were not separately tested — remaining item in Jira (To Do) |

## Dropped tickets (teacher-approved scope reduction)

| Ticket | Reason |
|---|---|
| T5 — DC02 | Teacher approved skipping a second DC for a 2-person group (2026-11-05) |
| T8 — Redundancy test | Dependent on DC02, dropped with it |
| T16, T17, T18 — RADIUS/WiFi/EAP-TLS | Scope reduced 24 Nov given time constraints; presented as design only |

## Known issues / facts to disclose in the presentation

- DC01's clock ran 8 hours fast with no external time source; corrected by hand on 2026-11-25. **Any event timestamped before 2026-11-25 01:13 is off by ~8 hours.**
- CLIENT01 has the same class of issue: its time zone is set to Pacific Standard Time (UTC−8), making its local clock read ~8h behind actual Swedish time. Timestamps from CLIENT01-side logs need `VM time − 8h` to line up with DC01/FILSRV01 events. Not fixed (time zone, not clock drift) — noted as a known artifact.
- VLAN/Fortigate gateway issue (T2) was found and fixed same evening (25 Nov) — root cause was an ESXi port-group mismatch, not a FortiGate config error. Presented honestly as "found it, fixed it, verified it," not hidden. T3 (firewall rules/DHCP specifically) remains not fully tested.
- NTLMv1 traffic observed from CA01 — a real finding, included as "discovered but not yet remediated" rather than omitted.
- `DC01 failed test SystemLog` in `dcdiag` before the network fix, traced to DNS Client/Schannel/OCSP timeouts reaching external hosts (no AD/replication errors) — cleared after the network fix, confirmed with a re-run.

## Demo candidates

Now that CLIENT01 is back on the network, these can run live from the client rather than only from the ESXi console:

1. Department user: own share works, another department's share is denied, event 5145 shows the denied attempt (covers T11 + T14 + T7) — demoed live from CLIENT01 as `ekonomi1`
2. Same department user connecting by IP vs. by hostname: NTLM v2 vs. Kerberos in the log (T19)
3. A group membership change on DC01 showing up as event 4728/4732 (add) and 4756/4757/4758 (universal group membership + deletion) (T14)
