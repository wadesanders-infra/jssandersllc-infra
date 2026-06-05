# Data Classification Model

This document defines the data classification standard for J.S. Sanders LLC. It is a reusable reference that establishes four classification tiers and the controls each tier requires. Access decisions across the environment (NTFS permissions on file shares, contractor access scoping, incident severity, and governance evidence) derive from this model rather than from ad hoc judgment.

The model is the standard. Its enforcement on any specific system (for example the SRV01 file shares) is recorded in the relevant journal entries and implementation documentation, not here, so that this standard stays independent of any one server's layout and remains valid as the environment grows.

This artifact realizes the architecture principle that access is justified by business need and classification, and traces to NIST 800-53 RA-2 (Security Categorization).

---

## Why Classification Comes First

The classification model is defined before file shares and permissions are configured, not after. Every NTFS permission, security group assignment, and contractor access scope is meant to trace back to a tier in this model. Defining the tiers first means access control logic answers to a deliberate standard rather than being reverse-engineered from whatever permissions happened to get set. The classification drives the access control, not the other way around.

---

## The Four Tiers

| Tier | Definition | Examples | Required Controls |
|---|---|---|---|
| Public | Information that can be disclosed freely with no harm to the business. | Marketing material, public-facing documents | Minimal restrictions. Readable by any authenticated user. Integrity protected (only authorized accounts can modify). |
| Internal | Operational information not intended for outside parties, but low sensitivity. Disclosure would be inconvenient, not damaging. | General operational documents, internal correspondence | Authenticated access only. Read access scoped to business personnel via security group membership. No anonymous access. |
| Confidential | Sensitive business information whose disclosure would cause real harm. | Contracts, financials, equipment records | Role-based access control via security groups. Access logging required. More tightly scoped than Internal: only roles with a business need, not all internal users. |
| Restricted | The most sensitive data. Disclosure, loss, or tampering would cause serious harm or compromise investigations. | Surveillance footage, HR records | Least privilege. Access limited to the minimum set of accounts. Access logging and periodic audit review required. |

The tiers are cumulative in rigor: each tier inherits the expectations of the tiers below it and adds to them. Internal is stricter than Public, Confidential is stricter than Internal, and Restricted is the strictest.

---

## The Internal and Confidential Distinction

Internal and Confidential are deliberately separate tiers and are intended to differ in practice, not just in name. Internal grants authenticated read access to business personnel broadly. Confidential narrows that to roles with a specific business need and adds access logging. Treating them as identical defeats the purpose of having two tiers. Where a current implementation grants them the same access, that is an implementation gap to close, not a redefinition of the model. The model defines the intended end state; implementations are expected to converge on it.

---

## Control Enforcement and Honest Status

This model states the controls each tier requires. Some of those controls are enforced today; others are required by the model but land in later workstreams. The model is written to the intended end state, with honest notes on what is not yet live.

- Authentication and RBAC (Public, Internal, Confidential, Restricted): enforced through Active Directory authentication, security group membership, and NTFS permissions. Implemented in Workstream 1.
- Integrity protection (all tiers): enforced through NTFS permissions that restrict modification to authorized accounts. Implemented in Workstream 1.
- Access logging (Confidential, Restricted): required by the model but not yet enforced. File and object access auditing feeds the SIEM, which is deployed in Workstream 4. Until then, the requirement is documented and the control is pending, not active.
- Periodic audit review (Restricted): required by the model. Depends on the access logging above and on the Wazuh alert rules and dashboard built in Workstream 4. Pending until then.

Stating a control as required while it is still pending is deliberate. The model defines what each tier needs; the workstreams show when each control comes online. A control that is documented but not yet enforced is an honest pending item, not a false claim of protection.

---

## How This Model Is Used Elsewhere

- File share NTFS permissions: each share tier maps to a classification tier, and its permissions reflect that tier's required controls.
- Contractor access scoping: contractors are scoped to their own project documents and explicitly excluded from Confidential and Restricted data (financials, surveillance). This traces to the contractor access design.
- Incident severity: the classification of data involved in an event informs how severe the incident is treated as being.
- Governance evidence: this model is the evidence artifact for RA-2 (Security Categorization) and supports the access control mappings (AC-3, AC-6) attached in Eramba.

---

## Maintenance

This standard is expected to outlive any particular file share layout or server. If a new tier is ever needed, or a tier's required controls change, this document is updated and the change is reflected in the implementations that trace to it. The model leads; the implementations follow.
