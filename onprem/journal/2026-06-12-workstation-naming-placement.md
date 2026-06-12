# 2026-06-12: Fixing a naming and placement gap in the workstation join process

**Phase:** 1
**Duration:** ~1.5h
**Goal:** Get the two business laptops named consistently and placed in the correct OUs, after noticing both had been domain-joined under their default Windows hostnames and left sitting in the default Computers container.

## What I Did

While moving the Pavilion's computer object in AD, I realized both laptops had the same problem: they were domain-joined and working, but each was still running its throwaway default hostname (DESKTOP-/WIN- style) and both objects were sitting in the default Computers container rather than an OU I had placed them in. The Latitude in particular needed this fixed because it is meant to receive a restricted GPO later, and computer-targeted GPOs apply based on OU placement.

Rather than just patch the two machines, I stopped to figure out why it kept happening and set a standard so it would not recur.

Root cause: the default hostname is assigned during the out-of-box setup. When I rushed through setup, I never set the name there. Domain-join with `Add-Computer` then joined each machine under whatever name already existed and dropped the computer object in the default Computers container. Joining and naming are separate operations and nothing forced the naming or placement step, so both laptops joined cleanly but kept their throwaway names and wrong placement. I had been treating domain-join as the finish line when the real sequence is name, join, place, verify.

I also reconsidered the naming convention. I had named the Pavilion JSS-FIELD01, which bakes the machine's role into its hostname. I decided against role-based names: a machine's role can change, and a role-based name becomes inaccurate the moment it does. I switched to role-neutral sequential names and express role through OU placement and GPO instead.

Corrective actions taken this session:

- Renamed the Pavilion from JSS-FIELD01 to JSS-WS01 with `Rename-Computer -NewName "JSS-WS01" -Restart`.
- Renamed the Latitude to JSS-WS02 with `Rename-Computer -NewName "JSS-WS02" -Restart`.
- Created a new OU: JSS > Workstations > Contractor-Workstations, so the restricted contractor GPO can target only contractor machines without affecting general workstations.
- Moved JSS-WS01 (field workstation) into JSS > Workstations.
- Moved JSS-WS02 (contractor workstation) into JSS > Workstations > Contractor-Workstations.
- Wrote a Workstation Build and Domain-Join SOP codifying the name, join, place, verify sequence and the naming convention, and created an sops/ folder for it since I expect to write more of these as I hit consistency issues.

## What I Learned

The role of a machine should not live in its hostname. Identity (what the machine is) and policy (what it is allowed to do) are cleaner kept separate: the name identifies the asset, the OU and GPO express the role. This is also why enterprises use generic sequential or asset-tag names rather than descriptive ones. Expressing role through OU placement means I can change a machine's policy by moving its object, without renaming the machine.

A workstation rename after joining is harmless, but the same mistake on a domain controller cost me two hours earlier (the DC rename ordeal). Same lesson, opposite stakes: on a member workstation the hostname is not woven into DNS SRV records and SPNs, so a post-join rename just updates the computer object cleanly. The severity of the "rename after join" mistake scales entirely with the role of the machine.

The bigger takeaway is about process. The gap was not a technology failure, it was a missing step in my own workflow. Writing the SOP turns a repeated mistake into a documented standard, and creating the sops/ folder reactively (one SOP per real consistency issue) means each standard encodes a lesson I actually learned rather than a hypothetical I guessed at.

## Issues & Troubleshooting

- **Symptom:** Both laptops domain-joined but running default hostnames and sitting in the default Computers container. **Cause:** name never set during rushed OOBE; join proceeded under the default name; objects defaulted to the Computers container. **Fix:** post-join rename of each machine (safe on workstations), creation of the Contractor-Workstations sub-OU, and manual move of each computer object to its correct OU.

## Open Questions

- Should I redirect the default computer-object location (redircmp) so future domain-joins land in JSS > Workstations automatically instead of the default Computers container, removing the manual placement step entirely?
- Does the README or outline need a pointer to the new sops/ folder so it is discoverable to a reviewer rather than something they stumble on?

## Next Session

- Build and link the restricted GPO targeting the Contractor-Workstations OU (JSS-WS02).
