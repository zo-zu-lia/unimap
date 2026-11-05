---
name: advisor-intake
description: Parses /advisor input paths and the --update and --execute flags, checks file types, and selects brainstorm, update, or execute. Use when the advisor agent starts a run or the user invokes /advisor.
disable-model-invocation: true
---

# Advisor intake

Read this at the start of every `/advisor` run. The advisor subagent is the caller. Do not start a brainstorm from an ordinary coding request.

## Parse

Take the text after `/advisor`. Split on whitespace. A token that starts with `--` is a flag.

- `--execute` is a boolean.
- `--update` requires the next token. That token is the path to an existing `audit/advisor/*.xml` file. It is not the brief.
- Exactly one remaining token is the input path. Zero or several input tokens is a parse error.
- Any other flag is a parse error.

On a parse error, stop. Quote the command shape and the tokens you could not place. Do not guess a file.

```text
/advisor <brief.md|brief.txt> [--update <audit/advisor/YYYY-MM-DD-ADVISOR-title.xml>] [--execute]
/advisor <audit/advisor/YYYY-MM-DD-ADVISOR-title.xml> --execute
```

## Select the mode

| `--execute` | `--update` | Input | Mode |
| --- | --- | --- | --- |
| no | no | `.md` or `.txt` | new brainstorm |
| no | yes | `.md` or `.txt` | update |
| yes | no | `.xml` inside `audit/advisor/` | execute |
| yes | yes | `.md` or `.txt` | update, then execute only if the gate passes |
| yes | no | `.md` or `.txt` | stop |
| no | no | `.xml` | stop |
| yes | yes | `.xml` | stop |

When you stop, say which mode was rejected and which command would be valid. Do not brainstorm and do not edit files.

## Check files

- The input file must exist. Read it. Refuse empty input.
- For `--update` or `--execute`, the XML must already exist at `audit/advisor/YYYY-MM-DD-ADVISOR-<title>.xml` and must contain `<pair><markdown>`. The paired file must be `local/advisor/` plus the same filename with `.md`. Reject a path that does not match that pattern.
- For a new brainstorm, name both files `YYYY-MM-DD-ADVISOR-<title>`. `YYYY-MM-DD` is today's date. The agent token is the literal `ADVISOR`. `<title>` is the brief filename without its extension, in lowercase, with spaces and underscores turned into `-`, and with every character outside letters, digits, and `-` removed. Collapse repeated hyphens and trim them from the ends. Example: brief `example.md` on 2026-11-05 becomes `2026-11-05-ADVISOR-example.xml` and `2026-11-05-ADVISOR-example.md`. If the title is empty, stop and ask for a filename to use. If either destination exists, stop and tell the user to rerun with `--update` and that XML path. Do not rename a file on update or execute.
- Create `audit/advisor/` and `local/advisor/` only when you are about to write a new pair.

## Then

- No flags: read `advisor-brainstorm` and stay in its question loop. Do not read `advisor-handoff` and do not write a pair until that skill says the problem in the input is fully covered.
- Update without execute: read `advisor-brainstorm`, then `advisor-handoff`. Do not run the question loop.
- Execute only: read `advisor-execute`. The XML is the only input. Do not read a brief unless the XML cites one as evidence.
- Update plus execute: finish the update and write both files. Do not run the question loop. Then apply the execute gate to the updated pair. If the gate fails, stop after the update.
