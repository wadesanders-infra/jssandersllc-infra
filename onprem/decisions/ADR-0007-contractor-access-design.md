# ADR-0007: Pre-Create Contractor Accounts Disabled With Scoped Access and Expiration Backstops

**Date:** 2026-06-03
**Status:** Accepted
**Workstream:** 1

## Context

The business engages short-term contractors who need access to their own project documents but must never reach financials, surveillance, or other sensitive resources. Contractor access is intermittent: an account is needed for the duration of an engagement and should not be active outside it. The question was how to provision and manage these accounts so that access is correctly scoped, never left active when it should not be, and reliable to bring online when an engagement starts.

## Options Considered

### Option A: Create contractor accounts on demand, delete when the engagement ends
- Provision a fresh account, OU placement, GPO, and scoped permissions each time a contractor starts, then remove it when they leave.
- Pros: No dormant accounts sitting in the directory between engagements.
- Cons: Every engagement repeats the full setup, often under time pressure when a contractor needs access now. Doing scoped-permission and GPO configuration in a rush is exactly when mistakes happen: a wrong group, a missed restriction, an over-broad grant. The risky work is repeated at the worst possible moment, every time.

### Option B: Pre-create accounts disabled, enable per engagement, with expiration and scoping built in
- Create the contractor accounts now, fully configured (Contractors OU, restricted GPO, scoped to SG-Contractor-Projects, no access to financials or surveillance), but disabled. Enable per engagement and set an expiration date at activation.
- Pros: The account exists exactly the way I want it before it is ever needed. The scoped permissions, OU, and GPO are designed and verified once in calm conditions, so the per-engagement action is a single low-risk step (enable, set expiration) rather than error-prone setup under time pressure. Disabled-by-default means the resting state is safe.
- Cons: Dormant disabled accounts exist in the directory between engagements (a minor, well-controlled footprint).

## Decision

Contractor accounts are pre-created disabled, fully configured with scoped access, and enabled only for the duration of an engagement with an expiration date set at activation. I wanted the system in place from the start: the accounts exist the way I want them and I turn them on when I need them. If I had to create an account in a rush there is no telling what mistakes I might make, so the design moves all the risky configuration work to a calm, one-time setup and leaves only safe activation for the moment of need. Two accounts exist: one as the primary contractor account and a second as planned spare capacity in case a second contractor is needed at the same time.

## Consequences

- Access is layered, not dependent on a single control. An account is disabled by default, scoped by group membership and a restricted GPO, and carries an expiration date once active. Even if I forget to disable an account when an engagement ends, the expiration date disables it as a backstop, so an account is never left silently active.
- The per-engagement workflow is low-risk: enable the account and set an expiration, rather than build and scope an account from scratch. The risky work is not repeated under time pressure.
- Contractor access is confined to their own project documents (SG-Contractor-Projects) and explicitly excludes financials and surveillance, which traces to the data classification model. A contractor's reach is defined before they ever log in.
- The Dell Latitude E6500 serves as the contractor workstation, placed in the contractor OU with the restricted GPO and scoped file share access, so the endpoint matches the account's restrictions.
- Two accounts is deliberate excess capacity. If concurrent contractors are ever needed the second account is ready; if not, it stays disabled and costs nothing.
- Dormant disabled accounts are an accepted, minor footprint. At this scale they are trivial to audit (the Contractors OU shows exactly what exists and its state), which is the trade for never doing rushed provisioning.
