# ASA ACL Ruleset Documentation

**Last updated:** 2026-06-12
**Device:** Cisco ASA 5506-X (perimeter firewall, inter-VLAN router)
**Status:** Draft. Entries marked **[VERIFY]** must be checked against the running config (`show running-config access-list`, `show running-config access-group`, `show nameif`) before this document is treated as authoritative. Everything not marked is sourced from journal entries and ADRs where the commands were recorded as entered.

## Enforcement Model

The ASA is the single enforcement point for inter-VLAN traffic (router-on-a-stick over the 802.1Q trunk to the SG350; the switch does L2 tagging only). Two layers govern what passes:

1. **Security levels (implicit policy).** Traffic from a higher security level to a lower one is permitted by default; lower to higher is denied by default.
2. **Interface ACLs (explicit policy).** The moment an ACL is applied to an interface, the implicit security-level behavior on that interface is replaced by the ACL, which ends in an implicit deny.

The steady-state philosophy is default-deny between segments with explicit least-privilege permits (ADR-0003). One documented temporary exception is in effect (ADR-0008, below).

## Interfaces and Security Levels

| Interface | VLAN | Subnet | Security Level | Source |
|---|---|---|---|---|
| OUTSIDE | - | upstream (double-NAT) | 0 **[VERIFY]** | assumed convention |
| MGMT | 10 | 10.10.10.0/24 | **[VERIFY]** | not recorded in journals |
| SERVERS | 20 | 10.10.20.0/24 | 80 | 2026-05-11 journal |
| CLIENTS | 30 | 10.10.30.0/24 | **[VERIFY]** | not recorded in journals |
| USER | 50 | 10.10.50.0/24 | 60 | 2026-05-11 journal |

VLAN 40 does not exist; the numbering gap is deliberate and documented in ADR-0003.

## Active ACLs

### USER_IN (inbound on USER interface)

Applied with:

```
access-group USER_IN in interface USER
```

Current entries:

```
access-list USER_IN extended permit udp 10.10.50.0 255.255.255.0 host 10.10.20.10 eq domain
access-list USER_IN extended permit tcp 10.10.50.0 255.255.255.0 host 10.10.20.10 eq domain
access-list USER_IN extended permit ip 10.10.50.0 255.255.255.0 any
```

| # | Rule | Purpose | Status |
|---|---|---|---|
| 1 | permit udp/tcp to 10.10.20.10 eq domain | DNS resolution against DC01 | Permanent |
| 2 | permit ip 10.10.50.0/24 any | Broad buildout permit for the development workstation | **Temporary.** Documented technical debt per ADR-0008. Removal trigger: end of Workstream 1, or sooner if the ASA becomes the true edge. |

History: the broad permit replaced three port-specific permits (www, https, ftp) added on 2026-05-11. The replacement reasoning and risk acceptance are recorded in ADR-0008; the risk is bounded because the double-NAT upstream means there is currently no inbound path from the internet.

### Other interfaces

**[VERIFY]** No ACLs on MGMT, SERVERS, or CLIENTS are recorded in the journals, which would mean those interfaces still run on implicit security-level policy. Confirm with `show running-config access-group` and document any entries found here.

## Known Deliberate Behaviors

- **ICMP is not permitted through the ASA.** Outbound web traffic works while ping fails; this is expected, not a defect (2026-05-06 journal). Open question: whether to permit ICMP scoped to MGMT for troubleshooting.
- **NAT** provides outbound internet for all production VLANs. The ASA is not yet the true edge; the network double-NATs through the upstream connection. This must be resolved before Workstream 3 remote access work.

## Re-Restriction Plan (ADR-0008 close-out)

At end of Workstream 1:

1. Inventory the flows the USER workstation actually needs (observed during buildout).
2. Remove `permit ip 10.10.50.0 255.255.255.0 any`.
3. Reinstate scoped permits for only those flows.
4. Record the reversal in the journal and update this document.

## Verification Commands

```
show running-config access-list
show running-config access-group
show nameif
show running-config interface
```

Re-run these after any ACL change and reconcile this document against the output. The running config is the source of truth; this document explains why each rule exists.
