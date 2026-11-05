---
name: advisor-brainstorm
description: In a no-flag /advisor run, questions the developer continuously in Plan mode until the input problem is fully covered, then decides the next steps. Also grounds an --update without that interview. Use when the advisor agent handles a new brief or --update.
disable-model-invocation: true
---

# Advisor brainstorm

## Default mode, no flags

This is the brainstorm. Question the developer continuously until the problem in the input is fully covered. Do not use this loop for `--update` or `--execute`.

### Plan mode

1. Call SwitchMode with `target_mode_id` `plan` before the first question. Explain that the brainstorm stays in planning until the input problem is covered.
2. If the mode switch is declined, stop. Do not question in Agent mode and do not write files.
3. While any gap remains, do not edit files, do not decide the next steps, and do not read `advisor-handoff`.

### Covered

The problem in the input is covered only when every part of it can be stated without guessing, from the input, the repo, or an answer the developer gave in this brainstorm:

- the outcome
- what is in scope and what is out of scope
- constraints and non-goals
- facts the decision needs that are not in the repo
- which parts only the developer can do
- how to tell the problem is solved

A guess, a plausible default, or an unanswered question means it is not covered.

### Loop

1. Read the input. Read only the repo facts needed to see what is still unknown. Do not invent files, commands, or project facts. Do not copy secrets. Do not ask about a fact the repo or the input already states.
2. Ask the remaining questions that block coverage. Ask in the language of the input, then wait.
3. The developer's next message continues this same brainstorm. Check the answers against the input. Ask again about anything still uncovered, including a gap the new answer opened.
4. Do not end the brainstorm on a turn that still needs an answer. Do not write either output file, and do not present the final steps, before coverage is complete.

### After coverage

Decide the steps with the owner rules below. Call SwitchMode with `target_mode_id` `agent` and explain that the only writes are the advisor XML and markdown pair. If that switch is declined, give the decision in chat and say the files were not written. Otherwise read `advisor-handoff` and write the pair. Do not implement the steps.

Record the developer's answers that are not already in the repo inside the XML `<context>`, so a later agent does not treat them as guesses.

## Update

`--update` is not the question loop. Do not interview. The new brief is the source of truth. Read that brief, the current pair, and the repo the change depends on. Keep superseded steps in the XML history. If another open advisor handoff overlaps this decision, cite its path in context. Then decide the steps and read `advisor-handoff`.

## Decide the steps

Each step has one owner:

- `developer` — a person must do it. Accounts, secrets, purchases, legal or university actions, physical devices, and approvals outside the repo are developer steps. Also use this when the repo evidence is missing and only the developer knows the answer.
- `delegated-agent` — a project subagent other than `advisor` has a description that covers the step. Name that agent. Do not assign work to `advisor`.
- `agent` — no other subagent fits, and the step can be done from this repo once its dependencies are done.

Order steps so every developer dependency comes first. Give each step an acceptance check that a later agent can verify. Drop a step you cannot justify from the brief, the repo, or a developer answer in this brainstorm.
