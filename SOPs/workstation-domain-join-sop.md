# Workstation Build and Domain-Join SOP

This procedure governs how every new Windows endpoint is brought into the J.S. Sanders LLC domain. It exists because the join operation alone is not the finish line: a machine can be domain-joined and still be running a throwaway hostname sitting in the wrong OU, which breaks naming consistency and causes computer-targeted GPOs to miss it. The sequence below is name, join, place, verify, in that order, and skipping a step is how endpoints end up misconfigured.

## Naming Convention

Hostnames are role-neutral. The machine's name identifies the asset, not its job. Role and policy are expressed through OU placement and GPO, never through the hostname.

- Workstations: `JSS-WSNN` (e.g., JSS-WS01, JSS-WS02), zero-padded, sequential.
- Servers: class-plus-sequence names (`DCNN`, `SRVNN`); the existing DC01 and SRV01 conform. The full rationale is recorded in ADR-0010.
- The reason for role-neutral names: a machine's role can change. A contractor workstation may later become a general workstation. If the role is baked into the hostname, the name becomes a lie the moment the role changes. Keeping names generic and expressing role through OU and GPO keeps identity and policy cleanly separated.

Record every assigned name and its asset in the Eramba asset inventory so machines can be told apart without relying on the hostname to carry meaning.

## Prerequisites (confirm before joining)

1. **Edition.** Run `winver`. Must be Windows 10/11 Pro, Enterprise, or Education. Home cannot join a domain. If a machine shows Home, reimage to Pro (use an EI.cfg in the installer's sources folder specifying Professional if the firmware carries a Home key).
2. **Network and DNS.** Run `ipconfig /all`. Confirm:
   - An IP on the correct VLAN subnet (workstations: 10.10.30.0/24).
   - Default gateway present and correct (10.10.30.1 for CLIENTS). A missing gateway means local-only connectivity and the join will fail to reach the DC.
   - DNS server pointing to DC01 (10.10.20.10). This is mandatory. Domain join fails immediately without DNS resolving to the domain controller.
3. **Time.** Confirm the machine's clock is within five minutes of DC01. Kerberos authentication fails on greater skew, and the failure looks like a wrong-password error rather than a time error. Domain members sync time from the PDC emulator automatically once joined, but the join itself needs the clock close beforehand.

## Procedure

### 1. Name (while standalone, before joining)

Rename the machine to its final hostname before joining. Naming before joining is the clean order: it avoids a post-join rename and ensures the computer object is created in AD under the correct name from the start.

```
Rename-Computer -NewName "JSS-WSNN" -Restart
```

If a machine was already joined under a default name (the mistake this SOP prevents), a post-join rename on a workstation is harmless and supported, unlike on a domain controller. Rename it, let it reboot, then continue to placement.

### 2. Join

```
Add-Computer -DomainName "ad.jssandersllc.org" -Credential ad\wsanders -Restart
```

Authenticate as wsanders. The credential authorizes the act of creating a computer object in AD; it does not make the machine belong to that user. Joining is a privileged operation by design, which reinforces the tiered admin model: provisioning is privileged, daily use is not.

### 3. Place

Move the computer object out of the default Computers container into the correct OU. Computer-targeted GPOs apply based on where the computer object lives, so placement is what determines which policies the machine receives.

- General workstations: `JSS > Workstations`
- Contractor / restricted workstations: `JSS > Workstations > Contractor-Workstations` (or the designated restricted sub-OU), so the restricted GPO targets only those machines without affecting general workstations.

In Active Directory Users and Computers on DC01: expand the domain, click the **Computers** container, right-click the machine, **Move**, select the target OU.

### 4. Verify

Confirm all four steps took:

```
hostname
systeminfo | findstr /B /C:"Domain"
```

- `hostname` returns the intended JSS-WSNN name, not a default.
- The Domain line reads `ad.jssandersllc.org`.
- In ADUC, the computer object sits in the intended OU, not the default Computers container.
- After the next Group Policy refresh (`gpupdate /force`), confirm the machine received the GPOs expected for its OU.

## Why Each Step Matters (the failure mode this prevents)

The default hostname is assigned during the out-of-box setup. If setup is rushed, the name is never set. Domain-join then proceeds under whatever name exists and drops the computer object in the default Computers container. Nothing forces the naming or placement steps, so a machine can be fully joined yet wrongly named and wrongly placed. The result is inconsistent naming across the fleet and computer-targeted GPOs that silently miss the machine because it is in the wrong OU. Following name, join, place, verify in order closes that gap.
