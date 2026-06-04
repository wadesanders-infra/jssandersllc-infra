# ADR-0006: Centralize Service Accounts in a Dedicated OU Instead of Placing Them With Their Servers

**Date:** 2026-06-03
**Status:** Accepted
**Workstream:** 1

## Context

The project uses dedicated service accounts for its application services (svc-wazuh, svc-paperless, svc-entraconnect). These accounts need a consistent, restrictive policy: non-expiring passwords, no interactive logon, and no RDP, so a compromised service credential cannot be used to log in like a person. The question was where these accounts should live in the OU structure: gathered in one dedicated OU, or distributed alongside the servers and applications they each serve.

## Options Considered

### Option A: One centralized Service Accounts OU
- All service accounts live in a single dedicated OU, regardless of which application or server uses them.
- Pros: One GPO applied to one OU covers every service account uniformly. Auditing is trivial because every service account is in one place; I know all my services are here. Adding a new service account later means dropping it into the existing OU and it inherits the restrictive policy automatically.
- Cons: The OU does not mirror the application/server topology, so there is no structural link between an account and the host it serves (that relationship lives in documentation and naming instead).

### Option B: Service accounts placed with their servers/applications
- Each service account sits in the same OU as the server or application it belongs to, following the workload around the structure.
- Pros: Structurally co-locates an account with what it serves. This is the kind of organization that helps when delegation is in play and different admins own different application areas.
- Cons: The deny-logon policy would have to be applied across multiple OUs or duplicated, and auditing service accounts means hunting through the structure rather than reading one OU. At this scale, with no delegation and a handful of accounts, the co-location buys nothing and the scattered policy application adds complexity and room for inconsistency.

## Decision

Service accounts are centralized in a single dedicated Service Accounts OU rather than distributed with their servers. At this scale the centralized approach simply functions better and is easier to manage; the nesting/co-location principles that pay off as an environment grows add unneeded complexity here. This is the same design philosophy applied throughout the foundation: the environment is built so that services and functions can be added easily, while the structures that only become necessary as headcount scales (nesting, delegation-oriented placement) are deliberately left out. A single restrictive GPO, GPO-ServiceAccounts-DenyLogon, applies deny-local-logon and deny-RDP to the whole OU at once.

## Consequences

- One GPO governs every service account. The deny-logon and deny-RDP policy is applied uniformly and cannot drift between accounts, because there is only one place it is applied.
- Auditing is straightforward: every service account is in one OU, so answering "what service accounts exist and what is their policy" is a single look rather than a structure-wide search.
- The accounts were pre-created with the restrictive policy already in place, before the applications that use them (Wazuh, Paperless-ngx, Entra Connect) are deployed in later workstreams. This means each service credential is locked down from the moment its application comes online, rather than the lockdown being applied after the fact. The secure state precedes the service.
- New service accounts added in future workstreams inherit the OU policy automatically just by being placed there, so onboarding a new service does not require re-deciding or re-applying the security baseline.
- The structural link between an account and the host it serves lives in naming (svc-<service>) and documentation rather than in OU placement, which is an acceptable trade given there is no delegation that would benefit from co-location.
