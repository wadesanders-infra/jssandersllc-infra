# 2026-06-11 - Renaming DC01 from its auto-generated hostname

**Phase:** 1
**Duration:** ~2h
**Goal:** Bring the domain controller's actual hostname in line with the name I have been calling it everywhere else (DC01), after discovering it was still running under the Windows-assigned WIN- name because I promoted it before renaming.

## What I Did

While setting up the time sync hierarchy I noticed the DC was reporting its hostname as WIN-AFPG3INH7RJ, not DC01. The Hyper-V VM is labeled DC01 and I had been referring to it as DC01 in every config and document, but the machine itself had never been renamed before promotion. So the computer object, SPNs, and DNS records all carried the auto-generated name.

I chose to rename it properly rather than rebuild or live with the mismatch. Renaming a domain controller is not a normal rename: the name is embedded in DNS SRV records, SPNs, and the AD database, so the supported path is `netdom computername`, which adds the new name as an alternate, promotes it to primary, then removes the old one so DNS and SPNs update cleanly.

The actual sequence that happened:

- Ran `netdom computername <oldname> /add:DC01.ad.jssandersllc.org`. It failed with "the specified domain either does not exist or could not be contacted."
- Checked services and found the root cause: the Netlogon service was stopped. With Netlogon down, the DC could not present or locate its own domain, so netdom and `nltest /dsgetdc` both failed (ERROR_NO_SUCH_DOMAIN, 0x54b).
- Tried to start Netlogon manually. It failed to start. The System log showed event 5602: "An internal error occurred while accessing the computer's local or network security database."
- Confirmed SYSVOL and NETLOGON were still shared and DFSR/NTDS were running, so this was not the SYSVOL-not-shared problem I first suspected.
- Did a clean reboot as the lowest-risk step, since 5602 after repeated reboots is sometimes transient.
- After the reboot, Netlogon came up on its own and stayed running. `nltest /dsgetdc:ad.jssandersllc.org` now returned DC: \\DC01.ad.jssandersllc.org, meaning the earlier netdom add/makeprimary had actually registered before Netlogon died, and the reboot let them finish applying.
- Ran `dcdiag /v` to confirm real health rather than trusting a single Netlogon check. All core tests passed (Connectivity, Advertising, MachineAccount, KnowsOfRoleHolders, NetLogons, Replications, RidManager, Services, LocatorCheck). Every SPN and FSMO role now reads DC01.
- Fixed NTP, which had two problems: a typo in the manual peer list (ox8 instead of 0x8) and DNS resolution failures during the Netlogon outage. Re-ran `w32tm /config /manualpeerlist:"time.nist.gov,0x8 pool.ntp.org,0x8" /syncfromflags:manual /reliable:yes /update`, restarted w32time, and resynced. Status then showed Source: time.nist.gov, Stratum 2, current sync time.
- Removed the old WIN- name with `netdom computername DC01.ad.jssandersllc.org /remove:WIN-AFPG3INH7RJ.ad.jssandersllc.org` to finish the rename cleanly. The old alternate name is now gone.

## What I Learned

The lesson that started this whole detour: rename a Windows Server before promoting it to a domain controller, not after. Renaming a member server is a one-command operation; renaming a DC is a multi-step supported procedure because the hostname is woven into DNS SRV records, SPNs, and the directory itself. The cost of getting the order wrong is the two hours this took.

The dcdiag SystemLog "failure" taught me to read event timestamps before reacting. The failure looked alarming, but almost every error in it (the GroupPolicy 041E failures, the 5602, the DNS timeouts) was timestamped inside the 20:03 to 20:29 window when Netlogon was down. They were symptoms of the outage I had already fixed, sitting in the log. dcdiag flags SystemLog if anything errored recently, so a passing DC can still "fail" that one test on stale entries. The core AD tests are the ones that actually describe current state.

I also now understand the Netlogon dependency chain more concretely. Netlogon is what registers the DC's SRV records and answers domain-location queries. When it is down, the DC cannot find itself, which cascades into DNS registration failures, Group Policy failures, and netdom/nltest errors that all look like separate problems but trace to one stopped service.

## Issues & Troubleshooting

- **Symptom:** netdom rename failed with "domain does not exist or could not be contacted." **Cause:** Netlogon service stopped. **Tried:** manual start, which failed with event 5602 (internal error accessing the local/network security database). **Fix:** a clean reboot brought Netlogon up and let the previously-registered name changes finish applying.
- **Symptom:** NTP would not set its peers (event 0x86, "No such host is known"). **Cause:** a typo (ox8 vs 0x8) in the original peer list plus DNS resolution failing during the Netlogon outage. **Fix:** re-ran the w32tm config with the correct 0x8 flag after the DC was healthy; Source now shows time.nist.gov.
- **Resolved as risk acceptance:** Repeated disk write-cache warnings (event 80040020) on the DC's virtual disk. On a DC this is a real durability risk for the AD database on an unclean shutdown, and I have already had one surge-protector incident power the whole lab off. A UPS is the correct mitigation but is not affordable right now, so I am formally accepting the risk: the DC runs on consumer hardware without battery backup, and an unclean power loss could corrupt the AD database. The fallback is the Hyper-V checkpoint taken after this session, which provides a known-good restore point.

## Open Questions

- The leftover "DC01" value in the OptionalNames registry key (event 800009CA) may still be present even after removing the old alternate name with netdom. It is being ignored so it is harmless, but worth confirming it does not resurface.
- Should the DFSR member object keeping the old CN (CN=WIN-AFPG3INH7RJ in the VerifyReferences output) be left as-is long term, or does it warrant cleanup before Workstream 4 when SYSVOL replication starts mattering more? The references appear correct and only the label is old.

## Next Session

- Domain-join the HP Pavilion x360 as the field workstation on the CLIENTS VLAN.
