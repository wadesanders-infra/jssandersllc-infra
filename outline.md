# J.S. Sanders LLC: IT Infrastructure & Security Program

## What Triggered This Project

J.S. Sanders LLC is a land development company operating across active job sites in the Awendaw, SC area. The business has generated over $3 million in revenue. It manages high-value physical assets (heavy equipment, vehicles, materials) distributed across sites with limited physical oversight.

Despite the business's scale and exposure, it had no formal IT infrastructure. No network segmentation, no centralized identity, no document management, no logging, and no security program.

A piece of equipment worth over $5,000 was stolen from an active job site. No suspect was identified. No evidence was recovered. The business had no centralized surveillance with retention policies, no access controls on footage, no logging of system or user activity, and no way to correlate events across systems. The incident could not be investigated or attributed. Other scouting attempts at active sites reinforced that the business was operating with zero visibility into physical security events and zero capacity to respond.

Despite a small user population (two permanent employees plus scoped contractor accounts), the business operates across active job sites with high-value physical assets and has already experienced theft. This project treats the environment as **high-risk, low-maturity**, not low-risk, and prioritizes controls accordingly.

## What This Project Builds

Everything in this repository exists because that failure happened. The infrastructure is designed so that a similar event would be detected in near real-time, logged across relevant systems, investigable after the fact, and attributable to a specific identity or access path.

This project builds all of it from scratch: network, identity, business systems, monitoring, and security program. It is driven by real business requirements, and then secures, tests, and validates the result. An isolated lab environment on separate hardware enables adversary simulation and detection validation without risking production systems. Every workstream produces artifacts that feed the next, resulting in a repository that documents not just technical implementation but the reasoning behind each decision.

## Two Contexts

The project operates in two contexts, and it's honest about what each one provides.

**Production** is the business environment. Every system in it exists because the business needs it. Decisions are driven by real constraints: budget, headcount, operational requirements. The user population is small and the Active Directory structure reflects that honestly. The complexity comes from access control logic, investigative capability, and bridging physical and IT security, not from headcount. The value here is judgment, accountability, and operational decision-making.

**Lab** is a cloned copy of the production environment running on physically separate hardware (the primary desktop) with no network path back to production. It exists to validate detection rules and hardening controls before they touch production. The lab carries a deliberately expanded user population (service accounts with SPNs, users with delegated privileges, nested group structures) designed to create the conditions needed for realistic attack simulation. These simulated accounts exist only on the cloned domain controller and never in production AD. The value here is technical depth in offensive and defensive security. The lab is a tool inside the security program, not a standalone project.

## Domain

The AD forest root is `ad.jssandersllc.org`, a subdomain of the real owned domain `jssandersllc.org`. This avoids split-brain DNS, `.local` conflicts, and public CA certificate issues. `jssandersllc.org` is added as a UPN suffix so user accounts read as `user@jssandersllc.org`.

-----

## Hardware

| Device | Role |
|---|---|
| Cisco ASA 5506-X | Perimeter firewall and inter-VLAN router. Primary enforcement layer: NAT, inter-VLAN ACLs with security levels, syslog to Wazuh. |
| Cisco SG350-10 | Managed switch. Layer 2 VLAN tagging only. No routing, no ACL enforcement. |
| Dell OptiPlex 9020 | Hyper-V host for production VMs. NIC configured as 802.1Q trunk carrying VLAN-tagged traffic to VMs via Hyper-V vSwitches. |
| Primary desktop (i7-12700K, RTX 5070 Ti) | Lab host (cloned production VMs + Kali on isolated internal-only vSwitch), development workstation, documentation, Ollama + Open WebUI. Sits on the USER VLAN for production network access; lab VMs run on a separate internal-only vSwitch with no external uplink. |

The ASA is the enforcement layer. The SG350 handles tagging. The OptiPlex is production compute. The desktop is lab compute and development. These roles are distinct and shouldn't be confused.

## Network Architecture

### Production Network

| VLAN | ID | Subnet | Purpose |
|---|---|---|---|
| MGMT | 10 | 10.10.10.0/24 | Infrastructure management, monitoring (Wazuh) |
| SERVERS | 20 | 10.10.20.0/24 | Domain controller, document management, video management |
| CLIENTS | 30 | 10.10.30.0/24 | Business workstations |
| USER | 50 | 10.10.50.0/24 | Main desktop |

All production VLANs are enforced by ASA ACLs with security levels governing inter-VLAN traffic.

### Lab Network

The lab runs on the primary desktop on an internal-only Hyper-V vSwitch with no external uplink. There is no physical or logical network path between the lab and the production network. This isolation is mandatory because the cloned domain controller carries the same domain name and SIDs as production, and connecting it to the production network would cause AD replication conflicts and SID collisions.

## Virtual Machines

### Production (OptiPlex 9020)

| VM | VLAN | Role |
|---|---|---|
| DC01 | SERVERS | Domain controller (AD DS, DNS, DHCP) |
| SRV01 | SERVERS | Member server: Entra Connect Sync, document management |
| Wazuh | MGMT | SIEM/XDR manager |
| Jumpbox | MGMT | Admin workstation for management access |
| Win11-01 | CLIENTS | Business workstation (hybrid-joined) |
| Win10-02 | CLIENTS | Business workstation |

### Lab (Primary Desktop, Internal-Only vSwitch)

| VM | Role |
|---|---|
| DC01-Lab | Cloned domain controller with lab user population |
| Win10-Lab | Cloned workstation (lateral movement target) |
| Kali | Attack platform |

Lab VMs are cloned from production at the end of Workstream 1 and maintained as a stable baseline. The lab user population (`OU=LabUsers` with 15–20 simulated accounts) is created exclusively on DC01-Lab after cloning.

## Cloud Footprint

A free Entra ID tenant extends the on-prem domain into Azure via Entra Connect Sync running on SRV01 (never on the DC). One cloud-only Global Admin with MFA serves as the break-glass account and is never synced from on-prem. Sync is scoped to a dedicated `OU=SyncUsers` to keep cloud projection deliberate.

-----

## How the Project Is Organized

The project is organized into four workstreams. Each workstream groups related work that shares a common purpose. The workstreams are sequential at a high level (you can't deploy business systems without infrastructure, can't secure what doesn't exist yet) but individual tasks within a workstream can overlap.

| Workstream | Purpose | Estimated Duration |
|---|---|---|
| 1. Foundation | Build the network and identity platform everything else runs on | 1–2 weeks |
| 2. Business Systems | Deploy the services that solve real operational problems | 2–3 weeks |
| 3. Secure Access | Make business systems safely accessible from the field | 1 week |
| 4. Security Program | Monitor, assess, test, harden, measure, and close out governance | 6–8 weeks |

Total estimated timeline: 10–14 weeks with part-time effort. This runs alongside WGU coursework and an active job search.

### Threads That Run Throughout

A few things are deployed early and maintained across the entire project rather than introduced at the end:

**Governance threading.** Eramba Community Edition gets deployed during Workstream 1. At the end of every workstream, the controls just implemented get mapped to their applicable NIST 800-53 controls with evidence attached. The control alignment is built incrementally so that the final report at the end of Workstream 4 is being *finished*, not started from scratch. This produces a living governance artifact that shows how a security program matures over time.

**README maintenance.** A skeleton README goes up on day one: project summary, architecture overview, workstream index with links, how to navigate the repo. It gets updated at the end of each workstream so the repo is navigable for anyone who visits mid-project.

**Snapshot discipline.** Hyper-V snapshots are taken at each workstream boundary with consistent naming. Before starting, estimate snapshot sizes per VM and verify available disk on the OptiPlex. Certain snapshots are permanent anchors (clean build, vulnerable baseline, hardened state). Running out of disk mid-project is avoidable, so plan for it.

-----

# Workstream 1: Foundation

## Purpose

The business previously had no separation between systems, no centralized accounts, and no controlled access to anything. A user, a contractor, and a server all sat on the same flat network with no visibility into who accessed what. Before any business system, monitoring, or security program can exist, the foundational layers (network segmentation and identity) have to be in place.

This workstream builds the network and identity platform that every subsequent service depends on. At the end of it, J.S. Sanders LLC has a segmented business network, a functioning Active Directory domain, domain-joined workstations, the baseline group policy and access control structure that everything else inherits, and the cross-project tools deployed and ready.

This is the workstream I'm executing right now. Everything below is scoped to be actionable.

## Network

- Cisco ASA 5506-X deployed as perimeter firewall and inter-VLAN router
- Cisco SG350-10 configured for 802.1Q VLAN tagging
- Hyper-V host provisioned with virtual switches mapped to VLANs
- Production VLANs (MGMT, SERVERS, CLIENTS, USER) operational with ACLs enforcing segmentation
- Internet access via NAT through the ASA

## Identity (Production)

- Windows Server Domain Controller (DC01) with AD DS, DNS, and DHCP
- AD forest root: `ad.jssandersllc.org`
- Organizational Unit structure reflecting J.S. Sanders LLC's actual roles and access needs
- User accounts for business personnel with appropriate privilege levels
- Contractor accounts scoped to specific project resources (their own project documents, not financials or surveillance)
- Temporary accounts with expiration dates for short-term contractors
- Service accounts for application services deployed in later workstreams
- Group Policy Objects enforcing password policy, audit logging, and logon restrictions
- Security groups that will govern access to documents (Workstream 2), surveillance (Workstream 2), and administrative functions
- Domain-joined business workstations

The identity layer is honest about scale. Two permanent users, a handful of contractor and service accounts. The complexity comes from access control logic (who can see what and why) not from headcount.

## Lab Environment Setup

At the end of this workstream, production VMs are cloned onto the primary desktop to establish the lab environment:

- DC01 and one workstation cloned to the desktop's Hyper-V instance
- Clones connected to an internal-only vSwitch with no external uplink
- Kali deployed on the same internal-only vSwitch
- Cloned environment verified functional (AD services, DNS resolution, domain logon)
- Hyper-V snapshot taken as lab baseline: `Lab: Clean Clone`

The lab user population (`OU=LabUsers`) is not created at this stage. It is built on the cloned DC at the start of Workstream 4 when attack simulation begins. Production AD contains only real accounts.

## Cross-Project Tool Deployment

- Eramba Community Edition deployed on the SERVERS VLAN; asset inventory created; initial control mapping started
- Skeleton README committed
- First journal entry documenting starting state: hardware inventory, goals, expectations

## ADRs for This Workstream

1. **Domain naming rationale:** why `ad.jssandersllc.org` as a subdomain of a real owned domain rather than `.local`, covering split-brain DNS avoidance, public CA compatibility, and UPN suffix considerations
2. **Hypervisor selection:** why Hyper-V over Proxmox or ESXi given hardware constraints and licensing
3. **VLAN design rationale:** segment justification and ACL philosophy
4. **OU design rationale:** production OU structure reflecting actual business roles and access needs; no lab users in production AD
5. **Contractor access design:** how external parties receive scoped access to specific file shares, what restrictions are applied, why this approach over alternatives
6. **Lab isolation on dedicated hardware:** why the lab runs as cloned VMs on the desktop rather than as a VLAN on the production network, covering physical isolation, resource constraints, and SID/replication conflict avoidance

## Exit Criteria

- [ ] All production VMs deployed and domain-joined
- [ ] Production OU structure with two real user accounts and appropriate permissions
- [ ] Contractor access mechanism configured with scoped permissions
- [ ] GPOs enforcing password policy and audit logging
- [ ] File shares with NTFS permissions configured
- [ ] Eramba deployed with asset inventory
- [ ] Production VMs cloned to desktop; lab environment verified functional on internal-only vSwitch
- [ ] Lab baseline snapshot taken: `Lab: Clean Clone`
- [ ] ADRs committed (domain name, hypervisor, VLAN design, OU design, contractor access, lab isolation)
- [ ] Skeleton README committed
- [ ] First journal entry committed
- [ ] Hyper-V snapshot: `WS1: Clean Build` (production, on OptiPlex)

## Deliverables

- Operational segmented network with domain-joined endpoints
- Network diagram (versioned)
- ASA ACL ruleset documentation
- OU diagram and GPO summary
- User and group inventory
- Contractor access design documentation
- Eramba: initial asset inventory and control mapping
- ADRs documenting the decisions listed above
- Journal entries in `onprem/journal/`

-----

# Workstream 2: Business Systems

## Purpose

The business runs on physical documents and autonomous surveillance hardware. Permits, contracts, and compliance records live in filing cabinets, unsearchable from the field. Cameras record footage that auto-deletes with no retention controls, no access restrictions, and no integration with anything else. When the theft occurred, neither system could contribute to an investigation.

This workstream deploys the services that solve those operational problems. These systems are the reason the infrastructure exists. They authenticate against AD, they generate logs, and they serve the business.

## Document Management

Paper documents (permits, contracts, surveys, regulatory compliance records, equipment logs) aren't searchable and aren't accessible from the field. When you're on a job site and need to pull a permit number or check a contractor's scope of work, you can't.

What gets built: a self-hosted document management platform (Paperless-ngx is the leading candidate; platform selection warrants an ADR) with scanning/ingestion pipeline, OCR for full-text search, metadata taxonomy (document type, date, property/lot, contractor, status), AD-integrated authentication, role-based access controls, and a backup/recovery strategy.

## Video Surveillance

The Swann camera system runs autonomously. Footage records, auto-deletes on a fixed cycle, and sends undifferentiated motion alerts to a phone. There's no retention policy, no access controls, no audit trail, no documentation of camera placement rationale, and no integration with any other system. It's hardware that happens to be running, not a security program. This is the system that was in place when the theft occurred, and the reason no evidence was recoverable.

What gets built: not a camera upgrade, but a retained evidence system. A written physical security policy covering retention, access, alerting, and incident procedures. Centralized management and remote access. Alert tuning replacing blanket motion notifications. Access controls on footage with audit logging so that who accessed what footage and when is itself a recoverable record. Event logging configured for ingestion by the monitoring stack in Workstream 4, enabling correlation between physical security events and system activity. Camera placement documentation. Platform evaluation (stay, migrate, or augment; ADR to follow).

## Deliverables

- Operational document management system with documented taxonomy, access controls, and ingestion workflow
- Backup and retention policy documentation
- Written physical security policy
- Alert tuning and camera placement documentation
- ADRs: document management platform selection, surveillance platform decision
- Eramba: control mappings for document management and physical security controls with evidence
- Journal entries in `onprem/journal/`

-----

# Workstream 3: Secure Access

## Purpose

The business operates across active job sites, not from a single office. The theft happened at a job site. Contractors work at job sites. The documents, footage, and systems built in Workstream 2 are valuable on the local network, but they only reach full utility when they're accessible from the field, securely, with identity attribution and access controls that extend beyond the building.

This workstream makes business systems accessible from the field without sacrificing the visibility and accountability the rest of the project is designed to provide.

## What Gets Built

- Entra ID tenant provisioned and linked to on-prem AD
- Entra Connect Sync installed on SRV01 (member server, not the DC; ADR documenting this decision)
- Sync scoped to `OU=SyncUsers`; only accounts that need cloud access are projected
- Cloud-only Global Admin with MFA as break-glass account
- Secure remote access to business services (VPN, Entra-integrated access, or both; ADR to follow)
- Conditional Access policies: MFA enforcement, device compliance checks, legacy auth blocking
- Win11-01 hybrid-joined to Entra ID
- SSO validated for document management and video surveillance access

## Deliverables

- Operational hybrid identity with secure field access
- Conditional Access policy documentation
- ADRs: Entra Connect placement (member server vs. DC), remote access approach
- Eramba: control mappings for hybrid identity and access controls with evidence
- Journal entries in `onprem/journal/` and `hybrid/journal/`

-----

# Workstream 4: Security Program & Governance

## Purpose

The theft proved three things: the business couldn't detect a security event, couldn't investigate it after the fact, and couldn't attribute it to anyone. Workstreams 1–3 build the systems that make detection, investigation, and attribution technically possible. This workstream proves they actually work and closes the gaps they don't cover.

This is where the security program comes together. Monitoring, vulnerability assessment, attack simulation in the lab, detection engineering, incident response, hardening, measurable validation, and final governance reporting. These are components of one security program, not separate projects. The NIST 800-53 control alignment that has been building incrementally since Workstream 1 gets closed out here.

The lab lives inside this workstream. It's the staging environment where detections and hardening controls are validated before being pushed to production.

## Lab Population Setup

Before attack simulation begins, the cloned DC (DC01-Lab) is populated with the simulated user environment:

- `OU=LabUsers` created on DC01-Lab with 15–20 simulated user accounts distributed across roles (IT, Finance, HR, Operations) with varying privilege levels
- Kerberoastable service accounts with SPNs registered
- Overprivileged helpdesk users and nested group structures creating BloodHound attack paths
- Weak passwords on select accounts
- Hyper-V snapshot taken as vulnerable baseline: `Lab: Vulnerable Baseline`

This expanded identity layer is a testing construct. It generates the conditions that enterprise environments create organically through years of user provisioning, role changes, and administrative drift. The production environment is honest about its scale; the lab is honest about its purpose.

## Monitoring

- Wazuh SIEM/XDR deployed on the MGMT VLAN
- Wazuh agents and Sysmon (SwiftOnSecurity config) on all business endpoints
- ASA syslog forwarded to Wazuh
- Document management and video surveillance logs ingested
- Entra ID sign-in and audit logs forwarded to Wazuh
- Alert rules targeting real business threats: unauthorized document access, failed AD logon patterns, denied ASA traffic, physical security events correlated with network access, anomalous Entra sign-ins
- Operational dashboard for security posture visibility

## Vulnerability Assessment

- OpenVAS/Greenbone deployed on the SERVERS VLAN
- Full credentialed scan against all business hosts
- Baseline report documenting findings with severity ratings (the "before" snapshot)
- Eramba updated: controls mapped to findings with degraded status

## Attack Simulation & Detection Engineering (Lab)

This work operates entirely on the lab environment running on the primary desktop. It does not touch production systems. The lab has no network path to the production network.

The lab runs cloned production VMs with the expanded user population from `OU=LabUsers`: service accounts with SPNs, users with delegated privileges, nested group structures creating the conditions for realistic attack simulation.

**On-prem attack chain (Kali → DC01-Lab on internal-only vSwitch):** LLMNR/NBT-NS poisoning → hash cracking → AD enumeration (BloodHound) → Kerberoasting → lateral movement → domain compromise. Each step mapped to MITRE ATT&CK technique IDs that carry through into Sigma rules and incident documentation.

**Hybrid/cloud attack chain:** ROADtools enumeration → MSOL_ account abuse → on-prem-to-cloud lateral movement.

**Purple team loop (per detection):** Attack → check (did the Sigma rule fire?) → tune (false positives, evasion variants) → re-attack (validate) → document (iteration history). Deliverables show the iteration history, not just the final rule.

**Detection promotion:** Validated rules deploy to the production Wazuh instance. This pipeline connects the lab to production and makes the lab operationally meaningful.

## Incident Response

For each detected technique, work the full NIST SP 800-61 lifecycle:

- Alert-specific runbooks written before cases are opened
- Cases documented with ATT&CK mapping, severity classification, and observable enrichment
- Detection-to-response workflow documented: what fired, what was investigated, what changed
- Executive-level incident summaries for non-technical audience
- Post-incident review informing hardening priorities
- IR plan revised based on lessons learned

## Hardening

Remediate findings from vulnerability assessment and control gaps identified during attack simulation.

**On-prem:** Disable LLMNR/NBT-NS via GPO. Enable SMB signing. Enforce strong password policy. Remove overprivileged group memberships. Restrict file share permissions to least-privilege. Enable additional audit logging per CIS Benchmark. Restrict outbound traffic from SERVERS VLAN.

**Cloud:** Conditional Access gaps closed. Entra Connect service account hardened.

## Validation

- Re-run attack chains in the lab against the hardened clone to confirm controls break the attack paths
- Run the same OpenVAS credentialed scan as the baseline
- Produce a side-by-side comparison: findings resolved, findings remaining, residual risk accepted with documented justification

The measurable delta between baseline and post-hardening scans is one of the strongest artifacts the project produces.

## Governance Closeout

Eramba has been receiving control mappings and evidence since Workstream 1. This phase finishes the assessment:

- Complete the mapping of all applicable NIST 800-53 controls to assets
- Verify all evidence artifacts from Workstreams 1–4 are attached to corresponding controls
- Conduct risk assessments with likelihood and impact ratings
- Document risk treatment: mitigated, accepted, or transferred
- Generate control posture report
- Produce executive summary for non-technical audience

**Key control mappings (built across all workstreams):**

| Control | Title | Source | Evidence |
|---|---|---|---|
| AC-2 | Account Management | WS 1, 4 | AD user/group inventory, GPO docs |
| AC-6 | Least Privilege | WS 1, 4 | Role-based access controls, hardening |
| AU-6 | Audit Record Review | WS 4 | Wazuh alert rules and dashboard |
| CP-9 | System Backup | WS 2 | Document system backup documentation |
| IA-5 | Authenticator Management | WS 1, 3, 4 | Password policy GPO, MFA, Conditional Access |
| IR-4 | Incident Handling | WS 4 | IR runbooks, documented cases, post-incident review |
| PE-6 | Monitoring Physical Access | WS 2 | Surveillance policy, retention, access controls |
| RA-5 | Vulnerability Monitoring | WS 4 | OpenVAS baseline and delta reports |
| SC-7 | Boundary Protection | WS 1, 4 | ASA ACLs, VLAN segmentation |
| SI-4 | System Monitoring | WS 4 | Wazuh + Sysmon + Entra logs |

The business is small, but the methodology is the same one GRC analysts apply at any scale. The rigor of the process matters more than the size of the environment. Aligning NIST controls to infrastructure you built and operate for a real business, with real evidence attached, is what structured governance actually looks like.

## Deliverables

- Operational SIEM with alert rules and dashboard
- OpenVAS baseline and post-hardening scan reports with comparison
- Sigma rule files tagged with ATT&CK technique IDs
- Converted Wazuh rules and KQL queries
- ATT&CK Navigator heatmap showing detection coverage and gaps
- Purple team loop documentation with iteration history
- IR runbooks and documented incident cases
- Hardening checklist with GPO and configuration changes
- Lab re-attack validation results
- Residual risk documentation
- Eramba control posture report (PDF export)
- Executive summary: one-page governance posture for non-technical audience
- Risk assessment with ratings and treatment decisions
- Control mapping table with evidence index
- Journal entries in `onprem/journal/`, `hybrid/journal/`, and `lab/journal/`

-----

## GitHub Repository Structure

```
jssandersllc-infra/
├── README.md
├── outline.md
├── onprem/
│   ├── journal/
│   ├── decisions/
│   ├── artifacts/
│   └── diagrams/
├── hybrid/
│   ├── journal/
│   ├── decisions/
│   ├── artifacts/
│   └── diagrams/
├── lab/
│   ├── journal/
│   ├── decisions/
│   └── artifacts/
└── templates/
    ├── adr-template.md
    └── journal-template.md
```

Documentation follows the ADR methodology (globally sequential `ADR-NNNN-short-slug.md`) and session-based journal entries (`YYYY-MM-DD-short-slug.md`). The Documentation SOP governs writing standards for both. The repository name reflects the business context. Lab work has its own subdirectory but lives inside the same repo because it serves the production environment.

