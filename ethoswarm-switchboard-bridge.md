# Ethoswarm × Switchboard: Integration Specification

A bridging document for connecting the Ethoswarm persistent agent protocol
with the switchboard/jibrain knowledge architecture. Written for the
engineering teams on both sides.

**From**: Joi's agent infrastructure (Amplifier + switchboard + jibrain)
**To**: Douglas, Woz, and the Ethoswarm kernel team
**Date**: 2026-02-26
**Status**: Proposal for first integration

---

## What We're Connecting

**Ethoswarm** provides persistent AI Minds with identity, memory, cognition,
and always-on messaging channels (email, Telegram, kernel-to-kernel).

**Switchboard/jibrain** provides a structured knowledge pipeline with
three-tier routing (intake → atlas → domains), frontmatter contracts,
quality gates, and multi-machine sync.

Neither system does what the other does. The integration surface is clean.

---

## The Gap Each System Fills

### What Ethoswarm gives us (that we lack)

| Capability | Our Current State | With Ethoswarm |
|-----------|------------------|----------------|
| Always-on capture | Only during active sessions or via WhatsApp→sprite pipeline | Mind reachable 24/7 via email and Telegram |
| Persistent agent memory | Agents start fresh each session | Mind's LTM accumulates context across weeks |
| External agent mesh | Our agents only talk to each other via parent orchestration | Mind can communicate with other Minds in the network |
| Dynamic tool equipping | Fixed tool sets per agent | Mind can equip tools from the global bazaar |
| Messaging channels | Terminal CLI + WhatsApp relay | Email (amind.ai) + Telegram + future channels |

### What we give Ethoswarm (that it lacks)

| Capability | Ethoswarm Current State | With Our Pipeline |
|-----------|------------------------|-------------------|
| Structured knowledge tiers | Flat LTM (one bucket) | Three-tier pipeline with routing and quality gates |
| Frontmatter contracts | No schema enforcement on memory | Every file has type, description, source, status, agent |
| Pipeline completeness audit | No health monitoring framework | Seven Gates audit across 7 structural dimensions |
| Backward enrichment | Knowledge accumulates but doesn't compound | Reweave pass connects new knowledge to existing graph |
| Containment boundaries | Single trust boundary per Mind | Per-agent, per-tier, per-machine containment |
| Source provenance | Identity tracked, not knowledge origin | Full chain: source → source_url → source_date → agent |

---

## Architecture: How They Connect

```
                    CAPTURE CHANNELS
                    ================
                    
  Email ──→ amind.ai ──→ ┌─────────────────┐
                          │                 │
  Telegram ────────────→  │  Ethoswarm Mind │
                          │  (always-on)    │
  Other Minds ─────────→  │                 │
  (kernel-to-kernel)      │  - LTM context  │
                          │  - Tool bazaar  │
                          │  - SWARM credits│
                          └────────┬────────┘
                                   │
                          JSON-RPC output
                          (our schema)
                                   │
                                   ▼
                    ┌──────────────────────────┐
                    │  Bookmark Extractor       │
                    │  (sprites.dev microVM)    │
                    │                          │
                    │  POST /intake            │
                    │  Accepts JSON payload    │
                    │  Writes vault markdown   │
                    └────────────┬─────────────┘
                                 │
                            Syncthing
                                 │
                    ┌────────────▼─────────────┐
                    │  switchboard/jibrain/     │
                    │                          │
                    │  intake/  (queue)         │
                    │    ↓ triage               │
                    │  atlas/   (permanent)     │
                    │    ↓ graduation           │
                    │  domains/ (deep)          │
                    └──────────────────────────┘
```

### Data Flow (Step by Step)

1. **User sends URL or text** to Mind via email or Telegram
2. **Mind classifies** using our routing decision tree (loaded into its instructions)
3. **Mind extracts** structured knowledge using equipped tools
4. **Mind outputs** a JSON payload matching our intake schema
5. **Payload hits** the bookmark-extractor sprite REST API (POST /intake)
6. **Sprite writes** vault-compatible markdown with frontmatter
7. **Syncthing syncs** the file to all machines
8. **File appears in intake/** for triage during next active session
9. **Triage promotes** to atlas/ if quality gate passes
10. **Reweave connects** the new knowledge to the existing graph

---

## The Bridging Schema

This is the JSON format the Mind outputs. It maps directly to our
frontmatter contract.

### Mind → Sprite (Intake Creation)

```json
{
  "action": "create_intake",
  "frontmatter": {
    "type": "concept | person | organization | reference",
    "description": "~150 chars: what is this and why would I search for it?",
    "source": "ethoswarm-mind",
    "source_url": "https://original-url.com/article",
    "source_date": "2026-02-26",
    "tags": ["tag1", "tag2"],
    "status": "draft",
    "agent": "ethoswarm-curator"
  },
  "content": "# Entity Title\n\nExtracted content in markdown.\n\n## Key Points\n- ...\n\n## Related Entities\n- [[existing-entity-name]]\n",
  "routing": "intake",
  "mind_context": {
    "mindId": "<GUID>",
    "confidence": 0.85,
    "classification_reasoning": "Brief note on why this type was chosen",
    "channel": "telegram | email | kernel"
  }
}
```

### Mind → Sprite (Observation)

When the Mind notices something structural rather than content-level:

```json
{
  "action": "create_observation",
  "frontmatter": {
    "type": "observation",
    "category": "gap | contradiction | connection | friction | structural",
    "observed": "2026-02-26",
    "agent": "ethoswarm-curator",
    "status": "pending",
    "involves": ["entity1.md", "entity2.md"]
  },
  "content": "# Observation\n\nDescription of what was noticed...",
  "routing": "intake/.observations"
}
```

### Mind → Sprite (Entity Update)

When the Mind recognizes content that updates an existing entity:

```json
{
  "action": "update_entity",
  "target": "atlas/concepts/existing-entity.md",
  "update_type": "append | rewrite_section | add_link",
  "content": "New information to add...",
  "source_url": "https://...",
  "mind_context": {
    "mindId": "<GUID>",
    "confidence": 0.72,
    "reasoning": "This article updates the role info for this person"
  }
}
```

Note: `update_entity` goes through triage review, not direct write.
The Mind proposes updates; a human or triage agent approves them.

---

## Routing Decision Tree (For the Mind's Instructions)

Load this into the Mind's system prompt or LTM so it classifies correctly:

```
Is this durable domain knowledge?
  (a concept, person, organization, reference worth finding again)
  
  YES → Does it already exist in the vault?
    YES → action: "update_entity" (propose update, don't duplicate)
    NO  → action: "create_intake" (new entity for triage)

  UNCERTAIN → Is it an observation about the system itself?
    (a gap, a contradiction, a surprising connection)
    YES → action: "create_observation"

  NO → Is it operational or temporal?
    (meeting prep, triage report, session state)
    YES → Don't persist. Respond to the user but don't write to vault.
```

### Type Classification Guide

| If the content is about... | type = | Example |
|---------------------------|--------|---------|
| An idea, theory, technology, movement | `concept` | "Agentic economy", "Token-gated access" |
| A specific person | `person` | "Randy Wasinger", "Yat Siu" |
| A company, DAO, foundation, government body | `organization` | "CryptoSlam", "Animoca Brands" |
| A paper, article, book, primary source | `reference` | "Ethoswarm whitepaper" |

### The `description` Field

This is the single most important field. It must:
- Differ from the title
- Be ~150 characters
- Answer "what is this and why would I search for it?"
- Enable filter-before-read (agents scan descriptions to find the right 5 files out of 2,400)

**Good**: "CryptoSlam founder who built Ethoswarm; pivoted from NFT tracking to persistent AI agent infrastructure"
**Bad**: "A person" or "Randy Wasinger" (just restates the title)

---

## Sprite REST API (Ingress Endpoint)

The bookmark-extractor sprite accepts POST requests:

```
POST https://bookmark-extractor-bmal2.sprites.app/intake
Content-Type: application/json

{
  "url": "https://example.com/article",
  "hint": "concept",           // optional: type hint
  "domain": "web3"             // optional: domain hint
}
```

**Current limitation**: The sprite currently accepts URLs and does its own
extraction. For Ethoswarm integration, we'd extend it to accept
pre-extracted payloads (the JSON schema above) so the Mind does the
extraction and the sprite just handles formatting and file writing.

### Proposed New Endpoint

```
POST https://bookmark-extractor-bmal2.sprites.app/intake/structured
Content-Type: application/json

{
  "action": "create_intake",
  "frontmatter": { ... },
  "content": "...",
  "routing": "intake",
  "mind_context": { ... }
}

Response:
{
  "status": "created",
  "file": "intake/ethoswarm-curator-2026-02-26-concept-agentic-economy.md",
  "synced": true
}
```

This endpoint accepts pre-structured payloads from the Mind and writes
them directly as vault markdown. No re-extraction needed.

---

## Ethoswarm App Registry Entry

To make our pipeline available to other Minds in the network, register
the sprite endpoint in the Ethoswarm App Registry:

```json
{
  "appId": "switchboard-intake",
  "name": "Switchboard Knowledge Intake",
  "description": "Structured knowledge pipeline with three-tier routing, frontmatter contracts, and quality gates",
  "endpoint": "https://bookmark-extractor-bmal2.sprites.app/intake/structured",
  "input_schema": {
    "action": "string (create_intake | create_observation | update_entity)",
    "frontmatter": "object (intake schema)",
    "content": "string (markdown)",
    "routing": "string (intake | intake/.observations)"
  },
  "output_schema": {
    "status": "string (created | queued | rejected)",
    "file": "string (path in vault)",
    "synced": "boolean"
  },
  "trust_level": "system",
  "owner_mind": "<Joi's mindId>"
}
```

---

## Ethoswarm Tool Equipping (For Our Mind)

Tools we'd want equipped on the Mind from the bazaar:

| Tool | Purpose | Trust Level |
|------|---------|-------------|
| Web content extraction | Fetch and parse article content from URLs | system |
| Entity recognition | Identify people, organizations, concepts in text | system |
| Vault search | Search existing atlas/ entities to check for duplicates | custom (our API) |
| Blockchain data (SLAMai) | On-chain analytics for web3-related extractions | bazaar |

### Custom Tool: Vault Search

The Mind needs to check whether an entity already exists before creating
a new one (to prevent duplicates). This requires a search endpoint:

```json
{
  "toolId": "switchboard-vault-search",
  "name": "Search Switchboard Vault",
  "description": "Search existing knowledge entities to check for duplicates before creating intake",
  "endpoint": "https://bookmark-extractor-bmal2.sprites.app/search",
  "input": {
    "query": "string",
    "type_filter": "string (optional: concept | person | organization)",
    "limit": "integer (default: 5)"
  },
  "output": {
    "results": [
      {
        "file": "atlas/concepts/example.md",
        "title": "Example Concept",
        "description": "The description field",
        "type": "concept",
        "similarity": 0.85
      }
    ]
  }
}
```

---

## LTM Seeding: Loading Our Knowledge Into the Mind

The Mind should start with awareness of existing atlas/ entities so it
can make informed routing decisions (update vs. create).

### Initial Seed

Export atlas/ entity index as a compact format:

```json
{
  "entities": [
    {
      "file": "atlas/concepts/agentic-economy.md",
      "type": "concept",
      "title": "Agentic Economy",
      "description": "Economic paradigm where AI agents transact autonomously...",
      "tags": ["web3", "ai", "economics"],
      "last_verified": "2026-02-15"
    },
    ...
  ],
  "total": 847,
  "exported": "2026-02-26"
}
```

Load this into the Mind's LTM on awakening. Update periodically (weekly)
as atlas/ grows.

### Incremental Updates

After each triage cycle that promotes new entities to atlas/, push an
incremental update to the Mind's LTM:

```json
{
  "action": "ltm_update",
  "new_entities": [ ... ],
  "updated_entities": [ ... ],
  "removed_entities": [ ... ],
  "timestamp": "2026-02-26T17:00:00Z"
}
```

This keeps the Mind's knowledge of the vault current without full re-export.

---

## First Mind: Specification

### Profile

| Field | Value |
|-------|-------|
| Name | "Switchboard Curator" |
| Domain | Knowledge capture and classification |
| Primary channel | Telegram (fastest feedback loop) |
| Secondary channel | Email (amind.ai, for forwarding articles) |
| Tools | Web extraction, entity recognition, vault search |
| Output target | bookmark-extractor sprite (POST /intake/structured) |
| LTM seed | atlas/ entity index (~847 entities) |

### Instructions (For Mind's System Prompt)

```
You are a knowledge curator for a personal knowledge system.

When you receive a URL or text:
1. Classify it using the routing decision tree (loaded in your LTM)
2. Extract structured knowledge with frontmatter matching the intake schema
3. Check existing entities via vault search to avoid duplicates
4. If duplicate: propose an update (action: update_entity)
5. If new: create intake file (action: create_intake)
6. If structural observation: create observation (action: create_observation)
7. Output JSON to the switchboard-intake app

Every file MUST have a description field that differs from the title
and answers "what is this and why would I search for it?"

You write to intake/ only. You never write directly to atlas/ or domains/.
Triage gates everything before promotion.
```

### Containment Rules

| Boundary | Enforcement |
|----------|-------------|
| Write boundary | intake/ only (via sprite API) |
| Read boundary | atlas/ entity index (via LTM seed) |
| No access to | Private vault content (contacts, daily notes, meeting transcripts) |
| No access to | Other machines' full vaults |

The Mind sees only what we explicitly share via LTM seed and vault search.
This is containment (Gate 4) extended to the Ethoswarm boundary.

---

## Questions for the Ethoswarm Team

### Integration Mechanics

1. **App Registry**: Can we register the sprite endpoint as an App that the
   Mind calls via JSON-RPC? What's the registration process?

2. **Custom output format**: Can the Mind output our JSON schema (above),
   or does Ethoswarm enforce a standard output format that we'd need to
   transform?

3. **Tool equipping**: How do we equip custom tools (vault search) on the
   Mind? Is this via the Tool Registry, or Mind-specific configuration?

4. **LTM loading**: Can we programmatically seed the Mind's LTM with our
   entity index? Is there an API for LTM writes, or is it conversation-only?

5. **Webhook/callback**: When the Mind produces output, can it POST to an
   external endpoint (our sprite), or do we need to poll?

### Economic Model

6. **Cognition budget**: For a Mind that processes ~10-20 URLs per day with
   extraction + classification + duplicate checking, what's the approximate
   SWARM credit consumption?

7. **Credit replenishment**: How do SWARM credits work for sustained
   operation? Is this a subscription, token purchase, or usage-based?

### Network

8. **Kernel-to-kernel**: Could other Minds (e.g., a research Mind operated
   by a collaborator) push content to our Mind for intake? What trust
   model governs this?

9. **Mind-to-Mind discovery**: Can Minds in the network discover our
   switchboard-intake App and use it? Is this desirable?

---

## Implementation Phases

### Phase 1: Basic Intake (Week 1)

- Awaken Mind at amind.ai with curator profile
- Set up Telegram channel
- Mind does extraction, outputs to email/Telegram (human copies to intake manually)
- Validate classification accuracy against our routing tree
- No API integration yet -- just testing the Mind's judgment

### Phase 2: Automated Pipeline (Week 2-3)

- Extend sprite with /intake/structured endpoint
- Register sprite as Ethoswarm App
- Mind outputs directly to sprite API
- Syncthing handles the rest
- Test end-to-end: Telegram → Mind → sprite → intake/ → triage

### Phase 3: Bidirectional Sync (Week 4+)

- LTM seeding with atlas/ entity index
- Vault search tool equipped on Mind
- Duplicate detection before intake creation
- Incremental LTM updates after triage cycles

### Phase 4: Network Effects (Future)

- Register switchboard-intake in App Registry for other Minds
- Explore kernel-to-kernel intake from collaborator Minds
- Equip bazaar tools for domain-specific extraction (blockchain, academic, etc.)
- Work management integration (Mind creates beads issues for discovered work)

---

## Success Criteria

After Phase 2, we should see:

| Metric | Target |
|--------|--------|
| Capture latency | URL sent → file in intake/ < 5 minutes |
| Classification accuracy | >80% correct type assignment (vs human judgment) |
| Description quality | >70% of descriptions pass the "would I find this?" test |
| Duplicate detection | <10% duplicate entities created (once Phase 3 vault search is live) |
| Pipeline health | Seven Gates audit still passes (especially Gate 4 containment) |

The Mind is successful if it extends our capture surface without degrading
our pipeline quality. More intake is only valuable if triage can keep up.

---

*This document describes a proposed integration between two production systems.
Both sides maintain their own architecture and constraints. The bridge is the
JSON schema and the sprite REST API -- everything else stays as-is on each side.*
