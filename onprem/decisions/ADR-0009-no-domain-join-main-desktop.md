# ADR-0009: Do Not Domain-Join the Main Desktop

**Date:** 2026-06-03
**Status:** Accepted
**Workstream:** 1

## Context

The primary desktop (i7-12700K) has the most compute of any machine available and is used occasionally to assist the buildout, in addition to being my personal computer. It sits on the USER VLAN. Every other endpoint in the environment (the business workstations, the contractor laptop) is being domain-joined so it falls under centralized identity, GPO, and management. The question was whether the desktop should be joined to the domain like the other endpoints, or kept off it.

## Options Considered

### Option A: Domain-join the desktop like the other endpoints
- Join the desktop to ad.jssandersllc.org so it is centrally managed under GPO and AD authentication.
- Pros: Consistent with how every other endpoint is handled. The machine would fall under the same policy and management as the rest of the environment.
- Cons: The desktop is my personal device. Joining it would place a personal machine inside the company's identity and management boundary, subjecting personal use to domain policy and pulling a personal device into the business trust domain. It would also put the future lab host under production AD control, which works against keeping the lab cleanly separate.

### Option B: Keep the desktop off the domain
- Leave the desktop unjoined, authenticating locally, while it assists the buildout from the USER VLAN.
- Pros: Keeps a personal device out of the company infrastructure, which is the outcome I want. Avoids subjecting a personal machine to business domain policy. Also keeps the machine that will later host the isolated lab outside production AD, reinforcing the production/lab separation.
- Cons: The desktop is not centrally managed and is an exception to the otherwise-uniform domain-join approach for endpoints.

## Decision

The main desktop is not domain-joined, and will never be. The driving reason is that it is my personal device; I did not want to join a personal computer to the company infrastructure even though I use it to assist the buildout. A secondary, reinforcing reason is that this machine will later host the isolated lab, and keeping it out of production AD supports that separation. This is a permanent decision, not a temporary state.

## Consequences

- A personal device stays outside the business identity and management boundary. Personal use is not subject to domain policy, and the company's trust domain does not extend onto a machine I own and use personally.
- The desktop is a deliberate, documented exception to the uniform domain-join approach for endpoints. It is the only unjoined machine assisting the environment, and the reason is recorded here so the exception is not mistaken for an oversight.
- The future lab host is kept out of production AD from the start, which reinforces the lab isolation principle: the machine that will run cloned production VMs on an internal-only vSwitch is itself not a member of the production domain.
- Because the desktop is unmanaged by GPO, anything it needs (its permissive USER VLAN access during buildout, its tooling) is configured locally and at the network layer rather than through domain policy. The USER VLAN's isolation from the business segments is what contains this unmanaged machine rather than GPO.
- The decision is permanent. If the machine's role ever changed such that it stopped being a personal device, that would be a new decision requiring its own record; as the environment stands, it will not be joined.
