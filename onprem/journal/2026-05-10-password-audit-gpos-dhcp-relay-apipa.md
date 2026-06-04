# 2026-05-10: Password and Audit GPOs, DHCP Server, and a DHCP Relay That Would Not Hand Out Addresses

**Phase:** 1
**Duration:** Not recorded (did not properly notate session duration at the time; catching up on notes after the fact)
**Goal:** Finish the security GPOs, stand up the DHCP server with per-VLAN scopes, and configure DHCP relay on the ASA so clients across VLANs can pull addresses from DC01.

## What I Did

Got some time in the evening and worked through the security GPOs:

- `GPO-Domain-PasswordPolicy` - password requirements
- `GPO-Domain-AuditPolicy` - event recording

Stood up the DHCP server role on DC01 and created a scope per VLAN:

**SERVERS (VLAN 20)**
- Range: 10.10.20.100 to 10.10.20.200
- Subnet mask: 255.255.255.0
- Default gateway: 10.10.20.1
- DNS server: 10.10.20.10 (DC01 itself)

**CLIENTS (VLAN 30)**
- Range: 10.10.30.100 to 10.10.30.200
- Subnet mask: 255.255.255.0
- Default gateway: 10.10.30.1
- DNS server: 10.10.20.10

**USER (VLAN 50)**
- Range: 10.10.50.100 to 10.10.50.200
- Subnet mask: 255.255.255.0
- Default gateway: 10.10.50.1
- DNS server: 10.10.20.10

DC01 sits on VLAN 20, so without help only devices on VLAN 20 would ever receive a DHCP response from it. DHCP discovery is a broadcast and does not cross VLAN boundaries on its own. To get CLIENTS and USER served, I configured DHCP relay on the ASA to forward their requests across to DC01 on the SERVERS VLAN:

```
dhcprelay server 10.10.20.10 SERVERS
dhcprelay enable CLIENTS
dhcprelay enable USER
dhcprelay timeout 60
```

Then tested from the main desktop on VLAN 50 to confirm DHCP was working end to end.

## What I Learned

DHCP relay is the piece that makes a single centralized DHCP server viable across a segmented network. The server lives on one VLAN, but the relay agent on the ASA turns the client's broadcast discovery into a unicast forward to the server and carries the response back, so one DHCP server can serve every segment without a server sitting on each one. The `dhcprelay server` line names where to forward to; the per-interface `dhcprelay enable` lines say which segments to relay for. Without the relay, segmentation silently breaks DHCP for every VLAN except the one the server is on.

## Issues & Troubleshooting

**Main desktop pulled an APIPA address instead of a lease.** Symptom: tested DHCP from the desktop on VLAN 50 and it fell back to an APIPA (169.254.x.x) address, meaning it never got a DHCP response.

Worked the path from the ASA inward. The ASA was in fact receiving DHCP requests from the desktop (11 requests by the time I was writing this), so the relay was forwarding. The problem was downstream: DC01 was receiving nothing back, or rather was not responding. The requests were getting to the server side but no lease was coming out.

Went back to DC01 and found a configuration alert: the DHCP server was not authorized in Active Directory. A Windows DHCP server has to be authorized in AD before it will hand out leases; until then it stays silent even though the role is installed and scopes exist. Enabled authorization and immediately saw a connection, followed right away by a disconnect as I was writing this up.

Status at end of session: still not fully resolved. The connect-then-disconnect points at a remaining DC01-side configuration problem rather than the relay (the relay is demonstrably forwarding). Leaving it here for the night. In an active production environment this would be a very high priority issue, since no working DHCP means no clients get on the network.

Also worth flagging: there is a decent chance I overlooked an option somewhere in the DHCP wizard. I will verify functionality and correct any flaws during testing rather than assuming the scopes are perfect.

## Open Questions

- Why the connect-then-disconnect after authorizing DHCP in AD? The relay is forwarding and the server is now authorized, so the remaining fault is on DC01. Need to dig into the DC01 DHCP config in the morning. (One candidate I want to rule in or out: ASA security levels. ACLs are not implemented yet, so low-to-high traffic could be getting dropped by default and interfering, but the symptom here looks more like a DHCP-server-side problem than an ACL drop. Will confirm.)
- Did I miss a setting in the DHCP scope wizard? Verify each scope's options against what a client actually needs during testing.

## Next Session

- Resolve the DHCP issue on DC01. Confirm whether the remaining fault is server-side config or ASA security-level / ACL behavior, then validate that the desktop and a CLIENTS-VLAN device both pull correct leases.
