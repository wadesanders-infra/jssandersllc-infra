# 2026-05-29: Returning to the Buildout: New Hardware, VM Standup, and Cutting the Windows 11 Endpoint

**Phase:** 1
**Duration:** Not recorded (did not properly notate session duration at the time; catching up on notes after the fact)
**Goal:** Get back into the buildout after an extended break. Stand up two more virtual machines (Windows 11 and Windows 10) and account for the new hardware I picked up while I was away.

## What I Did

Took a while to get back to this. Personal factors and work kept me away from the buildout for an extended stretch.

Started in on standing up two more VMs, a Windows 11 and a Windows 10. Partway through I caught myself about to proceed as if nothing had changed, when in fact the hardware picture is different now. While I was out of town I came into new equipment: two mini OptiPlex machines (courtesy of Goodwill) that need RAM and SSDs added, and two laptops received for free as payment for helping someone move. Both laptops will work well enough as field devices. I have updated the project outline to incorporate these changes.

The new hardware changes the VM plan. With the two laptops available as real endpoints, the planned Windows 11 VM endpoint is being cut due to hardware constraints. Both laptops can only support Windows 10. One of those two may also need to be dropped further back depending on how it performs; I will decide based on actual performance once they are running.

## What I Learned

Coming back from a long break, the instinct is to resume exactly where the notes left off, but the ground had shifted while I was gone. Catching the new hardware before I built VMs that the laptops now make redundant saved me from work I would have torn out. The broader lesson is that this buildout is opportunistic on hardware (Goodwill machines, a laptop earned by helping someone move) and the plan has to flex around what I actually have rather than what I sketched weeks ago. Updating the outline in the same session, rather than letting it drift from reality, is what keeps the documentation trustworthy.

Cutting the Windows 11 endpoint is a constraint-driven call, not a preference. Real laptops generating real telemetry on the CLIENTS VLAN are more valuable as endpoints than another VM, and the hardware will only run Windows 10, so the decision makes itself.

## Issues & Troubleshooting

**Two donated OptiPlex Micros need to be brought up to spec.** They arrived without sufficient RAM and without SSDs, so they are not deployable until I add both. Not a fault, just a prerequisite before they can take on a role in the buildout.

## Open Questions

- Will both laptops hold up as usable field/endpoint devices, or does the weaker of the two need to drop out? Decide on performance once they are imaged and running.
- Final role assignment for the two Micros now that they are part of the plan. (The outline reflects the intended roles; confirm against actual specs once RAM and SSDs are installed.)

## Next Session

- Image the two laptops. Their existing passwords are unknown, so they need to be reimaged before they can be brought into the domain.
