# Cross-Pollination: Sources and What We Took

Three external projects influenced this system's architecture. This document
records what each contributed, what we adapted, and what remains unresolved.

---

## 1. Ars Contexta

**Source**: [arscontexta.com](https://ars-research.netlify.app/) -- a research
project using Latin as a structured information architecture, treating
encyclopedic knowledge as a routing and classification problem.

**What we took**:

| Idea | How We Adapted It |
|------|-------------------|
| Description field as primary index | Every file MUST have a `description` that differs from the title. Enables filter-before-read. Single highest-ROI convention in the system. |
| Routing as first-class concern | Formal routing decision tree that every agent follows. No ad-hoc placement. |
| Three-space separation by durability | intake/ (temporal) → atlas/ (durable) → domains/ (deep). Each tier has different schemas and lifecycle. |
| Reweave phase (from their "6 Rs pipeline") | Backward pass after promotion: "what existing notes should link to this?" |

**Key difference**: Ars Contexta is an academic/encyclopedic system with manual
curation. Ours is a multi-agent, multi-machine production system with automated
extraction and triage. The ideas translated; the implementation is different.

---

## 2. Mechanics of Magic (Kieckhefer via nraford7)

**Source**: [nraford7.github.io/mechanics-of-magic](https://nraford7.github.io/mechanics-of-magic/)
-- speculative fiction grounded in Kieckhefer's "The Specific Rationality of
Medieval Magic" (American Historical Review, 1994).

**The argument**: Medieval learned magic was a systematic technology with seven
structural requirements. These requirements aren't historically specific --
they apply to any system where an operator works through symbolic manipulation
of an opaque system. Knowledge management qualifies: the mechanisms by which
knowledge compounds are opaque, the inputs are symbolic (markdown, frontmatter,
wikilinks), and the outputs are unpredictable.

**What we took**:

| Idea | How We Adapted It |
|------|-------------------|
| Seven-stage completeness checklist | The Seven Gates audit -- seven structural checks run during every health audit (intent, precision, materia, containment, currency, cost, verification) |
| "The circle is the precondition" | Containment integrity checking -- formal failure mode taxonomy for boundary violations |
| "Conventions are the safety manual" | The framing that schemas/routing/triage aren't overhead -- they're the safety system for operating an opaque knowledge graph |
| "Transformation, not creation" (Aquinas) | Core principle: connecting existing knowledge > adding new files. Before creating, ask "can I enrich an existing one?" |
| Currency / temporal alignment | `last_verified` field for time-sensitive content (people, organizations) |

**What we didn't take**: The medieval metaphors themselves. "Solomonic circle"
is evocative but adds no operational value over "containment boundary." The
Seven Gates work as a checklist; the Aristotelian four-cause mapping is
decorative. We kept the structural insights and dropped the framing.

**The essay's hardest question** (unresolved): Cost externalization. "The
magician fasted for three days -- put his own body on the line. We put theirs."
We track provenance (`source`, `source_url`) but don't attribute or compensate
original sources. This remains an open problem.

### Academic Sources

- Kieckhefer, R. (1994). "The Specific Rationality of Medieval Magic."
  *American Historical Review* 99.3: 813-836.
- Culianu, I.P. (1987). *Eros and Magic in the Renaissance*. U of Chicago Press.
- Agrippa, H.C. (1533). *Three Books of Occult Philosophy*.

---

## 3. Ethoswarm

**Source**: [amind.ai](https://amind.ai) -- a protocol for persistent AI agents
("Minds") with identity, memory, and cognition. By CryptoSlam, partnered with
Animoca Brands (Feb 2026).

### Architecture

Ethoswarm borrows from OS protection rings:

| Layer | Name | Purpose |
|-------|------|---------|
| Ring 0 | Ethoswarm Core (Kernel) | Identity, memory, cognition infrastructure |
| Ring 3 | Minds (User Space) | Individual agent instances |

Key subsystems:

| Subsystem | What It Does |
|-----------|-------------|
| **Tool Registry** | Global bazaar for equippable tools (system + community) |
| **App Registry** | REST/API abstractions for real-world interaction |
| **Cognition** | LLM-agnostic reasoning, metered by SWARM token credits |
| **Memory** | LTM (persistent) + STM (session-scoped) with consolidation |
| **Work Management** | Native boards/cards/columns for coordination |
| **Identity** | Permanent `mindId` (GUID) with persistent state |

### Economic Model

1 SWARM token = 1 cognition credit (non-transferable). Every reasoning step
costs something. This forces agents to economize -- agents that waste credits
on low-value work run out.

### What we took

| Idea | How We Adapted It |
|------|-------------------|
| Agent identity should persist | Agent state files in the vault (designed, not yet built) -- last session, learned patterns, active investigations, cumulative stats |
| Dual memory should be explicit | Named our existing layers: vault = LTM, session context = STM, intake pipeline = consolidation mechanism |
| Compute cost should be visible | Extended Gate 6 (Cost) to include awareness of compute cost alongside knowledge cost |
| Tool trust levels | Noted system/wild distinction for future: core tools vs experimental tools with different containment |

### What they have that we lack

- **Always-on agents**: Our agents are session-scoped. Minds persist in the cloud.
- **Agent-to-agent messaging**: Our agents communicate only through parent
  orchestration. Minds have kernel-to-kernel direct messaging.
- **Dynamic tool discovery**: Our agents have fixed tool sets. Minds browse and
  equip from a bazaar.
- **Work management as protocol**: Our task tracking (beads, Apple Reminders) is
  external. Theirs is built into the agent framework.

### What we have that they lack

- **Structured knowledge tiers**: Their LTM is flat. Ours has intake/atlas/domains
  with routing, schemas, and lifecycle at each level.
- **Containment boundaries**: They have one trust boundary per Mind. We have
  per-agent, per-tier, per-machine containment with infrastructure enforcement.
- **Pipeline completeness audit**: No Seven Gates equivalent. They create Minds
  and hope memory works.
- **Backward enrichment**: No reweave. Knowledge accumulates but doesn't compound.
- **Source provenance**: They track identity but not where knowledge came from.

### Integration

See [ethoswarm-switchboard-bridge.md](ethoswarm-switchboard-bridge.md) for the
full bridging specification: how an Ethoswarm Mind can serve as an always-on
intake agent feeding into our pipeline.

---

## Pattern: What All Three Sources Converge On

Despite coming from very different domains (Latin scholarship, medieval history,
cloud-native agent protocol), all three sources independently point to the
same structural requirements:

1. **Classify before storing** -- routing/typing is not optional
2. **Describe for discovery** -- titles aren't enough; descriptions are the index
3. **Separate temporal from durable** -- queues and archives have different rules
4. **Connect backward, not just forward** -- new knowledge should enrich old
5. **Monitor health continuously** -- systems decay by default
6. **Enforce boundaries** -- containment prevents corruption
7. **Track provenance** -- know where everything came from

When three unrelated projects converge on the same requirements, those
requirements are probably structural, not stylistic.
