---
name: delegating-code-work
description: Use when orchestrating a code change with the code-agents plugin (code-explorer, code-quick-implementer, code-implementer, code-validator, code-reviewer, spec-reviewer), before dispatching the first agent, when a brainstormed design is approved, and when an agent's report comes back.
---

# Delegating code work

## Overview
The orchestrator routes, keeps conclusions, and commits. Agents do the reading, writing, running and reviewing. Every report ends with a fixed slot the orchestrator merges into `AGENTS.md`, so the repo's rules grow as the work does.

## Before code: route by brainstorming path
Design happens in your own context with `superpowers:brainstorming`; it needs the user, so it is never delegated. Once the path is classified:

| Path | Sequence |
|---|---|
| Spike | `code-explorer`, report a recommendation, keep nothing |
| Bounded | design in chat, user approval, then the entry-point table below. If the design touches shared state, a file-plus-state protocol, or a trust boundary, dispatch `spec-reviewer` on the in-chat design first; otherwise it is optional |
| Architectural | brainstorming writes the spec, then `spec-reviewer` on the spec (mandatory, before `superpowers:writing-plans`); fix BLOCKING findings in the spec and re-review; then writing-plans; then the loop below once per plan task, in place of the generic implementer and reviewer in `superpowers:subagent-driven-development` |

A `spec-reviewer` verdict of BLOCKED means the spec changes, not the plan. Never start writing-plans on a BLOCKED spec.

## Entry point
| Situation | Dispatch |
|---|---|
| You cannot name the files and contracts involved | `code-explorer` first |
| Well specified, one or two files, no invariant touched | `code-quick-implementer` |
| Anything else that changes code | `code-implementer` |
| Only a question about the code, no change | `code-explorer`, then answer |

If `code-quick-implementer` escalates, re-dispatch the same brief to `code-implementer` without editing it.

Before dispatching any implementer, read the repo's follow-ups file (named under Conventions in `AGENTS.md`; default `TODO.md` at the root) for open items in the same area and put them in the brief as context. They are not scope unless the user says so.

## The loop
1. Implementer returns: changed files, tests it ran, a validation manifest, and a `Learned for AGENTS.md` slot.
2. `code-validator` runs the manifest. On failure classified as regression or test issue, resume the SAME implementer with the evidence; it repairs and returns a smaller manifest. On environment or pre-existing, fix the environment or note it and continue.
3. `code-reviewer` reviews the diff. `Verdict: BLOCKED` sends the findings back to the same implementer, then validator, then reviewer again (re-review reports only blocking status). `Verdict: CLEAR` exits the loop.
   **Round cap:** two BLOCKED rounds on the same area is the limit. On the third, ask the reviewer to name the design defect, then STOP: do not dispatch another repair. Report to the user with the defect named and the working tree left as is. A redesign is the user's decision; if they choose it, the route is brainstorming then `spec-reviewer`, never straight to an implementer.
4. Merge every `Learned for AGENTS.md` slot into `AGENTS.md` under the matching heading (Validation notes, Invariants, Conventions). One bullet per item, rule not story. If `AGENTS.md` does not exist, run `onboarding-repo-for-agents` first.
5. File and commit. Advisory findings go to the follow-ups file, one line each with the reviewer's summary and `file:line`, never into another review round. Any follow-up item this work closed moves to the closed-items file if the repo keeps one (Conventions in `AGENTS.md`), in the same commit.

## Rules
- Resume the agent that owns the files; do not hand a repair to a fresh one.
- Never commit on a BLOCKED verdict or on an unrun manifest.
- Never run a fourth repair round on the same area. Patching past the cap is how a design defect ships as a pile of special cases.
- `AGENTS.md` holds rules and commands only. No dated entries, no changelog; history lives in commits.
- Keep agent reports out of your reply to the user. Relay the outcome and the file list.

## Common mistakes
- Skipping the validator because the implementer said its own tests passed. Those were narrow selectors, by design.
- Treating ADVISORY findings as a reason to loop. They are filed, not fixed, unless the user asks.
- Pulling a related follow-up item into the slice because it was nearby. It goes in the brief as context; scope changes are the user's call.
- Writing the learned item as a paragraph about what happened instead of the rule that must hold.
- Counting a re-review that found a NEW blocking item in a different area as a round against the first area. Rounds are per area.
