# ADR-0003: Segment the Production Network Into Four VLANs With a Default-Deny ACL Philosophy

**Date:** 2026-06-03
**Status:** Accepted
**Workstream:** 1

## Context

Before this project the business ran on a flat network: a user, a contractor, and a server all sat in the same broadcast domain with no separation and no visibility into who could reach what. The foundation needed network segmentation so that traffic between classes of device could be controlled and logged. The questions were how many segments the environment actually called for, how to enforce traffic between them, and where my own personal/lab machine should sit relative to the business network.

## Options Considered

### Option A: Minimal practical segmentation: four VLANs
- Split the network into MGMT (10), SERVERS (20), CLIENTS (30), and USER (50), with the ASA as the inter-VLAN router and enforcement point and the SG350 doing Layer 2 tagging only.
- Pros: Separates the device classes that genuinely have different trust and access needs (management, servers, business endpoints, my personal/lab machine) without inventing segments the environment does not need. Each VLAN maps to a real purpose. Easy to reason about and to expand later if a real need appears.
- Cons: Four segments is the floor, not a richly subdivided design; some larger environments would break these down further (separate VLANs per device role, guest, IoT, etc.). If a future need appears it means adding VLANs rather than having them pre-built.

### Option B: More granular segmentation up front
- Pre-build additional VLANs (for example separate IoT, guest, or per-role segments) in anticipation of future needs.
- Pros: Headroom; segments exist before they are needed.
- Cons: Segments with nothing in them are overhead, not security. They add ACL surface and management burden with no current justification, and at this scale they would be documentation theater rather than real controls. The environment does not call for them today.

## Decision

The production network is segmented into four VLANs (MGMT, SERVERS, CLIENTS, USER), which is the bare minimum the environment actually calls for; if a real need appears I can add more rather than carrying empty segments now. The USER VLAN exists specifically for my personal computer, which currently shares some of this infrastructure and will host the isolated lab later. I put it on its own VLAN deliberately so that machine does not pollute the rest of the business network. Inter-VLAN traffic is enforced by the ASA using security levels, where traffic from a lower security level to a higher one is denied by default, with explicit ACLs permitting only the flows each segment legitimately needs. The intended steady state is default-deny between segments with least-privilege permits.

## Consequences

- Each device class is isolated and inter-VLAN traffic is controlled at the ASA, so a compromise or misbehavior in one segment does not have free reach into the others.
- The USER VLAN keeps my personal/lab machine off the business segments, which also gives the lab a clean separation story even before its dedicated isolated vSwitch is considered (lab isolation is recorded separately).
- The four-VLAN design is intentionally expandable. Adding a segment later is straightforward; the cost of starting minimal is that future needs require adding VLANs rather than populating pre-built ones, which is an acceptable trade.
- VLAN 40 does not exist in the production network. It was cut after the infrastructure was already built, and restructuring everything to reclaim the number was not warranted for what this environment is doing. The gap in numbering is deliberate and documented here so a reviewer is not left wondering. The number may be repurposed later. If skipping the restructure turns out to cause a problem down the line, that will be captured honestly in the journal.
- The USER VLAN currently runs a deliberately permissive ACL to ease diagnostics during the buildout, which is acknowledged as not least-privilege and is part of why that machine is isolated on its own VLAN. That temporary exception and the plan to restrict or remove it are recorded in their own ADR; the steady-state philosophy for the network as a whole remains default-deny with least-privilege permits. Not circling back on temporary exceptions is exactly how technical debt and vulnerabilities accumulate, so the reversal is tracked rather than assumed.
