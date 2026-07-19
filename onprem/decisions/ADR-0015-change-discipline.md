# ADR-0014: Adopt Risk-Tiered Change Discipline and a Standing Risk Register

**Date:** 2026-07-08
**Status:** Accepted
**Workstream:** All (adopted during WS1 close-out)



## Context

Two change-class failures are already on record. The broad USER VLAN permit went in under troubleshooting pressure on 2026-05-11, and its re-restriction survives on memory alone; the 05-11 journal itself notes that without a concrete trigger the deviation becomes permanent (ADR-0008 later supplied one). Separately, the domain controller ran under its default hostname from promotion until verification caught it on 2026-06-08, forcing a genuinely risky rename that still carries residue. Accepted risks now live scattered across ADRs, journals, and artifact docs with no single live view, rollback planning appears in exactly one journal entry, and as the only operator I have strong hindsight mechanisms (journals, SOPs, the verification runbook) but nothing that challenges a change before it runs.

## Options Considered

### Option A: Full change-ticket process (CHG records and CAB-style approval per change)

- Every change gets its own numbered record and a formal approval workflow, mirroring production ITSM practice.
- Pros: Closest replication of how change control works in a real organization; substantial portfolio artifact.
- Cons: Ceremony disproportionate to a two-VM, single-operator network. Adds a fourth artifact class alongside ADRs, journals, and SOPs. High abandonment risk, and an abandoned process is worse evidence of judgment than no process.

### Option B: Status quo

- Continue with journals, ADRs, staged work packages, and the verification runbook as they are.
- Pros: Zero added friction. The existing system did catch the naming miss.
- Cons: It caught the naming miss ten days late, after DFSR had baked in the wrong name. Expiry of accepted risk rests entirely on memory. The discipline degrades exactly when I am tired or frustrated, which is when the 05-11 shortcut happened.

### Option C: Thin extension of the existing session flow

- Changes are tiered (standard, normal, emergency). Normal-tier changes require written blast radius, rollback, and verification before execution, plus an adversarial review of the staged package against a fixed rubric. Any accepted risk gets a named exit trigger in a standing register. Repeatable changes graduate to a pre-approved standard list after two clean runs.
- Pros: Codifies what the DC rename session already practiced instinctively (checkpoint first, defer high-risk work when tired, verify after). One new artifact only (the register). Friction decays as competence is demonstrated.
- Cons: Review quality depends on the rubric having teeth. Enforcement is visibility only, since a solo operator can override himself. Roughly fifteen minutes of overhead per normal-tier session.

## Decision

Production changes run under Option C: normal-tier changes require written blast radius, rollback, and verification plus a rubric-based adversarial review before execution; every accepted risk carries a named exit trigger in `risk-register.md` at the repo root; standard changes graduate to a pre-approved list after two clean normal-tier runs. The review is performed by the LLM assistant against the staged package file only, under a fixed rubric with mechanical blocks, and my overrides of a denial are recorded rather than forbidden. This fits the actual failure evidence without taxing session time, which is the binding constraint.

## Consequences

- The register becomes the single live view of carried risk, checked at the start of every staged session. The ADR-0008 re-restriction trigger, the double-NAT constraint, and both power risks stop depending on memory.
- Rollback becomes a precondition for normal-tier changes instead of an instinct that shows up on good days.
- Emergency means service restoration only, with a retroactive record next session. The tell for misuse is an emergency that was not restoring anything.
- Normal tier covers GPO, AD, DNS, DHCP, ASA, Wazuh, hypervisor networking, domain joins and renames, and share or NTFS permission changes. The lab replica, once built, inherits the discipline at the promotion boundary: rehearsal inside the replica is free, promoting a change to production is not.
- Kill criterion, stated up front: if reviews start rubber-stamping or I start batching changes to dodge the process, the discipline gets thinned or retired by a superseding ADR rather than quietly ignored.
- Follow-ups: seed the register with the four risks already on record; adopt the review workflow in session staging.
