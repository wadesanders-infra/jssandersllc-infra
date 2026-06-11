## Journal Entries
 
### When to Write One
 
Write a journal entry for each working session where you made meaningful progress, hit a wall, or changed direction. The journal is your lab notebook — it captures **what happened**, not what you decided (that's what ADRs are for).
 
Good triggers:
- Completed a setup or configuration step
- Troubleshot an error (especially if it took more than 10 minutes)
- Ran an attack or detection test
- Realized something about how a technology actually works vs. how you expected it to work
- Ended a session with an open question to pick up next time
 
### Filename Convention
 
```
YYYY-MM-DD-short-slug.md
```
 
- Date-first so entries sort chronologically in a directory listing
- Slug is lowercase, hyphen-delimited, describes the session's main activity
- One entry per working session (if you do two sessions in a day, append `-a` / `-b`)
 
**Examples:**
```
onprem/journal/2026-04-18-dc01-initial-build.md
onprem/journal/2026-04-20-ou-structure-and-first-users.md
hybrid/journal/2026-05-10-entra-tenant-setup.md
```
 
### Template
 
```markdown
# YYYY-MM-DD — Title Describing the Session
 
**Phase:** 1–6  
**Duration:** ~Xh  
**Goal:** What you set out to accomplish this session.
 
## What I Did
 
Narrative account of the session. Walk through what you configured,
built, tested, or broke. Include:
 
- Commands or steps that matter (not every click, but the ones
  someone reproducing this would need)
- Screenshots or references to screenshots saved in artifacts/
- Config snippets worth preserving (or note where the full config
  lives in artifacts/)
 
## What I Learned
 
Key takeaways — things that clicked, misconceptions you corrected,
or connections to coursework / cert material. This section is where
the resume value lives. A hiring manager skimming your repo will
read these.
 
## Issues & Troubleshooting
 
Problems you hit and how you resolved them. Include:
 
- The symptom (what you saw)
- What you tried
- What actually fixed it (or if it's still open)
 
If nothing went wrong, delete this section rather than writing
"no issues" — that looks like you weren't paying attention.
 
## Open Questions
 
Anything you want to investigate or decide next session.
These become the starting point for your next entry.
 
## Next Session
 
One or two concrete next steps. Keep it specific enough that
future-you can pick up without re-reading the whole entry.
