# 2026-05-08: SIEM Hardware Realization; Paused at Password and Audit GPOs

**Phase:** 1
**Duration:** Not recorded (did not properly notate session duration at the time; catching up on notes after the fact)
**Goal:** Continue security group work and move into the password policy and audit logging GPOs.

## What I Did

Continued working on security groups. Mid-session I had a realization about SIEM hardware: I have an old laptop I could repurpose as a dedicated SIEM host. It is not industry-standard hardware for the role, but I am under serious financial constraints right now and much of this buildout is being assembled from hardware I already have. The plan at this point was to reimage the laptop with a Linux distribution with no GUI to conserve resources for the SIEM workload. Had not settled on a distro yet. Noted that the hardware decision deserves an ADR once I reach the documentation stage.

Got as far as the password policy GPO and audit logging before stopping. Had to step away for a couple of days to help family with a newborn and a move, so the session ended at the GPO stage rather than completing it.

## What I Learned

Budget is a design constraint, not just a background fact. Running the SIEM on hardware I already own changes what is realistic, and being honest about that is more useful than pretending to an enterprise reference architecture I cannot afford. The reasoning a reviewer should see is not "I bought the right box," it is "given these constraints, here is the host I chose and why it is adequate for the workload." A GUI-less distro is the obvious move to claw back RAM and CPU for the actual SIEM process on constrained hardware.

## Issues & Troubleshooting

No technical issues this session. Work paused for personal reasons (newborn and a move) rather than a blocker.

## Open Questions

- Which Linux distribution for the SIEM host? Leaning toward something minimal and GUI-less, but not decided.
- The SIEM hardware choice needs an ADR once I reach the documentation stage.

## Next Session

- Resume and complete the password policy GPO and audit logging GPO.

---

## Addendum: 2026-05-11

While I was out of town I managed to acquire two laptops and two mini OptiPlex machines, and I am reworking the hardware plan for the buildout around them. This changes the SIEM hardware decision recorded above: rather than repurposing the single old laptop, Wazuh will run on dedicated bare-metal hardware (one of the Micros). The "old laptop as SIEM" idea from this session is effectively superseded by the hardware restructure. The outline is being updated to incorporate the new devices. The original reasoning still holds (constrained budget, repurposed hardware, monitoring on hardware independent of what it monitors); the specific box changed.
