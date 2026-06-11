### Template
 
```markdown
# ADR-NNNN: Title as a Decision Statement
 
**Date:** YYYY-MM-DD  
**Status:** Accepted | Superseded by ADR-NNNN | Deprecated  
**Workstream:** 1-4
## Context
 
What situation or requirement prompted this decision? State the problem,
constraints, and any relevant facts. Keep it to 2–4 sentences.
 
## Options Considered
 
### Option A — [Name]
- How it works (1–2 sentences)
- Pros
- Cons
 
### Option B — [Name]
- How it works (1–2 sentences)
- Pros
- Cons
 
(Include only options you genuinely evaluated. Two is fine.)
 
## Decision
 
State the choice in one sentence. Then explain the reasoning in 2–3
sentences — connect it to the project's constraints, requirements,
and the workstream it serves.
 
## Consequences
 
- What this enables or simplifies going forward
- What trade-offs or limitations you're accepting
- Any follow-up actions this creates
```
 
### Writing Standards
 
- **Title is a decision, not a topic.** "Use Hyper-V over Proxmox for lab hypervisor" not "Hypervisor selection."
- **Context should stand alone.** Someone reading just the Context section should understand the problem without needing other documents.
- **Options Considered is not filler.** Only list alternatives you actually thought through. If there was really only one viable option, say so and explain why.
- **Consequences are forward-looking.** What does this decision mean for the next phase? What doors does it close?
- **Keep it short.** A good ADR is half a page to one page. If you're writing more, you're probably combining multiple decisions — split them.
 
### Status Lifecycle
 
```
Accepted  →  (remains Accepted unless revisited)
          →  Superseded by ADR-NNNN  (new ADR replaces this one)
          →  Deprecated  (decision no longer applies)
```
 
When superseding, update the old ADR's status line *and* reference the old one in the new ADR's Context section.
