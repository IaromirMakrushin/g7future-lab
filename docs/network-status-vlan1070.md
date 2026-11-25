# VLAN 1070 connectivity status

*Status as of the evening of 10 Nov 2026. Sanitized: no MAC addresses, public IPs or management addresses. "Verified" means someone ran the command and has the output or a screenshot. Everything else is marked as a hypothesis or "not verified".*

## Summary

The group's servers and client sit on VLAN 1070 (`10.7.1.0/24`, gateway `10.7.1.1` on the FortiGate). They reach each other, but **no frames from the VMs reach the FortiGate**, so there is no gateway, no internet from the servers and no inbound remote access. Sprint 1's domain goal is unaffected (it only needs L2 on the same VLAN). Backlog items T2 (VLANs + FortiGate interfaces) and T3 (FortiGate rules + DHCP) cannot be closed until this is fixed.

## What works (verified)

- DC01, FILSRV01, CA01 and CLIENT01 see each other on VLAN 1070 (ARP entries for the neighbours; domain join and domain login evidenced in `evidence/sprint1/`).
- ESXi (hypervisor01): port group has VLAN ID 1070, active uplink `vmnic0` at 100 Mbps full duplex, no failover-order override.
- FortiGate: the VLAN 1070 interface is up, `10.7.1.1/24`, ping allowed, parent interface is the hard-switch `internal` (no per-port VLAN filter listed).
- Exactly one FortiGate interface uses VLAN ID 1070 (no duplicate).

## What does not work (verified)

- `ping 10.7.1.1` from DC01 answers `Destination host unreachable` **from DC01's own address**. That is Windows failing ARP for the gateway locally. It is not a FortiGate policy or ICMP block.
- FortiGate packet captures filtered on DC01's traffic (on `internal` and on the VLAN interface): 0 packets. A control capture on `internal` toward the ESXi management address showed traffic, so the capture method works.
- The FortiGate interface counter for the VLAN 1070 interface shows 0 received frames, and the FortiGate ARP table has no `10.7.1.x` entries.
- Consequences: no internet from the servers, no inbound RDP through the port-forward, no way to reach the VMs from a laptop outside the lab.

## Interpretation (hypothesis, not verified)

Frames tagged 1070 are lost between hypervisor01 and the FortiGate. hypervisor01's link negotiates 100 Mbps while the FortiGate ports in use link at 1 Gbps, so another device (most likely a switch) sits in between. The leading candidate is a switch port profile that does not carry VLAN 1070 to hypervisor01's port. A bad cable or port (the 100 Mbps negotiation) may be a second fault on the same link. ESXi and FortiGate configuration do not explain the symptom. The group cannot change the switch, so this needs the instructor.

## Planned test before asking (not yet run)

Run a short ARP capture on DC01 (`pktmon`) while the FortiGate pings `10.7.1.2`:

| Result | Meaning | What to ask the instructor to check |
|---|---|---|
| DC01 sees the FortiGate's ARP request but the reply never arrives | Inbound path works, outbound is filtered | Port security / MAC limit on the switch port |
| DC01 sees no ARP from the FortiGate | VLAN 1070 is not carried to hypervisor01 at all | VLAN 1070 tagged on hypervisor01's port and on the uplink toward the FortiGate; network exists in the switch controller |

## Requests for the instructor

1. Which switch and port does hypervisor01 connect to, and is VLAN 1070 tagged on it and on the uplink?
2. Why does the link negotiate only 100 Mbps (cable or port)?
3. Confirm that `10.7.1.0/24` with gateway `10.7.1.1` is the intended network for group 7.
4. Is there a policy with NAT from the VLAN 1070 interface to the WAN, and may we create or edit FortiGate policies and DHCP for our own VLAN (needed for T3)?
5. A way into the lab from outside (lab VPN or port-forward) for remote administration and evidence collection.
6. Written confirmation that skipping DC02 is approved for a two-person group (needed in the report).

## Evidence to collect once the path works (all not verified until done)

`ping 10.7.1.1`, `ping 1.1.1.1`, `ipconfig /all` on every VM, `nltest /sc_verify:g7future.local` on FILSRV01 and CA01, DNS forwarder check on DC01, and the DC01 ARP table.
