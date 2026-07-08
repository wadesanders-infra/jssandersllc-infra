# Risk Register

Standing record of consciously carried risk, adopted in ADR-0014. Every open row is a risk I have accepted on purpose, with a named exit: the condition under which it stops being acceptable and gets closed. Checked at the start of every staged session; a fired trigger outranks new work. Closing a risk means moving its row to Closed with a date and a journal link, never deleting it.

## Open risks

| ID | Risk | Source | Accepted | Exit trigger | Status |
|---|---|---|---|---|---|
| RISK-001 | Broad USER VLAN permit (`permit ip 10.10.50.0 255.255.255.0 any`); compromise of one desktop reaches every internal segment | ADR-0008; journal 2026-05-11 | 2026-05-11 | End of Workstream 1, or immediately if the ASA becomes the true edge | Open |
| RISK-002 | ASA is not the true network edge (double-NAT through upstream home network); perimeter assumptions rest on equipment outside my control | onprem/artifacts/asa-acl-ruleset.md; network topology diagram | [CONFIRM: no dated acceptance on record; present since foundation] | Resolve before Workstream 3 remote access work | Open |
| RISK-003 | Surge protector rocker switch can cut power to the entire stack on incidental contact; occurred once | journal 2026-05-07 | 2026-05-07 | [CONFIRM: define the fix, non-switchable power path or relocation, and a date] | Open |
| RISK-004 | DC01 runs on consumer hardware without battery backup, with disk write-cache warnings (event 80040020); an unclean power loss could corrupt the AD database. Fallback: Hyper-V checkpoint from 2026-06-11 | journal 2026-06-11 (formal acceptance) | 2026-06-11 | UPS acquired and installed [CONFIRM: budget trigger] | Open |

## Interactions

These do not stand alone. RISK-001's acceptance is explicitly conditional on RISK-002 continuing to hold (no inbound path from the internet); if the double-NAT is resolved, RISK-001's trigger fires the same day. RISK-003 is a realized instance of the hazard class RISK-004 accepts: a switch bump plus cached writes is the AD corruption scenario, so fixing RISK-003 partially mitigates RISK-004 while the UPS remains unaffordable.

## Pre-approved standard changes

Changes on this list skip review and get a journal line only. A change type earns its place here after two clean normal-tier runs (ADR-0014).

- Hyper-V checkpoints, and reverting to a checkpoint taken in the same session
- Test objects created in a designated test OU and removed in the same session
- Memory or CPU adjustments on non-domain-controller VMs

## Closed risks

None yet.
