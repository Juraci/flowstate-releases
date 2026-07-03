# FlowState offline-docs conventions

Follow these so every technology saved with `save-it-offline` has a consistent shape.

## Node kind, tags, structure

| Node | kind | tags | parent |
|---|---|---|---|
| Technology node | **default** (do not pass `kind`) | `docs,<technology-slug>` | root, or the optional index |
| Offline content node | **default** (do not pass `kind`) | `docs,content` | its technology node |
| Optional index (only if the user groups docs) | **default** (do not pass `kind`) | `docs` | root |

- Never set `kind: solution` (or any explicit kind) — let FlowState apply its default node kind.
- `<technology-slug>` is a short lowercase name, e.g. `zod`, `drizzle-orm`, `rust`.
- Set `working: partial` only when the package has a known limitation worth flagging (e.g. a dormant/unmaintained wrapper); otherwise `working: yes`.

## Optional index

Only if the user keeps a single place for all their offline docs. Search for an existing index node the user points you at (or a node titled `Documentation`); if none exists and the user wants grouping, create one with tags `docs` and parent the technology nodes under it. Do **not** invent a project- or stack-specific index. If the user hasn't asked to group anything, create technology nodes at the root.

## Technology node body template

Title: `<Technology> <version>` (e.g. `zod 3.25`), or just `<Technology>` if version-agnostic.

```markdown
**Version:** `<resolved version>` — <the version the user requested, or the latest stable release and how it was resolved>.

**Official docs:** <home URL> — <how the site versions docs: per-major subdomains, /latest/ paths, single-version, README-only>.

**Key pages:**
- <Label>: [[<content-node-id>|<Technology> <Label> (offline)]] · source: <URL>
- <Label>: <plain URL>            ← pages deliberately not mirrored stay as bare URLs

**Changelog / releases:** <URLs>

**Migration / next version:** <next major status, pre-announced breaking changes,
migration-guide URL (link its offline node here too if mirrored), recommended upgrade path>.
```

Keep the body about the technology itself — its versions, docs, and upgrade path. Do **not** add project-specific notes, dependency pins, or "how this repo uses it" commentary. Cross-links land as trailing `[[id|title]]` tokens appended by `link_node`; leave them at the bottom.

## Offline content node body template

Title: `<Technology> — <Page label> (offline)`.

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
- `link_node(source, target)` appends a `[[target|title]]` token to the **source** body — idempotent, no self-links. Use it for technology↔technology relationship links; use inline body tokens for the Key pages bullets.
- When rewriting a technology node's Key pages, `get_node` first and change **only** the intended bullets — keep the rest of the body byte-identical.

## Source-fallback playbook

- Docs site JS-rendered / empty fetch → the project's GitHub README or `docs/` markdown (raw.githubusercontent.com), at the matching **tag**.
- URL 404s → check the repo's docs source tree and sitemap for the moved file; cite the working URL.
- Site documents a newer major than the target version → look for versioned archives (`vN.` subdomain, `/en/<version>/`, wayback as last resort) or the git tag's docs.
- Repo `main` is a newer major → always read the version tag.
