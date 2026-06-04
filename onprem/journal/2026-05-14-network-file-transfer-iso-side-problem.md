# 2026-05-14: Side Problem: Moving a Windows 11 ISO to the OptiPlex Over the Network

**Phase:** 1
**Duration:** Not recorded (did not properly notate session duration at the time; catching up on notes after the fact)
**Goal:** Limited time today, so I took on a small side problem: get a Windows 11 ISO from my main desktop over to the OptiPlex.

## What I Did

I have a Windows 11 ISO sitting on my main desktop and needed it on the OptiPlex. My usual habit in the past has been to drag the file onto a USB stick and carry it over, but that approach is not working for this, so I started looking into moving the drives/file across the network instead.

The motivation is bigger than this one transfer. Getting a reliable network file-transfer path between machines will be useful for the long-term workflow, not just for shuttling one ISO around. Solving it once properly beats reaching for a USB stick every time.

## What I Learned

The sneakernet habit (copy to USB, walk it over) does not scale and is not always available, and leaning on it has kept me from setting up a proper network path between hosts. Treating "how do files move between my machines" as its own infrastructure question, rather than an ad hoc per-file chore, is the right framing for a buildout I will be living in for a while.

## Open Questions

- What is the cleanest network transfer method here given the current segmentation? Options to weigh: an SMB share, something over the existing trunk, or another file-transfer approach. Want something repeatable rather than a one-off.

## Next Session

- Settle on and stand up a repeatable network file-transfer method, and use it to get the Windows 11 ISO onto the OptiPlex.
