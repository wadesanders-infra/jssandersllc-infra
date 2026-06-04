# ADR-0005: Use a Flat Security Group Structure Instead of AGDLP Nesting

**Date:** 2026-06-03
**Status:** Accepted
**Workstream:** 1

## Context

With the production users created, access to resources (documents, surveillance, contractor project files) needed to be governed by security groups. The Microsoft-recommended model for group structure is AGDLP: accounts go into global groups, global groups nest into domain local groups, and permissions are assigned to the domain local group. AGDLP is built to manage access cleanly at scale and across multiple domains. The question was whether to implement that nesting in a two-user, single-domain environment with no delegated administration, or to assign permissions through a single flat layer of groups.

## Options Considered

### Option A: Flat security groups
- One layer of groups, each tied to a resource and access level (for example SG-Documents-FullControl, SG-Documents-Read), with user accounts placed directly into them and permissions assigned to those groups.
- Pros: The blast radius of any group is visible at a glance; you read the membership and you know exactly who can do what. With no delegation and two real users, there is nothing for nesting to simplify. Naming groups by resource and access level keeps the flat set organized and self-documenting.
- Cons: Does not follow the AGDLP standard. Would not scale gracefully to a large, multi-domain, or delegated environment, where the lack of nesting would become a management burden.

### Option B: AGDLP nesting
- Implement the full nested model: global role groups nested into domain local resource groups, permissions on the domain local groups.
- Pros: The Microsoft best practice. Scales cleanly, supports delegation and multi-domain growth, and decouples "who is in a role" from "what a resource grants."
- Cons: At two users with no delegation and a business that will not grow into multiple domains or large headcount, the nesting is pure indirection. It adds layers to trace through with no corresponding benefit, making access harder to audit by eye rather than easier. The payoff AGDLP is designed for never arrives in this environment.

## Decision

Production uses a flat security group structure rather than AGDLP nesting. There is no delegation in place, and the production environment will never grow to the size where nesting becomes necessary, so the indirection AGDLP adds would cause more trouble than it solves at this scale. This is consistent with the earlier intention to build a structure that *expands* cleanly: I can add new groups without restructuring, but I deliberately do not pre-build *complexity* that only earns its keep at a scale this business will not reach. Expandable is not the same as over-engineered.

## Consequences

- Access is auditable by direct inspection. Reading a group's membership tells you its full blast radius, with no nested layers to chase, which is the right trade for an environment small enough to verify by eye.
- The group naming convention (resource plus access level, with a consistent prefix) keeps the flat set organized and makes each group's purpose obvious from its name. This is good practice independent of structure.
- If the business ever did grow substantially, adopting nesting later would mean reworking group membership and permission assignment. That cost is accepted, because the growth that would justify it is not realistically going to happen for this business.
- The lab is the place where nesting will be demonstrated. The expanded lab user population includes nested group structures precisely because attack simulation needs the conditions that nesting (and the privilege paths it can create) produces at scale. The lab will show why AGDLP and nesting are the right call in a large environment, which is the counterpart to this decision: flat is correct for production, nested is correct for the scale the lab simulates. That contrast will be documented when the lab population is built.
