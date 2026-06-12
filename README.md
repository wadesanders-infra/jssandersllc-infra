# J.S. Sanders LLC: IT Infrastructure & Security Program

J.S. Sanders LLC is a land development company operating across active job sites in Awendaw, SC. The business has generated over $3 million in revenue. It manages high-value physical assets (heavy equipment, vehicles, materials) distributed across sites with limited physical oversight.

Despite the business's scale and exposure, it had no formal IT infrastructure. No network segmentation, no centralized identity, no document management, no logging, and no security program.

> **Where the project stands:** Workstream 1 (network, identity, endpoints) is largely built, with verification in progress. Workstreams 2–4 (business systems, secure access, security program) are designed and sequenced but not yet built. Status is marked throughout this README; anything labeled *Planned* is a design target, not a finished system.

**Reviewing this repo?** Start with these four artifacts; they are representative of how the whole project is documented:

- [Network topology diagram](onprem/diagrams/network-topology.md): the current build state, rendered, with built versus planned marked
- [Workstation build and domain-join SOP](SOPs/workstation-domain-join-sop.md): a repeatable endpoint procedure written after a real process gap caused a real mistake
- [Journal: workstation naming and placement](onprem/journal/2026-06-12-workstation-naming-placement.md): root-cause analysis of that gap, the fix, and the standard that came out of it
- [ADR-0008: temporary permissive USER VLAN rule](onprem/decisions/ADR-0008-permissive-user-vlan-rule.md): how risk acceptance and technical debt are documented here, with a defined payoff trigger instead of a vague "later"

## What Triggered This Project

A piece of equipment worth over $5,000 was stolen from an active job site. No suspect was identified. No evidence was recovered.

The business had no centralized surveillance with retention policies, no access controls on footage, no logging of system or user activity, and no way to correlate events across systems. The incident could not be investigated or attributed. There was nothing to review, no one to hold accountable, and no way to determine whether it could have been prevented.

This was not the only incident. Other scouting attempts at active sites reinforced that the business was operating with zero visibility into physical security events and zero capacity to respond to them.

Despite a small user population (two permanent employees plus scoped contractor accounts), the business operates across active job sites with high-value physical assets and has already experienced theft. This project treats the environment as **high-risk, low-maturity**, not low-risk, and prioritizes controls accordingly.

## What This Project Builds

Everything in this repository exists because that failure happened.

The infrastructure is designed so that a similar event would be detected in near real-time, logged across relevant systems, investigable after the fact, and attributable to a specific identity or access path. The descriptions below are the design intent for the full program; the status table that follows shows what is built versus planned.

**Surveillance** is not cameras that happen to be recording. It's a retained evidence system with access controls, audit logging, tuned alerts, and event data feeding a centralized SIEM. When something happens, there is footage to review, a record of who accessed it, and a timeline to reconstruct.

**Centralized identity** ensures every access to a system, a file share, or a surveillance feed is attributable to a specific account. No anonymous access, no shared credentials, no ambiguity about who did what.

**Monitoring** correlates surveillance access events, authentication logs, endpoint activity, and network traffic into a unified timeline. If equipment goes missing, the question isn't "do we have logs." It's "what do the logs show."

**Document management** replaces paper permits, contracts, and compliance records with a searchable, field-accessible system. When a contractor's scope of work or a site access agreement needs to be verified from a job site, it's available, with access controlled and logged.

**Hybrid identity and conditional access** extend these controls to the field. Job sites are where the work happens. Systems that only function on-site don't protect assets that are distributed across sites.

**The lab** exists to validate that all of this actually works. Detections are tested against realistic attack scenarios on physically isolated cloned infrastructure before they touch production. Controls that haven't been tested are assumptions, not controls.

Every architectural decision is documented with rationale. Every workstream produces artifacts that feed the next. The repository documents not just what was built, but why each decision was made and what alternatives were rejected.

## Before → Target State

| Area | Before | Target | Status |
|---|---|---|---|
| **Network** | Flat topology, no segmentation | Four production VLANs enforced by Cisco ASA ACLs | ✅ Built: temporary permissive USER VLAN rule during buildout (ADR-0008) |
| **Identity** | No centralized accounts or access control | Active Directory with OU structure, GPOs, RBAC, and scoped contractor access | ✅ Built: core verified; contractor-scope verification continuing |
| **Endpoints** | Unmanaged personal devices | Domain-joined, GPO-managed Windows endpoints | 🔄 In progress: workstations joined, named, and OU-placed per SOP; contractor GPO pending |
| **Documents** | Paper in filing cabinets, not searchable, not field-accessible | Self-hosted DMS with OCR, full-text search, and AD-integrated access controls | 🔜 Planned |
| **Surveillance** | Cameras recording autonomously with no retention policy, no access controls, no audit trail | Retained evidence system with role-based access, tuned alerts, and logs feeding SIEM | 🔜 Planned |
| **Monitoring** | No logging, no alerting | Wazuh SIEM/XDR correlating endpoint, network, physical security, and cloud identity telemetry | 🔜 Planned: dedicated hardware staged, not yet deployed |
| **Field Access** | None; systems only usable on-site | Hybrid identity via Entra Connect Sync, Conditional Access, MFA | 🔜 Planned |
| **Incident Response** | No ability to detect, investigate, or attribute security events | Alert-driven detection, correlated logs in SIEM, defined response procedures | 🔜 Planned |
| **Security Testing** | Nothing to test | ATT&CK-mapped attack simulation on physically isolated lab clone, Sigma-based detections promoted to production | 🔜 Planned |
| **Vulnerability Mgmt** | No baseline | OpenVAS credentialed scans with measurable pre/post hardening delta | 🔜 Planned |
| **Governance** | No framework, no evidence | NIST 800-53 controls aligned to real infrastructure with evidence from each implementation phase | 🔜 Planned |

## Architecture

**Hardware:** Cisco ASA 5506-X (firewall/router) - Cisco SG350-10 (L2 switching) - Dell OptiPlex 9020 (production Hyper-V host) - 2× Dell OptiPlex 3050 Micro (Wazuh SIEM host; DVR integration host) - HP Pavilion x360 (field workstation) - Dell Latitude E6500 (contractor workstation) - Primary desktop i7-12700K (lab host)

**Production Network (built):**

| VLAN | ID | Subnet | Purpose |
|---|---|---|---|
| MGMT | 10 | 10.10.10.0/24 | Infrastructure management; monitoring host (planned) |
| SERVERS | 20 | 10.10.20.0/24 | Domain controller, member server; document and video management (planned) |
| CLIENTS | 30 | 10.10.30.0/24 | Business workstations |
| USER | 50 | 10.10.50.0/24 | Development workstation |

**Lab Network (planned):** Cloned production VMs + Kali will run on an internal-only Hyper-V vSwitch on the desktop with no network path to production. The cloned DC will carry the same domain name and SIDs as production; physical isolation is mandatory.

**Domain:** `ad.jssandersllc.org`, a subdomain of a real owned domain to avoid split-brain DNS, `.local` conflicts, and public CA issues.

**Cloud (planned, Workstream 3):** A free Entra ID tenant will be linked via Entra Connect Sync on a member server (not the DC). Sync will be scoped to `OU=SyncUsers`, with a cloud-only Global Admin with MFA as break-glass.

## Workstreams

| # | Workstream | Status |
|---|---|---|
| 1 | [Foundation](onprem/journal/): Network, AD, identity, lab clone, cross-project tooling | In progress |
| 2 | [Business Systems](onprem/journal/): Document management, surveillance program | Planned |
| 3 | [Secure Access](hybrid/journal/): Hybrid identity, Conditional Access, field access | Planned |
| 4 | Security Program & Governance: Lab population, SIEM, vuln assessment, attack simulation, detection engineering, hardening, validation, NIST 800-53 control alignment | Planned |

Governance threads throughout. Eramba deploys in Workstream 1 and accumulates control mappings incrementally so the control alignment is finished at the end of Workstream 4, not started from scratch.

## How This Repo Is Organized

```
jssandersllc-infra/
├── README.md
├── outline.md                  ← Full project plan with scope and rationale
├── onprem/
│   ├── journal/                ← Session-based lab notebook entries
│   ├── decisions/              ← Architecture Decision Records (ADRs)
│   ├── artifacts/              ← Configs, scripts, screenshots
│   └── diagrams/               ← Network, OU, and architecture diagrams
├── hybrid/
│   ├── journal/
│   ├── decisions/
│   ├── artifacts/
│   └── diagrams/
├── lab/
│   ├── journal/
│   ├── decisions/
│   └── artifacts/              ← Sigma rules, attack documentation, heatmaps (Workstream 4)
├── SOPs/                       ← Standing operating procedures
└── templates/
    ├── adr-template.md
    └── journal-template.md
```

**ADRs** follow a globally sequential numbering system (`ADR-NNNN-short-slug.md`) across all scopes. Each records the context, options considered, decision, and consequences.

**Journal entries** (`YYYY-MM-DD-short-slug.md`) capture what happened per working session: what was built, what broke, what was learned, and what's next. Entries are honest about failures, open questions, and pauses; that's the point of keeping them.

**SOPs** codify repeatable procedures after a process gap causes a real mistake. Each one encodes the corrected workflow; the first covers workstation naming, domain join, OU placement, and verification.

## Key Decisions

Architectural decisions are documented as ADRs. A few that shape the project:

- **Domain naming:** why `ad.jssandersllc.org` over `.local` or a fabricated domain
- **Hypervisor selection:** why Hyper-V over Proxmox or ESXi given hardware and licensing constraints
- **VLAN design:** segment justification and ACL philosophy
- **OU design:** production-only AD structure reflecting actual business roles; no simulated users in production
- **Lab isolation:** why the lab will run as cloned VMs on dedicated hardware rather than as a VLAN on the production network
- **Entra Connect placement:** why a member server and not the DC

See [`onprem/decisions/`](onprem/decisions/), [`hybrid/decisions/`](hybrid/decisions/), and [`lab/decisions/`](lab/decisions/) for the full set.
