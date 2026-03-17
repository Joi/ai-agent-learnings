# Knowledge Systems Architecture

> **⚠️ STALENESS WARNING (2026-03-17)**: This audit was a point-in-time snapshot from 2026-03-04. Several items listed as gaps or planned have since been resolved (e.g., `/intake/structured` is live, NanoClaw is deployed, bookmark-extractor sprite is deprecated). The Amplifier bundle system has matured significantly with 18+ tool modules, a talk-building pipeline, people harvest/enrichment, and design intelligence capabilities not reflected here. For current architecture, see `architecture.md`. A fresh audit is recommended.

> Comprehensive view of all knowledge extraction, storage, and retrieval systems.
> Created 2026-03-04 from full audit of jibot-docs, jibot-code, switchboard, amplifier, nanoclaw, and scattered design docs.

## Status: Active Reference Document

This document provides a single navigational map across all knowledge systems. It does NOT replace the canonical documents it references — it connects them.

**Canonical sources** (edit THESE for changes):
- Agent system: `jibot-docs/design/jibot-unified-architecture.md`
- NanoClaw migration: `jibot-docs/design/2026-02-23-nanoclaw-migration-plan.md`
- Knowledge pipeline: `switchboard/amplifier/JIBRAIN-ARCHITECTURE.md`
- Graph philosophy: `switchboard/amplifier/KNOWLEDGE-GRAPH-REDESIGN.md`
- Extraction config: `switchboard/amplifier/MULTI-DOMAIN-EXTRACTION.md`
- Vault conventions: `switchboard/jibrain/_STRUCTURE.md`

---

## 1. System Map

```
┌─────────────────────────────────────────────────────────────────────────┐
│                        KNOWLEDGE INPUT LAYER                           │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  WhatsApp/Signal ──► jibot (agent-mac) ──► bookmark-relay ──┐           │
│  Telegram ──────────► jimindbot (Ethoswarm Mind) ──────────┤           │
│  CLI ────────────────► bookmark-intake.sh ─────────────────┤           │
│                                                             │           │
│                                                             ▼           │
│                                              ┌──────────────────────┐  │
│                                              │ bookmark-extractor   │  │
│  PDF books ──► extract_book_chunked.py       │ sprite (FastAPI)     │  │
│  (Gemini vision)                             │ sprites.app          │  │
│                                              └──────────┬───────────┘  │
│  Japanese docs ──► knowledge-extract/                    │              │
│  (column-aware OCR)                                      │              │
│                                                          │              │
│  Daily notes ──► [!jibrain] callouts ──► meeting-extract │              │
│  Muesli transcripts ──► meeting-extract recipe           │              │
│                                                          │              │
└──────────────────────────────────────────────────────────┼──────────────┘
                                                           │
                                                           ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                    KNOWLEDGE STORAGE LAYER                              │
│                    ~/switchboard (Obsidian vault)                       │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  jibrain/                          ◄── "Obsidian IS the graph"         │
│    intake/          ◄── ALL new knowledge lands here (status: draft)   │
│      .meta/         ◄── JSON sidecar metadata                          │
│      .observations/ ◄── Agent friction signals                         │
│    atlas/           ◄── Promoted permanent knowledge (status: final)   │
│      concepts/      (8 files)                                          │
│      organizations/ (11 files)                                         │
│      people/                                                           │
│      references/                                                       │
│    domains/         ◄── Graduated deep-knowledge areas                 │
│      chanoyu/       (exemplar: 19+ concepts, seasonal guides, vocab)   │
│      ai-coding/     (3 files: QMD, agent patterns)                     │
│      poc-thebook/   (synced from git)                                  │
│    _review/         ◄── Staging for human review                       │
│                                                                         │
│  agents/                                                                │
│    curator/extractions/  ◄── Bookmark-extractor output landing zone    │
│    writer/drafts/        ◄── Talk/presentation drafts                  │
│    analyst/reports/      ◄── Analysis outputs                          │
│    research/findings/    ◄── Research agent outputs                    │
│    _shared/              ◄── Cross-agent handoffs, jimindbot prompt    │
│                                                                         │
│  concepts/          (30+ files, read directly by jibot-3)              │
│  organizations/     (read directly by jibot-3)                         │
│  people/            (880+ pages, PRIVATE, never synced)                │
│  chanoyu/           (curated export to jibot-docs)                     │
│  dailynote/         (PRIVATE, never synced)                            │
│  tools/knowledge-extract/  (Japanese OCR pipeline)                     │
│                                                                         │
└─────────────────────────────────────────────────┬───────────────────────┘
                                                   │
                          ┌────────────────────────┼────────────────┐
                          │ selective export        │ QMD indexing   │
                          ▼                         ▼                │
┌───────────────────────────────┐  ┌─────────────────────────────┐  │
│ ~/jibot-docs (shared repo)    │  │ QMD Search (MCP server)     │  │
│ ► chanoyu/ (curated)          │  │ 4,322 documents indexed:    │  │
│ ► concepts/ (planned)         │  │  - jibrain: 2,427 docs      │  │
│ ► organizations/ (planned)    │  │  - dailynote: 1,012 docs    │  │
│ ► people-public/ (planned)    │  │  - people: 883 docs         │  │
│ ► design/ (architecture docs) │  │                             │  │
│ ► inbox/outbox (async tasks)  │  │ Modes: keyword (BM25),      │  │
│                                │  │ vector (semantic), deep      │  │
│ git sync → agent-mac (5 min)   │  │ (hybrid + LLM rerank)       │  │
└───────────────────────────────┘  └─────────────────────────────┘  │
                                                                     │
┌─────────────────────────────────────────────────────────────────────┘
│
▼
┌─────────────────────────────────────────────────────────────────────────┐
│                      AGENT RUNTIME LAYER                                │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  Joi's Laptop (Amplifier)                                              │
│  ├── Amplifier sessions (Claude/OpenAI)                                │
│  │   ├── QMD search (vault_search, mcp_qmd_*)                         │
│  │   ├── vault operations (graph_health, enrichment)                   │
│  │   ├── jibrain recipes (triage, meeting-extract, reweave)            │
│  │   └── knowledge_synthesis module (Claude/Gemini extraction)         │
│  │                                                                     │
│  └── jibot-3 (TypeScript, NOT RUNNING)                                │
│      └── Reads ~/switchboard/concepts/ and organizations/ directly     │
│                                                                         │
│  agent-mac (Mac Mini M4 Pro, 64GB)                                     │
│  ├── OpenClaw gateway :18789 (CURRENT)                                 │
│  │   ├── owner agent (GPT-5.2, full access, 51 skills)                │
│  │   └── assistant agent (GPT-5.2, Docker sandbox, messaging only)    │
│  │                                                                     │
│  ├── NanoClaw (DEPLOYED 2026-03-13, running)                           │
│  │   ├── Claude Agent SDK (Sonnet for owner, Haiku for assistant)     │
│  │   ├── Signal, Telegram, Slack, WhatsApp channels live              │
│  │   ├── Per-group Docker containers with file-based IPC              │
│  │   ├── Per-group CLAUDE.md memory files                              │
│  │   └── jibrain intake hook wired into onMessage callback            │
│  │                                                                     │
│  ├── signal-cli daemon :8080 (standalone, preserved identity)          │
│  ├── Voice bridge :3100 (iOS → OpenAI Realtime, deferred)             │
│  └── ~/jibot-docs (synced from GitHub every 5 min)                    │
│                                                                         │
│  Ethoswarm (cloud)                                                     │
│  ├── jimindbot Mind (Switchboard Curator, ID: XXXXXXXX)               │
│  │   └── Telegram → classify → POST /intake/structured → sprite       │
│  └── amind.ai (email/messaging identity layer)                        │
│      └── curator@example-service.ai                                  │
│                                                                         │
│  Sprites (sprites.app, containerized)                                  │
│  ├── bookmark-extractor (FastAPI + Claude Sonnet)                      │
│  └── meeting-prep (exists, not fully documented)                       │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 2. Knowledge Pipeline Lifecycle

### Stage 1: Intake
Everything enters via `jibrain/intake/` with `status: draft`.

| Source | Path | Automation Level |
|--------|------|------------------|
| WhatsApp URL paste | jibot → relay → bookmark-extractor sprite | Fully automated |
| CLI bookmark | `bookmark-intake.sh` → sprite | Manual trigger |
| Telegram | jimindbot (Ethoswarm) → sprite | Fully automated |
| Daily note callouts | `[!jibrain]` → meeting-extract recipe | Recipe-triggered |
| Meeting transcripts | muesli/ → meeting-extract recipe | Recipe-triggered |
| PDF books | `extract_book_chunked.py` (Gemini vision) | Manual trigger |
| Japanese historical docs | `knowledge-extract/` pipeline | Manual trigger |
| Manual annotation | Direct write with frontmatter | Manual |

### Stage 2: Triage
Recipe scans intake, cross-references existing vault, generates `_review/pending.md`.

### Stage 3: Promote
Human review → items move to `atlas/` with `status: final`.

### Stage 4: Reweave
Backward pass connects new knowledge to existing notes (jibrain-reweave recipe).

### Stage 5: Domain Graduation
When a topic accumulates 15+ files with distinct vocabulary, it graduates from `atlas/` to `domains/` (modeled after `chanoyu/`).

---

## 3. Retrieval Systems

### QMD Search (Primary for Amplifier sessions)
- **What**: Local semantic + keyword search engine (BM25 + vector + LLM rerank)
- **Indexed**: 4,322 docs (jibrain: 2,427, dailynote: 1,012, people: 883)
- **NOT indexed**: concepts/, organizations/, chanoyu/, agents/, tools/
- **Access**: MCP server with 6 tools (search, vector_search, deep_search, get, multi_get, status)
- **Models**: All local (~2GB GGUF: embeddinggemma-300M, qwen3-reranker, qmd-query-expansion)

### Vault Operations (Amplifier tool)
- **What**: Graph analysis, orphan detection, enrichment, tagging
- **Operations**: graph_health, domain_health, people_search, chanoyu_search, knowledge_health
- **Access**: `vault()` tool in Amplifier sessions

### knowledge_synthesis Module (Python, extraction-focused)
- **What**: Claude/Gemini-powered concept extraction from articles and books
- **Output**: Per-file JSON + consolidated JSONL stream
- **Downstream**: `knowledge/` module builds NetworkX graph on top (3,587 nodes, 15,018 edges)
- **CLI**: `make knowledge-sync`, `make knowledge-gemini-extract`

### jibot-3 Direct Read (NOT RUNNING)
- **What**: TypeScript bot reads concepts/ and organizations/ directories directly
- **Access**: Slack commands (`explain <concept>`, `what is <org>`)

### agent-mac Agents (via jibot-docs)
- **What**: OpenClaw/NanoClaw agents read curated content from ~/jibot-docs/
- **Limitation**: Only sees what's been exported via the chanoyu pattern

---

## 4. Sync Topology

```
Joi's Laptop (source of truth)
  ~/switchboard ──► Syncthing ──► agent-mac ~/switchboard (subset)
                                             primary-mac ~/switchboard (subset)
  ~/jibot-docs  ──► GitHub    ──► agent-mac ~/jibot-docs (every 5 min via launchd)

Sprites (bookmark-extractor)
  Writes to ~/switchboard/agents/curator/extractions/
  ──► Syncthing distributes back to all machines

jibrain partition specifically:
  ~/switchboard/jibrain/ ──► Syncthing ──► all machines (non-private)
```

**Security boundary**: `~/switchboard` is the private vault. Only curated subsets (chanoyu/, concepts/, organizations/, people-public/) are exported to jibot-docs. Daily notes, private people notes, health/financial data NEVER leave the laptop.

---

## 5. Component Inventory

### Active and Maintained

| Component | Location | Language | Status |
|-----------|----------|----------|--------|
| jibrain pipeline | ~/switchboard/jibrain/ | Obsidian markdown | Active, growing |
| QMD search | MCP server (local) | Rust/Go | Active, 4,322 docs indexed |
| bookmark-extractor sprite | sprites.app | Python (FastAPI) | Active, deployed |
| knowledge_synthesis | ~/amplifier/amplifier/knowledge_synthesis/ | Python | Active, primary extraction engine |
| knowledge (graph) | ~/amplifier/amplifier/knowledge/ | Python (NetworkX) | Active, downstream of synthesis |
| vault operations | Amplifier tool-vault module | Python | Active |
| jibrain recipes | Amplifier recipes (triage, meeting-extract, reweave) | YAML | Active |
| OpenClaw | agent-mac :18789 | Node.js | Active (being replaced) |
| signal-cli | agent-mac :8080 | Java | Active, standalone |
| jimindbot (Ethoswarm) | cloud (Mind ID: XXXXXXXX) | Ethoswarm platform | Provisioned, testing |
| bookmark-relay | agent-mac :9999 | Node.js | Active |
| Syncthing | all machines | Go | Active |
| agent-mac-heartbeat | ~/scripts/ | bash (launchd) | Active |

### Planned / Not Yet Deployed

| Component | Location | Status |
|-----------|----------|--------|
| NanoClaw | github.com/qwibitai/nanoclaw | **DEPLOYED** on agent-mac, jibrain hook live |
| switchboard sharing (concepts, orgs, people-public) | jibot-docs | Designed, not implemented |
| Ethoswarm /intake/structured endpoint | bookmark-extractor sprite | Designed, needs implementation |
| Amplifier bridge (GTD data → agent-mac) | Not started | Design exists in upgrade docs |
| Proactive agent (morning briefing) | Not started | Designed in upgrade analysis |
| Voice bridge NanoClaw integration | ios-openclaw-voice | Deferred |

### Abandoned / Archived

| Component | Location | Why |
|-----------|----------|-----|
| knowledge_mining module | ~/amplifier/amplifier/knowledge_mining/ | Superseded by knowledge_synthesis; Makefile target deprecated |
| knowledge_integration module | ~/amplifier/amplifier/knowledge_integration/ | Never completed; no README, no Makefile targets |
| jibot-3 | ~/jibot-3/ | Installed but NOT RUNNING; unclear if it will be activated |
| 6 deprecated design docs | (deleted 2026-03-04) | See jibot-docs/design/DEPRECATIONS.md |

---

## 6. Integration Matrix

Which systems know about each other?

| System A | System B | Integration | Status |
|----------|----------|-------------|--------|
| WhatsApp | bookmark-extractor | via jibot relay | Working |
| Telegram | bookmark-extractor | via jimindbot (Ethoswarm) | Testing |
| bookmark-extractor | jibrain/intake | Writes vault markdown via Syncthing | Working |
| QMD | switchboard vault | Indexes jibrain, dailynote, people | Working |
| QMD | concepts/, organizations/, chanoyu/ | NOT indexed | Gap |
| Amplifier | QMD | MCP server tools (search, deep_search) | Working |
| Amplifier | vault operations | tool-vault module | Working |
| Amplifier | jibrain recipes | triage, meeting-extract, reweave | Working |
| Amplifier | knowledge_synthesis | Python module, CLI | Working |
| knowledge_synthesis | knowledge (graph) | Reads extractions.jsonl | Working |
| jibot-3 | switchboard | Reads concepts/ and organizations/ directly | NOT RUNNING |
| OpenClaw | jibot-docs | Reads curated exports via git sync | Working |
| OpenClaw | NanoClaw | Migration complete, NanoClaw deployed | Done |
| Amplifier GTD | agent-mac agents | NO integration (key gap) | Gap |
| jibrain | talk writing | Writer agent drafts exist, no pipeline | Gap |
| Ethoswarm | sprite /intake/structured | Needs implementation | Gap |
| knowledge_synthesis | jibrain | No automated flow between them | Gap |
| QMD | agent-mac agents | No access from agent-mac | Gap |

---

## 7. Data Flow Gaps (Critical)

### Gap 1: QMD indexing is incomplete
QMD indexes only 3 of 55+ vault directories: jibrain, dailynote, people. Missing: concepts/, organizations/, chanoyu/, agents/, Literature/, references/. This means semantic search can't find curated domain knowledge.

### Gap 2: No Amplifier → agent-mac bridge
GTD data (Apple Reminders, beads issues, daily focus) lives on Joi's laptop. agent-mac agents can't access it. The proactive agent (morning briefing, nudges) requires this bridge. Three designs exist (daily snapshot, REST API, hybrid) but none are built.

### Gap 3: knowledge_synthesis output doesn't flow to jibrain
The Python extraction engine produces JSONL and per-file JSON in `.data/`. This data isn't automatically promoted into jibrain/intake/. The two systems operate independently.

### Gap 4: Book ingestion → jibrain path not connected
PDF extraction via Gemini works. Output goes to source-specific directories. But there's no automated path from extracted book content → jibrain/intake/ for triage and atlas promotion.

### Gap 5: Talk/presentation writing pipeline is manual
Writer agent drafts exist in agents/writer/drafts/. Amplifier has blog_writer and blog_synthesize scenarios. But there's no connected pipeline from jibrain knowledge → talk outline → draft → review.

### Gap 6: Ethoswarm /intake/structured endpoint missing
jimindbot is provisioned and ready. The bookmark-extractor sprite has /intake but not /intake/structured (the richer schema jimindbot uses). This blocks Telegram-based automated intake.

### Gap 7: switchboard sharing not implemented
The "chanoyu pattern" for exporting concepts/, organizations/, and people-public/ to jibot-docs is designed but not built. agent-mac agents only see chanoyu/.

---

## 8. Python Module Status

```
~/amplifier/amplifier/
├── knowledge_synthesis/   ◄── PRIMARY: Claude/Gemini extraction, multi-domain
│   ├── 23 files, actively maintained (last commit: Jan 2025)
│   ├── CLI: make knowledge-sync / knowledge-gemini-*
│   └── Output: .data/extractions/*.json + .data/knowledge/extractions.jsonl
│
├── knowledge/             ◄── SECONDARY: NetworkX graph layer
│   ├── 8 files (graph_builder, graph_search, tension_detector, visualizer)
│   ├── CLI: make knowledge-graph-*
│   ├── Reads extractions.jsonl from knowledge_synthesis
│   └── Stats: 3,587 nodes, 15,018 edges, 49 tensions
│
├── knowledge_mining/      ◄── ABANDONED (Dec 2024, never updated)
│   ├── Makefile target explicitly deprecated
│   ├── Uses NetworkX as storage (rejected approach)
│   └── RECOMMENDATION: Archive or delete
│
└── knowledge_integration/ ◄── ABANDONED (Dec 2024, never completed)
    ├── No README, no Makefile targets
    ├── Over-complex unified approach, never reached usable state
    └── RECOMMENDATION: Archive or delete
```

---

## 9. Design Doc Registry

### Knowledge System Docs

| Document | Location | Status |
|----------|----------|--------|
| jibrain Architecture | switchboard/amplifier/JIBRAIN-ARCHITECTURE.md | **Canonical** |
| Knowledge Graph Redesign | switchboard/amplifier/KNOWLEDGE-GRAPH-REDESIGN.md | **Active** |
| Multi-Domain Extraction | switchboard/amplifier/MULTI-DOMAIN-EXTRACTION.md | **Active** |
| Obsidian CLI Integration | switchboard/amplifier/OBSIDIAN-CLI-INTEGRATION.md | **Active** |
| Future Knowledge Vaults | switchboard/amplifier/FUTURE-KNOWLEDGE-VAULTS-PLAN.md | **Proposal** |
| jibrain Conventions Skill | ~/.amplifier/skills/jibrain-conventions/SKILL.md | **Active** |
| Vault Structure | switchboard/jibrain/_STRUCTURE.md | **Living reference** |
| Vault Index | switchboard/jibrain/INDEX.md | **Living reference** |

### Agent System Docs

| Document | Location | Status |
|----------|----------|--------|
| Unified Architecture | jibot-docs/design/jibot-unified-architecture.md | **Canonical** |
| NanoClaw Migration | jibot-docs/design/2026-02-23-nanoclaw-migration-plan.md | **Active** |
| Switchboard Sharing | jibot-docs/design/switchboard-sharing.md | **Active** |
| Workflows | jibot-docs/design/workflows.md | **Active** |
| WhatsApp Commands | jibot-docs/design/whatsapp-commands.md | **Active** |
| Inbox Conventions | jibot-docs/design/inbox-conventions.md | **Active** |
| Deprecations | jibot-docs/design/DEPRECATIONS.md | **Active** |

### Integration Docs

| Document | Location | Status |
|----------|----------|--------|
| Ethoswarm Architecture | switchboard/jibrain/intake/ethoswarm-architecture.md | **Draft** |
| Ethoswarm Integration Thread | switchboard/jibrain/intake/.observations/ | **Active** |
| Ethoswarm Technical Brief | ~/Documents/ethoswarm_agentic_integration_brief_v1_0.md | **External** |
| jimindbot System Prompt | switchboard/agents/_shared/jimindbot-system-prompt.md | **Active** |
| amind.ai | switchboard/jibrain/intake/amind-ai.md | **Draft** |

---

## 10. Quick Reference: "How Do I..."

| Task | System | Command/Path |
|------|--------|--------------|
| Bookmark a URL | WhatsApp | Just paste the URL (bare URL auto-bookmarks) |
| Bookmark a URL (CLI) | bash | `~/jibot-code/scripts/bookmark-intake.sh <url>` |
| Search vault by keyword | Amplifier | `mcp_qmd_search(query="...")` |
| Search vault semantically | Amplifier | `mcp_qmd_vector_search(query="...")` |
| Deep search (hybrid) | Amplifier | `mcp_qmd_deep_search(query="...")` |
| Check vault health | Amplifier | `vault(operation="knowledge_health")` |
| Find stale contacts | Amplifier | `vault(operation="people_stale")` |
| Search chanoyu | Amplifier | `vault(operation="chanoyu_search", query="...")` |
| Extract from PDF book | CLI | `python ~/amplifier/scripts/extract_book_chunked.py <pdf>` |
| Extract Japanese docs | CLI | `cd ~/switchboard/tools/knowledge-extract && python batch_knowledge_extract.py` |
| Run jibrain triage | Amplifier | `recipes(operation="execute", recipe_path="@joi:recipes/jibrain-orient.yaml")` |
| Extract from meetings | Amplifier | `recipes(operation="execute", recipe_path="@joi:recipes/meeting-extract.yaml")` |
| Reweave knowledge | Amplifier | `recipes(operation="execute", recipe_path="@joi:recipes/jibrain-reweave.yaml")` |
| Build knowledge graph | CLI | `cd ~/amplifier && make knowledge-graph-build` |
| Search knowledge graph | CLI | `cd ~/amplifier && make knowledge-graph-search QUERY="..."` |
| Visualize tensions | CLI | `cd ~/amplifier && make knowledge-graph-viz` |