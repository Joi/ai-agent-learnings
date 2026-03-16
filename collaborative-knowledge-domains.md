# Collaborative Knowledge Domains: A File-Based Architecture for Humans and AI Agents

*Joi Ito, March 2026*

---

## The Problem

I run a personal knowledge system with ~4,300 indexed files across an [Obsidian](https://obsidian.md/) vault. Multiple AI agents -- a [Telegram-based research mind](https://amind.ai/), a chat-capture agent, a meeting transcript extractor, and various Amplifier session agents -- continuously deposit knowledge into this system. I curate it, promote the good stuff, and discard the noise. It works.

But I want to collaborate with other humans. Specifically, I want to share a "medtech" research domain with a colleague and have both of us -- along with our respective AI agents -- contribute research, with quality control that doesn't require either of us to manually merge everything.

The requirements are strict:

1. **Local-first.** No cloud dependency. Data lives on our machines.
2. **Privacy-preserving.** My collaborator sees only the shared domain, not my personal notes, contacts, or meeting transcripts.
3. **Human-readable.** Plain Markdown files, editable in any text editor or Obsidian.
4. **Agent-native.** AI agents can contribute by writing files -- no APIs, no databases, no custom protocols.
5. **Asynchronous.** Contributors work on their own schedules across time zones.
6. **Quality-controlled.** Not everything that goes in stays in. Someone (or something) curates.

No existing system satisfies all six. I evaluated seven alternative architectures before concluding that the one I'd already built -- extended with a few missing pieces -- was the right approach.

---

## What I Built

The architecture has three layers: **intake**, **curation**, and **atlas**. It's a file-based pipeline synced with [Syncthing](https://syncthing.net/).

### The Folder Structure

```
domains/medtech/
  _conventions.md          # Domain rules, schemas, tag vocabulary
  INDEX.md                 # Auto-generated navigation (maintained by curator agent)
  atlas/                   # Promoted durable knowledge
    concepts/
    people/
    organizations/
    references/
  intake/                  # Per-contributor write queues
    joi/                   # My agent deposits
    gal/                   # My collaborator's deposits
    curator/               # Automated research agent deposits
  requests/                # Curator -> contributor asks
  _review/                 # Items needing human judgment
```

### How It Works

**Contributing** is simple: write a Markdown file with YAML frontmatter to your intake folder. Your AI agents can do the same. Syncthing propagates it to all participants within seconds.

```yaml
---
type: concept
description: "LNP formulation advances enabling tissue-targeted mRNA delivery beyond liver"
contributor: gal
source: paper
source_url: https://doi.org/10.1038/...
source_date: 2026-03-10
tags: [mRNA, drug-delivery]
confidence: high
status: draft
---

# Tissue-Targeted Lipid Nanoparticles

Content here...
```

**Curation** is handled by a maintainer agent that polls all intake folders and runs a five-phase pipeline:

1. **Collect** -- scan intake folders, dedup against a processed manifest
2. **Validate** -- check required frontmatter (description, source, contributor, tags)
3. **Detect conflicts** -- semantic search against existing atlas to find overlapping or contradictory content
4. **Promote** -- move clean files to `atlas/{type}/`, add promotion metadata
5. **Index** -- regenerate the domain's navigation hub so all contributors can discover what exists

Files that fail validation get bounced back as requests. Files that conflict with existing knowledge go to `_review/` with both versions preserved for human resolution.

**Discovery** works through a generated `INDEX.md` that the maintainer updates after every promotion cycle. Contributors check it before writing to avoid duplicating existing knowledge. They also check `requests/` for open research asks from the maintainer.

### The Sync Layer

[Syncthing](https://syncthing.net/) handles distribution. It's peer-to-peer, encrypted in transit, works offline, and requires no server. Each domain is a separate Syncthing shared folder, so collaborators see only the domains they participate in -- not your personal vault.

Syncthing's conflict handling is primitive (it creates `.sync-conflict-*` files when two people edit the same file before syncing), but this is acceptable because the intake pattern means contributors write to their *own* folders. Conflicts only arise if two people edit the same atlas file simultaneously, which the maintainer agent detects and routes to human review.

### Trust Levels

Not all contributors are equal:

| Source | Trust | Auto-Promote? |
|--------|-------|---------------|
| Named human contributor | High | Yes, if all validation gates pass |
| Automated research agent (e.g., Curator Mind) | Medium | Yes, if gates pass AND no semantic conflict |
| Chat-capture agent | Low | Never -- always routed to human review |
| Unknown source | None | Rejected |

This graduated trust is critical. AI agents produce volume but variable quality. Humans produce less but with higher signal. The pipeline treats them differently.

### Bidirectional Communication

The early versions of this system were write-only: contributors deposited, the maintainer curated. This created a gap -- the maintainer couldn't ask for clarification or request specific research. The `requests/` folder closes that gap. The maintainer posts structured requests:

```yaml
---
type: request
requested_by: maintainer-agent
target_contributors: [gal]
status: open
---

## What's Needed
The two recent papers on CRISPR delivery vectors reach opposite conclusions
about AAV9 tropism in cardiac tissue. Can you check the methodology sections?
```

Contributors (and their agents) check `requests/` as part of their workflow.

---

## Why This Architecture (and Not Seven Others)

I spent a session evaluating alternatives in depth, drawing on recent papers, production frameworks, and deployed systems. Here's what I found.

### 1. Shared-State Agent Frameworks (CrewAI, LangGraph, AutoGen)

[CrewAI](https://www.crewai.com/) has the most sophisticated in-process memory: five cognitive operations, active contradiction detection, composite scoring. [LangGraph](https://www.langchain.com/langgraph) offers typed state with reducer functions and time-travel debugging. [Microsoft's AutoGen](https://github.com/microsoft/autogen) provides conversational shared memory.

**Why not:** All three are single-process, single-machine. They solve agent-to-agent collaboration within a session, not distributed human+agent collaboration across machines and time zones. There's no story for "Gal writes something on his laptop in Tel Aviv and I see it in Tokyo."

### 2. Knowledge Graphs (Neo4j GraphRAG, KARMA Pipeline)

[Neo4j](https://neo4j.com/developer/genai-ecosystem/) with GraphRAG provides multi-hop reasoning across structured knowledge. The [KARMA paper](https://arxiv.org/abs/2502.06472) (2025) defines a pipeline of nine specialized agents -- including dedicated conflict resolution and evaluation agents -- for knowledge graph enrichment.

**Why not:** High setup complexity, requires explicit ontology design, and the human authoring experience is poor. You can't just open a text editor and write. KARMA's pipeline architecture, however, validated my staged-curation approach -- conflict resolution and evaluation should be explicit roles, not afterthoughts.

### 3. Notion 3.0 + MCP

[Notion](https://www.notion.com/) is the strongest overall competitor. Native multiplayer, AI agents via MCP, structured databases, zero setup.

**Why not:** Cloud-only, proprietary format, total vendor dependency, no data sovereignty. For personal knowledge that includes contacts, meeting notes, and research -- unacceptable.

### 4. CRDTs / Local-First Libraries (Automerge, Yjs)

[Automerge](https://automerge.org/) and [Yjs](https://github.com/yjs/yjs) guarantee mathematical convergence of concurrent edits. The [Ink & Switch](https://www.inkandswitch.com/essay/local-first/) essay on local-first software articulates seven ideals that align perfectly with my requirements.

**Why not:** CRDTs solve sync, not curation. "You get eventual consistency of data, not eventual quality of knowledge." I need both. The intake/promote pipeline IS the curation layer that CRDTs lack. That said, [Obsidian + Relay](https://relay.md/) (CRDT-based real-time Obsidian collaboration) could replace Syncthing for the sync layer and add live cursors -- worth watching.

### 5. Vector Databases as Shared Memory (Weaviate, Mem0)

[Mem0](https://mem0.ai/) (49.9k GitHub stars) is purpose-built for multi-agent memory with automatic compression and graph+vector hybrid retrieval. [Weaviate](https://weaviate.io/) offers native multi-tenancy with agent-focused APIs.

**Why not:** No human authoring interface. These are agent memory layers, not knowledge management systems. Gal can't open a text editor and browse what we know about mRNA delivery.

### 6. Event Streaming (Apache Kafka)

[Kafka](https://kafka.apache.org/) provides O(n) connectivity, temporal decoupling, guaranteed delivery, full audit trail. Enterprise-proven at extreme scale.

**Why not:** Massive over-engineering for two people and a few AI agents. My intake folder IS a poor man's event queue. It works at this scale.

### 7. Wiki + MCP (Wiki.js)

[Wiki.js](https://js.wiki/) with MCP server bridges gives humans a wiki UI and agents a structured API. Git-backed, self-hosted.

**Why not:** Requires running a server. Less flexible than plain files. No real-time collaboration. Closer to what I want but adds infrastructure I don't need.

### The Convergence

The striking finding: **the enterprise world is converging on the same pattern at much greater complexity.** Google Research's [Blackboard Multi-Agent Systems](https://research.google/pubs/blackboard-multi-agent-systems-for-information-discovery-in-data-science/) (2025) validated the blackboard architecture -- a shared workspace where agents write contributions and a control unit decides what gets promoted -- outperforming alternatives by 13-57%. The [LbMAS paper](https://arxiv.org/abs/2507.01701) (CAS Beijing, 2025) adds specialized roles: Planner, Critic, Conflict-resolver, Cleaner, Decider. The [DARPA Collaborative Knowledge Curation](https://www.darpa.mil/research/programs/ckc) program treats knowledge curation as an explicitly collaborative process between human domain experts and AI.

My domain folder IS the blackboard. Intake folders are write queues. The maintainer agent is the control unit. The atlas is the consensus state. I'm doing with Markdown files and Syncthing what they're doing with Kafka, Neo4j, and custom infrastructure.

---

## What's Actually Hard

### Semantic Conflict Resolution

When two contributors write about the same topic with different conclusions, the system needs to detect the overlap. Filename dedup is trivial. Semantic overlap detection -- "these two files are about the same mRNA delivery mechanism but reach different conclusions" -- requires embeddings or LLM comparison. I use [QMD](https://qmd.io/) (a local semantic search index over Markdown files) for this, but it's the least mature part of the pipeline.

The [UCSD paper on multi-agent memory](https://arxiv.org/abs/2603.10062) (March 2026) frames this as analogous to cache coherence in hardware -- but harder, because conflicts are semantic rather than bitwise. Nobody has fully solved this. My approach: detect and preserve both versions for human review rather than attempting automated resolution.

### The Curator Bottleneck

One maintainer agent per domain centralizes quality control. At 2-3 contributors this is fine. At 10+ it becomes a chokepoint. KARMA's approach -- multiple specialized agents in a pipeline (ingestion, extraction, alignment, conflict resolution, evaluation) -- distributes the load better. I'll cross that bridge when the domains grow.

### Discovery at Scale

The generated `INDEX.md` works for a few hundred files. At thousands, contributors need full-text and semantic search over the domain. QMD provides this locally, but there's no good story for giving a remote collaborator search access to files that live on my machine and sync via Syncthing. This is the strongest argument for eventually adding a shared search endpoint.

---

## The Stack

Everything I use is open-source or self-hostable:

| Component | What It Does | Link |
|-----------|-------------|------|
| **Obsidian** | Knowledge authoring & browsing | [obsidian.md](https://obsidian.md/) |
| **Syncthing** | Peer-to-peer file sync | [syncthing.net](https://syncthing.net/) |
| **Amplifier** | AI agent framework (orchestrates the maintainer and contributor agents) | [github.com/microsoft/amplifier](https://github.com/microsoft/amplifier) |
| **QMD** | Local semantic search over Markdown | [qmd.io](https://qmd.io/) |
| **amind.ai** | Telegram-based research agent (Curator Mind) | [amind.ai](https://amind.ai/) |
| **Sprites** | Persistent cloud sandboxes for running extraction services | [sprites.dev](https://sprites.dev/) |

The critical architectural choice: **plain Markdown files as the universal interface.** Every component reads and writes them. No database, no API gateway, no schema migration. An AI agent writes a `.md` file to an intake folder. A human writes one in Obsidian. Syncthing moves it. The maintainer agent reads it. QMD indexes it. They're all working on the same artifact in the same format.

---

## What I'd Tell Someone Starting From Scratch

1. **Start with intake/atlas, not a database.** The two-tier structure (incoming vs. promoted) is the minimum viable knowledge pipeline. Everything else is optimization.

2. **Per-contributor intake folders from day one.** Even if you're the only contributor now. The separation costs nothing and makes multi-contributor collaboration trivial to add later.

3. **Require a `description:` field in all files.** This is the single most important convention. It enables filter-before-read across thousands of files. Without it, every agent and every search has to open every file to understand what it contains.

4. **Syncthing over git for knowledge vaults.** Git is designed for source code: small diffs, merge conflicts, branching. Knowledge vaults have large binary files (images, PDFs), frequent small edits, and no meaningful "branches." Syncthing handles this naturally. Use git for code, Syncthing for knowledge.

5. **Trust levels are not optional.** The moment you have AI agents contributing, you need graduated trust. An automated chat-capture agent and a human domain expert should not have the same promotion path.

6. **The maintainer agent is the key role.** Not the contributors, not the search index, not the sync layer. The maintainer decides what gets promoted, what gets bounced, and what conflicts need human attention. Invest your design effort here.

7. **Acknowledge the gaps honestly.** Semantic conflict resolution is unsolved. Discovery at scale is hard. The curator bottleneck is real. Building a knowledge system that pretends these problems don't exist will fail when it grows. Building one that routes around them -- by preserving conflicts for human review, generating indexes, and keeping the maintainer pipeline modular -- will survive.

---

## References

- Palangi et al., "Blackboard Multi-Agent Systems for Information Discovery in Data Science," Google Research, 2025. [research.google](https://research.google/pubs/blackboard-multi-agent-systems-for-information-discovery-in-data-science/)
- Han & Zhang, "LbMAS: LLM Blackboard Multi-Agent System," CAS Beijing, 2025. [arXiv:2507.01701](https://arxiv.org/abs/2507.01701)
- "KARMA: Multi-Agent Knowledge Graph Enrichment," 2025. [arXiv:2502.06472](https://arxiv.org/abs/2502.06472)
- Yu et al., "Multi-Agent Memory as Computer Architecture," UCSD, March 2026. [arXiv:2603.10062](https://arxiv.org/abs/2603.10062)
- Kleppmann et al., "Local-First Software: You Own Your Data, in spite of the Cloud," Ink & Switch, 2019. [inkandswitch.com](https://www.inkandswitch.com/essay/local-first/)
- "Multi-Agent Collaboration Mechanisms: A Survey of LLMs," 2025. [arXiv:2501.06322](https://arxiv.org/abs/2501.06322)
- DARPA Collaborative Knowledge Curation (CKC) program. [darpa.mil](https://www.darpa.mil/research/programs/ckc)
- "Agentic Knowledge Base Patterns," The New Stack, March 2025. [thenewstack.io](https://thenewstack.io/agentic-knowledge-base-patterns/)

*This document is part of [ai-agent-learnings](https://github.com/Joi/ai-agent-learnings), a collection of patterns from running a production AI-operated knowledge management system.*
