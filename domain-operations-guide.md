# Domain Operations Guide: Running a Collaborative Knowledge Domain

*Joi Ito, March 2026*

---

This is the practical companion to [Collaborative Knowledge Domains](collaborative-knowledge-domains.md). That document explains *why* this architecture works. This one explains *how to operate it* -- the folder structure, the workflows, the tooling, and the gaps you'll hit.

---

## The Complete Folder Structure

The earlier paper described the minimal structure (intake/atlas). In practice, a production domain needs more. Here is the full template, evolved from running two domain types: collaborative research domains (shared with other humans and agents) and confidential workstreams (solo with publication outputs).

```
domains/{topic}/
  _conventions.md           # Domain rules, schemas, tag vocabulary
  CONTEXT.md                # Session orientation -- read-first for any AI session
  INDEX.md                  # Auto-generated navigation hub (maintainer-managed)
  CHANGELOG.md              # Auto-generated edit log (maintainer-managed)
  .writing-style.md         # Voice/tone guide for publications (optional)

  atlas/                    # Promoted durable knowledge
    concepts/
    people/
    organizations/
    references/

  intake/                   # Per-contributor write queues
    {contributor-a}/
    {contributor-b}/
    curator/                # Automated research agent deposits

  requests/                 # Maintainer -> contributor asks
  _review/                  # Items needing human judgment

  publications/             # Versioned export documents
    drafts/                 # Working drafts of reports and summaries
    output/                 # Generated PDFs, slide decks, exports
    archive/                # Superseded versions (never delete, only archive)
    scripts/                # PDF/chart generation scripts
    assets/                 # Generated figures, charts, images

  .maintainer/              # Maintainer agent state (gitignored from Syncthing)
    processed.txt           # Dedup manifest
    last-run.txt            # Last pipeline run timestamp
    conflict-log.jsonl      # Conflict detection history
```

### What's New vs. the Minimal Structure

| Addition | Purpose | Learned From |
|----------|---------|-------------|
| `CONTEXT.md` | Session orientation for interactive tending | sankosh pattern |
| `CHANGELOG.md` | Machine-readable edit log | Gap: no audit trail in minimal version |
| `.writing-style.md` | Consistent voice across human and AI authors | sankosh `.writing-style.md` |
| `publications/` | Versioned reports, PDFs, exports | sankosh `output/` + `archive/` + `scripts/` |
| `.maintainer/` | Pipeline state files | maintainer-agent-spec |

---

## Three Modes of Operation

A domain is operated in three distinct modes. Most existing systems support only one. This architecture supports all three, and the key insight is that they share the same underlying files.

### Mode 1: Automated Curation (Maintainer Agent)

**What:** A background agent polls intake folders and runs the five-phase pipeline (collect, validate, detect conflicts, promote, index). It runs on a schedule -- every 15 minutes for active domains, daily for dormant ones.

**Where it runs:** A dedicated machine (in my case, a Mac Studio running macOS). The maintainer agent runs as a launchd job, calling an Amplifier recipe that executes the pipeline.

**What it produces:**
- Promoted files in `atlas/`
- Updated `INDEX.md` and `CHANGELOG.md`
- Bounced files in `requests/{contributor}/`
- Conflict pairs in `_review/`

**What it doesn't do:** Resolve semantic conflicts. Rewrite contributed content. Make editorial judgments about what's important. Those are human tasks, handled in Mode 2.

### Mode 2: Interactive Tending (Human + AI Session)

**What:** A human opens an Amplifier session (or Claude session) with the domain's `CONTEXT.md` loaded, and works interactively: asking questions about what the domain contains, editing atlas files, resolving conflicts in `_review/`, creating publication drafts, and promoting intake files that need human judgment.

**How it works:** The `CONTEXT.md` file at the domain root serves as the session's orientation. It tells the AI:
- What this domain is about and its scope boundaries
- Where everything lives (folder map with descriptions)
- What conventions to follow (link to `_conventions.md`)
- What's currently pending (link to `_review/`, `requests/`)
- What writing style to use (link to `.writing-style.md`)
- How to query the domain (which QMD collection, which search patterns)

A typical interactive tending session:

```
1. Load CONTEXT.md (automatic if using Amplifier with domain context bundle)
2. "What's new in intake?"     → Agent scans intake/ folders, summarizes
3. "Promote the LNP paper"     → Agent moves file, adds metadata, updates index
4. "Resolve the conflict in _review/" → Agent shows both versions, human decides
5. "Draft a summary report on {topic}" → Agent queries atlas, writes to publications/drafts/
6. "Export as PDF"              → Agent runs publication script
```

This is the mode where the human exercises editorial judgment that the maintainer agent cannot.

### Mode 3: Publication (Reports and Exports)

**What:** Transform domain knowledge into shareable documents -- PDF reports, slide decks, briefing papers. These live in `publications/` with proper versioning.

**The publication workflow:**

```
1. DRAFT    → Write/generate in publications/drafts/{slug}.md
2. VERSION  → Bump version in frontmatter, update CHANGELOG.md
3. REVIEW   → Human reviews, edits, approves
4. GENERATE → Run scripts/ to produce PDF/slides in output/
5. ARCHIVE  → Copy previous version to archive/ before overwriting
```

This is the pattern extracted from the sankosh workstream, generalized for any domain.

---

## The Versioning System

Every publication document carries its version in YAML frontmatter:

```yaml
---
title: "Medtech Research Summary"
version: "Draft 0.2.1 — 2026-03-16 14:30 JST"
last_updated: 2026-03-16 14:30 JST
status: draft | review | final
domain: medtech
confidential: false
---
```

### Version Numbering

Format: `Draft {major}.{minor}.{patch} — {YYYY-MM-DD} {HH:MM} {TZ}`

| Bump | When |
|------|------|
| **Major** (0.x → 1.0) | Document reaches "final" status for the first time |
| **Minor** (0.1 → 0.2) | Structural changes -- new sections, reorganization, significant rewrites |
| **Patch** (0.2.0 → 0.2.1) | Editorial -- corrections, updated figures, minor additions |

### Archive Naming

When bumping minor or major, archive the previous version:

```
archive/{slug}-draft-{version}-{YYYY-MM-DD}-{HHMM}.md
```

Example: `archive/medtech-summary-draft-0.1-2026-03-10-0930.md`

### Output Naming

Generated exports encode version + timestamp:

```
output/{slug}-{doc-type}-draft-{version}-{YYYY-MM-DD}-{HHMM}.pdf
```

Example: `output/medtech-summary-report-draft-0.2.1-2026-03-16-1430.pdf`

### CHANGELOG.md

The maintainer agent (or human during interactive sessions) appends to `CHANGELOG.md` after every significant change:

```markdown
## 2026-03-16

### Atlas
- Promoted: `concepts/federated-learning-privacy.md` (from intake/bob/)
- Promoted: `references/nature-lnp-review-2026.md` (from intake/curator/)
- Conflict detected: `concepts/aav9-tropism.md` vs intake/alice/ — routed to _review/

### Publications
- Updated: `medtech-summary.md` 0.2.0 → 0.2.1 (added section on delivery vectors)
- Generated: `output/medtech-summary-report-draft-0.2.1-2026-03-16-1430.pdf`
- Archived: `archive/medtech-summary-draft-0.2-2026-03-15-1100.md`
```

This log is append-only. It serves as the domain's audit trail -- who changed what, when, and why.

---

## The CONTEXT.md File (Session Orientation)

This is the most important operational file. It's what makes interactive tending sessions effective. Without it, every AI session starts from zero and has to rediscover what the domain is, where things are, and what conventions to follow.

### Template

```markdown
# {Domain Name} — Domain Context

> Read this first. This file orients any AI session working in this domain.

## What This Domain Is

{2-3 sentences: scope, why it exists, who contributes}

## Scope Boundaries

**In scope:** {list}
**Out of scope:** {list}

## Folder Map

| Path | What's There | How to Use It |
|------|-------------|---------------|
| `atlas/concepts/` | Promoted concept cards | Read for research; edit to update knowledge |
| `atlas/people/` | Key people in this field | Read for context; add via intake/ |
| `atlas/organizations/` | Companies, labs, institutions | Read for context; add via intake/ |
| `atlas/references/` | Source papers and articles | Read for citations; add via intake/ |
| `intake/{you}/` | Your deposit queue | Write new knowledge here |
| `intake/curator/` | Automated agent deposits | Review during tending sessions |
| `requests/` | Open research asks | Check for tasks; resolve and close |
| `_review/` | Conflicts needing human judgment | Resolve during tending sessions |
| `publications/drafts/` | Working report documents | Edit, version, export |
| `publications/output/` | Generated PDFs and exports | Share externally |

## Conventions

See `_conventions.md` for:
- Required frontmatter fields and schemas
- Tag vocabulary
- Filename conventions
- Trust levels by contributor

## Writing Style

{If .writing-style.md exists:}
See `.writing-style.md` for voice and tone guidance when writing or editing
atlas files and publications.

## Current State

{Updated by maintainer or during tending sessions:}
- Atlas: {N} files across {concepts, people, orgs, references}
- Intake pending: {N} files awaiting promotion
- Review queue: {N} conflicts needing resolution
- Open requests: {N} research asks outstanding
- Latest publication: {title} v{version} ({date})

## How to Query This Domain

{Depends on your search tooling:}
- **QMD:** `mcp_qmd_search(query="...", collection="{domain}")`
- **Vault search:** `vault_search(query="...", path="jibrain/domains/{domain}/")`
- **Direct browsing:** Open `INDEX.md` for the auto-generated navigation hub

## Session Checklist

When starting an interactive tending session:

1. [ ] Check `_review/` for conflicts needing resolution
2. [ ] Check `requests/` for open research asks
3. [ ] Scan `intake/` for files needing human judgment (low-trust sources)
4. [ ] Review `CHANGELOG.md` for recent automated changes
5. [ ] Update `Current State` section above if significantly changed
```

### Why CONTEXT.md Works

The sankosh workstream proved this pattern. Every AI session that loaded `CONTEXT.md` first could immediately:
- Navigate the folder structure without asking
- Follow conventions without being told
- Find the right files for the task
- Produce output in the right voice

Without it, the first 5-10 minutes of every session is wasted on orientation. With it, the session starts productive.

---

## PDF Generation

The sankosh pattern uses a Python [ReportLab](https://docs.reportlab.com/) script that parses Markdown and produces a styled A4 PDF with headers, footers, tables, and charts. This works but has a key limitation: the version string and file paths are hardcoded in the script.

### The Improved Pattern

Make the script config-driven:

```python
# scripts/generate_pdf.py
import sys
import yaml
from pathlib import Path

def load_source(source_path):
    """Read markdown file, extract frontmatter."""
    text = Path(source_path).read_text()
    # Parse YAML frontmatter between --- markers
    _, fm, body = text.split('---', 2)
    meta = yaml.safe_load(fm)
    return meta, body.strip()

def main():
    source = sys.argv[1]  # e.g., publications/drafts/medtech-summary.md
    meta, body = load_source(source)

    version = meta['version']
    slug = Path(source).stem
    timestamp = meta['last_updated'].replace(' ', '-').replace(':', '')

    output_path = f"publications/output/{slug}-{timestamp}.pdf"
    archive_path = f"publications/archive/{slug}-{version}.md"
    # ... generate PDF using version from frontmatter, not hardcoded
```

The version, title, and metadata all come from the source document's frontmatter. No more editing the script for each new version.

### Dependencies

```bash
pip install reportlab matplotlib pyyaml
```

### Chart Integration

Separate chart scripts in `publications/scripts/` output PNGs to `publications/assets/`. The PDF generator looks for `<!-- CHART: assets/filename.png -->` markers in the Markdown and inserts the image at that position.

---

## Syncthing Configuration for Domains

Each domain is a separate Syncthing shared folder. This is what makes the privacy boundary work -- collaborators see only the domains they participate in.

### Setup Checklist

1. **Create Syncthing folder** on the maintainer machine:
   - Folder ID: `jibrain-{domain}` (e.g., `jibrain-medtech`)
   - Path: `~/switchboard/jibrain/domains/{domain}/`
   - Type: Send & Receive

2. **Add collaborator's device** in Syncthing:
   - Exchange Device IDs (Actions > Show ID in Syncthing web UI)
   - Share only the domain folder, not the parent vault

3. **Configure ignore patterns** (`.stignore` in the domain root):
   ```
   .maintainer/
   .DS_Store
   *.sync-conflict-*
   ```

4. **Verify** the collaborator sees the domain folder syncing, not any parent directories.

### What Syncs vs. What Doesn't

| Syncs | Doesn't Sync |
|-------|-------------|
| `atlas/` (promoted knowledge) | `.maintainer/` (pipeline state) |
| `intake/` (all contributor queues) | Conflict files (`.sync-conflict-*`) |
| `requests/` and `_review/` | Parent vault directories |
| `publications/` (reports and PDFs) | Other domains |
| `INDEX.md`, `CHANGELOG.md` | Personal notes, daily notes, contacts |

---

## The Maintainer Agent

The maintainer agent runs as a scheduled job on a dedicated machine. It's an Amplifier recipe that executes the five-phase pipeline.

### Where It Runs

The maintainer should run on a machine that:
- Is always on (or wakes on schedule)
- Has Syncthing running (so it sees all contributor intake)
- Has QMD or another semantic search index (for conflict detection)
- Has LLM API access (for validation and semantic comparison)

In my setup, this is a Mac Studio (primary-mac) running macOS with launchd scheduling.

### Scheduling

```xml
<!-- ~/Library/LaunchAgents/com.jibrain.maintainer-{domain}.plist -->
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.jibrain.maintainer-{domain}</string>
    <key>ProgramArguments</key>
    <array>
        <string>/path/to/amplifier</string>
        <string>tool</string>
        <string>invoke</string>
        <string>recipes</string>
        <string>operation=execute</string>
        <string>recipe_path=@joi:recipes/domain-maintain.yaml</string>
        <string>context={"domain": "{domain}"}</string>
    </array>
    <key>StartInterval</key>
    <integer>900</integer>  <!-- Every 15 minutes -->
    <key>StandardOutPath</key>
    <string>/tmp/maintainer-{domain}.log</string>
    <key>StandardErrorPath</key>
    <string>/tmp/maintainer-{domain}.err</string>
</dict>
</plist>
```

### The Pipeline in Detail

**Phase 1: Collect**
```
Scan intake/*/ for files not in .maintainer/processed.txt
  → New files become the batch for this run
  → If batch is empty, exit early
```

**Phase 2: Validate**
```
For each new file:
  Required: type, description, contributor, tags, status
  description must be ≥30 chars and differ from title
  type must be promotable (concept | person | organization | reference)
  FAIL → move to requests/{contributor}/ with explanation
```

**Phase 3: Detect Conflicts**
```
For each validated file:
  Semantic search against atlas/ (using QMD vector search)
  If similarity > threshold:
    Same contributor → merge (newer replaces older, log in CHANGELOG)
    Different contributors + compatible → merge with attribution
    Different contributors + contradictory → route to _review/
```

**Phase 4: Promote**
```
For each clean file:
  Move to atlas/{type}/
  Add promotion metadata: promoted_by, promoted_date
  Update status: draft → final
  Append filename to .maintainer/processed.txt
```

**Phase 5: Index**
```
Regenerate INDEX.md:
  - By topic cluster (based on tags)
  - By contributor
  - By recency (last 20 promotions)
  - Open requests count
  - Review queue count
Append batch summary to CHANGELOG.md
Update .maintainer/last-run.txt
```

---

## Interactive Tending Workflows

These are the common tasks you'll do in interactive sessions. Each assumes CONTEXT.md is loaded.

### "What do we know about X?"

```
1. Search atlas: QMD semantic search scoped to the domain
2. If found: summarize relevant cards, show connections via wikilinks
3. If not found: check intake/ for pending files on the topic
4. If nowhere: suggest creating a research request in requests/
```

### "Promote this intake file"

```
1. Read the file from intake/{contributor}/
2. Validate frontmatter against _conventions.md schema
3. Check for semantic overlap with existing atlas files
4. If clean: move to atlas/{type}/, update metadata, append to CHANGELOG
5. If conflict: present both files, ask human to decide
```

### "Resolve the conflicts in _review/"

```
1. List files in _review/ with their conflict pairs
2. For each pair: show side-by-side comparison
3. Human decides: keep A, keep B, merge, or discard both
4. Execute decision, update CHANGELOG
5. Clear resolved files from _review/
```

### "Write a summary report on {topic}"

```
1. Query atlas for all files matching the topic (tags + semantic search)
2. Draft report in publications/drafts/{slug}.md with proper frontmatter:
   version: "Draft 0.1 — {date}"
   last_updated: {date}
   status: draft
3. Follow .writing-style.md if it exists
4. Human reviews and edits
5. When approved: generate PDF via scripts/, archive previous version
```

### "Update the existing report"

```
1. Read current version from publications/drafts/{slug}.md
2. Check what's new in atlas since last_updated date
3. Integrate new knowledge into the document
4. Bump version (patch for small edits, minor for structural changes)
5. Archive previous version to publications/archive/
6. Generate new PDF to publications/output/
7. Append to CHANGELOG
```

---

## Gaps and Honest Limitations

### Solved by This Guide

| Gap | Solution |
|-----|----------|
| No audit trail | `CHANGELOG.md` -- append-only edit log |
| No publication workflow | `publications/` tree with versioning, archive, scripts |
| No session orientation | `CONTEXT.md` -- loaded at session start |
| No way to query the domain interactively | QMD scoped collection + interactive tending workflows |
| Hardcoded paths in PDF scripts | Config-driven scripts reading from frontmatter |

### Still Open

| Gap | Status | Workaround |
|-----|--------|-----------|
| **Semantic conflict detection** | Requires embeddings (QMD vector search). Works locally but no shared index for remote collaborators. | Maintainer runs it centrally; collaborators rely on the maintainer's judgment. |
| **Real-time collaboration** | Syncthing is near-real-time (~seconds) but not true real-time. No live cursors, no presence indicators. | Acceptable for async research. For real-time co-editing, use a shared Obsidian session via [Relay](https://relay.md/). |
| **Discovery at scale** | INDEX.md works for hundreds of files. At thousands, need full-text + semantic search. | QMD handles this locally. Remote collaborators don't get search -- they get the synced INDEX.md. |
| **Cross-domain linking** | An atlas concept in `domains/medtech/` can't wikilink to `domains/ai-coding/`. Obsidian resolves wikilinks vault-wide, but Syncthing-shared collaborators only see their domain. | Use `source_url:` to reference cross-domain content by URL rather than wikilink. Or accept that cross-domain connections live in the parent vault, not in shared domains. |
| **Contributor identity verification** | The `contributor:` field is self-reported. Nothing prevents an agent from claiming to be a human contributor. | Social trust. At 2-5 contributors this is fine. At scale, you'd need signed commits or similar. |
| **Offline-first search for collaborators** | Collaborators can browse files in Obsidian but can't run semantic queries without their own QMD instance. | Each collaborator runs their own QMD over the synced domain folder. This works but requires setup on each machine. |

---

## Setting Up a New Domain

### Step 1: Create the Folder Structure

```bash
DOMAIN="medtech"
BASE=~/switchboard/jibrain/domains/$DOMAIN

mkdir -p $BASE/{atlas/{concepts,people,organizations,references},intake,requests,_review}
mkdir -p $BASE/{publications/{drafts,output,archive,scripts,assets},.maintainer}
touch $BASE/.maintainer/{processed.txt,last-run.txt}
```

### Step 2: Write _conventions.md

Define: scope, out-of-scope, file schema (required frontmatter), tag vocabulary, filename conventions, contributor list with trust levels. See the [medtech _conventions.md template](https://github.com/Joi/ai-agent-learnings) for a working example.

### Step 3: Write CONTEXT.md

Use the template from this guide. Fill in the domain-specific sections. This file is what makes interactive sessions productive -- invest time in it.

### Step 4: Create Contributor Intake Folders

```bash
mkdir -p $BASE/intake/{alice,bob,curator}
```

One folder per contributor (human or agent). Naming convention: lowercase, no spaces.

### Step 5: Configure Syncthing

Create a shared folder for the domain. Share only with the collaborators who should have access. Set up `.stignore` to exclude `.maintainer/`.

### Step 6: Schedule the Maintainer Agent

Create a launchd plist (macOS) or cron job (Linux) that runs the maintainer recipe on the target machine. Start with a 15-minute interval.

### Step 7: Do a Test Run

Have each contributor drop a test file into their intake folder. Verify:
- Syncthing propagates it to the maintainer machine
- The maintainer pipeline picks it up and promotes it
- The promoted file appears in atlas/ on all synced machines
- INDEX.md and CHANGELOG.md are updated

---

## Tools Referenced

| Tool | Purpose | Link |
|------|---------|------|
| **Obsidian** | Knowledge authoring & graph browsing | [obsidian.md](https://obsidian.md/) |
| **Syncthing** | Peer-to-peer encrypted file sync | [syncthing.net](https://syncthing.net/) |
| **Amplifier** | AI agent framework for maintainer and session agents | [github.com/microsoft/amplifier](https://github.com/microsoft/amplifier) |
| **QMD** | Local semantic search over Markdown files | [qmd.io](https://qmd.io/) |
| **ReportLab** | Python PDF generation library | [reportlab.com](https://docs.reportlab.com/) |
| **Relay** | CRDT-based real-time Obsidian collaboration (optional) | [relay.md](https://relay.md/) |
| **Sprites** | Persistent cloud sandboxes for extraction services | [sprites.dev](https://sprites.dev/) |
| **NanoClaw** | Lightweight agent gateway for chat-based agents | [github.com/nicholasgasior/nanoclaw](https://github.com/nicholasgasior/nanoclaw) |
| **Granola** | AI meeting notepad with speaker identification | [granola.so](https://www.granola.so/) |
| **muesli** | Meeting transcript sync and semantic search | [github.com/harperreed/muesli](https://github.com/harperreed/muesli) |

---

## Related Documents

- [Collaborative Knowledge Domains](collaborative-knowledge-domains.md) -- The architecture paper (why this works)
- [Knowledge Systems Audit](knowledge-systems-audit.md) -- Security and access tiers

*This document is part of [ai-agent-learnings](https://github.com/Joi/ai-agent-learnings), a collection of patterns from running a production AI-operated knowledge management system.*
