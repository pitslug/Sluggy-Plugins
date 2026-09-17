---
name: onboarding-repo-for-agents
description: Use when a repository has no AGENTS.md, when a code-validator or implementer reports that "Validation notes" or "Invariants" could not be found, or when setting up a new project so delegated agents know how to build, test and what never to break.
---

# Onboarding a repo for agents

## Overview
The custom agents (code-explorer, code-quick-implementer, code-implementer, code-validator, code-reviewer) read `AGENTS.md` at the repo root and grep for three fixed headings. This skill produces that file with exactly those headings, so no agent silently skips a section that was named slightly differently.

`AGENTS.md` is the tool-neutral file (Codex, Cursor, Copilot and others read it). `CLAUDE.md` imports it with one line, so Claude Code sees the same content.

## When to use
- Repo has no `AGENTS.md`.
- A validator reported "no Validation notes found" or classified a known blocker as a regression.
- A reviewer or implementer had no "Invariants" section to check against.
- Starting delegated work in a repo for the first time.

Not for: rewriting an existing, healthy `AGENTS.md`. Add to it under the same headings instead.

## Procedure
1. **Detect the stack.** From the tree and manifests, find: build command, test command, test runner, lint or type-check command, and whether test targets share build outputs (several test projects in one .NET solution, one Gradle build, one Cargo workspace). Do not guess a command you have not seen in a manifest or script.
2. **Ask the user the questions only they can answer.** Use AskUserQuestion or plain chat, one message:
   - Any process that locks files or ports while running (a desktop app, a dev server)?
   - Services or credentials tests need?
   - Commands that must never be run here (migrations, deploys, anything that touches live data)?
   - Rules that must never be broken (the "do not regress" list, if one exists elsewhere)?
3. **Write `AGENTS.md`** from the template below. Keep the three H2 headings verbatim. Fill every slot; write "None known" rather than deleting a slot. Every bullet must trace to something you read in the repo or something the user said; if neither supports it, the slot stays "None known".
4. **Import from `CLAUDE.md`.** If `CLAUDE.md` exists, add the line `@AGENTS.md` near the top if it is not already there. If it does not exist, create it containing only that line.
5. **Confirm.** Show the user the resulting file and stop. Do not commit.

## Template
```markdown
# Agent notes for <repo name>

<One paragraph: what this repo is and the stack.>

## Validation notes
- Build: `<command>`
- Test: `<command>`; focused selector form: `<example targeting one test file/class/project>`
- Lint / type-check: `<command or "None">`
- Shared build outputs: <"Yes: build once, then run selectors sequentially or with --no-build" | "No">
- Known blockers: <e.g. "A running <app> locks the build; quit it first or report blocked"> | "None known"
- Required services / env: <list | "None">
- Never run here: <list | "None">

## Invariants
<One bullet per rule that must not regress. Each bullet: the rule, then a short "why" in parentheses. Link to a longer doc if one exists.>
- None known

## Conventions
<Naming, layout, where tests live, commit style. Short bullets only.>
- Follow-ups file: `<path | "None">`; closed items file: `<path | "None">`; history file (dated engineering record, searched by area, never read whole): `<path | "None">`
- None known
```

## Common mistakes
- Renaming a heading ("Testing notes", "Do not regress"): the agents grep for the exact text. Keep the three H2s verbatim.
- Guessing a test command from the stack instead of reading the manifest or script.
- Pasting a whole existing CLAUDE.md into Invariants. Move only the rules; leave history where it is.
- Inventing Conventions or a "why" for an Invariant that nothing in the repo or the user's answers supports. "None known" is the correct output for an unobserved slot.
- Dropping the `@AGENTS.md` import, so Claude Code reads a stale CLAUDE.md while other tools read the new file.
