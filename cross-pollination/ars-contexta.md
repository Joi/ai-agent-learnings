# Ars Contexta: Cross-Pollination

What a Latin encyclopedic research framework teaches about knowledge routing,
description fields, and backward enrichment.

## Source

[Ars Contexta](https://ars-research.netlify.app/) -- a research project using
Latin as a structured information architecture, treating encyclopedic knowledge
as a routing and classification problem.

## Key Ideas We Adopted

### 1. The Description Field as Primary Index

Ars Contexta treats every entry's description as the primary discovery mechanism --
not the title, not tags, not the full text. The description must answer: "what is
this and why would I search for it?"

**What we implemented**: Every file in our system MUST have a `description` field
that differs from the title. This enables filter-before-read: agents scan
descriptions to decide which files to open, without loading all content.

This is the single highest-ROI convention in the entire system.

### 2. Routing as a First-Class Concern

Ars Contexta doesn't dump content into a single bucket. It routes content through
a decision tree based on content type, durability, and intended audience.

**What we implemented**: A formal routing decision tree that every agent follows.
Three-tier architecture (intake / atlas / domains) with explicit rules for what
goes where. No ad-hoc placement decisions.

### 3. The Reweave Phase (from the "6 Rs Pipeline")

Ars Contexta's pipeline includes a phase where new knowledge is woven back into
existing context -- not just stored alongside it.

**What we implemented**: The reweave pattern -- a backward pass after promotion
that asks "what existing notes should link to or be updated by this new content?"
This is how knowledge compounds through connection.

### 4. Three-Space Separation

Ars Contexta separates content into distinct spaces based on durability and purpose.

**What we implemented**: intake/ (temporal, disposable), atlas/ (durable, permanent),
domains/ (deep, specialized). Each tier has different schemas, different expectations,
different lifecycle.

## What We Do Differently

| Ars Contexta | Switchboard/jibrain |
|-------------|-------------------|
| Latin as structured language | YAML frontmatter as structured schema |
| Academic/encyclopedic focus | Personal knowledge + meeting notes + bookmarks |
| Single-user research | Multi-agent, multi-machine production system |
| Manual curation | Agent-assisted extraction, triage, and reweave |
| No sync/distribution model | Syncthing with containment boundaries per machine |
| Static site | Live Obsidian vault with graph visualization |

## The Key Takeaway

The description field and the routing decision tree. Everything else follows from
getting those two things right. Ars Contexta showed us that knowledge management
is fundamentally a routing and classification problem, not a storage problem.
