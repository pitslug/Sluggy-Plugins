# sluggy-plugins

A [Claude Code](https://claude.com/claude-code) plugin marketplace. Each folder under `plugins/` is one installable plugin. Add the marketplace once, then install only the plugins you want.

| Plugin | What it gives you |
|---|---|
| [`code-agents`](plugins/code-agents) | Six role-scoped coding agents, a skill that runs them as a pipeline, and a skill that prepares a repo for them |

## Install

```bash
claude plugin marketplace add pitslug/Sluggy-Plugins
```

```bash
claude plugin install code-agents@Sluggy-Plugins
```

If the plugin is enabled through claude.ai instead, it syncs to every machine on the account, but the sync does not poll GitHub: after each push, open claude.ai → Manage marketplaces → the three-dot menu on the marketplace → Check for updates, then close and reopen each Claude desktop app and check for updates on the plugin.

Installed agents appear namespaced, for example `code-agents:code-reviewer`. If you had loose copies in `~/.claude/agents/`, delete them after installing so only one version exists.

The plugin CLI has changed flags a few times; if the commands above don't match, `claude plugin --help` is the source of truth.

## code-agents

A small pipeline of agents, each with one job, one model, and one effort level. The orchestrating session delegates to them and keeps the conclusions, not the file dumps.

| Agent | Role | Model / effort |
|---|---|---|
| `code-explorer` | Read-only scout. Locates the relevant code and returns a condensed report with `path:line` references, contracts, gotchas and open questions. | Opus 4.8, low |
| `code-quick-implementer` | Small, mechanical, well-specified edits in one or two files. Runs its own narrow tests. Escalates anything ambiguous or invariant-touching to `code-implementer`. | Opus 4.8, low |
| `code-implementer` | Non-trivial slices. Test-first: writes the tests, watches them fail, implements, runs its own added tests, then hands a selector manifest to the validator. | Opus 4.8, high |
| `code-validator` | Read-only runner. Executes the assigned selectors, classifies failures (regression / test issue / pre-existing / flaky / environment), never edits. | Sonnet 4.6, low |
| `spec-reviewer` | Adversarial review of a design before any implementer sees it: lock map for shared resources, crash-point table for file-plus-state protocols, trust boundaries, invariants, and the test that catches each. Mandatory on architectural specs, optional on bounded designs. | Fable 5.1, high |
| `code-reviewer` | Adversarial pre-commit review. Finds the concrete input that breaks the change, with mandatory checks for shared resources, persisted values that drive file or process operations, file-plus-state crash points, and tests that cannot fail. Two severities: BLOCKING and ADVISORY. | Fable 5.1, high |

### How they fit together

```
brainstorm (in context)  ->  spec-reviewer  ->  writing-plans
                                                     |
explore  ->  quick-implementer | implementer  ->  validator  ->  reviewer  ->  commit
                                    ^                 |              |
                                    +---- repair -----+   loop until no BLOCKING (cap: 2 rounds per area)
```

- The implementer never claims green beyond the tests it ran itself; the validator reports the rest.
- The reviewer's verdict line is `Verdict: BLOCKED (n blocking)` or `Verdict: CLEAR (n advisory)`. Advisory items go to TODO after commit, not into another review round.
- A repo's dated engineering record (story, rejected alternatives, pinning tests) is named as the history file under `AGENTS.md` Conventions and searched by area, never read whole. Tested 2026-09-18: with the file named there, an implementer given a brief that re-proposed a recorded rejection found the entry in a 1,455-line file and refused, without any extra step in its definition; the rule that must actually be refused on still belongs under Invariants as a bullet.
- Design is never delegated: `superpowers:brainstorming` runs in the orchestrator's context because it needs the user. The spec it produces goes to `spec-reviewer` before any plan is written.
- Advisory findings are filed to the repo's follow-ups file (named in `AGENTS.md` Conventions, default `TODO.md`) and closed items move to its closed-items file, in the commit that ships the work. Open follow-ups in the same area go into an implementer's brief as context, never as scope.
- Two BLOCKED rounds on the same area is the cap. On the third the reviewer names the design defect and the orchestrator stops; a redesign is the human's call.
- Implementers and the reviewer end every report with a `Learned for AGENTS.md` slot. The orchestrator merges those into `AGENTS.md` before committing, so the repo's rules grow with the work.
- Nothing in the pipeline commits or pushes. That stays with the orchestrator and the human.

### AGENTS.md

Every agent reads `AGENTS.md` at the repo root first (falling back to `CLAUDE.md`) and looks for three headings:

- **Validation notes**: build and test commands, focused selector form, shared build outputs, known blockers such as a running app that locks files, commands never to run.
- **Invariants**: the rules that must not regress.
- **Conventions**: naming, layout, where tests live.

`AGENTS.md` is the tool-neutral file that Codex, Cursor, Copilot and others also read. A one-line `@AGENTS.md` in `CLAUDE.md` makes Claude Code see the same content.

### delegating-code-work (skill)

The orchestrator's side of the pipeline: which agent to dispatch first, the validate-review-repair loop, when to resume the same agent versus start a fresh one, merging the learned slots into `AGENTS.md`, and when to commit. Load it before dispatching the first agent on any code change.

### onboarding-repo-for-agents (skill)

Run it once in any repo that has no `AGENTS.md`, or when a validator reports it could not find "Validation notes". It detects the stack from the manifests, asks you the four questions only you can answer (locking processes, required services, never-run commands, do-not-regress rules), writes `AGENTS.md` with the exact headings above, and adds the import to `CLAUDE.md`. It does not commit.

### Models

Model IDs are pinned in each agent's frontmatter. The reviewer uses `claude-fable-5-1`; on an account without Fable access, change that line to `model: opus`. Everything else runs on generally available models.

## Adding a plugin

1. Create `plugins/<name>/.claude-plugin/plugin.json` with `name`, `version` and `description`.
2. Add `agents/`, `skills/` or `commands/` folders as needed. Skills are one folder each containing `SKILL.md`.
3. Add an entry to `.claude-plugin/marketplace.json`.
4. Bump that plugin's version on every change. Other plugins are unaffected.

Slice plugins by audience, not by file type: one plugin should be something a person wants all of.

## Licence

[MIT](LICENSE).
