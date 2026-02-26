# Dual Memory Model: LTM/STM for Knowledge Systems

How to think about memory layers in an AI-operated knowledge system.

## The Model

Every knowledge system operated by AI agents has two memory layers, whether
or not they're explicitly named:

| Layer | Name | Persistence | Scope | Example |
|-------|------|------------|-------|---------|
| **LTM** | Long-Term Memory | Survives sessions | Entire system | The vault (files, links, schemas) |
| **STM** | Short-Term Memory | Session-scoped | Current agent | Context window, tool results, working state |

Between them sits a **consolidation mechanism** that promotes important STM
content to LTM -- analogous to how sleep consolidates short-term memories
into long-term storage.

## Mapping to a Real System

### LTM: The Vault

```
atlas/           # Permanent entities (concepts, people, organizations)
domains/         # Deep knowledge areas
agent state/     # Agent memories across sessions
```

Properties:
- Persists indefinitely
- Structured with schemas
- Searchable and linkable
- Shared across all agents and sessions

### STM: Session Context

```
Context window   # What the LLM can currently "see"
Tool results     # File reads, search results, API responses
Working state    # Todo lists, intermediate analysis
```

Properties:
- Exists only during active session
- Limited capacity (context window size)
- Not shared across agents (unless explicitly passed)
- Lost on session end (unless consolidated)

### Consolidation: The Intake Pipeline

```
Agent notices something during session (STM)
  → Writes to intake/ (begins consolidation)
    → Triage evaluates (quality gate)
      → Promotes to atlas/ (becomes LTM)
```

The intake pipeline IS the consolidation mechanism. It transforms volatile
session observations into permanent knowledge. Without it, everything an
agent learns during a session disappears at session end.

## Why Naming This Matters

Most knowledge systems have these layers implicitly. Making them explicit
helps agents understand their own architecture:

1. **Agents know what persists and what doesn't** -- "If I don't write this
   to intake/, it will be lost when this session ends."

2. **Consolidation becomes a deliberate act** -- Instead of hoping important
   observations make it to the vault, agents have a named process for it.

3. **Memory health can be monitored** -- Is consolidation happening? Is the
   STM→LTM ratio healthy? Is important context being lost?

4. **Memory tiers can have different trust levels** -- LTM content has been
   triaged and validated. STM content is raw and unverified.

## The Consolidation Decision

Not everything in STM should become LTM. The consolidation gate asks:

| Question | If Yes → | If No → |
|----------|----------|---------|
| Will this matter in 30 days? | Write to intake/ | Let it expire |
| Does this update existing LTM? | Update atlas/ file directly | -- |
| Is this a structural observation? | Write to observations/ | -- |
| Is this operational metadata? | Write to _review/ or discard | -- |

## Anti-Patterns

| Anti-Pattern | Problem | Fix |
|-------------|---------|-----|
| No consolidation | Everything learned in sessions is lost | Add intake pipeline |
| Over-consolidation | Every session dumps files into LTM | Add triage gate |
| Flat LTM | All memories in one bucket | Add tiers (intake/atlas/domains) |
| Unnamed layers | Agents don't know what persists | Name and document the layers |
| No STM→LTM tracking | Can't tell if consolidation is working | Monitor intake queue |

## Inspiration

- **Ethoswarm**: Explicit LTM/STM model for persistent AI Minds
- **Neuroscience**: Hippocampal consolidation (STM→LTM during sleep)
- **Database architecture**: Write-ahead log (STM) → committed storage (LTM)
- **GTD methodology**: Inbox (STM) → organized lists (LTM) via clarify/organize
