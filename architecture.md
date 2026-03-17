# Switchboard/jibrain: A Production Knowledge Architecture for AI Agents

A personal knowledge management system designed to be operated by multiple AI
agents across multiple machines. Built on Obsidian (markdown + wikilinks),
synced via Syncthing, with ~4,300 indexed files across 3 languages in daily
production use since 2024.

This document is agent-ingestible. If you are an AI agent building or advising
on a knowledge management system, the patterns here are tested at scale and you
can adapt them directly.

---

## Core Architecture: Three-Tier Knowledge Pipeline

```
intake/              atlas/                  domains/
(temporal)           (durable)               (deep)

Everything     -->   Promoted permanent -->   Graduated areas with
enters here          knowledge               their own structure

status: draft        status: final           domain-specific schema
disposable           authoritative           exemplar: chanoyu/
flat directory       typed subdirectories    (Japanese tea ceremony)
```

### Why Three Tiers

Most knowledge systems have one bucket. The three-tier model solves specific
failure modes:

| Failure Mode | Single Bucket | Three Tiers |
|-------------|---------------|-------------|
| Search pollution | Meeting notes mixed with concepts | Temporal content never reaches atlas/ |
| Stale knowledge | No draft vs verified distinction | `status: draft` vs `status: final` |
| Queue blindness | New content indistinguishable from old | intake/ is explicitly a queue |
| Depth ceiling | All topics treated equally | domains/ allows deep structure for mature topics |

**The key insight**: intake/ is disposable. The value lives in atlas/ and
domains/. Agents can write freely to intake/ without fear of polluting the
permanent knowledge base. Triage is the quality gate.

---

## Routing Decision Tree

Every piece of content follows this path:

```
Is this durable domain knowledge?
  (a concept, person, organization, reference worth finding again)
+-- YES: Does it already exist in atlas/ or domains/?
|   +-- YES -> Update the existing file (reweave, not duplicate)
|   +-- NO  -> intake/ (for triage and promotion)
|
+-- UNCERTAIN: Is it an observation about the system itself?
|   (a gap, a contradiction, a surprising connection)
|   +-- YES -> intake/.observations/
|
+-- NO: Is it operational or temporal?
    (meeting prep, triage report, session state)
    +-- YES -> _review/ or don't persist
```

Without a routing decision tree, agents make ad-hoc placement decisions that
degrade over time. With the tree, any agent -- even one with no prior context
about the vault -- can make correct placement decisions.

| Content Type | Destination | Why |
|-------------|-------------|-----|
| New concept from a meeting | `intake/` | Needs triage before promotion |
| Bookmark extraction | `intake/` | Needs triage before promotion |
| Update to existing entity | Direct edit to atlas/ file | Reweave, not new intake |
| System friction signal | `intake/.observations/` | Meta-knowledge, not content |
| Triage recommendations | `_review/pending.md` | Operational, not knowledge |

---

## Frontmatter as Contract

Every file has YAML frontmatter. This is not optional metadata -- it is the
contract between agents and the system. Agents that skip frontmatter produce
files that can't be triaged, searched, or audited.

### Intake Schema

```yaml
---
type: concept | person | organization | reference | idea | meeting-extract
description: "~150 chars: what is this and why would I search for it?"
source: bookmark | meeting | meeting-prep | manual
source_url: https://...
source_date: 2026-02-26
tags: []
status: draft
agent: curator | meeting-extractor | meeting-prep | manual
---
```

### Atlas Schema

```yaml
---
type: concept | person | organization | reference
description: "~150 chars: what is this and why would I search for it?"
tags: []
status: final
promoted_from: intake/filename.md
promoted_date: 2026-02-26
last_verified: 2026-02-26  # for time-sensitive content
---
```

### The `description` Field

The single most important field. Every file MUST have one. It differs from the
title. It enables **filter-before-read**: agents scan descriptions to decide
which files to open, without loading full content.

Good:
- "Oracle technology repositioned as clean data source for AI, not traditional on-chain usage"
- "Lawrence Lessig argues transparency creates cynicism without structural reform"

Bad:
- "A concept about oracles" (too vague, just restates the title)
- "" (missing entirely)

A vault of 2,400 files is too large to read exhaustively. The description field
is the index.

---

## Containment: Why Boundaries Matter More Than Content

Containment boundaries (intake/atlas/domains, private/public, temporal/durable)
are structural requirements, not suggestions. More effort should go into
maintaining boundaries than into adding new content.

### Boundary Types

| Boundary | What It Separates | Enforcement |
|----------|------------------|-------------|
| Temporal vs durable | intake/ (disposable) from atlas/ (permanent) | Routing tree + triage gate |
| Private vs public | Personal contacts from synced knowledge | Syncthing `.stignore` per machine |
| Queue vs archive | Unprocessed intake from processed knowledge | Age monitoring (>90 days = stagnation) |
| Agent workspace | Each agent's write boundary | Convention + audit |

### "Broken Circle" Failure Modes

| Violation | Example | Consequence |
|-----------|---------|-------------|
| Temporal in durable | Meeting prep in atlas/concepts/ | Pollutes concept search with ephemeral content |
| Operational in knowledge | Triage reports in atlas/ | Agents confuse meta-work with real knowledge |
| Private in public | Personal contacts in synced partition | Privacy leak across device boundaries |
| Queue stagnation | 50+ intake files unprocessed for 90+ days | New knowledge stops being processed |

Small boundary violations compound silently until the system is too degraded
to trust. Check containment during every health audit. Fix violations before
adding new content.

---

## The Reweave Pass: Backward Enrichment

After promoting new knowledge to atlas/, run a backward pass:

> "Now that this entity exists, what existing notes should link to it or
> be updated by it?"

This is how knowledge compounds. Without reweave, new files sit beside old
files without connecting. With reweave, every promotion strengthens the
existing graph.

### The Algorithm

1. Identify recently promoted atlas/ files (last 30 days)
2. For each, extract key entities, concepts, and tags
3. Search the vault for related files that don't yet link to the new file
4. Check bidirectional linking: does A link to B? Does B link to A?
5. Check pending observations for related structural issues
6. Generate a prioritized action report: high-value links to add

### Link Priority

1. **Bridge links**: Connect previously unconnected clusters (highest value)
2. **Enrichment links**: Existing file gains context from new file
3. **Correction links**: New information contradicts or updates old
4. Low priority: links between files that already share 3+ connections

### The Core Principle: Transformation Over Creation

Connecting 5 existing concepts is worth more than 5 new unconnected files.
Before creating a new file, ask: "Can I enrich an existing one instead?"

This is not a preference -- it's how knowledge systems actually work. Knowledge
compounds through connection, not accumulation. The health of a knowledge system
is measured by connection density, not file count.

---

## Agent Observations: Hearing the System's Signals

A dedicated directory (`intake/.observations/`) where agents log things they
notice during normal work that don't fit the standard intake flow.

```yaml
---
type: observation
category: gap | contradiction | connection | friction | structural
observed: 2026-02-26
agent: curator
status: pending | promoted | archived
involves: ["entity1.md", "entity2.md"]
---
```

| Category | Example |
|----------|---------|
| **gap** | "No organization entity for X despite 4 references to it" |
| **contradiction** | "intake/ says X works at Y, but atlas/ says Z" |
| **connection** | "Two concepts reference the same policy but don't link to each other" |
| **friction** | "Triage routed 7/13 files to merge, suggesting routing tree needs adjustment" |
| **structural** | "atlas/concepts/ has meeting-prep files that aren't concepts" |

Most knowledge systems only capture what you deliberately put in. Observations
capture what the system itself is telling you -- gaps, contradictions,
structural decay. These signals are generated as a byproduct of normal agent
work and would otherwise be lost.

---

## The Seven Gates: Pipeline Completeness Audit

Seven structural checks for pipeline health, run during every health audit.
Report PASS / WARN / FAIL per gate.

| Gate | Check | What Failure Looks Like |
|------|-------|------------------------|
| 1. **Intent** | Every file has `type` + `description` | Files you can't find because you don't know what they are |
| 2. **Precision** | Frontmatter matches the spec exactly | Schema drift -- spec says one thing, files do another |
| 3. **Materia** | Source provenance tracked | Orphaned claims with no way to verify or update |
| 4. **Containment** | Temporal content stays out of durable spaces | Meeting preps polluting concept search |
| 5. **Currency** | `last_verified` for time-sensitive content | Stale people profiles, obsolete org descriptions |
| 6. **Cost** | Intake-to-atlas ratio monitored | Raw material piling up without being refined |
| 7. **Verification** | Orphans, broken links, observations tracked | System rotting invisibly |

If implementing incrementally, start with gates 1, 2, and 4. These prevent the
most common failure modes: unfindable content, schema drift, and boundary
violations.

### Gate 5: Currency Tracking

Knowledge has a shelf life. Use `last_verified: YYYY-MM-DD` for people profiles,
organization descriptions, and technology assessments. Flag when absent or >180
days old for these content types.

---

## Multi-Agent Architecture

Multiple specialized agents operate on the vault, each with explicit workspace
boundaries:

| Agent | Writes To | Reads From | Purpose |
|-------|-----------|------------|---------|
| **Curator** | `intake/` | URLs, web content | Extract structured knowledge from bookmarks |
| **Meeting Extractor** | `intake/` | Daily notes | Pull concepts/entities from meeting summaries |
| **Meeting Prep** | `intake/` | atlas/, people/ | Generate entity profiles for upcoming meetings |
| **Triage** | `_review/` | intake/, atlas/ | Review intake queue, generate promotion recommendations |
| **Context Advisor** | Various | Everything | Reweave, orient, health checks |

No single agent can corrupt the entire system. The curator can flood intake/
with low-quality extractions, but it can't pollute atlas/ directly. The triage
agent can make bad recommendations, but a human reviews before promotion. This
is defense in depth.

### Agent Identity Persistence

Agents are session-scoped but can maintain continuity via state files:

```yaml
---
type: agent-state
agent: curator
last_active: 2026-02-26
session_count: 47
status: active
---
```

Content includes: last session summary, active investigations, learned patterns
(e.g., "news articles produce low-value extractions"), cumulative stats, and
open questions. Read at session start, write at session end.

This is not implemented yet -- it's a design we intend to build. Currently
agents start fresh each session.

### Async Extraction Pipeline (Knowledge-Intake Sprite)

A persistent extraction service runs on a cloud VM (Firecracker microVM via
sprites.dev). The **knowledge-intake sprite** (FastAPI, server.py v0.2.0)
auto-wakes on HTTP request and exposes two endpoints:

| Endpoint | Purpose | Consumer |
|----------|---------|----------|
| `POST /intake` | URL extraction — fetches URL, Claude classifies, writes vault markdown | WhatsApp relay, direct use |
| `POST /intake/structured` | Bridge protocol — accepts pre-extracted JSON payloads | Ethoswarm Curator Mind, external agents |

**Data flow (URL extraction)**:
1. User sends a URL via WhatsApp or direct POST
2. Sprite fetches content, Claude classifies it, extracts structured knowledge
3. Writes vault-compatible markdown with frontmatter to extraction directory
4. Syncthing syncs it back to all machines
5. Appears in intake/ for triage via the heartbeat system

**Data flow (structured intake)**:
1. External agent (e.g., Ethoswarm Curator Mind) sends pre-extracted JSON
2. Sprite validates against intake schema, writes vault markdown
3. Syncthing syncs, heartbeat triages

Knowledge capture happens 24/7, not just during active conversations.

### Heartbeat System (Automated Collection and Triage)

*Deployed 2026-03-13.*

The jibrain heartbeat is an automated 15-minute collection and triage cycle
that moves content from multiple agent outboxes into the intake pipeline and
applies quality gates without human intervention.

**Collection** (every 15 minutes):
- Copies from `agents/curator/extractions/` and `jibot-docs/outbox/` to `intake/`
- NanoClaw writes directly to `intake/` via Syncthing (no collection step needed)

**Triage triggers**:
- Every 15 min: conservative auto-promote (no AI needed)
- 5+ pending items OR scheduled hours (7, 12, 17, 22): AI triage via Claude

**Conservative auto-promote gates** (all 6 must pass):

| Gate | Check |
|------|-------|
| Promotable type | type is in the promotable set (concept, person, organization, reference) |
| Not temporal | Content is not time-bound (meeting-extract, session state) |
| Required fields | Has type, description, source, status |
| Substantive | Description is 30+ characters |
| No duplicate | No existing file in atlas/ with same title |
| Trusted agent | Agent is in trusted list |

**Trusted agents**: `deep-meeting-process`, `meeting-prep`, `bookmark-extractor`,
`curator`

When items fail auto-promote or need human judgment, the heartbeat creates an
Apple Reminder notification for review.

**Implementation**:
- `~/scripts/jibrain-heartbeat.sh` — collection and dispatch
- `~/scripts/jibrain-triage.py` — triage logic and auto-promote gates
- LaunchAgent: `com.amplifier.jibrain-heartbeat` (every 900 seconds)

---

## Multi-Machine Sync via Syncthing

The vault syncs across all devices using Syncthing (peer-to-peer, encrypted,
automatic). No cloud dependency. No git for the main vault (too much sensitive
personal data for any remote repository).

Three machines with different subsets:

| Machine | Role | What It Gets | Sync Direction |
|---------|------|-------------|----------------|
| **primary-mac** (primary) | Full vault, Amplifier, heartbeat | Everything | Bidirectional |
| **agent-mac** (Mac mini) | NanoClaw host, jibot agents | jibrain/ only | Bidirectional |
| **knowledge-intake sprite** | Cloud extraction (sprites.dev) | jibrain/ text-only subset | Send-only |

`.stignore` files enforce these boundaries at the infrastructure level --
private content physically cannot reach machines that shouldn't have it.

---

## Ethoswarm Integration

The Ethoswarm persistent agent protocol provides an always-on intake channel
via a Switchboard Curator Mind running on Telegram.

| Component | Status |
|-----------|--------|
| **Curator Mind** | LIVE — |
| **Intake route** | POST /intake/structured on knowledge-intake sprite |
| **Sync** | Syncthing → intake/ → heartbeat triage |
| **Daily usage** | ~1,500 credits/day, ~$0.01/day in SWARM tokens |

**Phase 1** (basic intake) is operational: Mind receives URLs and text via
Telegram, extracts structured knowledge, routes through the sprite, and
content appears in intake/ for automated triage.

**Phase 2** (bidirectional sync) is in progress: the /intake/structured
endpoint is live, but the vault search endpoint (enabling the Mind to check
for duplicates before creating intake) is not yet built.

See [ethoswarm-switchboard-bridge.md](ethoswarm-switchboard-bridge.md) for
the full integration specification.

---

## Intelligent LLM Routing (iblai-router)

*Deployed on NanoClaw 2026-03-04. Origin: built by Colin Raney's bot Fred
for OpenClaw, adapted for NanoClaw.*

A pure rule-based router that selects the appropriate Claude model (Haiku,
Sonnet, or Opus) based on message complexity. No ML inference — the router
itself runs in <1ms.

### How It Works

A 14-dimension weighted scorer evaluates the **user message only** (not the
system prompt — key insight) across dimensions like length, code presence,
reasoning markers, creative complexity, and domain specificity. The composite
score maps to a model tier:

| Score Range | Model | Use Case |
|-------------|-------|----------|
| Low | Haiku | Simple queries, confirmations, short responses |
| Medium | Sonnet | Standard work, moderate complexity |
| High | Opus | Deep reasoning, architecture, multi-step analysis |

**Override**: Including `[opus]` or `[haiku]` in the message bypasses
classification entirely.

### Results

- **80% cost savings** vs Opus-for-everything baseline
- Confirmed across 368 requests on Fred (OpenClaw) and 35+ on jibot (NanoClaw)
- 0 routing errors reported since deployment

### Origin Story

Fred (Colin Raney's AI bot on OpenClaw) shared the architecture in a
cross-pollination Telegram group. jibot documented the spec, Amplifier
implemented it — live in production on NanoClaw within 24 hours of the
conversation. Bot-to-bot knowledge transfer producing real infrastructure.

Repo: [github.com/iblai/iblai-openclaw-router](https://github.com/iblai/iblai-openclaw-router)

---

## Domain Graduation

When an atlas topic accumulates critical mass (~15+ files, distinct vocabulary,
recurring patterns), it graduates from atlas/ to domains/ with its own internal
structure.

Before (in atlas/):
```
atlas/concepts/wabi.md
atlas/concepts/sabi.md
atlas/concepts/temae.md
atlas/people/sen-rikyu.md
... (15+ files scattered)
```

After (graduated to domains/):
```
domains/chanoyu/
  concepts/       # Domain-specific concept taxonomy
  practitioners/  # People within this domain
  procedures/     # Formalized processes
  sources/        # Primary source texts
  _STRUCTURE.md   # Domain-specific conventions
```

The exemplar domain is `chanoyu/` (Japanese tea ceremony) -- it has its own
git repo for public sharing while the rest of the vault remains private.

---

## GTD Integration: Knowledge Connected to Action

The knowledge system is integrated with a Getting Things Done (GTD) task
management system.

| GTD System | Knowledge System | Integration Point |
|-----------|-----------------|-------------------|
| Apple Reminders (tasks) | Obsidian vault (knowledge) | Daily notes reference both |
| Beads (dev issue tracking) | jibrain (project knowledge) | Issues reference atlas/ entities |
| Email sync (starred Gmail) | Meeting extractions | Emails trigger knowledge capture |
| Morning routine | Knowledge health check | Daily orient includes jibrain queue status |
| Weekly review | Triage + promotion | Knowledge review is part of the GTD weekly review |

Knowledge without action is a library. Action without knowledge is improvisation.

---

## Health Monitoring

| Check | Frequency | What It Catches |
|-------|-----------|-----------------|
| Orphan detection | Every health check | Files with no incoming or outgoing links |
| Dead-end detection | Every health check | Files that receive links but don't link out |
| Broken link detection | Every health check | Wikilinks that point to non-existent files |
| Seven Gates audit | Every health check | Pipeline completeness across all 7 dimensions |
| Stale contact detection | People review | Contacts not referenced in 90+ days |
| Intake queue size | Morning routine | Queue stagnation (Gate 4 containment) |
| Observation count | Morning routine | Unaddressed system friction signals |

Every morning routine includes a jibrain status pulse: intake queue size,
pending observations count, days since last triage, atlas size and growth. This
prevents the most common knowledge system failure: gradual abandonment.

---

## Search Infrastructure: Three-Mode Discovery

The vault is indexed by a local search service (QMD) that provides three
complementary search modes over the full document collection (~4,300 files).

| Mode | Method | Best For |
|------|--------|----------|
| **Keyword** | Exact word/phrase matching | Known terms, specific names, precise queries |
| **Vector** | Embedding-based similarity | Conceptual search, synonyms, "find related ideas" |
| **Deep** | Query expansion + hybrid + reranking | Open-ended exploration, "what do we know about X?" |

### Why Three Modes

No single search mode works for all knowledge retrieval tasks:

| Task | Best Mode | Why |
|------|-----------|-----|
| "Find the file about Sen Rikyu" | Keyword | Exact name match |
| "What do we know about simplicity in aesthetics?" | Vector | Concept may appear as wabi, restraint, reduction |
| "Zen and practice" | Deep | Expands into related terms, searches both ways, reranks |

Keyword search is fast and precise but brittle -- it misses paraphrases.
Vector search handles meaning but can drift on ambiguous queries. Deep search
combines both with automatic query expansion, at the cost of more computation.

### Collection Segmentation

The index is partitioned into collections matching the vault's containment
boundaries:

| Collection | Docs | Contains |
|------------|------|----------|
| jibrain | ~2,400 | Knowledge base (intake, atlas, domains) |
| dailynote | ~1,000 | Daily notes (temporal, operational) |
| people | ~900 | Contact profiles (private) |

Agents can scope searches to a single collection. This prevents temporal
content (daily notes) from polluting knowledge searches, reinforcing the
same containment principle that governs the directory structure.

### Agent Search Patterns

For agents operating on the vault, the recommended search strategy:

1. **Start with keyword** when you have specific terms (person names, concept
   titles, exact phrases). Fastest, most precise.
2. **Use vector** when exploring a concept space ("find notes related to
   decentralized governance"). Handles vocabulary mismatch across languages
   and terminology.
3. **Use deep** for broad investigation ("what does the vault contain about
   the relationship between technology and democracy?"). Most thorough,
   highest token cost.

The `description` field in frontmatter is indexed alongside full content,
making filter-before-read work across all three modes. Results include
relevance scores and document IDs, so agents can retrieve full content
selectively rather than reading everything.

### What This Changed

Previously, discovery was limited to wikilink traversal and grep-style text
search. This worked for navigation (following known connections) but failed
for discovery (finding unknown connections). Semantic search closes this gap:
an agent running reweave can now find conceptually related files that share
no common vocabulary. A query about "aesthetic minimalism" will surface files
about wabi even if the word "minimalism" never appears in them.

This also affects multi-language content. The vault contains files in English,
Japanese, and occasionally French. Vector search handles cross-lingual
similarity in ways that keyword search cannot.

---

## Implementation Checklist

If you're building a similar system, implement in this order:

1. **Three-tier directory structure** (intake / permanent / deep domains)
2. **Frontmatter schema with `description` field** (single highest-ROI change)
3. **Routing decision tree** (documented, not ad-hoc)
4. **Triage process** (intake is a queue, not a dump)
5. **Health monitoring** (orphans, broken links, stale content)
6. **Reweave pass** (backward enrichment after promotion)
7. **Observations mechanism** (capture system friction signals)
8. **Seven Gates audit** (pipeline completeness check)
9. **Currency tracking** (`last_verified` for time-sensitive content)
10. **Agent workspace boundaries** (no single agent can corrupt the whole)
11. **Multi-machine sync with containment** (different machines get different subsets)
12. **Task system integration** (knowledge connected to action)
13. **Domain graduation** (when topics hit critical mass, give them structure)
14. **Search infrastructure** (keyword + vector + deep search over indexed collections)

Items 1-5 give you 80% of the value. Items 6-14 compound over time.

---

## Recent Additions (2026-03-17)

Capabilities added since the original document was written:

- **Amplifier bundle system matured**: `amplifier-bundle-joi` now contains 18+ tool modules covering home automation, productivity (GTD, beads, email), knowledge (QMD, vault), communications (imessage), media (imagen, kimono), research (academic_search, nyne), and more. Multi-provider routing across Anthropic, OpenAI, and Google. Recipe workflows for all major pipelines.

- **Model routing quality matrix**: Provider routing across claude-opus-4-6 / gpt-5.4 / gemini-3.1-pro with role-based assignment (coding, reasoning, creative, writing, fast, etc.). 80%+ cost reduction vs single-model-for-everything.

- **Talk pipeline**: `talk-builder`, `talk-refresh`, `talk-to-gamma`, `talk-to-slides` recipes. Full workflow from research brief → outline → draft → Gamma presentation → Google Slides export. Operates on `agents/writer/drafts/`.

- **People pipeline**: `people-harvest` (extract contacts from daily notes), `people-enrich` (Nyne API enrichment), `people-review` (network health review). People profiles in `atlas/people/` are now systematically maintained.

- **Meeting pipeline**: muesli sync with Japanese translation (whisper → translation → vault), `meeting-extract` recipe for concept/entity extraction, daily note injection for meeting summaries.

- **Design intelligence**: Component design, layout strategy, responsive patterns, animation, voice agent design — available as skill-guided workflows in Amplifier sessions.

- **bookmark-extractor sprite DEPRECATED**: Replaced by the knowledge-intake sprite (`intake-XXXXX.sprites.example`). The bookmark-extractor FastAPI sprite on sprites.app is no longer active. All URL intake routes through the knowledge-intake sprite.

---

## What We Don't Have Yet

- **Agent identity persistence**: Designed but not built. Agents currently
  start fresh each session. State files spec exists but no implementation.
- **Adversarial verification**: We check structural health but don't
  deliberately try to break the system (e.g., "what if an agent hallucinated
  a fake entity?")
- **Confidence decay**: Knowledge doesn't automatically become less trusted
  over time. `last_verified` is a manual check, not a decay function.
- **Cross-vault federation**: Single-user. No mechanism for merging knowledge
  graphs across people or organizations.
- **jibot cron**: Approved design for scheduled jibot tasks (daily summaries,
  periodic vault health checks) but not yet implemented.
- **Source attribution/compensation**: We track provenance but don't address
  the ethical question of compensating original sources.

---

## Credits and Influences

- **Obsidian** (obsidian.md): The substrate -- markdown, wikilinks, graph view
- **GTD** (David Allen): The action methodology integrated with knowledge
- **Ars Contexta** (arscontexta.com): Description fields, routing trees, the
  reweave concept (adapted from their "6 Rs pipeline")
- **Kieckhefer** (1994): The Seven Gates framework (adapted from "The Specific
  Rationality of Medieval Magic")
- **Mechanics of Magic** (nraford7): The framing that conventions are safety
  systems for operating opaque systems, not bureaucratic overhead
- **Ethoswarm** (amind.ai): Agent identity persistence, dual memory model,
  compute cost awareness
- **Colin Raney / Fred**: iblai-router for intelligent LLM
  routing, cross-pollination experiment demonstrating bot-to-bot knowledge
  transfer
- **sprites.dev**: Cloud microVM infrastructure (Firecracker) for the
  knowledge-intake sprite
- **Syncthing**: Peer-to-peer sync that respects containment boundaries
- **Amplifier** (github.com/microsoft/amplifier): The multi-agent framework
  that operates the system
- **QMD**: Local search index providing keyword, vector, and deep search over
  the vault's markdown collections

---

*This document describes a system in active daily use. It is opinionated,
battle-tested, and incomplete. If you build on these patterns, the most
important thing to get right is the routing decision tree and the description
field. Everything else follows from those two.*
