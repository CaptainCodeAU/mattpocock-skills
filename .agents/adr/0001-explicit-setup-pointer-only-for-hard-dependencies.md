# Explicit `/setup-matt-pocock-skills` pointer only for hard dependencies

Engineering skills depend on per-repo config (issue tracker, triage label vocabulary, domain doc layout) seeded by `/setup-matt-pocock-skills`. Some skills cannot meaningfully function without that config — they have to publish to a specific issue tracker or apply a specific label string. Others only use it to sharpen output (vocabulary, ADR awareness) and degrade gracefully without it.

We split these into **hard-dependency** and **soft-dependency** skills:

- **Hard dependency** (`to-tickets`, `to-spec`, `triage`) — include an explicit one-liner: _"… should have been provided to you — run `/setup-matt-pocock-skills` if not."_ Without the mapping, output is wrong, not just fuzzy.
- **Soft dependency** (`diagnose`, `tdd`, `improve-codebase-architecture`) — reference "the project's domain glossary" and "ADRs in the area you're touching" in vague prose only. If the docs aren't there, the skill still works; output is just less sharp.

The split keeps soft-dependency skills token-light and avoids cargo-culting the setup pointer into places where it isn't load-bearing.

## Update, 2026-08-23

`to-tickets`, `to-spec`, and `triage` no longer block on missing config. Each now carries a graceful no-tracker fallback — say so, name `/setup-matt-pocock-skills`, and carry on with the local-markdown tracker, mentioned once in passing — mirroring the fallback `wayfinder` already had (and was never added to this ADR's hard-dependency list, which is why the drift here went unnoticed until now).

"Without the mapping, output is wrong, not just fuzzy" no longer holds for these three: without config, output is now a working local artifact, not wrong output. The **hard/soft split above is retired** as a description of current behaviour — every engineering skill that touches the tracker degrades gracefully rather than stopping. This note is left in place as history; do not edit the decision above it.
