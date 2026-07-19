# 2026-07-18: WS1 Verification Runbook Execution

## What I Did

Executed the full WS1 verification runbook in a single sitting, the read-only session the docs had been waiting on since the runbook was written. ASA first (interfaces, ACLs, access-groups, DHCP relay, NAT), then the SG350 MAC table, DC01 (identity spot checks, GPO inventory and links, time source, dcdiag, FSMO, SRV record), SRV01 (shares, time source), both workstations (hostname, domain, gpresult, time source), the contractor01 share tests from JSS-WS02, and a short ASA reconnect for the clock rows I missed on the first pass. Changed nothing, including no pre-flight password reset: I knew contractor01's password. That was luck, not process. There is no secured store where credentials live in this environment, and remembering one from weeks ago is not a control; a proper secrets store goes on the successor's baseline before its first user is provisioned.

Every phase produced a result. Five of the runbook's expected outcomes verified clean; seven findings came out of the rest, which is the point of running a verification session instead of assuming the docs are true.

## What I Learned

The recurring lesson across the two biggest findings is that I had verified existence, not effect. The password policy GPO existed, was linked, and appeared in gpresult on every machine, and none of it mattered because Default Domain Policy outranks it at the domain root: effective policy is minimum length 7, maximum age 42 days, lockout disabled. Same shape with contractor access: the group existed with the right members, and `icacls /findsid` proved it grants nothing anywhere on disk. Both passed every check I had done before today because none of those checks asked what actually happens.

Group Policy precedence is per setting, not per GPO. The effective policy showed the shadowed GPO's 30-minute lockout duration alongside Default Domain Policy's threshold of 0, because the winning GPO does not define duration. A threshold of zero makes the inherited duration inert, but seeing both GPOs' fingerprints in one effective policy made the mechanism concrete.

The share-versus-NTFS layering held exactly as designed for a low-privilege account: contractor01 connected to every share (broad share layer, as intended) and NTFS decided the rest: Public readable, Internal, Confidential, and Restricted denied.

Smaller ones: the switch MAC table is topology ground truth (the ASA trunk announces itself by appearing on three VLANs from one port; the Hyper-V host by carrying two 00:15:5d guest MACs); pasting a block of PowerShell commands locks the output formatter onto the first object type's columns and silently eats later output, so verification commands get run one at a time; Windows 10 Fast Startup makes systeminfo boot times meaningless.

## Issues & Troubleshooting

**F1: undocumented broad permit on CLIENTS.**
- Symptom: `show running-config access-list` returned CLIENTS_IN, an ACL no document mentions, mirroring USER_IN: DNS to DC01 plus `permit ip 10.10.30.0 255.255.255.0 any`.
- Cause: the buildout-era permissiveness ADR-0008 documents for the USER VLAN was evidently applied to CLIENTS as well and never recorded.
- Fix: recorded, not fixed, per the session rules. It also answers Phase 2: CLIENTS reaches SERVERS because the ACL permits it, not via security levels. Both broad permits die together in the new era's ACL rebuild (ADR-0008 amendment, ADR-0014).

**F4: password policy GPO shadowed.**
- Symptom: `Get-ADDefaultDomainPasswordPolicy` reports minimum length 7 and LockoutThreshold 0, while GPO-Domain-PasswordPolicy (14 characters, 90 days, lockout 5/30) is linked and applied everywhere.
- Cause: Default Domain Policy outranks it at the domain root, and account policy resolves per setting in favor of the higher-precedence GPO.
- Fix: recorded. The domain ran on stock defaults, without lockout, for its entire life. The exit criterion had been checked on the GPO's existence. Carried into the new-era baseline as a rule: one authoritative account-policy GPO, link order verified, effective policy read back with the cmdlet.

**F5: contractor access mechanism exists only in the directory.**
- Symptom: `icacls E:\Shares /findsid ad\SG-Contractor-Projects /t /c` returned no matches.
- Cause: the group and its members were provisioned; the NTFS grant that would give the group meaning never was.
- Fix: recorded. contractor01's effective access is Public, identical to any authenticated user. The new-era entitlement model provisions group and grant together and tests effect.

**F8: the contractor-lifecycle GPO never existed.**
- Symptom: the ou-structure summary listed a "contractor account lifecycle" GPO assumed at the Contractors OU; the full `Get-GPO -All` inventory contains no such object.
- Cause: the 2026-05-07 control (contractor accounts rest disabled, enabled per engagement) is procedural, and the documentation later assumed a GPO that was never built.
- Fix: recorded; the GPO summary now lists the actual five GPOs exhaustively and marks this one as nonexistent. Same lesson family as F4 and F5.

**F2/F3/F6: the switch was never finished.**
- Symptom: hostname is factory default (`switchd5be16`), management sits on VLAN 1, log timestamps read June 2019.
- Cause: the SG350 got VLAN configuration during buildout and nothing else; no hostname, no SNTP, no clock, management left on the default VLAN.
- Fix: recorded. F6 matters most going forward: a device whose logs are dated seven years wrong cannot participate in monitoring.

**F7: ASA clock is hand-set.**
- Symptom: `show clock` is correct; `show running-config ntp` is empty.
- Cause: the clock was set manually at some point and no NTP source was ever configured.
- Fix: recorded. The runbook anticipated exactly this branch. Time sync verified clean across all four Windows hosts (DC01 external stratum 2, everything else on DC01); the network devices are the gap.

**Swallowed GPO output during the DC01 pass.**
- Symptom: Get-GPO and the GPOReport link extraction printed nothing when pasted in a block with the AD queries.
- Cause: PowerShell's output formatter locked onto the DistinguishedName table from the first command; objects without that property rendered blank.
- Fix: reran the commands individually with explicit expansion; full inventory captured.

Observation, not an issue: both workstation gpresults ran as wsanders, a Domain Admin, interactively logged into endpoints including the contractor machine. That is the tiered-admin gap made visible, and it goes with that criterion into the new era.

## Open Questions

- Contractor02 in SG-Contractor-Projects initially read as a surprise; resolved during doc reconciliation: both contractor accounts are documented (2026-05-07 journal, ou-structure summary). No discrepancy.
- gi7 = JSS-WS01 is assigned by OUI and boot timing; a `getmac /v` on JSS-WS01 would make it evidence rather than inference.
- The DVR leg of the time-sync criterion is retired with ADR-0014; the criterion closes with the ASA and switch findings noted rather than resolved, since both devices' fixes belong to the new era.

## Next Session

Documentation reconciliation: fill the runbook results table, resolve the [VERIFY] markers across the four affected docs, flip the verified amber zones in the diagrams, tick the closed exit criteria, commit the evidence artifact, then the close-out commit set (ADR-0014, ADR-0008/0011 amendments, outline re-baseline) and the `production-era-final` tag.

One honest deviation from the rules of engagement: PuTTY session logging never got enabled, so the ASA and switch evidence is assembled from the console output captured by copy-paste during the session, labeled as such in `../artifacts/2026-07-18-asa-sg350-verification-output.md`. The clock and NTP rows were re-checked live on 2026-07-19 with identical results. Next time, logging goes on before the first command, not after the session.
