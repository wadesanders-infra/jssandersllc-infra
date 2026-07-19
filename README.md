# J.S. Sanders LLC: IT Infrastructure & Security Program

J.S. Sanders LLC is a land development company operating across active job sites in Awendaw, SC. The business has generated over $3 million in revenue. It manages high-value physical assets (heavy equipment, vehicles, materials) distributed across sites with limited physical oversight.

Despite the business's scale and exposure, it had no formal IT infrastructure. No network segmentation, no centralized identity, no document management, no logging, and no security program.

> **Where the project stands (2026-07-19): this era is closed.** Workstream 1 (network, identity, endpoints) was built, then verified against the running configuration in a single evidence session on 2026-07-18. That session produced seven findings, documented rather than hidden; two of them were controls that existed on paper but had never taken effect, which is exactly what verification exists to catch. I am leaving the company, and no one remains to operate or extend the environment, so the production program ends here, proven rather than abandoned midstream (ADR-0014). Workstreams 2 through 4 were designed but never built; each is visibly retired or carried forward in [outline.md](outline.md). The repository continues as the flagship for a successor project on the same hardware: a clearly fictional synthetic healthcare enterprise range (Genetix Health Group) built for detection engineering and identity security work. The tag `production-era-final` marks the production era's last state.

**Reviewing this repo?** Start with these four artifacts; they are representative of how the whole project is documented:

- [WS1 verification runbook, executed](SOPs/ws1-verification-runbook.md): the evidence session that closed the era, with the results table filled and every claim checked against the running config
- [Journal: verification runbook execution](onprem/journal/2026-07-18-ws1-verification-runbook-execution.md): the seven findings and what they taught, including the password policy that was linked everywhere and effective nowhere
- [ADR-0014: end the production era](onprem/decisions/ADR-0014-end-production-era-continue-as-synthetic-range.md): how the era ends, what it proved, and why the repo continues as a synthetic range
- [ADR-0008: temporary permissive USER VLAN rule](onprem/decisions/ADR-0008-permissive-user-vlan-rule.md): a complete technical-debt lifecycle: accepted with a defined trigger, corrected in place when its blast radius was understated, amended at close when the trigger changed

## What Triggered This Project

A piece of equipment worth over $5,000 was stolen from an active job site. No suspect was identified. No evidence was recovered.

The business had no centralized surveillance with retention policies, no access controls on footage, no logging of system or user activity, and no way to correlate events across systems. The incident could not be investigated or attributed. There was nothing to review, no one to hold accountable, and no way to determine whether it could have been prevented.

This was not the only incident. Other scouting attempts at active sites reinforced that the business was operating with zero visibility into physical security events and zero capacity to respond to them.

Despite a small user population (two permanent employees plus scoped contractor accounts), the business operates across active job sites with high-value physical assets and has already experienced theft. This project treats the environment as **high-risk, low-maturity**, not low-risk, and prioritizes controls accordingly.

## What This Project Built

Everything in this repository exists because that failure happened.

The infrastructure was designed so that a similar event would be detected in near real-time, logged across relevant systems, investigable after the fact, and attributable to a specific identity or access path. Workstream 1 (the network, identity, and endpoint foundation) was built and verified; the rest of the program below stands as the design record, retired or carried forward per ADR-0014 when the era closed. The status table shows the honest final state of each area.

**Surveillance** is not cameras that happen to be recording. It's a retained evidence system with access controls, audit logging, tuned alerts, and event data feeding a centralized SIEM. When something happens, there is footage to review, a record of who accessed it, and a timeline to reconstruct.

**Centralized identity** ensures every access to a system, a file share, or a surveillance feed is attributable to a specific account. No anonymous access, no shared credentials, no ambiguity about who did what.

**Monitoring** correlates surveillance access events, authentication logs, endpoint activity, and network traffic into a unified timeline. If equipment goes missing, the question isn't "do we have logs." It's "what do the logs show."

**Document management** replaces paper permits, contracts, and compliance records with a searchable, field-accessible system. When a contractor's scope of work or a site access agreement needs to be verified from a job site, it's available, with access controlled and logged.

**Hybrid identity and conditional access** extend these controls to the field. Job sites are where the work happens. Systems that only function on-site don't protect assets that are distributed across sites.

**The lab** exists to validate that all of this actually works. Detections are tested against realistic attack scenarios on physically isolated replica infrastructure before they touch production. Controls that haven't been tested are assumptions, not controls.

Every architectural decision is documented with rationale. Every workstream produces artifacts that feed the next. The repository documents not just what was built, but why each decision was made and what alternatives were rejected.

## Before → Target State

| Area | Before | Target | Final Status (2026-07-18) |
|---|---|---|---|
| **Network** | Flat topology, no segmentation | Four production VLANs enforced by Cisco ASA ACLs | ✅ Built and verified. Two temporary broad permits recorded as debt: USER (ADR-0008) and an undocumented CLIENTS mirror found during verification (F1) |
| **Identity** | No centralized accounts or access control | Active Directory with OU structure, GPOs, RBAC, and scoped contractor access | ✅ Built and verified, with findings owned honestly: the hardened password policy was shadowed by link order (F4) and the contractor group held no NTFS grant (F5) |
| **Endpoints** | Unmanaged personal devices | Domain-joined, GPO-managed Windows endpoints | ✅ Built and verified: joined, named, OU-placed per SOP; the contractor restriction GPO was never built and is retired (ADR-0014, design in ADR-0007) |
| **Documents** | Paper in filing cabinets, not searchable, not field-accessible | Self-hosted DMS with OCR, full-text search, and AD-integrated access controls | Retired unbuilt (ADR-0014) |
| **Surveillance** | Cameras recording autonomously with no retention policy, no access controls, no audit trail | Retained evidence system with role-based access, tuned alerts, and logs feeding SIEM | Retired unbuilt (ADR-0014); the founding problem stays unsolved, and the ADR owns that |
| **Monitoring** | No logging, no alerting | Wazuh SIEM/XDR correlating endpoint, network, and identity telemetry | Carried forward: Wazuh deploys on the same staged hardware in the successor project |
| **Field Access** | None; systems only usable on-site | Hybrid identity via Entra Connect Sync, Conditional Access, MFA | Retired unbuilt (ADR-0014); Entra work deferred to the successor's later phases |
| **Incident Response** | No ability to detect, investigate, or attribute security events | Alert-driven detection, correlated logs in SIEM, defined response procedures | Carried forward: becomes core scenario work in the successor range |
| **Security Testing** | Nothing to test | ATT&CK-mapped attack simulation on isolated replica, Sigma-based detections | Carried forward: the successor range replaces the replica entirely (ADR-0011 superseded) |
| **Vulnerability Mgmt** | No baseline | OpenVAS credentialed scans with measurable pre/post hardening delta | Carried forward to the successor's scale-out phase |
| **Governance** | No framework, no evidence | NIST 800-53 controls aligned to real infrastructure with evidence per phase | Carried forward, registers-first, in the successor project |

## Architecture

**Hardware:** Cisco ASA 5506-X (firewall/router) - Cisco SG350-10 (L2 switching) - Dell OptiPlex 9020 (production Hyper-V host) - 2× Dell OptiPlex 3050 Micro (Wazuh SIEM host; DVR integration host) - HP Pavilion x360 (field workstation) - Dell Latitude E6500 (contractor workstation) - Primary desktop i7-12700K (lab host)

**Production Network (built):**

| VLAN | ID | Subnet | Purpose |
|---|---|---|---|
| MGMT | 10 | 10.10.10.0/24 | Infrastructure management |
| SERVERS | 20 | 10.10.20.0/24 | Domain controller, member server |
| CLIENTS | 30 | 10.10.30.0/24 | Business workstations |
| USER | 50 | 10.10.50.0/24 | Development workstation |

**Lab Network:** The planned replica lab (ADR-0011) was never built and is superseded: the successor project turns the whole flagship into a synthetic range, which makes a separate replica redundant (ADR-0014). The rebuild-from-documentation test the replica was meant to provide transfers to the successor's scripted deployment.

**Domain:** `ad.jssandersllc.org`, a subdomain of a real owned domain to avoid split-brain DNS, `.local` conflicts, and public CA issues.

**Cloud:** The planned Entra ID integration (Connect Sync on a member server, scoped to `OU=SyncUsers`) was never built; the design carries to the successor project's later phases. The `SyncUsers` OU and `svc-entraconnect` account exist and were verified 2026-07-18.

## Workstreams

| # | Workstream | Final Status |
|---|---|---|
| 1 | [Foundation](onprem/journal/): Network, AD, identity, endpoints | ✅ Built and verified; closed 2026-07-18 with exit criteria re-baselined in [outline.md](outline.md) |
| 2 | [Business Systems](onprem/journal/): Document management, surveillance program | Retired unbuilt (ADR-0014) |
| 3 | [Secure Access](hybrid/journal/): Hybrid identity, Conditional Access, field access | Retired unbuilt; design carries to the successor's later phases |
| 4 | Security Program & Governance: SIEM, attack simulation, detection engineering, hardening, NIST 800-53 alignment | Carried forward: this is what the successor range exists to do |

Eramba was planned for Workstream 1 and never deployed; the successor project runs plain-file governance registers first and treats a GRC tool as optional tooling on top, not the system of record.

## How This Repo Is Organized

```
jssandersllc-infra/
├── README.md
├── outline.md                  ← Full project plan with scope and rationale
├── risk-register.md            ← Live register of consciously carried risk (ADR-0014)
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

**The risk register** (`risk-register.md`) is the live view of consciously carried risk. Every open row is a deliberate acceptance with a named exit trigger, checked at the start of each working session; closed rows keep their history rather than being deleted (ADR-0015).

## Key Decisions

Architectural decisions are documented as ADRs. A few that shape the project:

- **Domain naming:** why `ad.jssandersllc.org` over `.local` or a fabricated domain
- **Hypervisor selection:** why Hyper-V over Proxmox or ESXi given hardware and licensing constraints
- **VLAN design:** segment justification and ACL philosophy
- **OU design:** production-only AD structure reflecting actual business roles; no simulated users in production
- **Lab isolation:** why the lab was designed as an isolated generic replica on dedicated hardware (superseded at era close by ADR-0014)
- **Entra Connect placement:** why a member server and not the DC
- **Change discipline:** risk-tiered change control with a standing risk register, adopted after two documented change-class failures (ADR-0015)
- **End of the production era:** why the program closed verified rather than continuing without an operator, and what the repository becomes next (ADR-0014)

See [`onprem/decisions/`](onprem/decisions/), [`hybrid/decisions/`](hybrid/decisions/), and [`lab/decisions/`](lab/decisions/) for the full set.
