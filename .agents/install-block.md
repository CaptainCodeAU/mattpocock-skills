# The canonical install block

One install story, one wording. `README.md` and anything else in this repo that documents installation must say **this** and nothing else. Change it here first, then propagate.

This repo is a **personal fork** of [mattpocock/skills](https://github.com/mattpocock/skills). It is not the upstream distribution, and it is not listed in any public marketplace. It installs as its own single-plugin marketplace, built from `.claude-plugin/marketplace.json`.

## The identifiers, and why they are not upstream's

|                 | This fork        | Upstream                     |
| --------------- | ---------------- | ---------------------------- |
| Marketplace     | `captaincodeau`  | `claude-plugins-official`    |
| Plugin          | `cc-skills`      | `mattpocock-skills`          |
| Skill prefix    | `cc-skills:`     | `mattpocock-skills:`         |

These were deliberately renamed away from upstream's. Left as they were, adding this repo as a marketplace would register `mattpocock-skills@mattpocock` beside the official listing's `mattpocock-skills@claude-plugins-official` — the same plugin name under a near-identical suffix, making an unqualified `/plugin install mattpocock-skills` ambiguous and an accidental upstream install easy. **Never rename them back.**

## Claude Code — the plugin

<canonical-block name="claude-code">

```bash
claude plugin marketplace add CaptainCodeAU/mattpocock-skills
claude plugin install cc-skills@captaincodeau
```

Or, from inside a session:

```
/plugin marketplace add CaptainCodeAU/mattpocock-skills
/plugin install cc-skills@captaincodeau
```

Install at user scope so the skills reach every project. They arrive prefixed `cc-skills:`. Pick up changes pushed to this fork with `claude plugin update cc-skills@captaincodeau`.

</canonical-block>

## Not the install story

- **`claude plugins install mattpocock-skills`** installs the **upstream** set from Claude Code's official marketplace. This is the one command that must never appear in this repo as an instruction.
- **`npx skills@latest add mattpocock/skills`** likewise pulls upstream. The fork equivalent, `npx skills@latest add CaptainCodeAU/mattpocock-skills`, is **untested here** — verify it resolves to this repo before documenting it as a route.
- **`scripts/link-skills.sh`** symlinks every skill from the working copy into `~/.claude/skills` and `~/.agents/skills`. It is a dev convenience for hacking on the skills themselves, not a distribution: skills land unprefixed, and it links all of them rather than the promoted set the plugin ships.
