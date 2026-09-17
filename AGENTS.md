# Agent notes for Sluggy-Plugins

A Claude Code plugin marketplace owned by Chris Pitman. Each folder under `plugins/` is one installable plugin; `code-agents` is the first and holds six role-scoped coding agents plus the two skills that drive them. There is no build and no compiled code: the deliverables are Markdown agent definitions, `SKILL.md` files and two JSON manifests. Plugins are distributed by enabling the repo in claude.ai, which syncs them to every machine on the account; `claude plugin install` from the marketplace also works per machine.

## Validation notes
- Build: None (no compiled artefacts)
- Test: a with-and-without scenario pair for every behaviour change to an agent or skill: dispatch a fresh Opus agent with the scenario and the OLD text, then again with the NEW text, and compare what it does. Focused selector form: one scenario per changed rule (for example "reviewer returned BLOCKED for the third time on the same area; what next?"). Record the outcome in the commit message.
- Lint / type-check: `python -c "import json;json.load(open('.claude-plugin/marketplace.json'));json.load(open('plugins/code-agents/.claude-plugin/plugin.json'))"` (manifest JSON must parse); the three `AGENTS.md` headings the agents grep for must appear verbatim in the onboarding skill's template
- Shared build outputs: No
- Known blockers: an edit to an agent or skill is invisible to running sessions until the plugin is re-synced or updated AND the app restarts; the synced copy on this machine lives under `AppData/Roaming/Claude/local-agent-mode-sessions/.../rpm/plugin_*/` and does not show in `claude plugin list`
- Required services / env: None
- Never run here: nothing is off-limits; `git push` to main is the release step and is expected

## Invariants
- The three H2 headings `## Validation notes`, `## Invariants`, `## Conventions` are load-bearing: every agent greps a target repo's `AGENTS.md` for them by exact text, and the onboarding skill's template emits them. Never rename them in an agent, a skill or the template without changing all three together.
- Every behaviour change to an agent or skill ships with a scenario-pair test (see Validation notes). Wording that was not shown to change an agent's behaviour is not a fix.
- Every change to `plugins/code-agents/` bumps `version` in its `plugin.json` before push; the sync keys on the version, so an unbumped change never reaches an installed copy. Manifest-only wording changes may wait for the next bump.
- Reviewers use one severity vocabulary: BLOCKING / ADVISORY with the verdict line `Verdict: BLOCKED (n blocking)` or `Verdict: CLEAR (n advisory)`. The orchestrator loop keys on that line. Do not reintroduce P0–P3 or APPROVE / REQUEST_CHANGES.
- Every implementer and reviewer report ends with a `Learned for AGENTS.md` slot (a rule or `None`). It is a structural slot, not a reminder sentence; keep it in the report template.
- The repair loop is capped at two BLOCKED rounds per area; the third names the design defect and stops. Do not add a way for the orchestrator to run a fourth round.
- Design is never delegated: `superpowers:brainstorming` needs the user, so spec writing stays in the orchestrator's context. Do not add a spec-writing agent. `spec-reviewer` reviews a spec; it does not write one.
- Model and effort are pinned per agent in frontmatter and chosen per role: Opus 4.8 for explorer, quick-implementer (low) and implementer (high); Fable 5.1 high for `code-reviewer` and `spec-reviewer` (the reviews that found the serious problems were the long ones); Sonnet 4.6 low for `code-validator` because it only runs assigned commands. Change these deliberately, not as a side effect.
- Agents read `AGENTS.md` first and fall back to `CLAUDE.md`; the tool-neutral file is the primary one. Keep that order.

## Conventions
- Layout: `.claude-plugin/marketplace.json` at the root lists plugins; each plugin has `.claude-plugin/plugin.json`, `agents/*.md`, `skills/<name>/SKILL.md`. Slice plugins by audience (who wants all of it), not by file type.
- Agent files: YAML frontmatter (`name`, `description`, `model`, `effort`), then a one-line progress message naming model and effort, then `## Workflow` or `## Method`, `## Rules`, and a strict report format. Descriptions say when to use the agent, not how it works.
- Skills follow `superpowers:writing-skills`: `description` starts with "Use when", states triggers only, never summarises the workflow; body under ~500 words; fixed templates over prose reminders; a Common mistakes section.
- Line endings are LF (`.gitattributes`), commits use a conventional prefix (`docs:`, `code-agents 0.x.0:` for releases) and end with the Co-Authored-By line for the model that wrote them.
- The README is the user-facing description of the agents and the loop; update it in the same commit as the behaviour it describes.
