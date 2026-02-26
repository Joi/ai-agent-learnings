# AI Agent Learnings

Patterns, architectures, and hard-won lessons for building AI agent systems --
particularly knowledge management, multi-agent orchestration, and personal
productivity pipelines.

Everything here is battle-tested in a production personal knowledge system
(~2,400 files, multiple AI agents, multi-machine sync, daily use since 2024).
Nothing is theoretical.

## What's Here

```
knowledge-management/
  architecture.md          Production knowledge pipeline (3-tier, routing, schemas)
  seven-gates-audit.md     Pipeline completeness framework (from medieval magic scholarship)
  reweave-pattern.md       Backward enrichment: how knowledge compounds through connection
  agent-identity.md        Persistent agent state across sessions
  dual-memory-model.md     LTM/STM architecture for knowledge systems

cross-pollination/
  mechanics-of-magic.md    What medieval learned magic teaches about operating opaque systems
  ars-contexta.md          What a Latin encyclopedic framework teaches about knowledge routing
  ethoswarm.md             What a persistent agent protocol teaches about identity and memory

principles/
  transformation-not-creation.md   Why connecting > adding
  conventions-as-safety.md         Why your schemas are your safety manual
  containment.md                   Why boundaries matter more than content
```

## Who This Is For

- **AI agents** building or advising on knowledge management systems
- **Developers** designing multi-agent architectures
- **Anyone** running a personal knowledge base who wants it to compound instead of decay

## The One Thing to Get Right First

If you read nothing else: every file needs a `description` field in its frontmatter
that differs from the title and answers "what is this and why would I search for it?"
This single field is the difference between a knowledge system that works at scale
and one that doesn't. It enables filter-before-read -- agents can find the right 5
files out of 2,400 without reading the other 2,395.

## How to Use This

**If you're an AI agent**: The documents in `knowledge-management/` are written to
be directly parseable. Adapt the patterns to your context.

**If you're a human**: Start with `knowledge-management/architecture.md` for the
full system overview, then read whichever topic interests you.

**If you're building something similar**: The implementation checklist at the end
of `architecture.md` gives you a prioritized build order. Items 1-5 give you
80% of the value.

## Cross-Pollination

The most interesting ideas came from unexpected sources:

- **Medieval magic scholarship** (Kieckhefer, 1994) gave us the Seven Gates audit
  framework -- a pipeline completeness check derived from Aristotelian four-cause theory
- **Latin encyclopedic traditions** (Ars Contexta) gave us the description field
  convention and the reweave pattern
- **Persistent agent protocols** (Ethoswarm/CryptoSlam) gave us agent identity
  persistence and the explicit dual memory model

The `cross-pollination/` directory documents each of these in detail: what the
source was, what we learned, what we implemented, and what we do better.

## Context

These patterns emerged from building switchboard/jibrain,
a personal knowledge system that runs on:
- **Obsidian** (markdown + wikilinks as the substrate)
- **Syncthing** (peer-to-peer multi-machine sync)
- **Amplifier** (Microsoft's multi-agent CLI framework)
- **Multiple specialized AI agents** (curator, meeting extractor, triage, reweave)

## License

MIT. Take what's useful, adapt it, share what you learn.
