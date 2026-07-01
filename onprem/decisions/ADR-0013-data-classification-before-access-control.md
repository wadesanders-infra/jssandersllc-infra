# ADR-0013: Define a Four-Tier Data Classification Standard Before Configuring Any Access Control

**Date:** 2026-07-01
**Status:** Accepted
**Workstream:** 1

## Context

File shares on SRV01 needed NTFS permissions, security group scoping, and contractor exclusions, and every one of those is an access decision that has to be justified by something. The question was whether to write a formal classification standard first and derive access control from it, or configure permissions directly and let the structure emerge. The resulting standard lives in the data classification model artifact (`onprem/artifacts/data-classification-model.md`); this ADR records why that document exists and why it has four tiers.

## Options Considered

### Option A: No formal model
- Set NTFS permissions share-by-share as needs arise, without a written standard.
- Pros: Fastest path to working shares. No documentation overhead for a two-person company.
- Cons: Access control logic becomes whatever permissions happened to get set, reverse-engineered later instead of designed. Every future decision (contractor scoping, incident severity, audit scope) gets argued from scratch. There is no artifact for governance evidence (NIST RA-2), so Workstream 4's control alignment would start from nothing.

### Option B: Three tiers (Public / Internal / Restricted)
- A formal model, but without the Internal/Confidential split; arguably proportionate to the headcount.
- Pros: Simpler. Fewer boundaries to enforce and explain.
- Cons: Collapses the line where audit obligations begin. Confidential is the tier where access logging becomes required; without it there is no clean boundary between "authenticated staff can read this" and "only specific roles, and access is logged." The governance mapping also weakens: RA-2 and the AC-3/AC-6 evidence in Eramba work better against a standard with real gradations rather than one broad middle tier.

### Option C: Four tiers (Public / Internal / Confidential / Restricted)
- A written standard defined before any share is configured, with each tier carrying defined required controls, cumulative in rigor.
- Pros: Every permission traces to a tier instead of ad hoc judgment. The Internal/Confidential boundary marks where logging obligations start. The model doubles as governance evidence and outlives any particular server layout.
- Cons: More model than two permanent users strictly need today; the split is only meaningful if implementations actually treat the tiers differently, which becomes an obligation to converge on.

## Decision

Option C: the four-tier standard is defined first, and all access control derives from it. The classification drives the access control, not the other way around. The fourth tier earns its place on two grounds: the Internal/Confidential boundary is where the access logging requirement begins, giving audit obligations a defined starting line rather than a judgment call, and the governance mapping (RA-2 in Eramba, supporting AC-3 and AC-6) needs a standard with genuine gradations to map against.

## Consequences

- The SRV01 share permissions, security group scoping, and contractor exclusions all trace to tiers rather than to ad hoc choices; the enforcement details live in the implementation journals, keeping the standard server-independent.
- The Internal/Confidential distinction is a standing obligation: where an implementation treats them identically, that is a documented gap to close, not a redefinition of the model.
- Some required controls (access logging, periodic audit review) are pending until the SIEM lands in Workstream 4. The model deliberately states requirements ahead of enforcement, so those tiers carry honest pending items rather than false claims of protection.
- Workstream 4's governance closeout inherits a ready-made RA-2 evidence artifact instead of starting categorization from scratch.
- Any future data type gets classified into an existing tier before it gets permissions; if no tier fits, the model changes first and the implementations follow.
