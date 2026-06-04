# 2026-05-11: ASA Security Levels, Restoring Connectivity, and a Deliberately Permissive USER Rule

**Phase:** 1
**Duration:** Not recorded (did not properly notate session duration at the time; catching up on notes after the fact)
**Goal:** Resolve the connectivity problem left open from the previous session and get the USER VLAN desktop fully functional.

## What I Did

Picked up the connectivity issue from the prior session. The working theory this morning was ASA security levels: ACLs are not implemented yet, and traffic moving from a lower security level to a higher one is blocked by default. The USER interface is at level 60 and SERVERS is at level 80, so USER-to-SERVERS traffic from low to high would be dropped unless explicitly permitted.

Made some ASA config changes and ended up in a state where I had connectivity but could not actually do anything useful with it. Worked back from there and found the real cause: I had only ever permitted DNS from USER to DC01. Nothing else from the USER VLAN was allowed, so name resolution worked but no other traffic did, which is why earlier behavior looked broken in confusing ways.

Built out the USER inbound ACL. First the DNS permits and the access-group binding that were the starting point:

```
access-list USER_IN extended permit udp 10.10.50.0 255.255.255.0 host 10.10.20.10 eq domain
access-list USER_IN extended permit tcp 10.10.50.0 255.255.255.0 host 10.10.20.10 eq domain
access-group USER_IN in interface USER
```

Then added web and FTP egress to restore normal internet use:

```
access-list USER_IN extended permit tcp 10.10.50.0 255.255.255.0 any eq www
access-list USER_IN extended permit tcp 10.10.50.0 255.255.255.0 any eq https
access-list USER_IN extended permit tcp 10.10.50.0 255.255.255.0 any eq ftp
```

That restored connectivity. After some consideration I decided to replace the port-by-port permits with a single permissive rule for the desktop. The USER VLAN is my own development workstation and I am going to need a variety of network tools that will not fit neatly inside an HTTP/HTTPS/FTP allowlist. Removed the three port-specific permits and replaced them with a broad IP permit:

```
no access-list USER_IN extended permit tcp 10.10.50.0 255.255.255.0 any eq www
no access-list USER_IN extended permit tcp 10.10.50.0 255.255.255.0 any eq https
no access-list USER_IN extended permit tcp 10.10.50.0 255.255.255.0 any eq ftp
access-list USER_IN extended permit ip 10.10.50.0 255.255.255.0 any
```

This is a deliberate, temporary trade-off. The full reasoning and the plan to reimpose least privilege are in the ADR (see Open Questions). I will reinstate least-privilege rules on the USER VLAN once the infrastructure buildout is complete and I know exactly which tools and destinations the workstation actually needs.

Also decided against domain-joining the main desktop. Writing a separate ADR for the reasoning rather than burying it here.

## What I Learned

This is the concrete version of something I knew abstractly from cert material: on an ASA, security levels are an implicit policy. Traffic from a higher level to a lower level is permitted by default, low to high is denied by default, and the moment you apply an ACL to an interface the implicit behavior is replaced by what the ACL says (with an implicit deny at the end). My earlier "only DNS permitted" state was a perfect illustration: resolution worked, everything else silently died, and the symptom (some things work, most do not) is exactly what a too-narrow allowlist produces. Seeing it break this way taught me more than reading about default security levels ever did.

It also clarified the previous session's DHCP confusion. The relay was forwarding correctly the whole time; the problem was the USER interface ACL state, not a DC01 DHCP fault as I had assumed the night before. The lesson is to question my own diagnosis when the evidence (ASA receiving and forwarding requests) already pointed away from the server.

## Issues & Troubleshooting

**Connectivity present but nothing usable.** Symptom: after initial ASA changes I had a link but could not accomplish anything. Cause: only DNS was permitted from USER to DC01; the USER inbound ACL had no other permits, so everything except name resolution hit the implicit deny. Fix: built out the USER_IN ACL (DNS, then web/FTP), then consolidated to a single permissive IP permit for the workstation. Connectivity fully restored.

**Carried-over DHCP issue from 2026-05-10.** The connect-then-disconnect and APIPA behavior traced back to the same ACL/security-level situation rather than a DC01 DHCP server defect. Resolving the USER interface policy cleared the confusion.

## Open Questions

- Two decisions from this session need ADRs written at the documentation stage:
  - Permissive USER VLAN rule: why I am accepting a broad `permit ip` on my own workstation during buildout, and the trigger/plan for reinstating least privilege once infrastructure is complete.
  - Not domain-joining the main desktop: the reasoning for keeping the development/lab host off the production domain.
- The permissive rule is technical debt by design. I need a concrete definition of "infrastructure setup complete" so the least-privilege reimplementation actually happens rather than becoming permanent.

## Next Session

- Continue the buildout toward standing up the remaining workstations.
- When I reach the documentation stage, write the two ADRs flagged above.
