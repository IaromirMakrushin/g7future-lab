# Jira tickets — Group 7

Network: VLAN 1070–1079, 10.7.0.0/16, domain `g7future.local`. Scope reduced: no DC02.

**Correction (10 Nov 2026):** actual built servers/clients use `10.7.1.0/24` (gateway `10.7.1.1`), not `10.7.10.0/24` as originally planned — see README for details.

Paste each section below as one Jira issue (title = heading, description = bullet list). Suggested labels: `network`, `ad`, `fileserver`, `ca`, `radius`, `hardening`, `docs`.

---

## 1. Network setup (VLANs + Fortigate)
**Labels:** network
- Create VLAN 1070 (servers, 10.7.10.0/24) on HP switch, tag on ports 12, 13, 25
- Create VLAN 1071 (clients, 10.7.20.0/24) on HP switch, same ports
- Create VLAN 1072 (WiFi, 10.7.30.0/24) on HP switch, same ports
- Create matching interfaces in Fortigate (root firewall)
- Configure DHCP for VLAN 1071 (and 1072 once RADIUS/WiFi is set up)
- Verify VLAN tagging with a test VM on each segment
- **Evidence:** screenshot of switch VLAN config, Fortigate interface list

## 2. DC01 — Domain Controller
**Labels:** ad
- Install Windows Server (template: 2019, ISO: 2022) on DC01
- Assign static IP in 10.7.10.0/24
- Promote to Domain Controller, create domain `g7future.local`
- Install DNS role, verify resolution
- **Evidence:** `dcdiag` output, screenshot of AD Users and Computers

## 3. Users & groups (IGDLA)
**Labels:** ad
- Create 1 user per department: Ekonomi, Lager, IT, Sälj, VD
- Create Global groups: one per department (G_Ekonomi, G_Lager, G_IT, G_Salj, G_VD)
- Create Domain Local groups: one per folder/permission level (DL_Ekonomi_RW, etc.)
- Nest Global groups into Domain Local groups (IGDLA)
- **Evidence:** screenshot of group structure + membership

## 4. FILSRV01 — File server
**Labels:** fileserver
- Install Windows Server, join domain `g7future.local`
- Add extra virtual disk, format, dedicate to shares
- Create 5 folders (Ekonomi, Lager, IT, Sälj, VD), one share each
- Apply NTFS + share permissions via Domain Local groups only
- Restrict access to Domain Users (no Everyone/Anonymous)
- Enforce latest SMB version + SMB signing
- **Evidence:** screenshot of share permissions per folder, `Get-SmbServerConfiguration` output

## 5. CA01 — Certificate Authority
**Labels:** ca
- Install AD CS, configure as Enterprise Root CA
- Create/configure certificate templates (computer, user, LDAPS)
- Enable auto-enrollment via GPO
- Issue a test certificate to DC01 for LDAPS
- **Evidence:** screenshot of issued certificates, CA console

## 6. GPO hardening
**Labels:** hardening
- Password policy: 12 chars (users), 14 chars (admins)
- Account lockout: 3 failed attempts, 30 min timeout (or admin-unlock for sensitive accounts)
- Disable NTLM, enforce Kerberos only
- Enforce LDAPS for directory communication
- Enforce SMB signing via GPO
- **Evidence:** `gpresult /r`, screenshot of applied policies

## 7. Auditing on FILSRV01
**Labels:** fileserver, hardening
- Enable Object Access auditing (failed access attempts)
- Enable auditing for file/folder deletion
- Enable auditing for Administrators group membership changes
- Verify events appear in Security log
- **Evidence:** screenshots of audit events (access denied, delete, group change)

## 8. RADIUS01 + WiFi 802.1X
**Labels:** radius
- Install NPS role on RADIUS01
- Configure RADIUS client (WiFi AP/controller)
- Configure 802.1X policy, point to DC01 for auth
- Test WiFi login with a domain user
- **Evidence:** screenshot of NPS policy, successful 802.1X auth log

## 9. Admin count & audit review
**Labels:** hardening, docs
- Document how many admin accounts exist in the domain
- Verify Security log captures unauthorized admin-rights grants
- **Evidence:** screenshot of admin group membership + relevant log entry

## 10. Evidence package & presentation
**Labels:** docs
- Compile all screenshots/command output with dates into one document
- Mark any untested item clearly as "not verified"
- Prepare slides/demo for 26 Nov presentation
