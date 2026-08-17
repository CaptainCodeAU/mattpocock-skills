---
"mattpocock-skills": minor
---

**Breaking:** a `/wayfinder` effort is now **two issues** — a **map** and an **answer key** — instead of one map plus a child issue per unit. The unit is a **question**, and "decision ticket" is retired.

The map holds what is still open: the destination, the notes, a link to the answer key, and the numbered **open questions**, each carrying a `type:` field and a `blocked by:` field. The answer key holds what has been settled, one appended entry per answered question. Answering a question deletes it from the map, so **the map only ever shrinks and the effort is over when it is empty** — a completion signal the previous shape didn't have.

Why the split rather than one document: the map never grows, so loading it costs the same in session forty as in session one, while the answer key grows without bound and is read an entry at a time. That is what the old skill was already reaching for with "the map is an index, not a store" — but a decision genuinely lived in two places under the old shape, in the resolution comment *and* gisted on the map. Now it lives in one.

Two ordering rules carry the safety. Answering writes **the key first, then the map**, so a session that dies between the two writes loses nothing. And every session **reconciles** on load, deleting any open question that already has an entry — which is what lets the parallel `research` subagents each append their own answer without any of them editing the map, and what tidies a half-finished session away unnoticed.

What this deletes:

- **All native tracker machinery.** No sub-issues, no dependency links, no assignee-as-claim, no per-issue frontier query. Blocking is a `blocked by:` field read out of the map, so the frontier is one document read with no API calls. The GitHub template loses the recipe that needed the blocker's numeric *database id* via `gh api` — a documented failure point ([#513](https://github.com/mattpocock/skills/issues/513), where the agent asserted GitHub has no native blocking at all).
- **Tracker inequality.** GitLab's blocking links are Premium/Ultimate only and a self-hosted Gitea has none; both degraded wayfinder before and now behave exactly like GitHub. The fallback paragraphs in all three tracker templates are gone.
- **Three labels.** `wayfinder:<type>` is retired — type is a field on the question line — so the labels you must hand-create on GitHub drop from five to two: `wayfinder:map` and `wayfinder:answers`.
- **A two-pass create.** Charting used to create the tickets and then wire blocking in a second pass, because issues need ids before they can reference each other. Numbers in one body need no ids, so it is a single edit.
- **The confusion the vocabulary created.** The most-reported wayfinder failure is an agent reading a wayfinder ticket as an implementation ticket and writing product code inside the map. Nobody mistakes a **question** for a build slice.

What it costs, stated plainly in the skill and the docs page: **the frontier no longer renders in the tracker's UI.** Native blocking was chosen precisely so a human could see what was takeable without opening the map; now you open the map. There is also no per-question comment thread, and claim contention moves onto one document instead of N assignable issues — which is a second reason for the standing one-question-per-session rule.

Reopening a settled decision is a **new question** naming what it revisits, not an edit: its entry gets a `Supersedes Q4` line and Q4's gets `Superseded by Q12`. The old answer stays, matching how `teach` handles superseded learning records.

`CONTEXT.md` retires **Decision ticket** and gains **Question**, **Map** and **Answer key**; the carve-out in the `Issue` term's `_Avoid_` line, which existed only to protect "decision ticket", goes with it. `ask-matt`, both READMEs, and the docs pages for `wayfinder`, `research`, `prototype`, `to-spec`, `setup-matt-pocock-skills`, `grill-with-docs` and `grilling` are re-synced.

**Migration.** A map charted under the old model is not recognised by the new skill. Either finish it under the old scheme by hand, or migrate: create a `wayfinder:answers` issue, copy each closed ticket's resolution comment across as one comment per question, replace the map's `Decisions so far` section with a link to the new key, and move the still-open tickets into `## Open questions` with their blocking edges written out as text.
