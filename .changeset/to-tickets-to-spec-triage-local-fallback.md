---
"cc-skills": minor
---

**Breaking:** `to-tickets` no longer uses a tracker's native blocking or sub-issue relationships. "Blocked by" is now plain text in every ticket, on every tracker — closing two known bugs, [#513](https://github.com/mattpocock/skills/issues/513) and [#554](https://github.com/mattpocock/skills/issues/554) — at the stated cost that the frontier no longer renders in the tracker's own UI. Same fix `wayfinder` already applied to itself.

`to-tickets`, `to-spec`, and `triage` also gain wayfinder's graceful no-tracker fallback: each used to stop cold with "should have been provided to you" when no tracker was configured. Now each names `/setup-matt-pocock-skills` once, in passing, and carries on writing to `.scratch/<feature-slug>/`, adding the directory to `.gitignore` before writing into it. `to-spec` previously named no local path at all; `triage` previously delegated all of its local-mode behaviour to the tracker config doc.

Ticket ids in `to-tickets` are now stated as stable forever — assigned once, never renumbered or reused.

Re-synced: `ask-matt`, both READMEs, `docs/engineering/to-tickets.md`, `docs/engineering/to-spec.md`, `docs/engineering/triage.md`, and ADR-0001 (appended note retiring its hard/soft dependency split, now that all three degrade gracefully).

Explicitly out of scope: `to-tickets` still writes one file or issue per ticket — no board, no done log — and `/implement` is unchanged. A design stress-test found that a board redesign would need `/implement` rewritten first (it has no completion step today, confirmed on every tracker), and would reintroduce the shared-file race `docs/engineering/to-tickets.md` already records as a fixed bug — wayfinder's map is cheap because its payload lives in the append-only answer key, and a ticket's payload has to be readable *before* the work starts, so that economics don't carry over.
