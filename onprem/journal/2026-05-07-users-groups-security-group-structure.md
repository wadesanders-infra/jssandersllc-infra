# 2026-05-07: Users, Groups, and the Security Group Structure

**Phase:** 1
**Duration:** Not recorded (did not properly notate session duration at the time; catching up on notes after the fact)
**Goal:** Create the production user accounts and the security group structure that document and surveillance access will inherit. Establish the contractor account lifecycle.

## What I Did

Created the production user accounts and the security groups that govern access.

Users:

- `wsanders` - full control over documents and surveillance
- `jsanders` - read-only, no other permissions
- `contractor01`, `contractor02` - created disabled from the start, positioned to be temporarily enabled per engagement as needed

Security groups, with membership reflecting the access model above:

- `SG-Documents-FullControl` - `wsanders`
- `SG-Documents-Read` - `jsanders`
- `SG-Surveillance-FullControl` - `wsanders`
- `SG-Surveillance-Read` - `jsanders`
- `SG-Contractor-Projects` - `contractor01`, `contractor02`

Built a flat group structure rather than nesting groups inside one another. With no delegation in place and a user population this small, the blast radius of any single group is easy to identify by looking directly at its membership. Nesting would add indirection that buys nothing at this scale and makes it harder, not easier, to answer "who can touch this." Decided to centralize service accounts into a dedicated group for the same reason: it streamlines GPO targeting and keeps auditing straightforward.

Set the contractor accounts up disabled at creation. They exist only to be enabled on a temporary basis when a contractor is actively engaged, then disabled again when the engagement ends. Created GPOs to accommodate this lifecycle so that an enabled contractor account inherits the right restrictions automatically rather than requiring manual per-account configuration each time.

## What I Learned

The complexity in this identity layer is in the access logic, not the headcount. Two real users and a couple of dormant contractor accounts, but the structure still has to answer the questions that matter: who can read documents, who can write them, who can see surveillance, and what a contractor is scoped to. Getting the group design right at two users is the same discipline as getting it right at two hundred; the difference is that at this scale I can verify it by eye, which is exactly why a flat structure is the correct call here rather than a limitation.

Disabling contractor accounts from creation flips the default. Instead of an account that is enabled and needs to be remembered-to-be-disabled when the work ends (the failure mode that leaves stale active accounts lying around), the account is dormant by default and only live during the window it is actually needed. The safe state is the resting state.

## Issues & Troubleshooting

**Whole system powered off mid-session.** Symptom: came back to do a small bit of work and found the entire environment was off. The surge protector I bought has a rocker switch that is easy to trip; something dropping on it or the cat stepping on it (I am not certain which) cut power to everything. I lost time bringing the whole stack back up in order. This is a real availability risk, not just an annoyance: an accidental bump takes down the domain controller and everything depending on it. I need a power solution that cannot be switched off by incidental contact, or at minimum to relocate the strip out of reach. Logging it as a tracked physical risk rather than a one-off.

## Open Questions

- The surveillance and contractor groups created here are broader than what later workstreams strictly require yet. Worth confirming during Workstream 2 (document management and surveillance) that these group names and scopes still match the access model when those systems actually exist, and renaming or consolidating if they drifted.
- What is the right physical fix for the power switch problem? Options range from a different PDU without an exposed rocker, to relocating the strip, to a small UPS that also buys clean shutdown on power loss. Deciding later.

## Next Session

- Continue group policy work. Stand up the password policy GPO and audit logging GPO that the rest of the domain will inherit.
