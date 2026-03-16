# {Domain Name} — Domain Context

> Read this first. This file orients any AI session working in this domain.
> Copy this template to `CONTEXT.md` in your domain root and fill in the sections.

## What This Domain Is

{2-3 sentences: scope, why it exists, who contributes.}

Example: *This domain tracks research on medical technology and health innovation. It's maintained by Joi with contributions from Gal and automated research agents. The goal is a shared, searchable knowledge base that produces periodic summary reports for stakeholders.*

## Scope Boundaries

**In scope:**
- {topic area 1}
- {topic area 2}
- {topic area 3}

**Out of scope:**
- {explicitly excluded topic 1}
- {explicitly excluded topic 2}

## Contributors

| Name | Type | Trust | Intake Folder |
|------|------|-------|---------------|
| {you} | Human | High (auto-promote) | `intake/{you}/` |
| {collaborator} | Human | High (auto-promote) | `intake/{collaborator}/` |
| curator | Agent (research) | Medium (auto-promote if no conflict) | `intake/curator/` |
| nanoclaw | Agent (chat capture) | Low (always review) | `intake/nanoclaw/` |

## Folder Map

| Path | What's There | How to Use It |
|------|-------------|---------------|
| `atlas/concepts/` | Promoted concept cards | Read for research; edit to update knowledge |
| `atlas/people/` | Key people in this field | Read for context; add via intake/ |
| `atlas/organizations/` | Companies, labs, institutions | Read for context; add via intake/ |
| `atlas/references/` | Source papers and articles | Read for citations; add via intake/ |
| `intake/{you}/` | Your deposit queue | Write new knowledge here |
| `intake/curator/` | Automated agent deposits | Review during tending sessions |
| `requests/` | Open research asks from maintainer | Check for tasks; resolve and close |
| `_review/` | Conflicts needing human judgment | Resolve during tending sessions |
| `publications/drafts/` | Working report documents | Edit, version, export |
| `publications/output/` | Generated PDFs and exports | Share externally |
| `publications/archive/` | Superseded report versions | Reference only |
| `INDEX.md` | Auto-generated navigation hub | Browse to discover what exists |
| `CHANGELOG.md` | Edit log (append-only) | Review recent changes |

## Conventions

See `_conventions.md` for the full specification:
- Required frontmatter fields and schemas
- Tag vocabulary for this domain
- Filename conventions
- Trust levels by contributor

## Writing Style

{If .writing-style.md exists:}
See `.writing-style.md` for voice and tone guidance when writing or editing
atlas files and publications.

{If no writing style defined:}
No domain-specific writing style defined. Use clear, concise prose.
State facts with sources. Avoid hedging unless confidence is genuinely low.

## Current State

<!-- Update this section during tending sessions or via maintainer agent -->
- **Atlas:** {N} files ({concepts: N, people: N, orgs: N, references: N})
- **Intake pending:** {N} files awaiting promotion
- **Review queue:** {N} conflicts needing resolution
- **Open requests:** {N} research asks outstanding
- **Latest publication:** {title} v{version} ({date})

## How to Query This Domain

**Semantic search (QMD):**
```
mcp_qmd_search(query="...", collection="{domain}")
mcp_qmd_vector_search(query="...", collection="{domain}")
```

**Vault search (Obsidian CLI):**
```
vault_search(query="...", path="jibrain/domains/{domain}/")
```

**Direct browsing:** Open `INDEX.md` for the auto-generated navigation hub.

**File listing:**
```bash
find atlas/ -name "*.md" | head -20    # Recent atlas files
ls intake/*/                            # Pending intake by contributor
ls _review/                             # Conflicts awaiting resolution
ls requests/*/                          # Open research asks
```

## Session Checklist

When starting an interactive tending session:

1. [ ] Check `_review/` for conflicts needing resolution
2. [ ] Check `requests/` for open research asks
3. [ ] Scan `intake/` for files needing human judgment (low-trust sources)
4. [ ] Review `CHANGELOG.md` for recent automated changes
5. [ ] Update **Current State** section above if significantly changed

## Publication Workflow

When creating or updating reports:

1. **Draft** in `publications/drafts/{slug}.md` — include version frontmatter:
   ```yaml
   version: "Draft 0.1 — {YYYY-MM-DD} {HH:MM} {TZ}"
   last_updated: {YYYY-MM-DD} {HH:MM} {TZ}
   status: draft
   ```
2. **Review** — human reads and edits the draft
3. **Archive** previous version to `publications/archive/{slug}-draft-{old-version}-{date}-{time}.md`
4. **Bump** version in frontmatter
5. **Generate** PDF/export to `publications/output/` via `publications/scripts/`
6. **Log** the change in `CHANGELOG.md`

## Links

- Architecture: [Collaborative Knowledge Domains](https://github.com/Joi/ai-agent-learnings/blob/main/collaborative-knowledge-domains.md)
- Operations: [Domain Operations Guide](https://github.com/Joi/ai-agent-learnings/blob/main/domain-operations-guide.md)
