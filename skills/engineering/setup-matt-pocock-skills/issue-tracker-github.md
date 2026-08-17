# Issue tracker: GitHub

Issues and specs for this repo live as GitHub issues. Use the `gh` CLI for all operations.

## Conventions

- **Create an issue**: `gh issue create --title "..." --body "..."`. Use a heredoc for multi-line bodies.
- **Read an issue**: `gh issue view <number> --comments`, filtering comments by `jq` and also fetching labels.
- **List issues**: `gh issue list --state open --json number,title,body,labels,comments --jq '[.[] | {number, title, body, labels: [.labels[].name], comments: [.comments[].body]}]'` with appropriate `--label` and `--state` filters.
- **Comment on an issue**: `gh issue comment <number> --body "..."`
- **Apply / remove labels**: `gh issue edit <number> --add-label "..."` / `--remove-label "..."`
- **Close**: `gh issue close <number> --comment "..."`

Infer the repo from `git remote -v` — `gh` does this automatically when run inside a clone.

## Pull requests as a triage surface

**PRs as a request surface: no.** _(Set to `yes` if this repo treats external PRs as feature requests; `/triage` reads this flag.)_

When set to `yes`, PRs run through the same labels and states as issues, using the `gh pr` equivalents:

- **Read a PR**: `gh pr view <number> --comments` and `gh pr diff <number>` for the diff.
- **List external PRs for triage**: `gh pr list --state open --json number,title,body,labels,author,authorAssociation,comments` then keep only `authorAssociation` of `CONTRIBUTOR`, `FIRST_TIME_CONTRIBUTOR`, or `NONE` (drop `OWNER`/`MEMBER`/`COLLABORATOR`).
- **Comment / label / close**: `gh pr comment`, `gh pr edit --add-label`/`--remove-label`, `gh pr close`.

GitHub shares one number space across issues and PRs, so a bare `#42` may be either — resolve with `gh pr view 42` and fall back to `gh issue view 42`.

## When a skill says "publish to the issue tracker"

Create a GitHub issue.

## When a skill says "fetch the relevant ticket"

Run `gh issue view <number> --comments`.

## Wayfinding operations

Used by `/wayfinder`. An effort is **two issues** — a **map** and an **answer key**. There are no child issues and no dependency links.

- **Map**: a single issue labelled `wayfinder:map`, holding the Destination / Notes / Answer key / Open questions / Not yet specified / Out of scope body. `gh issue create --label wayfinder:map`; edit it with `gh issue edit <n> --body-file -`.
- **Answer key**: a single issue labelled `wayfinder:answers`, linked from the map. `gh issue create --label wayfinder:answers`. Each answered question is **one comment**: `gh issue comment <n> --body "<entry>"`. Read the key with `gh issue view <n> --comments`.
- **Blocking**: a `blocked by:` field on each question line in the map body. A question is answered when it has an answer-key comment, and unblocked when every question it names is answered. No native issue dependencies are used.
- **Frontier query**: read the map. The frontier is the open questions that are unblocked and not marked `claimed`; first in map order wins.
- **Claim**: mark the question line `claimed` in the map body — the session's first write.
- **Resolve**: `gh issue comment <answer-key> --body "<entry>"` **first**, then remove the question from the map's Open questions.

Both labels must exist before the first run — `gh issue create --label <missing>` fails rather than creating it. Run `gh label create wayfinder:map` and `gh label create wayfinder:answers` once per repo.
