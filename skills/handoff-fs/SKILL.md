---
name: handoff-fs
description: Compact the current conversation into a handoff document for another agent to pick up, saved as a FlowState node. Requires the FlowState MCP server.
argument-hint: 'What will the next session be used for?'
disable-model-invocation: true
---

## Precondition — FlowState MCP (hard gate)

Run `ToolSearch("select:mcp__flowstate__create_node")`.
If the `mcp__flowstate__*` tools are not available, **stop immediately** and tell the user the FlowState MCP server is not connected (enable it in FlowState settings / `claude mcp list`). Do not fall back to writing a file.

## Writing the handoff

Write a handoff document summarising the current conversation so a fresh agent can continue the work. Save it as a FlowState node via `create_node` — not to the workspace or the OS temporary directory:

- `kind: task`, `task_status: todo`, `tags: handoff`
- `title`: `Handoff: <short topic> — <YYYY-MM-DD>`
- `body`: the handoff document, in markdown

Include a "suggested skills" section in the document, which suggests skills that the agent should invoke.

Do not duplicate content already captured in other artifacts (PRDs, plans, ADRs, issues, commits, diffs). Reference them by path or URL instead.

Redact any sensitive information, such as API keys, passwords, or personally identifiable information.

If the user passed arguments, treat them as a description of what the next session will focus on and tailor the doc accordingly.

Finish by telling the user the created node's id and title, and that a fresh session can pick it up with `/resume-handoff`.
