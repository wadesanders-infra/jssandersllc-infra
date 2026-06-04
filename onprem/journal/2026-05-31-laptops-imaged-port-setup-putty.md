# 2026-05-31: Laptops Imaged; Port Setup and a PuTTY Problem

**Phase:** 1
**Duration:** Not recorded (did not properly notate session duration at the time; catching up on notes after the fact)
**Goal:** Reimage the two laptops and begin setting up their network access.

## What I Did

Imaged both laptops. Their existing passwords were unknown, so reimaging was the only way to get clean, usable devices.

Moved on to setting up their ports / network access. Ran into a practical wall: modern laptops do not have an Ethernet port. For server access up to now I had been working through switched PuTTY console operations, which is not ideal as a standing approach. Tried to continue that way and PuTTY was not behaving correctly. I am fairly sure it is user error on my end rather than a real fault, but I had other things I needed to work on, so I left it there for now rather than chasing it down this session.

## What I Learned

The lack of an Ethernet port on modern laptops is a real planning constraint for field/endpoint devices, not a triviality. Wired console and management habits built around machines that have an RJ-45 port do not carry over cleanly, and I need a deliberate plan for how these laptops get on the network and how I manage them, rather than improvising per device. The PuTTY console workflow I had been leaning on was always a stopgap; the laptops make that obvious.

## Issues & Troubleshooting

**PuTTY not working correctly.** Symptom: PuTTY was not behaving as expected while I was setting up laptop access. I did not fully diagnose it this session. My working assumption is user error rather than a genuine fault, but I deprioritized it to focus on other work and am leaving it open. Flagging it honestly rather than pretending it is resolved.

## Open Questions

- What is actually wrong with the PuTTY setup? Confirm whether it is configuration/user error (most likely) or something else, and settle on the right console/management path for laptops that have no Ethernet port.
- What is the intended network-access design for the two laptops as endpoints, given no wired port?

## Next Session

- Diagnose and resolve the PuTTY issue, and establish a working management/access path for the laptops.
