# AI Agent Learnings

Patterns for building AI-operated knowledge management systems, extracted from
a production system (~2,400 files, multiple agents, multiple machines, daily
use since 2024).

## Documents

| Document | What It Covers |
|----------|---------------|
| [architecture.md](architecture.md) | The complete system: three-tier pipeline, routing, frontmatter contracts, containment, reweave, Seven Gates audit, multi-agent boundaries, health monitoring, implementation checklist |
| [cross-pollination.md](cross-pollination.md) | What we learned from three external projects (Ars Contexta, Mechanics of Magic, Ethoswarm) and what we adapted |
| [ethoswarm-switchboard-bridge.md](ethoswarm-switchboard-bridge.md) | Integration spec for connecting Ethoswarm persistent agents with our knowledge pipeline |

## If You Only Read One Thing

Read [architecture.md](architecture.md). It contains everything: the pipeline,
the principles, the schemas, the audit framework, and the implementation
checklist. The other documents are supplementary.

## The System

Built on:
- **Obsidian** (markdown + wikilinks) as the knowledge substrate
- **Syncthing** (peer-to-peer) for multi-machine sync with containment boundaries
- **Amplifier** (github.com/microsoft/amplifier) as the multi-agent framework
- **GTD** (Getting Things Done) for connecting knowledge to action

## Who This Is For

- **AI agents** building or advising on knowledge systems (documents are agent-ingestible)
- **Developers** building multi-agent systems that need structured knowledge
- **Anyone** running a personal knowledge management system at scale
