---
name: wayfinder
description: Plan a huge chunk of work — more than one agent session can hold — as a shared map of open questions on your issue tracker, answered one at a time into a single answer key, until the way to the destination is clear.
disable-model-invocation: true
---

A loose idea has arrived — too big for one agent session, and wrapped in fog: the way from here to the **destination** isn't visible yet. Wayfinding is about finding that way, not charging at the destination. This skill charts the way as a **shared map** on the repo's issue tracker, then works its **questions** — each one resolving to a decision, not a slice of a build to execute — one at a time, recording each answer in the map's **answer key**, until the route is clear.

The destination varies per effort, and naming it is the first act of charting — it shapes every question. It might be a spec to hand off and iterate on, a decision to lock before planning starts, or a change made in place like a data-structure migration. The map is domain-agnostic — engineering work, course content, whatever fits the shape.

## Plan, don't do

Wayfinder is **planning** by default: each question resolves to a decision, and the map is done when the way is clear — nothing left to decide before someone goes and does the thing. The pull to just do the work is usually the signal you've reached the edge of the map and it's time to hand off. An effort can override this in its **Notes** — carrying execution into the map itself — but absent that, produce decisions, not deliverables.

## Refer by name

Every question carries a number and a title. In everything the human reads — narration, the answer key — refer to it as `Q7 — Which auth provider?`, never a bare `Q7`. A wall of `Q4, Q5, Q6` is illegible; titles read at a glance. The number doesn't vanish — it's what blocking edges and answer-key entries line up on — but it rides *inside* the name, never stands in for it. The map and the answer key carry names too — an issue title, or a file path; use them the same way.

## The map and the answer key

An effort is **two** documents, and no more. The **map** holds what is still open. The **answer key** holds what has been settled.

| | The map | The answer key |
| --- | --- | --- |
| Holds | Destination, Notes, open questions, fog, out of scope | One entry per answered question |
| Grows | No — it shrinks as questions are answered, and is finished when empty | Yes, without bound |
| Written by | Editing the body | Appending an entry |
| Read | Once per session, in full | Entry titles once per session; bodies on demand |

That split is what keeps the map cheap to load however long the effort runs. **The answer key is the source of truth for what has been answered**; the map's open-question list is a cache of what has not. A question is answered when, and only when, it has an answer-key entry.

**Whether the two are issues or files, and how an entry is appended, is tracker-specific.** Consult the tracker doc's "Wayfinding operations" section for how _this_ repo expresses them. If no tracker has been provided, say so, name `/setup-matt-pocock-skills` as the way to configure one, and carry on with the local-markdown tracker unless the user redirects you — a repo with no tracker configured still gets a map.

### The map body

The whole map at low resolution, loaded once per session.

```markdown
## Destination

<what reaching the end of this map looks like — the spec, decision, or change this effort is finding its way to. One or two lines; every session orients to it before choosing a question.>

## Notes

<domain; skills every session should consult; standing preferences for this effort>

## Answer key

<a link to this map's answer key>

## Open questions

- **Q4 — <title>**
  type: research · blocked by: none

- **Q5 — <title>**
  type: grilling · blocked by: Q4 · claimed

## Not yet specified

<!-- see "Fog of war": in-scope fog you can't phrase as a question yet; graduates as the frontier advances -->

## Out of scope

<!-- see "Out of scope": work ruled beyond the destination; never graduates -->
```

### Questions

A question is a numbered entry under `## Open questions`, sized to one 100K token agent session. It carries three fields:

- **`type:`** — one of `research`, `prototype`, `grilling`, `task`. See [Question Types](#question-types).
- **`blocked by:`** — the numbers of the questions that must be answered before this one can start, or `none`.
- **`claimed`** — present once a session has taken it.

Numbers are assigned in order and are **stable forever**: an answered `Q7` stays `Q7` in the answer key, and a retired number is never reused. That is what lets a blocking edge and an answer-key entry still refer to a question long after it has left the map.

A question is **unblocked** when every question it names has an answer-key entry. The **frontier** is the open questions that are unblocked and unclaimed — the edge of the known. Reading the map is the whole frontier query; there are no tracker dependency links to consult, on any tracker.

A session **claims** a question by marking its line `claimed`, **first**, before any work, so concurrent sessions skip it. The map is the one contended write in an effort, which is the reason for the one-question-per-session rule below.

### The answer key

One entry per answered question, **appended**, never rewritten in place. The entry carries the question as well as the answer, because answering deletes the question from the map — the entry has to stand on its own.

```markdown
## Q4 — <title>

**Question:** <the question, as it read on the map>

**Answer:** <the decision>

**Assets:** <links to whatever was created while resolving it — a prototype branch, a research branch, a file. Linked, never pasted in.>
```

Reopening a settled decision is a **new question** that names what it revisits, not an edit to the old entry. When the new one is answered, add a `Supersedes Q4` line to its entry and a `Superseded by Q12` line to Q4's. Both writes are rare, and keeping the old entry preserves how the effort changed its mind.

## Question Types

Every question is either **HITL** — human in the loop, worked _with_ a human who speaks for themselves — or **AFK**, driven by the agent alone. A HITL question only resolves through that live exchange; the agent never stands in for the human's side of it (a grilling agent that answers its own questions has broken this).

- **Research** (AFK): Reading documentation, third-party APIs, or local resources like knowledge bases to surface a fact a decision waits on. Resolved by a subagent that calls the Skill tool with "research". Use when knowledge outside the current working directory is required.
- **Prototype** (HITL): Raise the fidelity of the discussion by making a cheap, rough, concrete artifact to react to — an outline, a rough take, a stub, or UI/logic code, by calling the Skill tool with "prototype". Links the prototype as an asset. Use when "how should it look" or "how should it behave" is the key question.
- **Grilling** (HITL): Conversation. The default case. Always call the Skill tool twice, for "grilling" and "domain-modeling".
- **Task** (HITL or AFK): Manual work that must happen before a _decision_ can be made — nothing to decide, prototype, or research, but the discussion is blocked until it's done. Signing up for a service so its API can be judged, provisioning access, moving data so its shape can be seen. This is the one type that _does_ rather than decides — and it earns its place by unblocking a decision, not by delivering the destination. The agent drives it alone where it can (AFK); otherwise it hands the human a precise checklist (HITL). Resolved when the work is done; the answer records what was done and any resulting facts (credentials location, new URLs, row counts) later questions depend on.

## Fog of war

The map is _deliberately_ incomplete: don't chart what you can't yet see. Beyond the live questions lies the **fog of war** — the dim view of decisions and investigations you can tell are coming but can't yet pin down, because they hang on questions still open. Answering a question clears the fog ahead of it, graduating whatever's now specifiable into fresh questions — one at a time, until the way to the destination is clear and no questions remain.

The map's **Not yet specified** section is where that dim view is written down: the suspected question, the area to revisit later. It's the undiscovered frontier _toward_ the destination — everything here is in scope, just not sharp enough to phrase as a question. Write as loosely or as fully as the view allows; it doubles as a signpost for collaborators reading where the effort is headed.

**Fog or question?** The test is whether you can state it precisely now — _not_ whether you can answer it now.

- **A question when** it's already sharp — even if it's blocked and you can't act on it yet.
- **Not yet specified when** you can't yet phrase it that sharply. Don't pre-slice the fog into question-sized pieces: it's coarser than a question, and one patch may graduate into several questions, or none, once the frontier reaches it.

**Not yet specified** excludes what's already answered (it's in the answer key), what's already an open question, and what's out of scope (the next section).

## Out of scope

Fog only ever gathers _toward_ the destination. The destination fixes the scope, so work beyond it is **out of scope** — it isn't fog, and it doesn't belong in **Not yet specified**. It gets its own **Out of scope** section on the map: work you've consciously ruled out of _this_ effort. Scope, not sharpness, lands it here.

Out-of-scope work never graduates — the frontier stops at the destination — so it returns only if the destination is redrawn, and then as a fresh effort, not a resumption.

Ruling something out of scope is a scoping act, not a step on the route. When a question that already exists turns out to sit past the destination — mis-scoped in while charting, or exposed by an answer — **delete it** from **Open questions** and leave one line in the **Out of scope** section: the gist plus why it's out of scope, keeping its number so nothing reuses it. It gets **no answer-key entry**; the key records the route actually walked, and a scope boundary isn't a step on it.

## Invocation

Two modes. Either way, **never answer more than one question per session** — with the exception of research questions.

### Chart the map

User invokes with a loose idea.

1. **Name the destination.** Call the Skill tool twice, for "grilling" and "domain-modeling", to pin down what this map is finding its way to — the spec, decision, or change. The destination fixes the scope, so it's settled first.
2. **Map the frontier.** Grill again, **breadth-first** this time: fan out across the whole space rather than deep on any one thread, surfacing the open decisions and the first steps takeable now. **If this surfaces no fog** — the way to the destination is already clear, the whole journey small enough for one session — you don't need a map. Stop and ask the user how they'd like to proceed.
3. **Create the two documents**: the map, with Destination and Notes filled in and the fog sketched into **Not yet specified**; and its answer key, empty. Link each from the other.
4. **Write the questions you can specify now** into the map's **Open questions** — numbered from `Q1`, each with its `type:` and its `blocked by:` edges, in a single edit. Numbers are yours to assign, so nothing needs creating before it can be referenced. Everything you can't yet specify stays in the fog.
5. **Fire the research subagents.** For each `research` question, mark it `claimed` and spin up a subagent that calls the Skill tool with "research" to resolve it in parallel, capturing its findings on a throwaway `research/<name>` branch. Each subagent **appends its own answer-key entry** when it finishes, and **does not touch the map** — appending entries concurrently is safe, editing the map concurrently is not. The next session reconciles the map (step 1 below).
6. Stop — charting is one session's work; it hand-answers nothing.

### Work through the map

User invokes with a map (URL or number). A question is **optional** — without one, you pick the next one, not the user.

1. Load the **map** — the low-res view — and the answer key's **entry titles**. **Reconcile**: delete from **Open questions** any question that already has an entry. This is how research answered in parallel, or a session that died mid-record, gets tidied up.
2. Choose the question. If the user named one, use it. Otherwise take the first frontier question in order. **Claim it**: mark its line `claimed` before any work.
3. Resolve it — **zoom as needed**: read the answer-key entries for the questions this one builds on, not the whole key; call the Skill tool for whichever skills the `## Notes` block names, and for this question's type. If in doubt, call the Skill tool twice, for "grilling" and "domain-modeling".
4. Record the answer, **in this order**: **append the answer-key entry first**, then delete the question from the map's **Open questions**. Key first means a session that dies between the two writes has lost nothing — the next session's reconcile finishes the job. The reverse order loses the answer outright.
5. Add newly-surfaced questions, numbering on from the highest ever used; graduate any fog the answer has made specifiable, clearing each graduated patch from **Not yet specified** so it lives only as its question. If the answer reveals a question — this one or another — sits beyond the destination, **rule it out of scope** rather than answering it on the route. If the answer invalidates other parts of the map, update or delete those questions.

The user may run unblocked questions in parallel, so expect other sessions to be editing the map concurrently.
