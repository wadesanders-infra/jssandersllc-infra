# ADR-0010: Use Role-Neutral, Class-Based Sequential Hostnames; Express Role Through OU Placement

**Date:** 2026-06-12
**Status:** Accepted
**Workstream:** 1

## Context

Both business laptops were domain-joined under their default Windows hostnames and left in the default Computers container. Correcting them forced a naming-convention decision before more endpoints join the domain. The first correction attempt named the field laptop JSS-FIELD01, which bakes the machine's business role into its hostname, and that choice prompted a reconsideration of what a hostname should carry.

## Options Considered

### Option A: Role-based names (JSS-FIELD01, JSS-CONTRACTOR01)
- The hostname declares the machine's business role.
- Pros: Self-documenting at a glance; no inventory lookup needed.
- Cons: Roles change, and the name becomes wrong the moment one does. Fixing it means a rename, and rename cost scales with the machine's importance (a workstation rename is one command; the DC rename was a two-hour supported procedure). Couples identity to policy.

### Option B: Role-neutral class and sequence (JSS-WSNN; servers DCNN / SRVNN)
- The hostname identifies the asset class and a sequence number only. Role is expressed through OU placement and GPO.
- Pros: Names stay accurate through role changes. Identity (what the machine is) and policy (what it is allowed to do) stay cleanly separated. Matches common enterprise practice.
- Cons: The name alone says nothing about role; an asset inventory has to carry the mapping.

### Option C: User- or location-based names
- The hostname ties to the assigned user or physical site.
- Pros: Easy to identify whose machine or where it lives.
- Cons: Users and sites change even more often than roles, and in a field operation machines move between sites routinely. Every reassignment forces a rename or leaves a lying name.

## Decision

All machines use role-neutral, class-based sequential hostnames: workstations are `JSS-WSNN`, and servers follow the class-plus-sequence pattern the existing names already conform to (DC01, SRV01). The deciding reason is that a machine's role can change while its name should not have to: role lives in OU placement and GPO, identity lives in the hostname. The laptops were renamed JSS-WS01 (field) and JSS-WS02 (contractor) under this convention, and the Workstation Build and Domain-Join SOP encodes it as the standard for every future endpoint.

## Consequences

- A role change becomes an OU move instead of a rename. Policy follows the object, not the name.
- The name-to-asset mapping needs a formal home. None exists yet; the mapping will be recorded in the Eramba asset inventory when it deploys. Until then the workstation SOP and AD OU placement are the de facto record. This is a known gap, not an oversight.
- The convention constrains future naming decisions: new asset classes get a class prefix and sequence, not a descriptive role name.
- JSS-FIELD01 existed for less than a session and was corrected before anything depended on it.
