# 2026-05-06: Network Foundation: ASA Wipe, Switch Cabling, and Baseline Connectivity

**Phase:** 1
**Duration:** Not recorded (did not properly notate session duration at the time; catching up on notes after the fact)
**Goal:** Establish a clean network baseline. Wipe the ASA to a known state, cable the ASA and switch correctly, confirm the physical topology, and verify that hosts behind the firewall can reach the internet.

## What I Did

Started from the console. A few Cisco CLI mechanics that were worth internalizing before doing anything else: Ctrl+H backspaces (the terminal emulator was not passing a normal backspace), and commands can be abbreviated as long as the abbreviation is unique, so `termina` resolves to `terminal` because nothing else matches. This matters more than it sounds; once I trusted unique-prefix matching I stopped typing full keywords.

Decided to wipe the ASA and rebuild from a clean baseline rather than work on top of whatever residual config was on it. Working from a known-good starting point removes a whole class of "is this my config or leftover state" questions later.

Corrected the physical cabling and locked down the actual topology:

- ASA port 2 to switch port 1 (trunk uplink)
- Switch port 2 to the main desktop
- Switch port 3 to the Hyper-V host

While building the diagram I caught that I had the firewall and switch reversed in my earlier network diagram. The device positions were swapped relative to how they are actually cabled and how traffic actually flows. Updated the diagram to match reality (ASA as the edge/router, SG350 downstream doing L2).

Brought up VLAN subinterfaces and NAT on the ASA (from prior sessions) and verified that both the main desktop and the VMs could reach the internet through the firewall.

## What I Learned

**Internet connectivity and ICMP are independent.** Both the desktop and the VMs could load Google but could not ping it. I did not have ICMP permitted through the firewall, so outbound web traffic worked fine while echo requests were dropped. The takeaway is that "ping fails" is not the same as "no connectivity," and using ping as a first-line reachability test will lie to you if ICMP is not explicitly allowed. For real reachability testing behind this firewall I need to test the actual service port, not rely on ICMP.

**The ASA and the SG350 console at different baud rates.** The ASA is 9600 and the switch is 115200. I had to set the terminal correctly per device or the console output is garbage. Getting this wrong wastes time staring at a dead or scrambled console wondering if the cable is bad.

**Unique-prefix command matching** is a real time saver on the Cisco CLI once you trust it, but it is also a trap: an abbreviation that is unique today can become ambiguous as you learn more commands, so the habit is fine interactively but full keywords belong in any saved config.

## Issues & Troubleshooting

**Reversed firewall/switch in the network diagram.** Symptom: my existing diagram showed the firewall and switch in swapped positions relative to the physical cabling and traffic flow. I worked through the topology by hand against the actual port connections and corrected the diagram. Worth noting: I had been relying on AI assistance while building this out, and at no point did it flag that the firewall was in the wrong place on the diagram. The error was mine to catch, and it was a good reminder that AI output does not substitute for verifying the topology against the hardware in front of me.

**Incorrect baud rate guidance.** Symptom: console output was unusable until the terminal baud rate matched the device. The AI assistance I was using initially had the baud rates wrong (it did not correctly distinguish 9600 for the ASA from 115200 for the switch). I confirmed the correct rates against the devices themselves. Another case where the hardware is the source of truth.

**Cannot ping out despite working internet.** Symptom: web traffic succeeded, ICMP echo to 8.8.8.8 failed from both the desktop and the VMs. Cause: ICMP is not permitted through the ASA. Not a defect; this is expected with the current ruleset. Documenting it so future-me does not chase a connectivity ghost when the real answer is "ICMP is just not allowed here."

## Open Questions

- The network is currently double-NATting through the upstream home network. I do not have the time budget to resolve this now. It needs to be resolved before remote access work (Workstream 3), because the ASA is not the true network edge until then. Deferred deliberately, not forgotten.
- Do I want to permit ICMP through the ASA for my own troubleshooting convenience, and if so, scoped to which sources? Leaning toward allowing it from MGMT only rather than broadly, but not deciding today.

## Next Session

- Begin identity work: create the production users and security groups, and stand up the group structure that document and surveillance access will inherit.
