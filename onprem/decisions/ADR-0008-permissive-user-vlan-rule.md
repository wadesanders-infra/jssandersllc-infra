# ADR-0008: Run a Temporarily Permissive USER VLAN Rule During Buildout, Re-Restricted at End of Workstream 1

**Date:** 2026-06-03
**Status:** Accepted
**Workstream:** 1

## Context

During the foundation buildout I am doing most of the hands-on work from the desktop on the USER VLAN, using a variety of network tools to configure, test, and troubleshoot the rest of the environment. The default-deny ACL philosophy (recorded in the VLAN design ADR) means each flow has to be explicitly permitted, and during active buildout the set of destinations and ports I need keeps changing. The network is not yet exposed to the internet; the double-NAT through the upstream home network means the ASA is not the true edge and there is no inbound path from outside. The question was whether to keep building out narrow per-port permits as I go, or open the USER VLAN broadly for the duration of the buildout and re-restrict it once the foundation is done.

## Options Considered

### Option A: Maintain narrow least-privilege permits throughout buildout
- Add specific permits for each port and destination as the need arises, keeping the USER VLAN least-privilege at all times.
- Pros: Never deviates from least privilege. No cleanup step later.
- Cons: During active buildout the required flows change constantly. Every new tool or test means hitting a block, diagnosing whether it is the firewall or something else, and adding a permit before continuing. That overhead repeatedly interrupts production work for a risk that, while the network has no internet exposure, is low.

### Option B: Permit broadly during buildout, re-restrict at end of Workstream 1
- Replace the per-port permits with a single broad permit for the USER VLAN to remove firewall friction during buildout, then reinstate least-privilege rules once the foundation is complete.
- Pros: Removes the firewall as a source of friction and false leads while building, so troubleshooting time is spent on the actual problem rather than on chasing ACL blocks. The trade is acceptable specifically because the network is not yet internet-exposed, so the broad rule is not reachable from outside during the window it exists.
- Cons: Deviates from least privilege on the USER VLAN for the duration. Requires a disciplined re-restriction step, and if that step is skipped the deviation becomes permanent technical debt.

## Decision

The USER VLAN runs a single permissive rule (`permit ip 10.10.50.0 255.255.255.0 any`) for the duration of the foundation buildout, to be re-restricted to least privilege at the end of Workstream 1. The reasoning is that I wanted full access before the network is exposed to the internet, to speed up production and reduce blocks while building. The risk is bounded because the network currently has no inbound internet path, so this broad rule is not externally reachable during the window it exists. End of Workstream 1 is the defined trigger to switch it back, which keeps this from becoming open-ended debt.

## Consequences

- Buildout friction drops. The firewall stops being a source of repeated blocks and false leads during configuration and troubleshooting, so time goes to the real work rather than to ACL chasing.
- This is acknowledged technical debt with a defined payoff date. The re-restriction is tied to the end of Workstream 1, not to a vague "later," precisely because not circling back on temporary exceptions is how debt and vulnerabilities accumulate.
- The risk acceptance is explicitly conditional on no internet exposure. If the perimeter situation changes before Workstream 1 closes (for example if the double-NAT is resolved and the ASA becomes the true edge sooner than planned), the re-restriction has to happen at that point rather than waiting, because the condition that makes the broad rule acceptable would no longer hold.
- *(Corrected 2026-07-01; the original text here understated the exposure.)* The broad permit grants the USER VLAN reachability into every internal segment, not just outbound to the internet: an interface ACL replaces security-level policy, so `permit ip ... any` includes the business VLANs. The blast radius is one machine's compromise reaching everything internal, bounded during the window only by the absence of an inbound path from the internet. The trust and ACL flow diagram (`onprem/diagrams/trust-acl-flow.md`) shows this accurately.
- A concrete close-out task is created: at end of Workstream 1, remove the broad permit and reinstate least-privilege rules scoped to what the workstation actually needs, then record that reversal in the journal.
- *(Amended 2026-07-18 per ADR-0014.)* Workstream 1 closes with the production era, and the re-restriction closes with it in a different form: the broad permit is removed when the new era rebuilds the full ACL set for its zones, not as a standalone reversal against the old ruleset. The debt is not erased, its payoff moves to that rebuild, and the risk acceptance's condition (no inbound internet path) must continue to hold on the live ASA until the rebuild happens.
