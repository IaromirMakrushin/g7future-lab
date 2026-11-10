# Advanced IT Security — Lab Project (Group 7)

A secured company network built from scratch in a virtual lab, as part of the *Avancerad IT-säkerhet* course (TUC Yrkeshögskola, ITSÄK25LIN).

**Presentation:** 26 Nov 2026 · **Exam:** 27 Nov 2026

## Repo structure

- [`docs/jira-tickets.md`](docs/jira-tickets.md) — full backlog as Jira-ready ticket text
- `evidence/sprint1/`, `evidence/sprint2/`, `evidence/sprint3/` — screenshots and command output proving each requirement, organized by sprint
- `scripts/` — PowerShell/config snippets used during the build

> This repo documents the design, build steps, and evidence for the project. It intentionally contains **no credentials, IP-reachable login pages, or anything that could be used to access the live lab** — see [Security notes](#security-notes).

---

## Team

| Name | Role (rotates each sprint) |
|---|---|
| Elis | Sprint 1: Product Owner |
| Iaromir | Sprint 1: Scrum Master |
| *(3rd member — TBC)* | Sprint 3: Product Owner |

Working as an agile team (Scrum), 2–3 people. Roles rotate each sprint so everyone experiences each responsibility.

---

## Scenario

A fictional company with 5 departments needs a secured Windows Active Directory environment:

- **Ekonomi** (Finance)
- **Lager** (Warehouse)
- **IT**
- **Sälj** (Sales)
- **VD** (CEO)

## Architecture

| Server | Role | IP |
|---|---|---|
| **DC01** | Primary Domain Controller, DNS | `10.7.1.2` |
| **DC02** | *Dropped: teacher approved skipping DC02 for a two-person team — see [Open questions](#open-questions)* | — |
| **FILSRV01** | File server — one share per department | `10.7.1.3` |
| **CA01** | Enterprise Root CA — certs for machines/users/LDAPS | `10.7.1.4` |
| **RADIUS01** | NPS/RADIUS — 802.1X WiFi auth | TBD (not built yet) |
| **CLIENT01** | Test client proving domain login (Sprint 1) | `10.7.1.10` |

### Network

| VLAN | Purpose | Subnet |
|---|---|---|
| 1070 | Servers + clients (actual, per Fortigate config) | `10.7.1.0/24` |

> **Correction (10 Nov 2026):** the original plan below called for separate `10.7.10.0/24` (servers) / `10.7.20.0/24` (clients) / `10.7.30.0/24` (WiFi) subnets. Elis found the real Fortigate interface config uses gateway `10.7.1.1` on a single `10.7.1.0/24` subnet, not `10.7.10.x`. All built machines (DC01, FILSRV01, CA01, CLIENT01) were re-IP'd to `10.7.1.0/24` to match. The planned per-purpose VLAN split (1070/1071/1072) below is the *original design intent* and may still apply once WiFi/RADIUS (VLAN 1072) is built — treat the table above as what's actually running today.

<details>
<summary>Original planned VLAN split (superseded for servers/clients, kept for reference)</summary>

| VLAN | Purpose | Subnet |
|---|---|---|
| 1070 | Servers | `10.7.10.0/24` |
| 1071 | Client workstations | `10.7.20.0/24` |
| 1072 | WiFi (802.1X) | `10.7.30.0/24` |

</details>

- Full allocated range: **VLAN 1070–1079, `10.7.0.0/16`**
- Domain: **`g7future.local`**
- Firewall: Fortigate 70F (shared `root` VDOM)
- Switch: HP (VLANs tagged on ports 12, 13, 25)

---

## Security requirements

- Password policy: **12 chars** (users) / **14 chars** (admins), lockout after **3 failed attempts**
- **Kerberos only** — NTLM disabled
- **LDAPS** enforced for directory communication
- **SMBv1 disabled**, latest SMB + **SMB signing** required
- **Auditing** on the file server: failed access attempts, deletions, admin group membership changes
- Admin account count tracked and documented
- **802.1X / EAP-TLS** for WiFi via RADIUS

---

## Sprint plan

| Sprint | Dates | Goal |
|---|---|---|
| 1 — Foundation | 5–11 Nov | A client can log in to the domain |
| 2 — Services & policy | 12–18 Nov | Department shares (SMB signed), certificates issued, LDAPS working |
| 3 — WiFi & verification | 19–25 Nov | Secure WiFi, NTLM blocked, full test matrix, presentation rehearsed |

Reviews/retrospectives: 11, 18, 25 Nov.

---

## Backlog

### Sprint 1
- [x] Create Jira board + backlog
- [ ] Configure VLANs on switch + Fortigate interfaces
- [ ] Fortigate firewall rules + client DHCP
- [x] DC01 — static IP, VMware Tools, `g7future.local` domain
- [x] OUs + one user per department + global groups
- [x] Join a Windows client to the domain — CLIENT01, logged in as `ekonomi1`, verified via `whoami`
- [x] FILSRV01 — built, static IP, extra disk, VMware Tools, snapshot
- [x] CA01 — built, static IP, VMware Tools, snapshot
- [x] FILSRV01 + CA01 joined to domain, domain login verified

### Sprint 2
- [ ] CA01 — Enterprise Root CA, certificate templates, auto-enrollment
- [ ] LDAPS on DCs, verified with `ldp.exe` on port 636
- [ ] FILSRV01 — 5 department shares via IGDLA, Domain Users only
- [ ] Password policy (12/14 chars) + PSO for admins
- [ ] SMBv1 disabled, SMB signing enforced
- [ ] Auditing (SACLs + Security Group Management) — events 4663, 4660, 4728, 4732, 4756
- [ ] Admin account documentation

### Sprint 3
- [ ] RADIUS01 — NPS, certificate from CA01, AD group policy
- [ ] WiFi SSID (WPA2/3-Enterprise) on VLAN 1072, AP as RADIUS client
- [ ] EAP-TLS tested on a real device
- [ ] NTLM audit mode → review → **Deny**
- [ ] Full test matrix with evidence
- [ ] Presentation + topology diagram + rehearsal

---

## Open questions

- **DC02**: dropped. The teacher approved skipping DC02 for a two-person team, so the project has one domain controller. This must be confirmed again if the team becomes three.
- Third team member — to be confirmed.
- UniFi access for WiFi AP configuration — to be requested from teacher.
- Whether the 30GB/VM disk guidance applies if a Windows 11 client (needs more) is added.

---

## Evidence policy

Every completed item gets a screenshot or command output, dated, stored under `/evidence/<sprint>/`. Anything not actually tested is marked **"not verified"** — never claimed as done without proof.

---

## Security notes

- Lab access (ESXi/Fortigate URLs, login) is **not included here** — provided separately by the instructor, shared only within the team.
- No real credentials, API keys, or personal data beyond first names are stored in this repo.
- Only resources inside `10.7.0.0/16` / VLANs 1070–1079 are touched — this is a shared school firewall.
