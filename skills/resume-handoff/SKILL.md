---
name: resume-handoff
description: Resume work from a handoff node previously saved in FlowState by /handoff-fs. Use when the user says "resume the handoff", "pick up the handoff", "continue where we left off", or points at a FlowState handoff node. Requires the FlowState MCP server.
argument-hint: '[node id or search terms to locate the handoff]'
---

## Precondition — FlowState MCP (hard gate)

Run `ToolSearch("select:mcp__flowstate__search_nodes,mcp__flowstate__get_node,mcp__flowstate__update_node")`.
If the `mcp__flowstate__*` tools are not available, **stop immediately** and tell the user the FlowState MCP server is not connected (enable it in FlowState settings / `claude mcp list`).

## Locating the handoff node

1. If the argument looks like a node id, fetch it directly with `get_node`.
2. Otherwise `search_nodes` for the argument terms; with no arguments, search for `Handoff`.
3. Consider only handoff nodes (title starts with `Handoff:` / tagged `handoff`) that are still open (`task_status` of `todo` or `doing`). One clear match → use it. Several plausible matches → list their titles and dates and ask the user which to resume. None → say so and stop.

## Resuming

1. Read the node body in full. It is a compacted summary of a previous session: current state, remaining work, and a "suggested skills" section.
2. Read any artifacts the handoff references by path or URL (PRDs, plans, ADRs, issues, commits, diffs) before acting on claims about them — the handoff deliberately does not duplicate their content.
3. Invoke the skills listed in the "suggested skills" section where they apply to the work ahead.
4. Mark the handoff as picked up: `update_node` with `task_status: doing`.
5. Briefly restate to the user where the previous session left off and what you'll do next, then continue the work.

When the handed-off work is complete, close the loop: `update_node` with `task_status: done`.
