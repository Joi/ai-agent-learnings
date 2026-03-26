# Multi-Interface Knowledge Layer

*March 2026*

How to project a personal knowledge vault into a team-facing system where
multiple interfaces (a document platform, a chat bot, Slack, email) all
feed events into a single normalized pipeline -- without duplicating policy
logic or losing cross-channel identity.

---

## The Problem

A personal vault with thousands of files is powerful for the owner, but
opaque to collaborators. Teams need:

1. A browsable, searchable document interface (we used [Onyx](https://github.com/onyx-dot-app/onyx), an open-source RAG platform)
2. A chat bot that can answer questions about the corpus
3. Slack integration for async collaboration
4. Email for review comments and approvals

Each interface generates events: someone reads a document, posts a comment,
approves a change. These events must flow into a single audit trail and
policy engine without each interface reimplementing access control, dedup,
or identity resolution.

---

## Architecture: The Envelope Pattern

Every event from every interface is normalized into a `NormalizedEnvelope`
before anything else touches it:

```
Interface          Normalizer           Envelope              Policy Engine
─────────          ──────────           ────────              ─────────────
Onyx event    →    onyx_normalizer  →   NormalizedEnvelope →  check access
Jibot message →    jibot_normalizer →   NormalizedEnvelope →  check access
Slack message →    slack_normalizer →   NormalizedEnvelope →  check access
Email         →    email_normalizer →   NormalizedEnvelope →  check access
```

### The Envelope Schema

```python
class NormalizedEnvelope:
    envelope_id: str          # Unique per event instance
    channel: Channel          # ONYX | JIBOT | SLACK | EMAIL (enum)
    actor_id: str             # Canonical identity (resolved from any interface handle)
    action: str               # read | write | approve | review_comment | ...
    artifact_id: str          # What document/artifact was touched
    thread_id: str            # Conversation threading (interface-specific)
    dedupe_fingerprint: str   # SHA-256 of (channel + thread + actor + action + target)
    payload: dict             # Interface-specific data (email subject+body, Slack text, etc.)
    received_at: datetime     # When the event was received
    created_at: datetime      # When the envelope was created
```

**Key design decisions:**

- `channel` is part of the fingerprint, so the same person doing the same
  action via Onyx and Slack produces two distinct events (both are recorded)
- `actor_id` is always the *canonical* identity, never the interface handle
- `thread_id` is interface-specific: Slack uses `channel_id:thread_ts`,
  email uses the RFC 2822 thread root, the document platform uses its own thread IDs
- `payload` is a dict, not a typed class -- each interface puts what it needs
  there; the policy engine doesn't look inside it

---

## Cross-Interface Identity Resolution

People appear differently on each interface:

| Interface | Handle | Same Person? |
|-----------|--------|-------------|
| Document platform | `joi@example.org` (Google OAuth) | Yes |
| Chat bot | `joi` (handle) | Yes |
| Slack | `UJOI0001` (Slack user ID) | Yes |
| Email | `joi@example.org` or `joi@personal.com` | Yes |

### ActorIdentities + ActorResolver

Each person gets a YAML file with all their handles:

```yaml
# actors/joi.yaml
actor_id: joi
display_name: Joi
identities:
  google_email: joi@example.org
  slack_user_id: UJOI0001
  jibot_handle: joi
  email_addresses:
    - joi@example.org
    - joi@personal.com
domains:
  - main
  - research
roles:
  - owner
```

The `ActorResolver` builds reverse-lookup indexes on load:

```python
resolver = ActorResolver(actors_dir=Path("actors/"))
resolver.resolve_by_google_email("joi@example.org")  # → ActorRecord(actor_id="joi")
resolver.resolve_by_slack_user_id("UJOI0001")         # → same
resolver.resolve_by_email("joi@personal.com")          # → same (secondary email)
resolver.resolve_by_jibot_handle("joi")                # → same
```

**Pattern**: Build the index once at startup, then resolution is O(1) dict
lookups. Each adapter calls the appropriate `resolve_by_*` method before
normalizing, so the envelope always has the canonical `actor_id`.

---

## Cross-Channel Deduplication

### Why Channel is Part of the Fingerprint

We considered making the fingerprint channel-agnostic (so the same action via
any interface would dedupe). We rejected this because:

1. **Audit trail**: You want to know *which* interface was used
2. **Different payloads**: A Slack message about a document and an Onyx read
   of the same document are semantically different events
3. **Rate limiting**: Per-channel rate limits make more sense than global ones

### How It Works

```python
fingerprint = sha256(f"{channel}:{thread_id}:{actor_id}:{action}:{target}")
```

The `DedupeRegistry` is a persistent set backed by the filesystem:

```python
registry = DedupeRegistry(store_dir=Path("dedupe/"))
if registry.is_new(envelope.dedupe_fingerprint):
    registry.mark_seen(envelope.dedupe_fingerprint)
    process(envelope)  # First time seeing this event
else:
    log_duplicate(envelope)  # Skip
```

Persistence means a server restart doesn't replay already-processed events.

---

## The Adapter Pattern

Every interface adapter follows the same structure:

```
adapters/
  onyx/
    __init__.py
    normalizer.py    # normalize_onyx_event(raw, resolved_actor_id) → NormalizedEnvelope
    adapter.py       # OnyxAdapter.process_event(raw, resolved_actor_id) → NormalizedEnvelope
  jibot/
    __init__.py
    normalizer.py    # normalize_jibot_event(raw, resolved_actor_id) → NormalizedEnvelope
    adapter.py       # JibotAdapter.process_event(raw, resolved_actor_id) → NormalizedEnvelope
  slack/
    ...
  email/
    ...
```

### Normalizer Rules

1. **No policy logic** in normalizers -- they translate raw events, nothing more
2. **Actor resolution happens before the normalizer** -- the adapter receives
   `resolved_actor_id`, not a raw handle
3. **Interface-specific quirks stay in the normalizer** -- Slack's
   `channel_id:thread_ts` threading, email's thread-ID-fallback-to-message-ID
4. **Fingerprint computation is shared** -- all normalizers call the same
   `compute_fingerprint()` function

### The Adapter is a Thin Wrapper

```python
class SlackAdapter:
    def process_event(self, raw_event, resolved_actor_id) -> NormalizedEnvelope:
        return normalize_slack_event(raw_event, resolved_actor_id)
```

Why have the wrapper at all? It's a stable entry point for dependency injection,
middleware, and future concerns (logging, metrics) without touching the
normalizer's pure-function signature.

---

## Policy Engine Separation

The policy engine receives `NormalizedEnvelope` objects and makes decisions:

```python
class DomainAccessPolicy:
    def check_access(self, actor: ActorRecord, artifact_path: str) -> bool
    def allowed_domains(self, actor: ActorRecord) -> list[str]

class ApprovalPolicy:
    def requires_approval(self, artifact_path: str) -> bool
    def check_approval(self, artifact_path: str) -> ApprovalStatus
```

**Critical separation**: Adapters never call policy. Policy never knows which
interface the event came from. The envelope is the contract between them.

This means adding a new interface (say, a mobile app) requires only:
1. A new normalizer
2. A new adapter (thin wrapper)
3. An entry in the Channel enum

Zero changes to policy, dedup, or identity resolution.

---

## Vault Projection Pipeline

The personal vault contains far more than what should be visible to the team.
A "projection" pipeline selects and transforms content:

```
Personal Vault (thousands of files)
    │
    ▼
Projection Runner
    │ - Filter by domain/workstream
    │ - Strip confidential frontmatter fields
    │ - Skip files > 1MB (binary artifacts)
    │ - Emit ProjectionRecord per file
    ▼
Projected Corpus (team-visible subset)
    │
    ▼
Document Platform (Onyx, etc.)
```

### ProjectionRecord

```python
class ProjectionRecord:
    source_path: str        # Where it lives in the vault
    projected_path: str     # Where it appears in the team corpus
    content_hash: str       # For change detection
    projected_at: datetime  # When this projection was made
    metadata: dict          # Frontmatter that's safe to share
```

**Key insight**: The projection is one-way. Team members read projected content
through the document platform. Changes flow back as events (review comments,
approvals) through the adapter pipeline. The vault owner decides what to
incorporate.

---

## Deploying Onyx as the Document Interface

[Onyx](https://github.com/onyx-dot-app/onyx) is an open-source RAG platform
(formerly Danswer) that provides document search, chat, and agent capabilities.

### What We Learned

1. **Use vanilla upstream** -- no custom Docker images, no fork. All
   customization happens through environment variables and the admin panel.

2. **Key `.env` settings**:
   ```
   AUTH_TYPE=google_oauth          # Google SSO for your org domain
   GEN_AI_MODEL_PROVIDER=anthropic # Or openai, etc.
   GEN_AI_MODEL_VERSION=claude-sonnet-4-20250514
   ```

3. **Agent personas** are the main UX customization point. You define them
   as system prompts and paste them into the Onyx admin panel. Store them
   as markdown files in your repo for version control.

4. **Document connectors** (file upload, web scrape, etc.) are configured
   in the admin panel, not in code. The corpus preparation script runs
   locally, then you upload the projected files.

5. **The admin panel is the UI** -- there's no theme system, no CSS
   overrides, no template engine. If you want to change the interface
   significantly, you're forking the frontend (Next.js/React).

### Onyx Customization Map

| What You Want to Change | Where to Change It |
|------------------------|-------------------|
| Who can log in | `.env` → `AUTH_TYPE`, `GOOGLE_CLIENT_ID` |
| Which LLM | `.env` → `GEN_AI_MODEL_PROVIDER`, `GEN_AI_MODEL_VERSION` |
| Agent behavior/persona | Admin panel → Assistants (paste from your `agents/*.md` files) |
| What documents are searchable | Admin panel → Connectors + Document Sets |
| Visual branding/theme | Fork the Onyx frontend (no built-in theming) |
| URL/domain | `.env` → `WEB_DOMAIN` |

---

## What Worked Well

1. **The envelope pattern** eliminated interface-specific branching in the
   policy engine. Adding the fourth adapter (email) took 20 minutes.

2. **Actor YAML files** are human-readable and git-trackable. When someone
   joins or changes their email, you edit one file.

3. **Channel-aware fingerprints** preserved audit fidelity without creating
   false duplicates across interfaces.

4. **Projection as a one-way gate** kept the vault owner in control. Team
   members never write directly to the vault.

---

## What We'd Do Differently

1. **Automate projection scheduling** -- we run it manually. A cron job or
   file watcher would catch changes faster.

2. **Document the admin panel setup** -- the Onyx admin panel configuration
   (creating connectors, document sets, agents) is manual and undocumented.
   Capture it in code or at least in a runbook.

3. **Consider a lightweight message queue** -- the current design processes
   events synchronously. At scale (50+ users, high event volume), a queue
   between adapters and the policy engine would help.

4. **Test with real users earlier** -- we built all four adapters before
   testing with anyone. Two adapters plus real feedback would have been better.

---

## Implementation Stats

- 3 implementation phases, each tagged as a release
- Phase 1 (Foundation): schemas, enums, fingerprint, dedupe, audit — 296 tests
- Phase 2 (Document platform adapter + policy): normalizer, access control, approvals, end-to-end — 317 tests
- Phase 3 (Multi-interface): 3 additional adapters + identity resolution — 357 tests
- All code: Python, Pydantic models, pytest
- Zero external dependencies beyond Pydantic and PyYAML

---

*Last updated: 2026-03-26*
