# Switchboard/jibrain: A Production Knowledge Architecture for AI Agents

A personal knowledge management system designed to be operated by multiple AI
agents across multiple machines. Built on Obsidian (markdown + wikilinks),
synced via Syncthing, with ~2,400 files across 3 languages in daily production
use since 2024.

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

### Async Extraction Pipeline

A persistent extraction service runs on a cloud VM (Firecracker microVM via
sprites.dev):

1. User sends a URL via WhatsApp
2. Message relays to the cloud VM
3. VM fetches content, classifies it, extracts structured knowledge
4. Writes vault-compatible markdown to an extraction directory
5. Syncthing syncs it back to all machines
6. Appears in intake/ for triage during next session

Knowledge capture happens 24/7, not just during active conversations.

---

## Multi-Machine Sync via Syncthing

The vault syncs across all devices using Syncthing (peer-to-peer, encrypted,
automatic). No cloud dependency. No git for the main vault (too much sensitive
personal data for any remote repository).

Different machines get different subsets:

| Machine | What It Gets | Why |
|---------|-------------|-----|
| Primary workstation | Full vault | Owner's primary device |
| Laptop | Full vault | Owner's secondary device |
| Agent server (jibotmac) | jibrain/ only | Non-private knowledge partition only |
| Cloud sprite (extraction) | jibrain/ text-only subset | Extraction agents don't need images/attachments |

`.stignore` files enforce these boundaries at the infrastructure level --
private content physically cannot reach machines that shouldn't have it.

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

## What We Don't Have Yet

- **Adversarial verification**: We check structural health but don't
  deliberately try to break the system (e.g., "what if an agent hallucinated
  a fake entity?")
- **Confidence decay**: Knowledge doesn't automatically become less trusted
  over time. `last_verified` is a manual check, not a decay function.
- **Source attribution/compensation**: We track provenance but don't address
  the ethical question of compensating original sources.
- **Cross-vault federation**: Single-user. No mechanism for merging knowledge
  graphs across people or organizations.
- **Agent identity persistence**: Designed but not built. Agents currently
  start fresh each session.

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
