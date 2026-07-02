# WS1 Verification Runbook: Network Policy, Identity, and Time

**Type:** Runbook (read-only verification session)
**Estimated time:** 60 to 90 minutes on-site
**Re-run trigger:** After any ACL, interface, or share permission change; before closing Workstream 1
**Access required:** ASA console or SSH (enable mode), admin logon to DC01 and SRV01, local access to JSS-WS01 and JSS-WS02

## Purpose

Every **[VERIFY]** marker in `onprem/artifacts/asa-acl-ruleset.md` and `onprem/diagrams/trust-acl-flow.md`, plus the unchecked verification items in the Workstream 1 exit criteria, resolved in one structured session. The goal is to make the documentation match the running config, not the other way around.

## Rules of Engagement

1. **This session changes nothing.** Record findings; do not fix them inline. A discrepancy becomes a documented finding, then a separate change task. (The ADR-0008 re-restriction is a change task and is explicitly out of scope here.)
2. **Log everything.** Enable session logging in PuTTY before connecting to the ASA, and run `terminal pager 0` so output is not truncated. The sanitized session log gets committed to `onprem/artifacts/` as evidence.
3. **Close out the same day.** The [VERIFY] markers come out of the docs in the same commit that records what was found; a verified fact that never reaches the repo is still unverified as far as the documentation is concerned.

## Phase 1: ASA Interfaces and Policy (console/SSH, enable mode)

| Step | Command | Resolves | Expected |
|---|---|---|---|
| 1.1 | `show nameif` | MGMT, CLIENTS, OUTSIDE security levels | OUTSIDE 0, USER 60, SERVERS 80 confirmed; MGMT and CLIENTS recorded for the first time |
| 1.2 | `show running-config interface` | Subinterface-to-VLAN mapping | Subinterfaces match VLANs 10/20/30/50 on the trunk |
| 1.3 | `show running-config access-list` | Full ACL inventory | USER_IN with the three known entries; anything else is a new finding |
| 1.4 | `show running-config access-group` | Which interfaces have ACLs applied, and direction | Confirms whether MGMT/SERVERS/CLIENTS run on implicit security-level policy |
| 1.5 | `show running-config dhcprelay` | DHCP relay configuration | Relay to 10.10.20.10 on CLIENTS |
| 1.6 | `show running-config nat` | NAT rules serving outbound internet | Outbound NAT for production VLANs; no inbound statics |

Record each actual result in the results table below before moving on.

## Phase 2: Explain CLIENTS to SERVERS

The workstations demonstrably reach DC01 (joins, Kerberos, GPO), but the mechanism is unrecorded. Use Phase 1 results to close it:

- If an ACL is applied inbound on CLIENTS: record its entries in `asa-acl-ruleset.md`; done.
- Else if CLIENTS security level (step 1.1) is higher than 80: implicit high-to-low policy explains it; record that.
- Else the reachability is unexplained; trace it: `packet-tracer input CLIENTS tcp 10.10.30.11 49152 10.10.20.10 88 detailed` and record which phase permits the flow. An unexplained permit is a finding, not a footnote.

## Phase 3: Endpoint Identity Spot Checks (JSS-WS01, then JSS-WS02)

On each workstation:

```
hostname
systeminfo | findstr /B /C:"Domain"
gpresult /r
```

On DC01:

```
Get-ADComputer JSS-WS01 | Select-Object DistinguishedName
Get-ADComputer JSS-WS02 | Select-Object DistinguishedName
```

Expected: hostnames match the convention; domain reads `ad.jssandersllc.org`; JSS-WS01 sits in JSS > Workstations, JSS-WS02 in JSS > Workstations > Contractor-Workstations; applied GPOs match each OU. (The contractor restricted GPO is not built yet; its absence on JSS-WS02 is expected, not a finding.)

## Phase 4: File Share Access Model (contractor01)

Lesson from 2026-06-08 applies: use the logon name, and open a fresh shell if any password was recently reset.

```
runas /user:ad\contractor01 cmd
```

From that shell (fill the real share names from `net share` on SRV01 first):

| Target | Expected per classification model |
|---|---|
| `dir \\10.10.20.20\<contractor-project-share>` | Allowed |
| `dir \\10.10.20.20\Internal` | Denied (Internal is scoped to business personnel) |
| `dir \\10.10.20.20\<confidential-share (financials)>` | Denied |
| `dir \\10.10.20.20\Restricted` | Denied |

Any allow where the model says deny is a finding against the implementation, not a reason to bend the model (see the classification artifact's gap language).

## Phase 5: Time Synchronization Hierarchy

| Where | Command | Expected |
|---|---|---|
| DC01 | `w32tm /query /source` | External NTP (time.nist.gov or pool.ntp.org) |
| DC01 | `w32tm /query /status` | Healthy sync, correct local time and zone |
| SRV01, JSS-WS01, JSS-WS02 | `w32tm /query /source` | DC01 (domain hierarchy) |
| Any domain member | `w32tm /monitor` | Skew across members under a few seconds |
| ASA | `show clock` and `show running-config ntp` | Clock correct; note whether NTP is configured at all (if not, that is a finding) |

## Phase 6: DC Health Baseline (post-rename assurance)

```
dcdiag
netdom query fsmo
nslookup -type=SRV _ldap._tcp.dc._msdcs.ad.jssandersllc.org
```

Expected: dcdiag passes under the DC01 name, all five FSMO roles present, SRV record resolves to DC01. This is the standing proof that the 2026-06-11 rename left no residue.

## Results Table

| Step | Expected | Actual | Pass/Fail | Doc to update |
|---|---|---|---|---|
| 1.1 | | | | asa-acl-ruleset.md, trust-acl-flow.md |
| 1.2 | | | | asa-acl-ruleset.md |
| 1.3 | | | | asa-acl-ruleset.md, trust-acl-flow.md |
| 1.4 | | | | asa-acl-ruleset.md, trust-acl-flow.md |
| 1.5 | | | | asa-acl-ruleset.md |
| 1.6 | | | | asa-acl-ruleset.md |
| 2 | | | | trust-acl-flow.md (CLIENTS edge loses [VERIFY]) |
| 3 | | | | outline exit criteria |
| 4 | | | | outline exit criteria (contractor verification) |
| 5 | | | | outline exit criteria (time sync) |
| 6 | | | | journal only |

## Closeout (same session or same day)

1. Sanitize the ASA session log (remove any secrets/hashes) and commit it to `onprem/artifacts/` as the evidence record.
2. Update `asa-acl-ruleset.md`: fill actuals, remove resolved **[VERIFY]** markers, change Status from Draft to Verified with the date.
3. Update `trust-acl-flow.md` in the same commit: amber zones that verified clean become blue; the CLIENTS edge gets its mechanism recorded.
4. Tick the resolved Workstream 1 exit criteria in `outline.md`.
5. Write the journal entry: what verified clean, what surprised you, findings opened.
6. Findings that require a change get their own task (and an ADR if the fix involves a real decision).

Suggested commit: `docs: reconcile network and identity docs against running config`
