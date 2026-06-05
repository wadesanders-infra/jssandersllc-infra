# 2026-06-04: SRV01 Domain Join via PowerShell, Tiered File Shares, and NTFS Permissions

**Phase:** 1
**Duration:** ~3h
**Goal:** Finish bringing SRV01 into the domain and stand up the tiered file shares with NTFS permissions tied to the data classification model.

## What I Did

Picked up SRV01 from where it was left. Hit a block immediately: I could not change SRV01 from workgroup to domain through the GUI, likely a permissions issue. Rather than spend the session fighting the GUI, I sidestepped it with PowerShell:

```powershell
Rename-Computer -NewName "SRV01" -Restart
Add-Computer -DomainName "ad.jssandersllc.org" -Credential ad\wsanders -Restart
```

That worked. Logged in as wsanders to do the rest.

Added the File Server role to SRV01. Before creating any shares, I added a second virtual disk to keep file shares separate from the system drive, so an OS problem does not put the data at risk and a data problem does not put the OS at risk. I could not assign the D label (already in use), so the data volume came up as E.

Created the share root and a folder per data classification tier:

```powershell
New-Item -Path "E:\Shares" -ItemType Directory
New-Item -Path "E:\Shares\Public" -ItemType Directory
New-Item -Path "E:\Shares\Internal" -ItemType Directory
New-Item -Path "E:\Shares\Confidential" -ItemType Directory
New-Item -Path "E:\Shares\Restricted" -ItemType Directory
```

Broke NTFS inheritance on each tier folder so they do not retain parent permissions, converting inherited rules to explicit so existing access was preserved at the point of the break rather than stripped:

```powershell
$folders = @("E:\Shares\Public","E:\Shares\Internal","E:\Shares\Confidential","E:\Shares\Restricted")
foreach ($f in $folders) {
    $acl = Get-Acl $f
    $acl.SetAccessRuleProtection($true,$true)
    Set-Acl $f $acl
}
```

Then set explicit per-folder NTFS permissions. I wrote these out as separate per-folder blocks rather than looping, deliberately, so the documentation is clearer and each tier is easy to modify later without untangling shared code. Identities used:

```powershell
$you       = "ad\wsanders"
$docsRead  = "ad\SG-Documents-Read"
$authUsers = "NT AUTHORITY\Authenticated Users"
$system    = "NT AUTHORITY\SYSTEM"
$admins    = "BUILTIN\Administrators"
```

The resulting access model per tier:

- Public: wsanders, Administrators, SYSTEM get FullControl; Authenticated Users get ReadAndExecute.
- Internal: wsanders, Administrators, SYSTEM get FullControl; SG-Documents-Read gets ReadAndExecute.
- Confidential: wsanders, Administrators, SYSTEM get FullControl; SG-Documents-Read gets ReadAndExecute.
- Restricted: wsanders, Administrators, SYSTEM get FullControl only. No SG group, no Authenticated Users.

Each block cleared existing access rules first, then added the tier's rules with ContainerInherit,ObjectInherit so the permissions flow down into the folder contents.

Finally created the SMB shares so the folders are reachable over the network:

```powershell
New-SmbShare -Name "Public"       -Path "E:\Shares\Public"       -FullAccess "Authenticated Users"
New-SmbShare -Name "Internal"     -Path "E:\Shares\Internal"     -FullAccess "Authenticated Users"
New-SmbShare -Name "Confidential" -Path "E:\Shares\Confidential" -FullAccess "Authenticated Users"
New-SmbShare -Name "Restricted"   -Path "E:\Shares\Restricted"   -FullAccess "Authenticated Users"
```

Stopped here, before verification testing. Testing is the first task next session.

## What I Learned

When the GUI fights you over something that should be routine, the CLI is often the faster path and not a workaround to be ashamed of. The workgroup-to-domain join through the GUI was blocked, but Add-Computer did it cleanly. Knowing the PowerShell equivalent of a GUI operation turns a dead end into a one-liner.

The share-permissions versus NTFS-permissions split finally made concrete sense doing it by hand. The SMB shares are created with FullAccess for Authenticated Users on purpose: the share layer is left broad and the NTFS layer is where access is actually enforced. The effective permission is the more restrictive of the two, so anyone authenticated can connect to the share, but NTFS decides what they can actually read or change once connected. This is why Restricted can be an SMB share open to Authenticated Users at the share level while its NTFS permissions allow only me, Administrators, and SYSTEM.

Separating the data volume from the system drive is a small habit with real payoff: it decouples the failure domains of the OS and the data. This is likely overkill at this scale, but I want good security and resilience practices in place wherever they do not actively get in the way.

Breaking inheritance and writing explicit per-folder rules trades brevity for clarity. A loop would have been shorter, but explicit blocks make each tier auditable on its own and easy to change without side effects on the others. For a permission structure that traces to the data classification model, clarity is worth more than concision.

## Issues & Troubleshooting

**Could not join SRV01 to the domain via the GUI.** Symptom: changing from workgroup to domain through the GUI was blocked, probably a permissions issue. Rather than diagnose the GUI path under time pressure, I used PowerShell (Rename-Computer then Add-Computer with explicit ad\wsanders credentials), which completed the join cleanly. Resolved by sidestepping, though the underlying GUI permission cause was not investigated.

**D drive label unavailable for the data volume.** Symptom: could not assign D to the new virtual disk because the label was already in use. Used E instead. No impact beyond the share paths living under E:\Shares.

## Open Questions

- Verification is untested. Next session must confirm the access model actually behaves: jsanders (SG-Documents-Read) should be able to read Internal and Confidential, Public should be readable by any authenticated user, and Restricted should be denied to jsanders entirely. Until tested, these permissions are assumptions, not confirmed controls.
- Internal and Confidential currently grant the same access (SG-Documents-Read, ReadAndExecute on both). Confirm whether that is the intended final model or whether Confidential should be more tightly scoped than Internal, since the classification model treats them as distinct tiers.
- The GUI domain-join block was sidestepped, not diagnosed. Worth understanding why the GUI path failed in case it signals a permissions or policy issue that resurfaces elsewhere.
- Share-level permissions are broad (Authenticated Users FullControl) by design, with NTFS as the real control. Confirm this is the intended model for Restricted specifically, or whether the share-level permission should also be tightened as defense in depth.

## Next Session

- Verify the file share access model from a client: test read access as jsanders against each tier, confirm Restricted is denied, and confirm Public is readable by an authenticated user.
- Resolve any permission behavior that does not match the intended model before moving on.
