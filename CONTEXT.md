# Matt Pocock Skills

A collection of agent skills (slash commands and behaviors) loaded by Claude Code. Skills are organized into buckets and consumed by per-repo configuration emitted by `/setup-matt-pocock-skills`.

## Language

**Issue tracker**:
The tool that hosts a repo's issues — GitHub Issues, Linear, a local `.scratch/` markdown convention, or similar. Skills like `to-tickets`, `to-spec`, and `triage` read from and write to it.
_Avoid_: backlog manager, backlog backend, issue host

**Issue**:
A single tracked unit of work inside an **Issue tracker** — a bug, task, spec, or slice produced by `to-tickets`.
_Avoid_: ticket (use only when quoting external systems that call them tickets)

**Question**:
A `wayfinder` unit — a numbered entry on a **Map** whose resolution is a decision, not a slice of a build to execute. Answering one moves it off the **Map** and into the **Answer key**.
_Avoid_: decision ticket, ticket

**Map**:
The single **Issue** holding one `wayfinder` effort's destination, its open **Questions**, and its fog. Labelled `wayfinder:map`.
_Avoid_: decision map

**Answer key**:
The single **Issue** holding one entry per answered **Question** for one **Map**. Append-only, and the source of truth for what has been answered. Labelled `wayfinder:answers`.

**Triage role**:
A canonical state-machine label applied to an **Issue** during triage (e.g. `needs-triage`, `ready-for-afk`). Each role maps to a real label string in the **Issue tracker** via `docs/agents/triage-labels.md`.

## Relationships

- An **Issue tracker** holds many **Issues**
- An **Issue** carries one **Triage role** at a time
- A **Map** and its **Answer key** are two **Issues** — one of each per `wayfinder` effort
- A **Question** lives on a **Map** until it is answered, and in the **Answer key** thereafter

## Flagged ambiguities

- "backlog" was previously used to mean both the *tool* hosting issues and the *body of work* inside it — resolved: the tool is the **Issue tracker**; "backlog" is no longer used as a domain term.
- "backlog backend" / "backlog manager" — resolved: collapsed into **Issue tracker**.
- "decision ticket" named a `wayfinder` unit that was also a real child **Issue** — resolved: wayfinder no longer creates an **Issue** per unit, so the unit is a **Question** on a **Map** and "decision ticket" is no longer used.
