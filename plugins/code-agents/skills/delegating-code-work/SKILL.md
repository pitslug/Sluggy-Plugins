---
name: delegating-code-work
description: Use when orchestrating a code change with the code-agents plugin (code-explorer, code-quick-implementer, code-implementer, code-validator, code-reviewer), before dispatching the first agent, and again when an agent's report comes back and the next step is unclear.
---

# Delegating code work

## Overview
The orchestrator routes, keeps conclusions, and commits. Agents do the reading, writing, running and reviewing. Every report ends with a fixed slot the orchestrator merges into `AGENTS.md`, so the repo's rules grow as the work does.

## Entry point
| Situation | Dispatch |
|---|---|
| You cannot name the files and contracts involved | `code-explorer` first |
| Well specified, one or two files, no invariant touched | `code-quick-implementer` |
| Anything else that changes code | `code-implementer` |
| Only a question about the code, no change | `code-explorer`, then answer |

If `code-quick-implementer` escalates, re-dispatch the same brief to `code-implementer` without editing it.

## The loop
1. Implementer returns: changed files, tests it ran, a validation manifest, and a `Learned for AGENTS.md` slot.
2. `code-validator` runs the manifest. On failure classified as regression or test issue, resume the SAME implementer with the evidence; it repairs and returns a smaller manifest. On environment or pre-existing, fix the environment or note it and continue.
3. `code-reviewer` reviews the diff. `Verdict: BLOCKED` sends the findings back to the same implementer, then validator, then reviewer again (re-review reports only blocking status). `Verdict: CLEAR` exits the loop.
4. Merge every `Learned for AGENTS.md` slot into `AGENTS.md` under the matching heading (Validation notes, Invariants, Conventions). One bullet per item, rule not story. If `AGENTS.md` does not exist, run `onboarding-repo-for-agents` first.
5. Commit. Advisory findings go to the repo's TODO file, not into another review round.

## Rules
- Resume the agent that owns the files; do not hand a repair to a fresh one.
- Never commit on a BLOCKED verdict or on an unrun manifest.
- `AGENTS.md` holds rules and commands only. No dated entries, no changelog; history lives in commits.
- Keep agent reports out of your reply to the user. Relay the outcome and the file list.

## Common mistakes
- Skipping the validator because the implementer said its own tests passed. Those were narrow selectors, by design.
- Treating ADVISORY findings as a reason to loop. They are filed, not fixed, unless the user asks.
- Writing the learned item as a paragraph about what happened instead of the rule that must hold.
