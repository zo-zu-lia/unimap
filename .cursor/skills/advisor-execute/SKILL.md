---
name: advisor-execute
description: Executes advisor XML steps that no other agent can take, after the developer has finished every step in the paired markdown. Use when the advisor agent is invoked with --execute.
disable-model-invocation: true
---

# Advisor execute

`--execute` accepts only `audit/advisor/YYYY-MM-DD-ADVISOR-<title>.xml` as input, for example `2026-11-05-ADVISOR-example.xml`, unless intake already completed an `--update` in this same run. Do not open a new brainstorm. Do not rename the pair.

## Mode

Call SwitchMode with `target_mode_id` `agent` before changing the repo. Explain that the advisor is executing a saved plan. If the switch is declined, stop without edits.

## Gate

Read the XML and the paired markdown from `<pair><markdown>`.

The developer gate is the list under `<!-- advisor:developer-steps -->`, up to the next heading:

- `None.` means the gate is open.
- Every developer item must be `- [x]` or `- [X]`.
- An empty box, a missing marker, or a developer step still `pending` in the XML closes the gate.
- If a checked step names a repo path, that path must exist. A checked box with a missing path closes the gate.

When the gate is closed, do not edit the repo, the XML, or the markdown. Name each unfinished developer step and point at the markdown file. Do not tick checkboxes.

## Delegation

Read descriptions in `.cursor/agents/*.md`, excluding `advisor`.

If one subagent clearly covers the remaining `agent` work, do not implement it. Set the step `owner` to `delegated-agent`, fill `<delegate-to>`, set the document `status` to `delegated`, and add one history entry. Tell the user the slash command to run. Update the markdown Next steps line so it names that agent. Leave developer checkboxes unchanged.

If several agents fit, or none does, the work is not delegable. Continue.

## Do the work

Execute pending `owner="agent"` steps in `id` order. Skip developer and delegated steps. Stop before a step whose `<depends-on>` is not `done`.

Stay inside the step's `<details>` and `<acceptance>`. Do not take adjacent work. After each step, verify its acceptance and set that step to `done`. If verification fails, set the document `status` to `partial`, describe the failure in a history entry, and stop.

When you stop at a later developer step, set `status` to `partial`. When every agent step is done, set `status` to `executed`.

## Record

Read `advisor-handoff`. Update the same XML and markdown. Set `mode` to `execute`, increment `<revision>`, and append a history entry naming the steps you finished or why you stopped. In the markdown, add a short "Agent log" of what changed. Do not mark developer boxes yourself.
