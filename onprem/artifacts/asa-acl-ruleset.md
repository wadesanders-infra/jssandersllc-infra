# ASA ACL Ruleset Documentation

**Last updated:** 2026-07-18
**Device:** Cisco ASA 5506-X (perimeter firewall, inter-VLAN router)
**Status:** Verified against the running config 2026-07-18 (WS1 verification runbook, Phases 1 and 2). Evidence: the 2026-07-18 journal and `2026-07-18-asa-sg350-verification-output.md` in this directory (transcribed console output; PuTTY logging was not enabled). The verification surfaced one undocumented ACL (CLIENTS_IN, below). The running config remains the source of truth.

## Enforcement Model

The ASA is the single enforcement point for inter-VLAN traffic (router-on-a-stick over the 802.1Q trunk to the SG350; the switch does L2 tagging only). Two layers govern what passes:

1. **Security levels (implicit policy).** Traffic from a higher security level to a lower one is permitted by default; lower to higher is denied by default.
2. **Interface ACLs (explicit policy).** The moment an ACL is applied to an interface, the implicit security-level behavior on that interface is replaced by the ACL, which ends in an implicit deny.

The steady-state philosophy is default-deny between segments with explicit least-privilege permits (ADR-0003). One documented temporary exception is in effect (ADR-0008, below).

## Interfaces and Security Levels

| Interface | VLAN | Subnet | Security Level | Source |
|---|---|---|---|---|
| OUTSIDE | - | upstream (double-NAT) | 0 | verified 2026-07-18 (`show nameif`) |
| MGMT | 10 | 10.10.10.0/24 | 90 | verified 2026-07-18 (`show nameif`) |
| SERVERS | 20 | 10.10.20.0/24 | 80 | 2026-05-11 journal; verified 2026-07-18 |
| CLIENTS | 30 | 10.10.30.0/24 | 70 | verified 2026-07-18 (`show nameif`) |
| USER | 50 | 10.10.50.0/24 | 60 | 2026-05-11 journal; verified 2026-07-18 |

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

History: the broad permit replaced three port-specific permits (www, https, ftp) added on 2026-05-11. The replacement reasoning and risk acceptance are recorded in ADR-0008; the risk is bounded because the double-NAT upstream means there is currently no inbound path from the internet. Verified unchanged 2026-07-18.

### CLIENTS_IN (inbound on CLIENTS interface): found during verification, previously undocumented

Applied with:

```
access-group CLIENTS_IN in interface CLIENTS
```

Entries as found 2026-07-18:

```
access-list CLIENTS_IN extended permit udp 10.10.30.0 255.255.255.0 host 10.10.20.10 eq domain
access-list CLIENTS_IN extended permit tcp 10.10.30.0 255.255.255.0 host 10.10.20.10 eq domain
access-list CLIENTS_IN extended permit ip 10.10.30.0 255.255.255.0 any
```

| # | Rule | Purpose | Status |
|---|---|---|---|
| 1 | permit udp/tcp to 10.10.20.10 eq domain | DNS resolution against DC01 | Permanent |
| 2 | permit ip 10.10.30.0/24 any | Broad permit mirroring the USER buildout rule | **Temporary and previously undocumented** (2026-07-18 finding F1). No journal or ADR recorded its creation. It explains CLIENTS-to-SERVERS reachability: the ACL replaces security-level policy (70 to 80 would otherwise deny). Retired together with the USER broad permit in the new-era ACL rebuild (ADR-0008 amendment, ADR-0014). |

### Other interfaces

Verified 2026-07-18 (`show running-config access-group`): only CLIENTS_IN and USER_IN exist. MGMT and SERVERS carry no ACLs and run on implicit security-level policy (MGMT 90 and SERVERS 80 may initiate to lower levels; nothing lower may initiate to them).

## Known Deliberate Behaviors

- **ICMP is not permitted through the ASA.** Outbound web traffic works while ping fails; this is expected, not a defect (2026-05-06 journal). Open question: whether to permit ICMP scoped to MGMT for troubleshooting.
- **NAT** provides outbound internet for all production VLANs. The ASA is not yet the true edge; the network double-NATs through the upstream connection. Verified 2026-07-18: four object-network dynamic interface PATs (MGMT, SERVERS, CLIENTS, USER to outside), no static or inbound entries.
- **DHCP relay** verified 2026-07-18: `dhcprelay server 10.10.20.10 SERVERS`, relay enabled on CLIENTS and on USER (the USER relay was previously unrecorded), timeout 60.

## Re-Restriction Plan (ADR-0008 close-out)

*(Amended 2026-07-18 per ADR-0014 and the ADR-0008 amendment: the plan below now covers both broad permits, USER_IN's documented one and CLIENTS_IN's undocumented one (F1), and executes as part of the new era's full ACL rebuild rather than as a standalone end-of-WS1 reversal.)*

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
