# ADR-0001: Use ad.jssandersllc.org as the AD Forest Root Over a .local or Fabricated Domain

**Date:** 2026-06-03
**Status:** Accepted
**Workstream:** 1

## Context

J.S. Sanders LLC needed an Active Directory forest root name before the domain controller could be promoted. The name is effectively permanent; renaming an AD forest after the fact is disruptive and risky, so the choice had to be right at promotion time. I also wanted the naming to reflect current enterprise practice rather than a dated convention, so the decision would hold up under review by someone who has dealt with domain-naming consequences at scale.

## Options Considered

### Option A: A `.local` (or similar non-routable) internal domain
- Promote the forest as something like `jssanders.local`, a name with no relationship to any real public domain.
- Pros: No domain purchase required. Historically common, so plenty of older documentation assumes it.
- Cons: `.local` collides with mDNS/Bonjour and is no longer recommended. A public certificate authority will not issue certificates for a `.local` name, which forecloses clean TLS for internal services later. It also invites split-brain DNS confusion if the business ever runs a public presence.

### Option B: A subdomain of a real, owned public domain (`ad.jssandersllc.org`)
- Purchase `jssandersllc.org` and use the `ad.` subdomain as the internal forest root, while adding `jssandersllc.org` as a UPN suffix so accounts read as `user@jssandersllc.org`.
- Pros: No split-brain DNS, because the internal forest lives under a subdomain that is delegated/separate from anything published publicly. A public CA can issue certificates against names under the owned domain. UPN suffix gives clean, professional user principal names. Matches current Microsoft guidance.
- Cons: Requires owning a real domain (a small recurring cost).

A fabricated public-style name (e.g. inventing a domain I do not own) was never seriously considered, because using a domain you do not control risks colliding with a real registered domain and cannot back real certificates. The genuine choice was between `.local` and a subdomain of an owned domain.

## Decision

The forest root is `ad.jssandersllc.org`, a subdomain of `jssandersllc.org`, a domain purchased specifically for this buildout and not used for anything else. I chose to buy a real domain to keep the naming clean and to avoid the `.local` pitfalls outright rather than work around them later. `jssandersllc.org` is added as a UPN suffix so user accounts read as `user@jssandersllc.org`. The environment is small enough that a `.local` domain would have functioned, but the point of this project is to demonstrate that I understand the enterprise-scale consequences of naming decisions even when my environment does not force the issue.

## Consequences

- Public certificate authorities can issue certificates for names under the owned domain, so internal services can use clean TLS later without self-signed-cert workarounds.
- No split-brain DNS to manage: the internal forest lives under `ad.`, distinct from any public zone, so internal and external resolution do not conflict.
- User accounts get professional UPNs (`user@jssandersllc.org`), which also keeps on-prem identities consistent with how they will project into a cloud tenant in Workstream 3.
- Accepts a small recurring domain-registration cost, which is the trade-off for avoiding the `.local` limitations.
- The forest name is now fixed; this decision is foundational and everything built on the domain inherits it.
