---
name: advisor-handoff
description: Writes the advisor English XML handoff under audit/advisor and the shorter developer markdown under local/advisor. Use when the advisor agent saves or refreshes a decision, including after --execute.
disable-model-invocation: true
---

# Advisor handoff

Write both files in one turn. The XML is the contract for later agents. The markdown is the shorter note for the local developer.

## Paths

- XML: `audit/advisor/YYYY-MM-DD-ADVISOR-<title>.xml`
- Markdown: `local/advisor/YYYY-MM-DD-ADVISOR-<title>.md`
- Example: `2026-11-05-ADVISOR-example.xml` and `2026-11-05-ADVISOR-example.md`
- Intake chooses the filename. On update or execute, keep the paths already stored in `<pair>`. Do not rename.

## XML

UTF-8. English prose only. Element names stay as in the template. Escape `&`, `<`, and `>` in text. Put long text in elements. Use ISO-8601 UTC timestamps.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<advisor-output>
  <meta>
    <revision>1</revision>
    <updated>2026-11-05T00:00:00Z</updated>
    <mode>brainstorm</mode>
    <status>proposed</status>
    <source-input>path/to/brief.md</source-input>
    <pair>
      <xml>audit/advisor/2026-11-05-ADVISOR-example.xml</xml>
      <markdown>local/advisor/2026-11-05-ADVISOR-example.md</markdown>
    </pair>
  </meta>
  <context>
    <project-state>What was true in the repo, with paths.</project-state>
    <input-summary>What the brief asked for.</input-summary>
    <related-handoff>audit/advisor/2026-11-05-ADVISOR-other.xml</related-handoff>
  </context>
  <decision>
    <goal>One outcome.</goal>
    <rationale>Why this sequence, including tradeoffs you rejected.</rationale>
  </decision>
  <next-steps>
    <step id="1" owner="developer" status="pending">
      <title>Short name</title>
      <details>Enough for a later agent to continue without this chat.</details>
      <acceptance>How to tell the step is done.</acceptance>
      <depends-on></depends-on>
      <delegate-to></delegate-to>
    </step>
  </next-steps>
  <delegation>
    <candidate agent="name" feasible="false">Why it does or does not fit.</candidate>
  </delegation>
  <history>
    <entry revision="1" at="2026-11-05T00:00:00Z">Created.</entry>
  </history>
</advisor-output>
```

Allowed values:

- `mode`: `brainstorm`, `update`, `execute`
- `status`: `proposed`, `ready`, `blocked`, `delegated`, `executed`, `partial`
- `owner`: `developer`, `agent`, `delegated-agent`
- step `status`: `pending`, `done`, `blocked`, `skipped`, `superseded`

Use `proposed` when any developer step is pending. Use `ready` when every remaining step is `agent` or `delegated-agent`. Use `blocked` when the brief lacks a fact you cannot infer. Use `delegated` when the remaining work was handed to another agent. Use `executed` or `partial` only after an execute pass.

On `--update`, increment `<revision>`, change `<updated>`, set `mode` to `update`, append one `<history><entry>`, and mark replaced steps `superseded` instead of deleting them. Move superseded steps out of `<next-steps>` into `<history>` as a one-line entry that includes the old step id and title.

Omit `<related-handoff>` when there is none. Leave `<depends-on>` and `<delegate-to>` empty when unused. `delegate-to` is required for `owner="delegated-agent"`.

## Markdown

Write it in the language of the brief. If the brief mixes languages, use the language the user wrote the request in.

Keep it shorter than the XML. Include the decision, the ordered steps in one line each, and the developer-only steps. Omit rationale, rejected alternatives, delegation essays, and file-by-file evidence.

The developer section lists only work the agent must not do. Those checkboxes are the execute gate. Leave them unchecked. If there is no such work, write `None.` under the marker.

```markdown
# Brief title

<!-- advisor:pair audit/advisor/2026-11-05-ADVISOR-example.xml -->

- Status: proposed
- Input: `path/to/brief.md`
- Updated: 2026-11-05

## Decision

Two to five sentences.

## Next steps

1. Developer: create the API key and put it in `.env`.
2. Agent: add the client once the key exists.

## Developer steps
<!-- advisor:developer-steps -->

- [ ] Create the API key and put it in `.env`. Do not commit the file.
```

Preserve the two HTML comments verbatim. They are anchors, not translations. On update, refresh the body but keep checked developer boxes that are still required. Add new developer steps unchecked. Remove a developer step only when the new brief makes it obsolete, and say that in the Decision section in one sentence.

## After writing

Tell the user both paths, the status, and the unchecked developer steps. Do not repeat the XML.
