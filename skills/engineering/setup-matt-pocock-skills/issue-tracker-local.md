# Issue tracker: Local Markdown

Issues and specs for this repo live as markdown files in `.scratch/`.

## Conventions

- **Add `.scratch/` to `.gitignore` before writing into it.** Everything under it is local and disposable, the way a spec is: it records what was known at the time, and a stale one is worse than none. Read the repo's `.gitignore` and add the line when it is absent — and describe `.scratch/` as ignored only once that is actually true, since a fresh repo will not have it. Once ignored, nothing here reaches a collaborator, so where a skill calls its artifact a "shared" map, it is shared only on a real tracker. Reach for GitHub or GitLab when more than one person needs to see it.
- One feature per directory: `.scratch/<feature-slug>/`
- The spec is `.scratch/<feature-slug>/spec.md`
- Implementation issues are one file per ticket at `.scratch/<feature-slug>/issues/<NN>-<slug>.md`, numbered from `01` — never a single combined tickets file
- Triage state is recorded as a `Status:` line near the top of each issue file (see `triage-labels.md` for the role strings)
- Comments and conversation history append to the bottom of the file under a `## Comments` heading

## When a skill says "publish to the issue tracker"

Create a new file under `.scratch/<feature-slug>/` (creating the directory if needed).

## When a skill says "fetch the relevant ticket"

Read the file at the referenced path. The user will normally pass the path or the issue number directly.

## Wayfinding operations

Used by `/wayfinder`. An effort is **two files** — a **map** and an **answer key**. There is no file per question.

- **Map**: `.scratch/<effort>/map.md` — the Destination / Notes / Answer key / Open questions / Not yet specified / Out of scope body.
- **Answer key**: `.scratch/<effort>/answers.md`, pointed at from the map's Answer key section. Each answered question is one entry appended to the bottom of the file, under its own `## Q<n> — <title>` heading.
- **Blocking**: a `blocked by:` field on each question line in `map.md`. A question is answered when `answers.md` holds an entry for it, and unblocked when every question it names is answered.
- **Frontier**: read `map.md`. The frontier is the open questions that are unblocked and not marked `claimed`; first in map order wins.
- **Claim**: mark the question line `claimed` in `map.md` and save, before any work.
- **Resolve**: append the entry to `answers.md` **first**, then remove the question from `map.md`'s Open questions.
