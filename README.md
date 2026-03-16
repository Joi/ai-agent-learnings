# AI Agent Learnings

Patterns for building AI-operated knowledge management systems, extracted from
a production system (~4,300 indexed files, multiple agents, multiple machines,
daily use since 2024).

*Last updated: 2026-03-13*

## Documents

| Document | What It Covers |
|----------|---------------|
| [architecture.md](architecture.md) | The complete system: three-tier pipeline, routing, frontmatter contracts, containment, reweave, Seven Gates audit, multi-agent boundaries, heartbeat triage, search infrastructure, LLM routing, implementation checklist |
| [knowledge-systems-audit.md](knowledge-systems-audit.md) | Full system audit: component inventory, retrieval systems, sync topology, integration matrix, data flow gaps, Python module status, design doc registry |
| [tiered-access-design.md](tiered-access-design.md) | Multi-agent security model: four access tiers, signal group architecture, OpenClaw routing, document airgap strategy, migration phases |
| [cross-pollination.md](cross-pollination.md) | What we learned from four external projects (Ars Contexta, Mechanics of Magic, Ethoswarm, Colin Raney & Fred) and what we adapted |
| [ethoswarm-switchboard-bridge.md](ethoswarm-switchboard-bridge.md) | Integration spec for connecting Ethoswarm persistent agents with our knowledge pipeline (Phase 1 live, Phase 2 in progress) |
| [collaborative-knowledge-domains.md](collaborative-knowledge-domains.md) | Architecture for multi-human, multi-agent collaborative knowledge domains using file-based intake/curation/atlas pipeline synced with Syncthing -- evaluated against 7 alternative architectures |

## If You Only Read One Thing

Read [architecture.md](architecture.md). It contains everything: the pipeline,
the principles, the schemas, the audit framework, and the implementation
checklist. The other documents are supplementary.

## System Status (2026-03-13)

The system is **OPERATIONAL** with four major infrastructure components deployed:

| Component | Status | What It Does |
|-----------|--------|-------------|
| **Heartbeat** | Live (deployed today) | Automated 15-min collection and triage cycle |
| **Ethoswarm Curator Mind** | Live | Always-on knowledge intake via Telegram |
| **QMD Search** | Live (~4,300 files) | Three-mode search: keyword, vector, deep |
| **iblai-router** | Live (since Mar 4) | Intelligent LLM routing, 80% cost savings |

## The System

Built on:
- **Obsidian** (markdown + wikilinks) as the knowledge substrate
- **Syncthing** (peer-to-peer) for multi-machine sync with containment boundaries
- **Amplifier** (github.com/microsoft/amplifier) as the multi-agent framework
- **GTD** (Getting Things Done) for connecting knowledge to action
- **QMD** for three-mode search (keyword, vector, deep) over indexed collections
- **sprites.dev** for cloud microVM infrastructure (knowledge-intake sprite)

## Who This Is For

- **AI agents** building or advising on knowledge systems (documents are agent-ingestible)
- **Developers** building multi-agent systems that need structured knowledge
- **Anyone** running a personal knowledge management system at scale
