# Agent Identity Persistence

How to give session-based agents continuity across runs.

## The Problem

Most agent frameworks (including ours) spawn agents fresh each session. The
agent inherits its instructions, tools, and context from configuration -- but
it has no memory of previous runs. It doesn't know:

- What it did last time
- What patterns it has noticed over many sessions
- What open questions or investigations it's tracking
- How its outputs have been received (were they useful? were they wrong?)

This means every session starts from zero context. The agent re-discovers
things it already knew. It re-makes mistakes it already learned from.

## The Pattern: Agent State Files

Give each agent a persistent state document in the knowledge base. The agent
reads this file at session start and writes updates at session end.

### Schema

```yaml
---
type: agent-state
agent: curator          # which agent this belongs to
last_active: 2026-02-26
session_count: 47
status: active
---
```

### Content Structure

```markdown
# Agent: Curator

## Last Session
- Date: 2026-02-26
- Actions: Extracted 3 URLs, created 3 intake files
- Duration: ~2 minutes
- Notes: Two URLs were paywalled, one extraction was thin

## Active Investigations
- [ ] Several recent extractions mention "agentic economy" -- potential concept cluster
- [ ] Person X appeared in 3 different extractions but has no atlas/ entity yet

## Learned Patterns
- News articles about partnerships produce low-value extractions (mostly PR copy)
- Academic papers produce high-value extractions but need more structured parsing
- Meeting transcripts with 3+ participants have higher entity density

## Cumulative Stats
- Total extractions: 342
- Promotion rate: 23% (79 promoted to atlas/)
- Most common types: concept (41%), person (28%), organization (19%)
- Average extraction quality (self-assessed): 3.2/5

## Open Questions
- Should we create a "partnership" entity type? Many extractions are about partnerships
  but they don't fit concept/person/org cleanly.
- Meeting extractions overlap with daily notes. Is there a deduplication strategy?
```

### Session Lifecycle

```
Session Start:
  1. Read agent state file
  2. Load active investigations and learned patterns
  3. Adjust behavior based on accumulated knowledge

Session Work:
  Normal agent operations, but with awareness of:
  - Previous patterns ("news articles are low-value, deprioritize")
  - Open questions ("should I create a partnership entity type?")
  - Active investigations ("look for more agentic economy references")

Session End:
  1. Update last session summary
  2. Add any new learned patterns
  3. Update or close active investigations
  4. Update cumulative stats
  5. Write agent state file back to vault
```

## Why This Works

Agent state files give session-based agents 80% of the value of persistent
agents without the infrastructure cost of always-on services:

| Capability | Always-On Agent | State File Agent |
|-----------|----------------|-----------------|
| Memory across sessions | Native | Via state file |
| Pattern recognition over time | Continuous | At session boundaries |
| Active investigations | Always tracking | Resumed at session start |
| Infrastructure cost | Continuous compute | Zero between sessions |
| Complexity | High (daemon, messaging, health checks) | Low (read file, write file) |

The key trade-off: always-on agents notice things between sessions (new data
arriving, external events). State file agents only notice things during active
sessions. For a personal knowledge system where sessions happen daily, this
trade-off is acceptable.

## Inspiration

- **Ethoswarm** (amind.ai): Persistent Mind identity with `mindId`, LTM, and
  profile telemetry -- showed us that agent continuity matters
- **Video game save files**: The simplest model of persistence -- serialize
  state, restore on load
- **Ops runbooks**: The pattern of "here's what the last operator learned"
  handed off between shifts
