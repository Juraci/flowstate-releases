# FlowState stack-docs conventions

These match the existing "FlowState Stack Documentation Hub" structure (created 2026-07-03). Follow them exactly so new technologies slot into the same graph.

## Node kinds, tags, fields

| Node | kind | tags | parent |
|---|---|---|---|
| Hub | `solution` | `docs,stack,reference` | root (none) |
| Tech node | `solution` | `docs,stack,<area>` | hub |
| Offline content node | `solution` | `docs,content` | its tech node |

- `<area>` is one of: `core`, `frontend`, `data`, `testing`, `misc` (plus extras like `editor`, `markdown`, `security`, `native`, `packaging`, `mcp` when apt).
- `working: yes` normally; `working: partial` when the package has a known limitation worth flagging (e.g. a dormant wrapper).
- `priority: high/med` only when there's real upgrade pressure (EOL, imminent major).

## Hub (create only if missing)

Title: `FlowState Stack Documentation Hub`. Body: what the hub is, snapshot date, the layer list (Core/build · Frontend · Data · Testing · Misc) naming each child tech, and a "Keeping this current" paragraph (re-check migration sections on major bumps; offline nodes are dated snapshots to re-fetch).

## Tech node body template

```markdown
**Version in project:** `<pkg> ^X.Y.Z` — <relation to npm latest / EOL warnings>.

**Official docs:** <home URL> — <how the site versions docs: per-major subdomains, /latest/ paths, single-version, README-only>.

**Key pages:**
- <Label>: [[<content-node-id>|<Tech> <Label> (offline)]] · source: <URL>
- <Label>: <plain URL>            ← pages deliberately not mirrored stay as bare URLs

**Changelog / releases:** <URLs>

**Migration / future-proofing:** <next major status, breaking changes, migration-guide URL
(link its offline node here too if mirrored), recommended upgrade path>.

**FlowState notes:** <how THIS vault's owner uses the package — repo gotchas, CLAUDE.md
rules, pins, interactions with other stack pieces>. Omit the section if there is nothing real to say.
```

Cross-links land as trailing `[[id|title]]` tokens appended by `link_node` — that's fine, leave them at the bottom.

## Offline content node body template

Title: `<Tech> — <Page label> (offline)`.

```markdown
> Condensed from <URL> — fetched <YYYY-MM-DD>. <Provenance caveat if content came from a
different location, e.g. "site is JS-rendered; sourced from the vX.Y.Z tag README on GitHub.">

<Condensed reference: every major section of the page, in your own words. Keep API
signatures, config option names/defaults/types, and the important short code examples
verbatim — those are functional. Target 500–1200 words (up to ~1400 for an architecture
"system guide" page).>
```

Do **not** reproduce pages verbatim wholesale; the value is a self-contained distilled reference plus the source link.

## Link mechanics

- `[[<node-id>|<label>]]` tokens in bodies are materialized by FlowState on save (`create_node`/`update_node` both do this).
- `link_node(source, target)` appends a `[[target|title]]` token to the **source** body — idempotent, no self-links. Use it for tech↔tech relationship links; use inline body tokens for the Key pages bullets.
- When rewriting a tech node's Key pages, `get_node` first and change **only** the intended bullets — keep the rest of the body byte-identical.

## Source-fallback playbook

- Docs site JS-rendered / empty fetch → the project's GitHub README or `docs/` markdown (raw.githubusercontent.com), at the matching **tag**.
- URL 404s → check the repo's docs source tree and sitemap for the moved file; cite the working URL.
- Site documents a newer major than the target version → look for versioned archives (`vN.` subdomain, `/en/<version>/`, wayback as last resort) or the git tag's docs.
- Repo `main` is a newer major → always read the version tag.
