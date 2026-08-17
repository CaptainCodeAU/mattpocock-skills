## What it does

`wayfinder` takes an effort too big for one agent [session](https://www.aihero.dev/ai-coding-dictionary/session) — an idea whose **destination** you can name but whose route you cannot yet see — and charts it as a shared **map** of open **questions** on your issue tracker, then answers them one at a time into a single **answer key** until the way is clear.

It plans, it does not do. Every question resolves to a decision, not a slice of a build to execute, and the map is finished when nothing is left to decide before someone goes and builds the thing. That one rule is what separates a wayfinder question from an ordinary implementation [ticket](https://www.aihero.dev/ai-coding-dictionary/ticket), and it is the rule agents break most often. When the map clears, wayfinder hands off; it does not carry on into code.

## When to reach for it

You invoke this by typing `/wayfinder` — the [agent](https://www.aihero.dev/ai-coding-dictionary/agent) won't reach for it on its own.

It is the heaviest, densest flow in the set, so the trigger is narrow: the effort has to be genuinely larger than one agent session can hold, and the route to the destination has to be foggy. The split is a clean one: `/grill-with-docs` for single-session planning, `/wayfinder` for multi-session planning.

| What you have in front of you | What to run |
| --- | --- |
| A well-scoped feature you can settle in one sitting | [grill-me](https://aihero.dev/skills-grill-me), or [grill-with-docs](https://aihero.dev/skills-grill-with-docs) when there is a codebase |
| A greenfield project, or a build spanning many sessions, with the route still unclear | `/wayfinder` |
| A thread where the deciding is already done | [to-spec](https://aihero.dev/skills-to-spec) — skip straight past the map |
| A cleared wayfinder map | [to-spec](https://aihero.dev/skills-to-spec), then [to-tickets](https://aihero.dev/skills-to-tickets) and [implement](https://aihero.dev/skills-implement) |
| An existing session that has already grown too big | say "hand off to `/wayfinder`" — [handoff](https://aihero.dev/skills-handoff) bridges into a map as well as out of one |

Greenfield is not a requirement. Wayfinder is used routinely on legacy and half-built codebases, and it is arguably sharper there, because a lot of the fog is "what is already true here" rather than "what should we do".

## Prerequisites

The map and its answer key live on the repo's issue tracker, so wayfinder needs the tracker wiring that [setup-matt-pocock-skills](https://aihero.dev/skills-setup-matt-pocock-skills) lays down. That step writes a "Wayfinding operations" section describing where the two live and how an answer gets appended, for GitHub, GitLab, or local markdown. Wayfinder resolves that doc through the pointer in your `CLAUDE.md` / `AGENTS.md` rather than a fixed path; with no tracker configured at all it falls back to local markdown files.

On GitHub, two labels have to exist before the first run — `wayfinder:map` and `wayfinder:answers` — because `gh issue create --label <missing>` fails rather than creating them, and setup does not create them for you.

What the tracker no longer has to provide is native dependency links. Blocking is a plain `blocked by:` field read out of the map, so a self-hosted Gitea, or GitLab's free tier where blocking links are a paid feature, now behaves exactly like GitHub. That gap is closed.

## The map, the answer key, and the frontier

An effort is **two documents**, and no more — two issues on a real tracker, or `map.md` and `answers.md` under `.scratch/<effort>/` on the local-markdown one.

The **map** holds what is still open; on a tracker it is an issue labelled `wayfinder:map`. The property that matters: **it only ever shrinks.** Answering a question deletes it from the map, so the map is finished when no questions are left on it.

The **answer key** is linked from the map, and on a tracker it is an issue labelled `wayfinder:answers`. Each answered question is appended to it — one comment, or one entry in the file — carrying the question, the answer, and links to whatever got built along the way. It is append-only, and it is the source of truth for what has been answered — a question counts as answered when, and only when, it has an entry there.

That split is what keeps a long effort cheap. The map never grows, so loading it costs the same in session forty as in session one; the answer key grows without bound, and you read only the entries you need.

Four things live on the map:

- **Destination** — what reaching the end of this map looks like. Naming it is the first act of charting, before any question exists, because the destination fixes the scope every question is measured against.
- **Open questions** — the live ones, numbered `Q1`, `Q2`… each with a `type:` and a `blocked by:` field. Numbers are stable forever and never reused, so a blocking edge or an answer-key entry still resolves long after a question has left the map.
- **Not yet specified** — the **fog of war**. Decisions you can tell are coming but cannot yet phrase sharply. The test for fog versus question is whether you can state it precisely *now*, not whether you can answer it. Answering a question clears the fog ahead of it and graduates whatever is now specifiable into fresh questions.
- **Out of scope** — work ruled beyond the destination. Fog only ever gathers *toward* the destination, so out-of-scope work is deleted from the open list and never graduates.

The **frontier** is the open questions that are unblocked and unclaimed — the edge of the known. A question is unblocked when every question it names has an answer-key entry, so computing the frontier is one read of the map plus the key's entry titles, with no tracker API calls at all. A session claims a question by marking its line `claimed` in the map before doing any work. Questions are referred to by number *and* title — `Q7 — Which auth provider?`, never a bare `Q7`; a wall of numbers is illegible in narration.

Two ordering rules do the safety work. Answering writes **the key first, then the map**, so a session that dies between the two writes has lost nothing. And every session **reconciles** on load, deleting any open question that already has an entry — which is how research answered in parallel, or a half-finished session, gets tidied away without anyone having to spot it.

## The four question types

Every question carries a `type:` field, and is either **[HITL](https://www.aihero.dev/ai-coding-dictionary/human-in-the-loop)** — worked with a human who speaks for themselves — or **[AFK](https://www.aihero.dev/ai-coding-dictionary/afk)**, driven by the agent alone. A HITL question only resolves through the live exchange; an agent that answers its own [grilling](https://www.aihero.dev/ai-coding-dictionary/grilling) questions has broken it.

| Type | Mode | Reach for it when | Resolved by |
| --- | --- | --- | --- |
| `grilling` | HITL | The default. The question can be settled by talking it through. | [grilling](https://aihero.dev/skills-grilling) plus [domain-modeling](https://aihero.dev/skills-domain-modeling), in a fresh session |
| `prototype` | HITL | "How should this look" or "how should this behave" — a question talking cannot settle. | [prototype](https://aihero.dev/skills-prototype), with the built artifact linked from the answer-key entry as an asset |
| `research` | AFK | A fact outside the working directory is blocking a decision. | A [research](https://aihero.dev/skills-research) [subagent](https://www.aihero.dev/ai-coding-dictionary/subagent), fired at charting time and burned down in parallel on a `research/<name>` branch |
| `task` | Either | Nothing to decide, but manual work blocks a decision — provisioning access, signing up for a service, moving data so its shape can be seen. | The agent alone where it can, otherwise a precise checklist for the human |

`task` is the only type that *does* rather than decides, and it earns its place by unblocking a decision — never by delivering a piece of the destination. This is the type that goes wrong most often in practice: agents interpret it as an implementation step and start writing product code inside the map.

Research is the only exception to *one question per session*.

## Common questions

**How is this different from `/grill-with-docs`? Which should I start with?**
Session count, not project size. `/grill-with-docs` is single-session planning; wayfinder is multi-session planning. If you can hold the whole thing in one conversation, grilling is the cheaper and better tool, and wayfinder is genuinely slower and denser for that case. The community shorthand that has settled on it: wayfinder only makes sense if the work doesn't fit into a single session. This is by a distance the most-asked wayfinder question, and it keeps being asked because the descriptions do not tell you where your own task sits on that line — you have to judge the session count yourself.

**When it asks for the "destination", does it mean the end of this session or the end of everything?**
The whole map — the destination of the entire map, not just the initial session. The question reads ambiguously because wayfinder is by definition a multi-session tool, so a session-scoped answer never makes sense. Typical destinations are a [spec](https://www.aihero.dev/ai-coding-dictionary/spec) to hand off, a decision to lock before planning starts, a proof of concept, or a change made in place like a data migration.

**The map is cleared. Why do I still need `/to-spec` and `/to-tickets` — didn't wayfinder already write the spec and make the tickets?**
No. Wayfinder's questions resolve to decisions, and by the time the map closes every one of them has moved into the answer key. What is left is an answer key full of decisions, which is not a build plan. [to-spec](https://aihero.dev/skills-to-spec) collapses them into one spec — `/to-spec #<map_issue>`, following the map's link to the key — and [to-tickets](https://aihero.dev/skills-to-tickets) slices that into tracer-bullet implementation tickets. Looping the map straight into [implement](https://aihero.dev/skills-implement) skips the collapse and throws the linked detail away. Go straight to implementation only when the effort turned out genuinely small. People do run the abbreviated pipeline and report it working; the two extra steps buy you an explicit spec artifact that a reviewer or a colleague can read, which matters more the less solo you are.

**My agent started writing production code in the middle of a wayfinder session.**
The most-reported failure with this skill, and there is a real hole behind it. Wayfinder's "plan, don't do" default can be overridden in the map's **Notes** — but the Notes are written by the agent, so the constraint and its exemption live in the same file the constrained party owns. One user watched an agent write "this map carries execution" into its own Notes and then read it back in later sessions as its own licence, building on a live server. There is no hard in-skill stop for "I meant the default." Until there is: read the Notes on any map you didn't chart yourself, keep implementation in its own sessions, and treat any question typed `task` that looks like a slice of the build as mis-typed.

**I charted 27 questions, and by the time I got to the thirteenth, the rest no longer made sense.**
A real and repeatedly-reported outcome, verbatim from a field report. Wayfinder's default instinct is to plan comprehensively, and a map whose later questions rest on assumptions the earlier ones invalidate is exactly the waterfall trap the skill is accused of. Two things push back on it. Scope the map to a bounded destination rather than to the whole product — practitioners consistently report that maps scoped to one defined epic behave better than a sprawling "implement V1", and planning something very big is not the goal in the first place — shipping small increments is. And [prototype](https://www.aihero.dev/ai-coding-dictionary/prototyping) aggressively: the whole reason the route stays current is that uncertainty is flushed out by cheap concrete artifacts before implementation depends on it. Wayfinder is "prototypemaxxing", not "planmaxxing".

**Can I work several questions in parallel?**
The frontier is built to show you what is takeable, and blocking edges are there so parallel work is safe on paper. In practice one-at-a-time is the safer default, and there is now a second reason for it: the map is a single document, so two sessions claiming or deleting questions at the same moment can clobber each other's edit. The answer key cannot be clobbered — appending an entry is safe from any number of sessions at once, which is precisely why parallel research works — but the map can. Beyond that, users working two grilling questions at once get asked in one session a question they just answered in the other, because the sessions share no [context](https://www.aihero.dev/ai-coding-dictionary/context). There is also a known gap on prototype questions: an agent has been reported building three UI variations, choosing one itself, and writing its own answer into the key — the selection is yours to make, and the skill does not currently say so loudly enough. If you do run in parallel, review the blocking edges yourself first.

**Do I have to use GitHub Issues?**
No — any issue tracker works, and they are now all equal. Wayfinder needs nothing more than somewhere to keep two documents and a way to append to one of them, so GitHub's native sub-issues and dependency links have stopped being load-bearing and a tracker without them is no longer second-class. GitLab, Linear, Jira, Gitea and local markdown all get used.

One caveat survives: local markdown puts the artifacts in your repo, which tends to lead to accidental persistence. What used to push open-source maintainers toward it — public trackers filling with agent-generated planning tickets — is a much smaller problem now, because an effort is two documents rather than one plus one per question.

**The grilling is exhausting. Every question is three paragraphs long.**
This is the sharpest live complaint about wayfinder and it is not resolved. The decomposition one user gave: the verbosity itself causes decision exhaustion, and the length strips out *why* a question is being asked, so you lose the chain from decision to decision as the map gets longer. The verbosity looks like a property of the current set of [models](https://www.aihero.dev/ai-coding-dictionary/model) rather than of the skill, and no fix has landed. Practitioner mitigations in circulation: run a lower [reasoning effort](https://www.aihero.dev/ai-coding-dictionary/effort), and put a plain-language instruction in your global `CLAUDE.md`. Expect to spend real thought here regardless — the amount of thinking wayfinder demands from you is not a defect, it is most of what it is for.

**A decision I already recorded turned out to be wrong. Do I edit the entry or raise a new question?**
A new question. The answer key is append-only, so reopening a settled decision means a fresh numbered question naming what it revisits; when that one is answered, its entry gets a `Supersedes Q4` line and Q4's gets a `Superseded by Q12` line. The old answer stays, because how the effort changed its mind is worth keeping — the same reasoning the [teach](https://aihero.dev/skills-teach) skill applies to its learning records.

The agent's instinct is still unhelpful: it tends to design around a bad decision rather than challenge it, so you have to steer manually. What works is telling wayfinder plainly what changed — it updates the map and revises the affected questions. Scope changes mid-map are recoverable. A map you *designed* to change is a scoping smell.

**I have a map with a dozen child issues from before this changed. Does it migrate?**
Not automatically, and the skill will not recognise the old shape. Either finish an in-flight map under the old model — nothing breaks, you are just working the previous scheme by hand — or migrate it: create a `wayfinder:answers` issue, copy each closed ticket's resolution comment across as one comment per question, replace the map's `Decisions so far` section with a link to the new key, and move the still-open tickets into `## Open questions` with their blocking edges written out as text.

**Where did `decision-mapping` go?**
It is this skill, renamed to `wayfinder` in v1.1 and invoked as `/wayfinder`. "Decision map" was jargon and was also inaccurate, since only one of the four types is really a decision by itself. The reframe gave the skill one coherent vocabulary — destination, fog of war, frontier, the map — instead of an invented term layered on top.

The unit has since been renamed as well. It used to be a **decision ticket**, a real child issue, and the "decision" qualifier was there to stop people reading it as an implementation ticket. Now that an effort creates no issue per unit, it is simply a **question** — which retires the term and the confusion it was fighting in one go, because nobody mistakes a question for a build slice.

## It's working if

- The destination is written down and agreed before a single question exists.
- The whole effort produces exactly two documents — two issues on a tracker, or `map.md` and `answers.md` on local markdown. One artifact per question means the skill is running the old shape.
- Every open entry reads as a question. Anything that reads "build the X" is either mis-typed or belongs downstream of the map.
- The map gets shorter, not longer. Questions leave it as they are answered, and the effort is over when **Open questions** is empty.
- A session answers one question: it appends an entry to the answer key, *then* deletes that question from the map. In that order. Then it stops.
- **Not yet specified** shrinks over time. A patch of fog that graduates into a question disappears from that section rather than living in both places.
- When the opening breadth-first grill turns up no fog at all, the skill stops and tells you the effort is small enough to skip the map.
- The session that finishes the map hands you toward a spec, not a pull request.

## Where it fits

`wayfinder` is a **situational on-ramp**, not the default front door. The grill-led idea → ship chain is still where most work starts; wayfinder is what you climb onto when the idea is too big to hold in one session, and it merges back onto that chain at [to-spec](https://aihero.dev/skills-to-spec), because a cleared map hands off rather than builds.

Underneath, it is mostly other skills wearing wayfinder's scheduling: [grilling](https://aihero.dev/skills-grilling) and [domain-modeling](https://aihero.dev/skills-domain-modeling) answer the default question type, [prototype](https://aihero.dev/skills-prototype) answers the questions that talking cannot, and [research](https://aihero.dev/skills-research) runs as a subagent so its reading never lands in your session. [handoff](https://aihero.dev/skills-handoff) is the bridge in and out — into a map from a conversation that outgrew itself, out of one when a side quest appears mid-session. For anything else, [ask-matt](https://aihero.dev/skills-ask-matt) routes over the whole set.
