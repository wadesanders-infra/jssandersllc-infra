# ADR-0014: End the Production Era and Continue the Flagship as a Synthetic Healthcare Range

**Date:** 2026-07-18
**Status:** Accepted
**Workstream:** 1

## Context

Workstream 1 built what it set out to build: a segmented network, a domain with enforced policy, managed endpoints, and documentation that held up under a verification session. I am leaving the company, and no one remains to operate, maintain, or extend the environment; infrastructure without an operator does not serve a business, it slowly becomes a risk to one. My next phase of work (detection engineering, identity security) requires day-to-day activity, identity lifecycle, and security signal, plus attack and failure scenarios no production network should host. This ADR supersedes ADR-0011 along the way (a replica of a static environment reproduces its stillness) and decides what the flagship becomes once the era closes verified.

## Options Considered

### Option A: Continue the production build-out as planned
- Finish the remaining WS1 items (Wazuh, Eramba, tiered admin), then run Workstreams 2 through 4 against production plus the ADR-0011 replica.
- Pros: Original mission intact. The real-stakes narrative keeps growing. No rework.
- Cons: Builds out systems for a company I am leaving, with no one to operate or maintain them after my departure. Every security exercise still requires building the replica first, doubling the build effort. Months of work remain before any security artifact exists.

### Option B: Freeze this repo and start a separate range repo
- Tag the JSS work as finished and build a cyber range in a fresh repository.
- Pros: Clean separation of eras. The new docs carry no legacy baggage.
- Cons: Splits one story across two repos. The new repo starts with none of the accumulated evidence of working discipline. Reviewers see fragments instead of an arc.

### Option C: End the production era in place; continue the same repo as a synthetic healthcare range
- Close WS1's verification debt via the runbook, re-baseline the exit criteria honestly, tag `production-era-final`, then rebaseline the repo around Genetix Health Group, a clearly fictional precision-medicine company on the same hardware.
- Pros: One continuous arc with the history preserved as evidence. Hardware reused. Synthetic personas solve the no-activity problem. Attack simulation, deliberate misconfiguration, and identity lifecycle failures become possible, which production forbids.
- Cons: Production consequence pressure is lost and must be replaced by self-imposed discipline. The founding theft-response mission is formally retired unfulfilled. Part of the story becomes fiction and must be disclosed as such.

## Decision

The flagship continues as the Genetix Health Group synthetic healthcare range, and the production era ends with Workstream 1's verification session rather than with its full exit criteria. The build is complete and proven, and with my departure there is no operator to carry it further; closing the era verified beats leaving it to decay unmaintained. The range carries the same hardware and the same discipline forward into an environment that generates the activity, telemetry, and failure modes the next phase of my work requires. Verification still gates the close, because the era's value is that its claims check out.

## Consequences

- The WS1 exit criteria in `outline.md` are re-baselined: verification items close via the runbook; every unbuilt item is marked retired or carried forward with its destination. Nothing is silently dropped.
- ADR-0008's re-restriction trigger is amended: the broad USER permit comes out when the new era rebuilds the ACL set for its zones, not as a standalone reversal. Its risk condition (no inbound internet path) must continue to hold until then.
- ADR-0011 is superseded. The range replaces the replica, and the rebuild-from-docs test transfers to the new era's scripted deployment.
- DVR-to-SIEM integration and Eramba's WS1 deployment are retired. The theft that started the project remains uninvestigated; what it taught shapes what gets built instead.
- After the verification reconciliation commit, the repo is tagged `production-era-final`. The Genetix rebaseline (ADR batch, README, outline) follows as the new era's first work.
