# 2026-06-03: Console Cable Realization, a Missing Default Gateway, Wrong Windows Edition, and Standing Up SRV-01

**Phase:** 1
**Duration:** ~2.5h
**Goal:** Get the laptops onto the network and domain-joinable, resolve the leftover PuTTY/console problem, and make progress on file-share hosting.

## What I Did

Started by resolving the console problem I had left open. I had convinced myself there was a hardware bottleneck: only one laptop has an Ethernet port and I only have one Ethernet-to-USB adapter. Then I remembered that my console cable is itself an Ethernet-to-USB cable, the literal Cisco-made console cable. There was no bottleneck; I had talked myself into one. The actual PuTTY problem turned out to be physical: I checked the connection and saw I was not plugged into the console port at all. Once I was on the right port, PuTTY worked. Classic reason to work the OSI model from the bottom up. I had been chasing it as an application-layer (PuTTY config) issue when the fault was at Layer 1.

Configured the laptops on switch ports 7 and 8. They pulled IP addresses but had no internet. My first assumption was the firewall ACLs, so I consoled into the ASA to check, and the ACLs were configured correctly. My assumption was wrong. Went back to an endpoint and ran `ipconfig /all`, which showed the default gateway was missing. That pointed at DHCP, not the firewall, so I moved over to DC01.

On DC01 I found the cause: I had left option 003 (Router) off the CLIENTS VLAN scope, and worse, even where it existed it had no value set. Set option 003 to 10.10.30.1 to match the rest of the network's gateway naming convention. Endpoints had full functionality after that.

Hit a new problem: I had reimaged both laptops with Windows Home instead of Pro, and Home cannot domain-join. Going with the nuclear option and reimaging with Windows Pro, left unactivated. These endpoints do not need what activation provides, and I cannot afford full Pro licenses right now, so unactivated Pro is the right trade-off for the role they play.

While the endpoints were reimaging, I used the time to start standing up SRV-01. The immediate driver was that I needed a file share, and rather than stand up a separate host for it I decided to put the share on SRV-01, where it fits well alongside the future services this member server will host. Reimaged the Latitude with Pro and got SRV-01 stood up, though not fully configured, before stopping.

The remaining time this session went to catching up on documentation.

## What I Learned

This session was a clean lesson in not trusting my first assumption and working a problem from the physical layer up. Twice in one session my first guess was wrong: the "hardware bottleneck" that was really me forgetting what cable I already owned, and the "firewall ACL" problem that was really a missing DHCP option. In both cases the fix came from checking the layer below the one I assumed was at fault, the physical connection in the first case and the actual IP configuration via `ipconfig /all` in the second. The OSI model is not just exam material; it is the order to check things in when something does not work.

The missing default gateway also reinforced how DHCP scope options actually drive client behavior. The endpoints got an address (so the scope and relay were working) but could not reach anything off-subnet, which is the exact signature of a missing or unset option 003. An address without a gateway is a device that can talk to its own VLAN and nowhere else.

The Windows edition mistake cost a reimage, but it clarified a real constraint: domain join requires Pro (or higher), Home will not do it, and given the budget the workable path is unactivated Pro for endpoints that do not need activation-gated features. Worth remembering before imaging anything else.

## Issues & Troubleshooting

**PuTTY would not connect (carried over from the previous session).** Symptom: PuTTY not working, which I had assumed was user error in the PuTTY config. Actual cause: I was not physically connected to the console port. I had also briefly convinced myself I was blocked by having only one Ethernet-to-USB adapter, forgetting that the Cisco console cable is itself Ethernet-to-USB. Fix: connected to the correct console port. Resolved.

**Laptops got IPs but no internet.** Symptom: endpoints on switch ports 7 and 8 received addresses but had no connectivity off-subnet. First assumption was the ASA ACLs; consoled into the ASA and confirmed the ACLs were correct, so that assumption was wrong. Ran `ipconfig /all` on an endpoint and saw the default gateway was missing. Cause: DHCP option 003 (Router) was left off the CLIENTS VLAN scope and had no value set. Fix: set option 003 to 10.10.30.1. Endpoints reached full functionality. Resolved.

**Laptops imaged with Windows Home, which cannot domain-join.** Symptom: could not domain-join the reimaged laptops. Cause: I had installed Windows Home instead of Pro. Fix: reimaging with Windows Pro, left unactivated, since these endpoints do not require activation-gated features and a full Pro license is not affordable right now. The Latitude has been reimaged with Pro; the other endpoint is still pending.

## Open Questions

- Does running the endpoints on unactivated Windows Pro create any practical limitation for the roles they need to fill (domain join, GPO application, telemetry generation)? Domain join is the requirement and Pro supports it; want to confirm nothing activation-gated bites me later.

## Next Session

- Reimage the remaining endpoint with Windows Pro so both laptops can be domain-joined.
- Continue configuring SRV-01 toward hosting file shares.
