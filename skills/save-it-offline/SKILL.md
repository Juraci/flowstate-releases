---
name: save-it-offline
description: Research the official documentation for a technology (optionally at a specific version) and save it into FlowState as linked nodes with offline content copies, following the Stack Documentation Hub conventions. Use when the user says "save it offline", invokes /save-it-offline, or asks to mirror/archive/save docs for a library, framework, or tool into FlowState. Requires the FlowState MCP server.
---

# save-it-offline

Mirror a technology's official documentation into FlowState: one **tech node** (versions, key pages, changelog, migration path) plus **offline content nodes** holding condensed copies of the key pages, all wired together with `[[id|title]]` links.

Input: `<technology> [version]` — e.g. `/save-it-offline drizzle-orm 0.45`, `/save-it-offline zod`.

## Precondition — FlowState MCP (hard gate)

Run `ToolSearch("select:mcp__flowstate__search_nodes,mcp__flowstate__create_node,mcp__flowstate__get_node,mcp__flowstate__update_node,mcp__flowstate__link_node")`.
If the `mcp__flowstate__*` tools are not available, **stop immediately** and tell the user the FlowState MCP server is not connected (enable it in FlowState settings / `claude mcp list`). Do not fall back to files.

## Workflow

1. **Resolve the version.** If the user gave one, use it. Otherwise find the latest **stable** release (npm registry, GitHub releases, or the official site — not betas/RCs). If the current project's `package.json` pins the technology, note that version too and prefer it as "version in project".

2. **Check for existing nodes.** `search_nodes` for the technology name and for "Stack Documentation Hub".
   - Hub found → new tech node's `parent_id` = hub id. No hub → create one first (see [CONVENTIONS.md](CONVENTIONS.md) § Hub).
   - Tech node already exists → **update it** (refresh versions, add missing pages) instead of creating a duplicate; re-fetch stale offline children only when the version changed.

3. **Research** (WebSearch + WebFetch):
   - Official docs site: home URL, **how the site versions its docs** (per-major subdomains? `/latest/` paths? single-version?) and the URL matching the resolved version.
   - 2–4 highest-value key pages (guide, API/config reference, migration guide to the next major).
   - Changelog / releases location.
   - **Public open-source repos (GitHub)**: README, `docs/` folder, release notes. Use these as the primary source when the docs site is JS-rendered, paywalled, or 404s — fetch raw files or the GitHub API, and pin to the version's **tag** (not `main`) when the repo has moved ahead.
   - Future-proofing: next major's status, pre-announced breaking changes, deprecations.

4. **Create the tech node**, then **one offline content node per key page**, then rewrite the tech node's Key pages bullets into `[[id|…]] · source: URL` links. Exact templates, tags, and link formats: [CONVENTIONS.md](CONVENTIONS.md).
   - Content nodes are **condensed references written in your own words** (structure, concepts, API signatures, config options, short code examples). Never paste pages verbatim wholesale. Always start with the provenance line (`> Condensed from <URL> — fetched <date>.`) and record any source substitution (e.g. "fetched from GitHub tag vX because the site is JS-rendered").
   - More than ~4 pages, or several technologies at once → fan out one background Agent per technology/page group; each agent owns its tech node end-to-end (create children, rewrite bullets) to avoid write conflicts.

5. **Cross-link** (`link_node`) the new tech node with related tech nodes that already exist in the vault — real dependency/usage relationships only (bundler ↔ framework, ORM ↔ driver, test runner ↔ utils), both directions when the relationship matters from both sides.

6. **Report**: node ids/titles created, version resolved (and how), pages mirrored, pages deliberately left as plain URLs, and any provenance caveats.

## Quality bar

- Version accuracy beats completeness: the docs saved must match the resolved version; when the live site documents a newer major, hunt for the versioned archive or the git tag.
- Every offline node must be reachable from its tech node's body; the tech node must be reachable from the hub (as its child).
- Broken/404 source URL → find the real location (repo search, sitemap) and cite the working URL; never save a dead link as "source".
