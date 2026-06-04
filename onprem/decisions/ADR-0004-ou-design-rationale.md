# ADR-0004: Structure Production AD OUs Around Real Business Roles With No Simulated Accounts

**Date:** 2026-06-03
**Status:** Accepted
**Workstream:** 1

## Context

Once the domain controller was promoted, the production Active Directory needed an organizational unit structure. OUs in AD primarily serve GPO targeting and delegation of administration. This environment has no delegation (I hold full administrative control) and a genuinely small user population, so the structure had to earn its keep through clean organization and GPO targeting rather than through an administrative hierarchy the business does not have. A second question ran alongside it: whether production AD should contain only the real business, or be padded with additional accounts and groups to make the environment look larger and create richer attack paths.

## Options Considered

### Option A: Production OUs reflect the real business; simulated complexity lives only in the lab
- Build OUs around the actual roles and object types the business has, and keep production AD populated exclusively with real accounts. Any simulated users, SPNs, nested groups, and attack paths are created only on the cloned DC in the lab.
- Pros: Production stays honest. Every object exists to serve a real purpose, and a reviewer can trust that the production AD reflects the actual business rather than a staged scenario. The lab carries all the manufactured complexity, which is where scale and attack-path testing belong. Cleanly separates "the real environment" from "the test environment."
- Cons: Production AD is small and looks small. It does not, on its own, demonstrate the kind of sprawling structure that attack simulation exercises.

### Option B: Pad production AD with simulated users and groups
- Add fabricated accounts, service principals, and nested group structures to production AD to make it larger and to create the conditions for attack simulation directly in production.
- Pros: A single environment that looks bigger and supports attack-path testing without a separate lab.
- Cons: Dishonest. Production would no longer reflect the real business, which undermines the entire premise that this is real infrastructure serving a real company. It also mixes test artifacts into the production trust boundary, which is exactly what the lab exists to avoid.

## Decision

Production OUs are structured around the real business and its actual object types, and production AD contains only real accounts. The structure (Users with Employees and Contractors beneath it, Service Accounts, Workstations, Servers, and SyncUsers) is driven by clean organization and GPO targeting. I hold full permissions across it; everyone else has read, and at most execute, scoped to the isolated sections relevant to them. Production reflects the business as it actually is, and the lab exists separately as the environment for testing scale and attack paths. The Service Accounts OU and the SyncUsers OU appear in the structure here, but their specific rationale is recorded in their own ADRs (service account placement, and Entra Connect sync scoping) rather than repeated in this one.

## Consequences

- GPOs can be targeted cleanly at the right object types (workstation policy at Workstations, server policy at Servers, contractor restrictions at Contractors) without fighting an over- or under-divided structure.
- Production AD is small and honest. Its credibility comes from reflecting the real business, not from looking large, and a reviewer can take every object at face value.
- All simulated users, SPNs, and nested-group attack paths are confined to the cloned DC in the lab (the LabUsers OU), so production is never polluted with test constructs. This reinforces the production/lab separation recorded in the lab isolation ADR.
- The permission model (full control for me, read/execute for others in scoped sections) is simple to audit at this scale because the blast radius of any grant is easy to see directly.
- The structure is expandable if the business grows; new roles or object types get new OUs without restructuring the existing ones.
