# 2026-06-08: File Share Verification, a Naming Mismatch, EI.cfg for Pro, and Discovering DC01 Was Never Named DC01

**Phase:** 1
**Duration:** ~4h
**Goal:** Verify the file share access model set up last session actually enforces as intended, then continue domain-joining endpoints and stand up the time synchronization hierarchy.

## What I Did

Started by testing whether last session's file share permissions actually work. Testing turned out to take far more effort than building did.

First problem surfaced immediately: `runas` on the contractor machine was not working. There were several possible causes (mistyped credentials, the machine never domain-joined, seclogon service disabled, or even a clock skew), so rather than guess I went to verify the most basic assumption first: was the Latitude even domain-joined? It was not. I had reimaged it to Pro and never completed the join step. Good reminder to check the simple things before running down a rabbit hole.

Joined the Latitude, then ran verification from SRV01 logged in as wsanders:

```
net view \\10.10.20.20
dir \\10.10.20.20\Restricted
```

As wsanders (effectively the master account) I could reach everything, as expected. To confirm the restrictions actually bite for a less-privileged account, I opened a shell as jsanders:

```
runas /user:ad\jsanders cmd
dir \\10.10.20.20\Internal      (allowed)
dir \\10.10.20.20\Restricted    (denied)
```

That is the intended behavior: jsanders can read Internal but is denied Restricted. The access model verified for the document-read role.

Went to verify the contractor account restrictions next:

```
runas /user:ad\contractor01 cmd
```

The password was rejected repeatedly. I re-entered it, then reset it on the domain controller and tried again, still rejected. Inspecting the account, I found the cause: the display name was contractor01 but the actual logon name was c01. I had taken some shortcut early on and it cost me time now. The second contractor account had the same mismatch. Fixed the logon names on both to match the display names, and disabled the first account again rather than waiting for it to time out.

Moved to the Pavilion, which had a separate problem: the embedded product key kept forcing a Windows Home install, so it could not be domain-joined. After researching, I created an `EI.cfg` (Edition ID config) file to force a Pro install:

```
[EditionID]
Professional
[Channel]
Retail
[VL]
0
```

EditionID says which edition to install, Channel is the licensing channel, and VL set to 0 tells the installer there is no volume license (1 would make it behave as part of a managed volume deployment). This should install unactivated Pro, the same state as the Latitude.

While the Pavilion installed, I worked on the time synchronization hierarchy. On the domain controller:

```
netdom query fsmo
```

to confirm all five FSMO roles were present including the PDC emulator. Then pointed the DC at external NTP sources:

```
w32tm /config /manualpeerlist:"time.nist.gov,0x8 pool.ntp.org,0x8" /syncfromflags:manual /reliable:yes /update
Restart-Service w32time
w32tm /resync /rediscover
w32tm /query /status
```

The time it pulled was correct but displayed three hours behind local (status read 2:09pm when it was 5:09pm EST), which is a time zone display issue, not a sync problem. Set the zone explicitly for my own sanity:

```
tzutil /s "Eastern Standard Time"
```

## What I Learned

Check the simple things first. The contractor `runas` failure could have been any of four causes, and the actual one was the most basic possible: the machine was never joined. Starting from the cheapest assumption to verify saved me from chasing seclogon and clock theories on a problem that was really an unchecked checkbox.

Credential changes do not propagate into an already-open shell. When resetting a password in AD, the existing PowerShell window keeps the old token; a fresh window is needed to use the updated credentials. This was wasting attempts until I realized it.

Naming consistency is not cosmetic. The c01-versus-contractor01 mismatch between logon name and display name cost real troubleshooting time, because `runas` needs the logon name and I was reading the display name. A shortcut taken early surfaced as a confusing failure later. This is the practical argument for a naming convention applied consistently from the start.

The `EI.cfg` mechanism is the clean way around an embedded-key edition lock: the installer reads it to decide which edition and licensing channel to use, so a machine that wants to install Home can be steered to Pro.

Time synchronization is about evidentiary consistency, not convenience. Getting every machine on the same correct time means logs across the environment share a timeline, which makes investigation and remediation simpler. A correct-but-wrong-zone display is harmless for correlation (the underlying time is right) but I corrected the zone anyway to avoid confusing myself during analysis.

## Issues & Troubleshooting

**Contractor `runas` rejected, machine never joined.** Symptom: `runas` failed on the contractor machine. Root cause: the Latitude was never domain-joined after the Pro reimage. Fix: completed the join. Lesson logged about verifying basic assumptions first.

**Contractor account logon name mismatch.** Symptom: `runas /user:ad\contractor01` rejected the password even after a reset. Cause: the account's logon name was c01, not contractor01, despite the display name showing contractor01. Both contractor accounts had this. Fix: corrected the logon names to match the display names; re-disabled the first account.

**Credential reset not taking effect.** Symptom: password resets were not recognized on retry. Cause: the open PowerShell session held the stale token. Fix: open a new shell after an AD password change.

**Pavilion forced into Windows Home by embedded key.** Symptom: installer kept selecting Home, blocking domain join. Fix: created an `EI.cfg` to force a Pro install. Sub-issue: I first created the file incorrectly because the extension was hidden and it saved as `EI.cfg.txt`; had to enable file name extensions in Explorer to strip the `.txt` so it was a real config file rather than a text document. Reattempted with the corrected file.

**DC01 was never actually named DC01.** Symptom: running `w32tm /query /source` on SRV01 led me to check the DC, and I discovered the domain controller still carried its default Windows machine name; it was never renamed to DC01. Every document and prior journal entry refers to it as DC01, but that is not its actual hostname. The forest is small and simple, so `netdom computername` should be able to correct it, but renaming a DC carries real risk of breaking things if done wrong. I took a checkpoint of the DC VM so I can roll back if the rename goes badly, and deferred the actual rename to next session rather than attempting a high-risk change at the end of a long session. This is the first task next time.

## Open Questions

- The DC rename is the first task next session. Confirm the safe procedure for renaming a domain controller (`netdom computername` to add the new name, make it primary, then remove the old), and validate AD DS, DNS, and replication health after.
- Once the DC is actually named DC01, decide whether any committed documentation needs a note. Right now the README, outline, ADRs, and earlier journals all reference DC01 as though it is the hostname, which has been aspirational rather than accurate. The cleanest path is to make the rename real so the docs become correct, rather than editing the docs to match a wrong name.
- Confirm the Pavilion installs Pro from the `EI.cfg` and domain-joins successfully, then fix its clock and time zone to match the rest of the environment.
- The file share model is verified for jsanders (Internal allowed, Restricted denied). Still need to verify contractor01 behaves correctly now that its logon name is fixed: contractor should reach its scoped project resources and be denied financials and surveillance.

## Next Session

- Rename the domain controller to DC01 using `netdom computername`, with the checkpoint as rollback insurance, then verify AD DS, DNS, and replication health.
- Confirm the Pavilion is on Pro and domain-joined, and bring its clock and time zone in line.
- Verify contractor01 access scope now that its logon name matches.
