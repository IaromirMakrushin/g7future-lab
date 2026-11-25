# Admin account overview (T15)

**Date checked:** 2026-11-25

## Domain-level admin groups

| Group | Members | Why |
|---|---|---|
| **Domain Admins** | `Administrator` (built-in) | Full control over the domain — used for all AD, GPO, and server administration tasks in this lab. |
| **Enterprise Admins** | `Administrator` (built-in) | Forest-wide admin rights — relevant mainly because this is a single-domain forest; no separate forest-level account was created. |

No other accounts are members of either group. A test account (`adm-psotest`) was briefly added to Domain Admins earlier in the project to test password policy enforcement (see T12/PSO) and has since been removed from the Domain Admins group and disabled — the account object itself was not deleted — see [`evidence/sprint2/dc01-admin-group-membership.png`](../evidence/sprint2/dc01-admin-group-membership.png).

**Finding:** `adm-psotest` still carries `adminCount=1` as a leftover attribute from when it was a Domain Admins member, even though it's no longer in the group and is disabled. This is expected AD behavior (AdminSDHolder doesn't automatically clear `adminCount` when an account is removed from a protected group) — flagged here as a known artifact rather than a live risk, since the account is disabled.

## Local admin accounts per server

All three servers (DC01, FILSRV01, CA01) and CLIENT01 use the same domain `Administrator` account for local administration — no separate local admin accounts were created, since this is a lab environment with a single administrator (shared between the two group members).

## Department users

The 5 department test users (Ekonomi, Lager, IT, Sälj, VD) are standard domain users — **not** members of any admin group. They only have access to their own department's file share via the IGDLA group structure (see T11).

## Why this matters for the security requirements

- Admin account count is minimal and documented (one shared `Administrator` account), satisfying the "admin account count tracked and documented" requirement.
- Domain Admins group membership changes are audited on DC01 (see T14, done by Elis) — any future addition/removal would generate a Security event (4728/4732).
- Evidence: [`evidence/sprint2/dc01-admin-group-membership.png`](../evidence/sprint2/dc01-admin-group-membership.png)
