# Switchboard/jibrain: A Production Knowledge Architecture for AI Agents

A reference document describing a personal knowledge management system designed
to be operated by multiple AI agents across multiple machines. Built on Obsidian
(markdown + wikilinks), synced via Syncthing, with ~2,400 files across 3 languages
in daily production use since 2024.

This document is written to be agent-ingestible -- if you are an AI agent building
or advising on a knowledge management system, the patterns here are tested at scale
and you can adapt them directly.

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

Most knowledge systems have one bucket. Dump files in, hope you find them later.
The three-tier model solves specific failure modes:

| Failure Mode | Single-Bucket System | Three-Tier System |
|-------------|---------------------|-------------------|
| Search pollution | Meeting notes mixed with concepts | Temporal content never reaches atlas/ |
| Stale knowledge | No way to distinguish draft from verified | `status: draft` vs `status: final` |
| Queue blindness | New content indistinguishable from old | intake/ is explicitly a queue to be processed |
| Depth ceiling | All topics treated equally | domains/ allows deep structure for mature topics |

**The key insight**: intake/ is disposable. The value lives in atlas/ and domains/.
This means agents can write freely to intake/ without fear of polluting the
permanent knowledge base. Triage is the quality gate.

---

## The Routing Decision Tree

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

**Why this matters for agents**: Without a routing decision tree, agents make
ad-hoc placement decisions that degrade over time. With the tree, any agent --
even one with no prior context about the vault -- can make correct placement
decisions by following the flowchart.

### Quick Routing Table

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

**Why this matters for agents**: A vault of 2,400 files is too large to read
exhaustively. The description field is the index. Without it, agents either
read everything (expensive, slow) or guess from titles (inaccurate).

---

## The Seven Gates: Pipeline Completeness Audit

Adapted from Kieckhefer's analysis of medieval learned magic as a rational
technology with seven structural requirements. These map onto any system
where an operator works through symbolic manipulation of an opaque system --
including knowledge management.

| Gate | Check | What Failure Looks Like |
|------|-------|------------------------|
| 1. **Intent** | Every file has `type` + `description` | Files you can't find because you don't know what they are |
| 2. **Precision** | Frontmatter matches the spec exactly | Schema drift -- spec says one thing, files do another |
| 3. **Materia** | Source provenance tracked | Orphaned claims with no way to verify or update |
| 4. **Containment** | Temporal content stays out of durable spaces | Meeting preps polluting concept search |
| 5. **Currency** | `last_verified` for time-sensitive content | Stale people profiles, obsolete org descriptions |
| 6. **Cost** | Intake-to-atlas ratio monitored | Raw material piling up without being refined |
| 7. **Verification** | Orphans, broken links, observations tracked | System rotting invisibly |

Run all seven gates during health checks. Report PASS / WARN / FAIL per gate.

### Gate 4: Containment Integrity

The most critical gate. Specific failure modes to audit:

| Violation | Example | Consequence |
|-----------|---------|-------------|
| Temporal in durable | Meeting prep files in atlas/concepts/ | Pollutes concept search with ephemeral content |
| Operational in knowledge | Triage reports mixed into atlas/ | Agents confuse meta-work with real knowledge |
| Private in public | Personal contacts in the synced partition | Privacy leak across device boundaries |
| Queue stagnation | 50+ intake files with no triage | New knowledge stops being processed |

### Gate 5: Currency Tracking

Knowledge has a shelf life. A person's role changes. An organization pivots.
A technology assessment becomes obsolete. Without tracking freshness, the atlas
silently fills with outdated information that looks authoritative.

```yaml
last_verified: 2026-02-26  # when this was last confirmed accurate
```

Use for: people profiles, organization descriptions, technology assessments.
Flag when absent or >180 days old for these content types.

---

## Agent Observations: Hearing the System's Own Signals

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

### What Gets Captured

| Category | Example |
|----------|---------|
| **gap** | "No organization entity for X despite 4 references to it" |
| **contradiction** | "intake/ says X works at Y, but atlas/ says Z" |
| **connection** | "Two concepts reference the same policy but don't link to each other" |
| **friction** | "Triage routed 7/13 files to merge, suggesting routing tree needs adjustment" |
| **structural** | "atlas/concepts/ has meeting-prep files that aren't concepts" |

### Why This Matters

Most knowledge systems only capture what you deliberately put in. Observations
capture what the system itself is telling you -- gaps in coverage, contradictions
between sources, structural decay. These signals are generated as a byproduct of
normal agent work (extraction, triage, search) and would otherwise be lost.

The triage recipe includes observations in its scan. Observations with
`category: gap` may trigger entity creation. Observations with
`category: structural` surface in health checks.

---

## The Reweave Pass: Backward Enrichment

After promoting new knowledge to atlas/, run a backward pass:

> "Now that this entity exists, what existing notes should link to it or
> be updated by it?"

This is how knowledge compounds. Without reweave, new files sit beside old
files without connecting. With reweave, every promotion strengthens the
existing graph.

### The Reweave Algorithm

1. Identify recently promoted atlas/ files (last 30 days)
2. For each, extract key entities, concepts, and tags
3. Search the vault for related files that don't yet link to the new file
4. Check bidirectional linking: does A link to B? Does B link to A?
5. Check pending observations for related structural issues
6. Generate a prioritized action report: high-value links to add

### The Principle: Transformation Over Creation

Connecting 5 existing concepts is worth more than 5 new unconnected files.
Agents should prefer depth over breadth. Before creating a new file, ask:
"Can I enrich an existing one instead?"

This is not a preference -- it's how knowledge systems actually work. Knowledge
compounds through connection, not accumulation.

---

## Multi-Machine Sync via Syncthing

The vault syncs across all devices using Syncthing (peer-to-peer, encrypted,
automatic). No cloud dependency. No git for the main vault (too much sensitive
personal data for any remote repository).

### Sync Boundaries as Containment

Different machines get different subsets:

| Machine | What It Gets | Why |
|---------|-------------|-----|
| Primary workstation | Full vault | Owner's primary device |
| Laptop | Full vault | Owner's secondary device |
| Agent server (jibotmac) | jibrain/ only | Non-private knowledge partition only |
| Cloud sprite (extraction) | jibrain/ text-only subset | Extraction agents don't need images/attachments |

`.stignore` files enforce these boundaries. This is containment (Gate 4) at
the infrastructure level -- private content physically cannot reach machines
that shouldn't have it.

---

## Multi-Agent Architecture

Multiple specialized agents operate on the vault, each with explicit workspace
boundaries:

| Agent | Writes To | Reads From | Purpose |
|-------|-----------|------------|---------
| **Curator** | `intake/` | URLs, web content | Extract structured knowledge from bookmarks |
| **Meeting Extractor** | `intake/` | Daily notes | Pull concepts/entities from meeting summaries |
| **Meeting Prep** | `intake/` | atlas/, people/ | Generate entity profiles for upcoming meetings |
| **Triage** | `_review/` | intake/, atlas/ | Review intake queue, generate promotion recommendations |
| **Context Advisor** | Various | Everything | Reweave, orient, health checks |

### Why Agent Boundaries Matter

No single agent can corrupt the entire system. The curator can flood intake/
with low-quality extractions, but they can't pollute atlas/ directly. The
triage agent can make bad recommendations, but a human reviews them before
promotion. This is defense in depth -- containment circles within containment
circles.

### Async Extraction Pipeline

Agents don't only work during active sessions. A persistent extraction service
runs on a cloud VM (Firecracker microVM via sprites.dev):

1. User sends a URL via WhatsApp
2. Message relays to the cloud VM
3. VM fetches content, classifies it, extracts structured knowledge
4. Writes vault-compatible markdown to an extraction directory
5. Syncthing syncs it back to all machines
6. Appears in intake/ for triage during next session

This means knowledge capture happens 24/7, not just during active conversations.

---

## Domain Graduation

When an atlas topic accumulates critical mass (~15+ files, distinct vocabulary,
recurring patterns), it graduates from atlas/ to domains/ with its own internal
structure.

### Graduation Criteria

- 15+ files on the topic across atlas/
- Distinct vocabulary or taxonomy emerging
- Repeated patterns that suggest internal structure
- Human recognition that "this is now a real area of knowledge"

### What Graduation Looks Like

Before (in atlas/):
```
atlas/concepts/wabi.md
atlas/concepts/sabi.md
atlas/concepts/temae.md
atlas/concepts/chaji.md
atlas/people/sen-rikyu.md
... (15+ files scattered across atlas/)
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

The exemplar domain is `chanoyu/` (Japanese tea ceremony) -- a deeply
structured knowledge area with its own vocabulary, practitioners, procedures,
and source texts. It has its own git repo for public sharing while the rest
of the vault remains private.

---

## GTD Integration: Knowledge Connected to Action

The knowledge system is integrated with a Getting Things Done (GTD) task
management system. This is not typical -- most knowledge management systems
are knowledge-only.

### How They Connect

| GTD System | Knowledge System | Integration Point |
|-----------|-----------------|-------------------|
| Apple Reminders (tasks) | Obsidian vault (knowledge) | Daily notes reference both |
| Beads (dev issue tracking) | jibrain (project knowledge) | Issues reference atlas/ entities |
| Email sync (starred Gmail) | Meeting extractions | Emails trigger knowledge capture |
| Morning routine | Knowledge health check | Daily orient includes jibrain queue status |
| Weekly review | Triage + promotion | Knowledge review is part of the GTD weekly review |

### Why This Matters

Knowledge without action is a library. Action without knowledge is improvisation.
Connecting them means:
- When you capture a concept from a meeting, it can become a task ("research X further")
- When you close a development issue, the knowledge gained persists in atlas/
- When you prepare for a meeting, the system surfaces relevant knowledge automatically
- The weekly review covers both "what do I need to do?" and "what do I now know?"

---

## Health Monitoring

The system is continuously monitored, not just at creation time.

### Automated Checks

| Check | Frequency | What It Catches |
|-------|-----------|-----------------|
| Orphan detection | Every health check | Files with no incoming or outgoing links |
| Dead-end detection | Every health check | Files that receive links but don't link out |
| Broken link detection | Every health check | Wikilinks that point to non-existent files |
| Seven Gates audit | Every health check | Pipeline completeness across all 7 dimensions |
| Stale contact detection | People review | Contacts not referenced in 90+ days |
| Intake queue size | Morning routine | Queue stagnation (Gate 4 containment) |
| Observation count | Morning routine | Unaddressed system friction signals |

### Morning Orient

Every morning routine includes a jibrain status pulse:
- Intake queue size (how many files awaiting triage)
- Pending observations count
- Days since last triage
- Atlas size and growth

This prevents the most common knowledge system failure: gradual abandonment.
If the queue is growing and triage isn't happening, the morning routine
surfaces it before it becomes a problem.

---

## Philosophical Principles

### 1. Transformation, Not Creation

Agents don't create knowledge. They transform and recombine existing information.
Every "new" atlas file is a recombination of things that already existed.
Connecting and enriching existing knowledge is more valuable than adding new files.

### 2. Conventions Are the Safety Manual

The frontmatter schemas, the routing tree, the observation mechanism, the triage
process -- these aren't bureaucratic overhead. They are the safety system for
operating an opaque knowledge graph. When agents skip them, the system degrades
in specific, predictable ways.

### 3. Filter Before Read

With 2,400+ files, no agent can read everything. The `description` field,
frontmatter types, and directory structure exist to enable filtering. An agent
should be able to find the right 5 files out of 2,400 without reading the
other 2,395.

### 4. The Circle Must Hold

Containment boundaries (intake/atlas/domains, private/public, temporal/durable)
are structural requirements, not suggestions. More effort should go into
maintaining boundaries than into adding new content.

### 5. Hear the Hum

The system generates signals through its own structure -- gaps, contradictions,
surprising connections, friction. The observations mechanism exists to capture
these signals. A healthy knowledge system is one where agents notice and report
what's wrong, not just what's new.

---

## Implementation Checklist

If you're building a similar system, here's what to implement in order:

1. **Three-tier directory structure** (intake / permanent / deep domains)
2. **Frontmatter schema with `description` field** (the single highest-ROI change)
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

Items 1-5 give you 80% of the value. Items 6-13 are what make it compound
over time instead of decaying.

---

## What We Don't Have Yet

Documenting gaps honestly:

- **Adversarial verification**: We check structural health but don't deliberately
  try to break the system (e.g., "what if an agent hallucinated a fake entity?")
- **Confidence decay**: Knowledge doesn't automatically become less trusted over
  time. `last_verified` is a manual check, not a decay function.
- **Semantic search**: Discovery is wikilink-graph and text-search based. No
  embedding-based similarity search for finding conceptually related but
  differently-worded content.
- **Source attribution/compensation**: We track provenance but don't address the
  ethical question of compensating original sources whose work feeds the system.
- **Cross-vault federation**: The system is single-user. No mechanism for merging
  knowledge graphs across people or organizations.

---

## Credits and Influences

- **Obsidian** (obsidian.md): The substrate -- markdown, wikilinks, graph view
- **GTD** (David Allen): The action methodology integrated with knowledge
- **Ars Contexta** (arscontexta.com): Description fields, routing trees, the
  reweave concept (adapted from their "6 Rs pipeline")
- **Kieckhefer** (1994): The Seven Gates framework (adapted from "The Specific
  Rationality of Medieval Magic")
- **Syncthing**: Peer-to-peer sync that respects containment boundaries
- **Amplifier** (github.com/microsoft/amplifier): The multi-agent framework
  that operates the system

---

*This document describes a system in active daily use. It is opinionated,
battle-tested, and incomplete. If you build on these patterns, the most
important thing to get right is the routing decision tree and the description
field. Everything else follows from those two.*
