# Ethoswarm: Cross-Pollination

What a cloud-native persistent agent protocol teaches about identity,
memory architecture, and economic models for AI agents.

## Source

[Ethoswarm](https://amind.ai) by CryptoSlam -- a protocol for persistent
AI agents ("Minds") with their own identity, memory, and cognition.
Partnered with Animoca Brands to launch Animoca Minds (Feb 2026).

Technical manifest (Artifact ID: AE6B1C07-E612-F111-AD1D-0EA9A5017E89)
analyzed against our production knowledge system.

## The Architecture

Ethoswarm borrows from OS protection rings:

| Layer | Name | Purpose |
|-------|------|---------|
| Ring 0 | Ethoswarm Core (Kernel) | Identity, memory, cognition infrastructure |
| Ring 3 | Minds (User Space) | Individual agent instances |

**Topology**: Distributed mesh with centralized registry.

**Messaging channels**: Email (amind.ai), Telegram, direct kernel-to-kernel
(restricted to trusted circle).

### Key Subsystems

| Subsystem | What It Does |
|-----------|-------------|
| **Tool Registry** | Global "bazaar" for equippable tools -- both system tools and "wild tools" (community-contributed) |
| **App Registry** | REST/API abstractions for real-world interaction (services, blockchain, etc.) |
| **Cognition** | LLM-agnostic reasoning, metered by SWARM token credits |
| **Memory** | Dual-layer: LTM (Long-Term Memory, persistent) + STM (Contextual Stream, session-scoped) |
| **Work Management** | Native boards/cards/columns for agent-to-agent and agent-to-human coordination |

### Economic Model

1 SWARM token = 1 cognition credit (non-transferable). Every reasoning step
costs something. This isn't just billing -- it's a structural constraint that
forces agents to be economical with compute. Agents that waste cognition
credits on low-value work run out.

### Identity Model

Each Mind gets a permanent `mindId` (GUID) with:
- Profile metadata
- Headline telemetry
- Persistent memory state
- Tool/app equipment loadout

The Mind persists in the cloud -- always on, always reachable. Not a session
that starts and stops, but a standing entity with continuity.

## What We Learned

### 1. Agent Identity Should Persist Across Sessions

**Ethoswarm pattern**: Every Mind has a permanent `mindId`. It accumulates
memory, preferences, and operational history. It doesn't start fresh each
time -- it resumes.

**Our gap**: Our agents (curator, meeting-extractor, triage) are ephemeral.
Each session spawns them fresh. They inherit instructions from their bundle
definitions but have no memory of previous runs. The curator doesn't remember
what it extracted last week or what patterns it noticed.

**What we implemented**: Agent identity files in the vault. Each agent now has
a persistent state document that tracks:
- What it last did and when
- Patterns it has noticed across sessions
- Active investigations or open questions
- Cumulative statistics

This isn't full Ethoswarm-style persistent identity (we don't have always-on
agents), but it gives session-based agents continuity between runs.

### 2. Dual Memory Architecture (LTM/STM) Should Be Explicit

**Ethoswarm pattern**: Explicit separation between Long-Term Memory (persists
forever) and Short-Term Memory / Contextual Stream (session-scoped). With a
consolidation mechanism: important STM gets promoted to LTM.

**Our system**: We already have this, but we never named it or formalized
the consolidation:
- **LTM** = the vault (atlas/, domains/, agent state files)
- **STM** = session context (Amplifier context window, tool results)
- **Consolidation** = agent observations, intake files, triage

**What we clarified**: Named and documented the dual memory model explicitly.
The vault IS our LTM. Session context IS our STM. The intake pipeline IS our
consolidation mechanism. Making this explicit helps agents understand their
own architecture.

### 3. Compute Cost Should Be Visible (Not Just Knowledge Cost)

**Ethoswarm pattern**: 1 SWARM = 1 cognition credit. Every reasoning step has
a cost. The agent can see its balance and must economize.

**Our system**: Gate 6 (Cost) in the Seven Gates tracks the knowledge cost
(intake-to-atlas ratio: are we refining or just accumulating?) but not the
compute cost (how many API calls, tokens, and agent-hours are we spending?).

**What we implemented**: Extended Gate 6 to include compute cost awareness.
Not token-level metering (that's infrastructure), but awareness of the
cost/value ratio: "This agent session consumed X compute to produce Y
knowledge artifacts. Is that ratio healthy?"

### 4. Tool Discovery Should Be Dynamic

**Ethoswarm pattern**: A "bazaar" of equippable tools. Agents can browse
available tools and equip what they need. Includes both "system tools"
(platform-provided) and "wild tools" (community-contributed, less trusted).

**Our system**: Agents have fixed tool sets defined in their bundle
configurations. The curator always has the same tools. There's no discovery
or dynamic equipping.

**What we noted (not yet implemented)**: The system/wild tool distinction is
interesting -- trusted tools vs. community tools with different trust levels.
This maps to a pattern we could adopt: core tools (vault operations, GTD)
vs. experimental tools (new extractors, untested integrations) with
different containment boundaries.

### 5. Work Management as First-Class Citizen

**Ethoswarm pattern**: Native boards/cards/columns built into the agent
framework. Agents coordinate through structured work items, not just
message passing.

**Our system**: We have beads (issue tracking) and Apple Reminders (GTD
tasks), but they're external to the agent framework. Agents can read/write
them via tools, but work management isn't part of the agent protocol itself.

**What we noted**: The tight integration between agent cognition and work
management is worth considering. When an agent creates a sub-task, it
shouldn't have to "call the beads tool" -- task management should be as
natural as memory access.

## What We Do Better

### 1. Structured Knowledge Pipeline vs. Flat Memory

Ethoswarm has LTM and STM. That's two buckets. We have:

```
intake/ (temporal, disposable, queue semantics)
  → triage (quality gate)
    → atlas/ (durable, typed, schema-enforced)
      → domains/ (deep, specialized, graduated)
```

Four tiers with explicit routing, schemas, and lifecycle at each level.
Ethoswarm's LTM is a flat store. Our LTM has internal structure that
prevents the most common failure modes (search pollution, boundary
violations, stale knowledge).

### 2. Containment Boundaries

Ethoswarm runs in the cloud with a single trust boundary (the Mind has
access to everything it's been given). We have:

- Per-agent read/write boundaries (curator writes to intake/, not atlas/)
- Per-machine sync boundaries (agent server gets jibrain/ only, not contacts)
- Per-tier containment (temporal stays out of durable spaces)
- Infrastructure-level enforcement (.stignore, not just convention)

This is defense in depth. No single agent failure can corrupt the entire system.

### 3. The Seven Gates Audit

Ethoswarm has no pipeline completeness framework. You create a Mind, give it
memory, and hope it works. We audit seven structural dimensions continuously:
intent, precision, materia, containment, currency, cost, and verification.

This is the difference between "we have memory" and "we know our memory
system is healthy."

### 4. Backward Enrichment (Reweave)

Ethoswarm agents accumulate knowledge but don't have a mechanism for new
knowledge to strengthen existing knowledge. Our reweave pass -- "now that
this exists, what should connect to it?" -- is how knowledge compounds
instead of just accumulating.

### 5. Observation Mechanism

Ethoswarm agents have no way to report structural concerns about their
own system. Our observation mechanism (gap, contradiction, connection,
friction, structural) captures signals that the system generates during
normal operation. These are the immune system of the knowledge base.

### 6. Source Provenance

Ethoswarm tracks identity but not source provenance for knowledge.
Where did this information come from? When? From which agent? Our
frontmatter schema (source, source_url, source_date, agent) enables
tracing any claim back to its origin.

## Implementation Checklist (For Ethoswarm-Style Systems)

If you're building a persistent agent platform and want to adopt our patterns:

1. **Add knowledge tiers** -- don't use flat LTM. Separate temporal from durable.
2. **Add frontmatter schemas** -- every memory entry needs type, description, source.
3. **Add containment audit** -- check that boundaries hold, not just that data exists.
4. **Add a reweave mechanism** -- new knowledge should enrich old knowledge.
5. **Add observations** -- let agents report structural concerns about the system.
6. **Add source provenance** -- track where every piece of knowledge came from.

## Implementation Checklist (For Our System, From Ethoswarm)

What we took from Ethoswarm and implemented:

1. **Agent identity files** -- persistent state across sessions
2. **Named dual memory model** -- explicitly documented LTM/STM/consolidation
3. **Extended Gate 6** -- compute cost awareness alongside knowledge cost
4. **System/wild tool distinction** -- noted for future implementation

What we noted for future consideration:

5. Work management as part of agent protocol (not external tool)
6. Dynamic tool discovery and equipping
7. Always-on agents (not just session-scoped)
8. Agent-to-agent direct messaging (not just parent orchestration)

## Sources

- Technical manifest: Artifact ID AE6B1C07-E612-F111-AD1D-0EA9A5017E89
- Animoca Brands press release (Feb 5, 2026)
- amind.ai -- Ethoswarm Mind deployment platform
- SLAMai/SWARM token documentation (theswarm.slamai.xyz)
- CryptoSlam API portal (cryptoslam.apiable.io)
