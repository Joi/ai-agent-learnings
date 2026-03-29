# Deploying Onyx as a RAG Platform for Organizational Knowledge

*March 2026*

How we deployed [Onyx](https://github.com/onyx-dot-app/onyx) (formerly Danswer) as a self-hosted RAG platform to project curated subsets of a personal knowledge vault into a team-facing search and chat interface -- without forking the codebase, without exposing the full vault, and with a governance layer that runs on flat files.

---

## The Problem

A personal vault with thousands of files is useful to the owner. It is opaque to everyone else. When a partner organization needed searchable access to curated knowledge -- research summaries, entity profiles, domain references -- we had three options: share the vault (unacceptable), copy-paste into shared docs (unmaintainable), or project a filtered subset into a platform the team could use without direct vault access.

We chose projection. The vault owner controls what is projected. The team sees only what has passed through the sensitivity filter. The vault remains the source of truth.

---

## Why Onyx

[Onyx](https://github.com/onyx-dot-app/onyx) is an open-source RAG and chat platform, YC W24 batch, formerly known as Danswer. It provides the full stack: document ingestion, vector search (backed by [Vespa](https://vespa.ai/)), a chat UI, an agent framework with custom personas, and connectors for file upload, Slack, Google Drive, and web scraping.

We chose it for four reasons:

1. **Self-hosted.** The corpus includes organizational research that cannot live on a third-party cloud. Self-hosting on our own GCP VM is non-negotiable.
2. **Full-featured out of the box.** A single Docker Compose deployment (~12 containers) gives you ingestion, embedding, search, chat, and admin. No glue code required.
3. **Zero-fork strategy.** We run vanilla upstream. All customization happens through environment variables and the admin panel. This means upstream updates are a `git pull && docker compose up -d` -- no merge conflicts, no rebasing.
4. **Active development.** Frequent releases, responsive community, YC backing. The project is moving fast.

---

## Deployment Architecture

```
Personal Vault (source of truth)
    |
    |  Projection Pipeline (Python)
    |  - Parse markdown frontmatter
    |  - Normalize document types
    |  - Sensitivity filter (skip confidential/restricted, >1MB)
    |  - Stage to temp directory
    |
    v
Onyx VM (GCP e2-standard-4, 4 vCPU / 16 GB RAM)
    +-- Docker Compose stack
    |   +-- Web UI (:3000)
    |   +-- API server
    |   +-- Background workers
    |   +-- Vespa (vector index)
    |   +-- PostgreSQL
    |   +-- Redis
    |   +-- Model server (embeddings)
    |   +-- Nginx (reverse proxy)
    +-- Google OAuth SSO
    +-- Claude via Anthropic API (LLM backend)
```

The personal vault never touches the VM. The projection pipeline runs locally, produces a filtered corpus, and uploads it via Onyx's file upload connector. The VM sees only the projected output.

---

## The Projection Pipeline

The pipeline bridges the vault and the Onyx instance. It is a one-way gate: vault content flows out through the filter, but nothing flows back into the vault.

### How It Works

1. **VaultReader** scans designated directories in the vault, producing `VaultArtifact` objects for each markdown file.
2. **ArtifactLoader** parses YAML frontmatter from each file, extracting `type`, `sensitivity`, `domains`, and `tags`.
3. **TYPE_MAP** normalizes 11 vault type strings (concept, person, organization, reference, meeting-extract, etc.) to a canonical `ArtifactType` enum. This prevents drift between the vault's evolving vocabulary and the projection system's expectations.
4. **OnyxProjectionRunner** transforms artifacts into projection records, applying two filters:
   - **Sensitivity filter**: Anything marked `confidential` or `restricted` in frontmatter is excluded. No exceptions.
   - **Size filter**: Files larger than 1 MB are skipped (binary artifacts, embedded images).
5. Output is staged to a temp directory and uploaded to Onyx via the file upload connector.

### ProjectionRecord

```python
class ProjectionRecord:
    source_path: str        # Where it lives in the vault
    projected_path: str     # Where it appears in the team corpus
    content_hash: str       # SHA-256 for change detection
    projected_at: datetime  # When this projection was made
    metadata: dict          # Frontmatter that's safe to share
```

The `content_hash` field exists for future incremental sync -- upload only files that changed since the last projection. We haven't built this yet. Currently, each projection uploads the full corpus. This is the single biggest operational pain point.

---

## Customizing Onyx Without Forking

The zero-fork strategy means every customization must fit through one of two doors: environment variables (`.env` file) or the admin panel (browser UI). If it doesn't fit through either door, we don't do it.

### The Customization Map

| Layer | How | Where |
|-------|-----|-------|
| Authentication | `AUTH_TYPE=google_oauth` | `.env` file |
| LLM provider | `GEN_AI_MODEL_PROVIDER=anthropic` | `.env` file |
| LLM model | `GEN_AI_MODEL_VERSION=claude-sonnet-4-20250514` | `.env` file |
| Agent personas | System prompts | Admin panel |
| Document sets | Groups of documents for scoped search | Admin panel |
| Connectors | File upload, Slack, Google Drive, web | Admin panel |
| Branding | Logo, display name | Admin panel |
| URL/domain | Reverse proxy + custom domain | Nginx / Cloudflare |

### What This Means in Practice

- **Agent personas are the main UX customization point.** You define them as system prompts in the admin panel. There is no theme system, no CSS overrides, no template engine. The system prompt is your product design lever.
- **Document sets scope what each agent can see.** A general knowledge agent searches everything; a domain-specific agent searches only its assigned document set. This is configured in the admin panel, not in code.
- **Per-agent model overrides are supported.** You can run a fast model for quick Q&A and a stronger model for deep analysis, configured per persona in the admin panel.

---

## Custom AI Agents

We run two agents inside Onyx, both configured as personas in the admin panel.

### Knowledge Assistant

Cross-references entities across the projected corpus, cites specific source documents, and flags when information may be stale (based on `last_verified` dates in frontmatter). Its system prompt instructs it to always cite the document it's drawing from and to say "I don't have information on that" rather than hallucinate.

### Domain Analyst

Scoped to a specific project's document set. References domain-specific document types (technical assessments, stakeholder profiles, regulatory references) and is instructed to reason about relationships between entities rather than simply summarizing individual documents.

### Version Control for Prompts

We store the full text of each agent's system prompt as a markdown file in the code repository. When we update a prompt, we edit the file, commit it, and manually paste the new version into the Onyx admin panel. This is not ideal -- the admin panel has no API for agent configuration -- but it means the prompt history lives in git, not in a database we don't control.

```
agents/
  knowledge-assistant.md    # System prompt text
  domain-analyst.md         # System prompt text
```

---

## Multi-Interface Integration

Onyx is one interface in a four-interface architecture. The full picture:

| Interface | Role | Protocol |
|-----------|------|----------|
| Document platform (Onyx) | Browse, search, RAG chat | Web UI + REST API |
| Chat bot | Conversational Q&A | Messaging APIs |
| Slack | Async team collaboration | Slack Events API |
| Email | Review comments and approvals | IMAP/SMTP |

All four interfaces normalize events through an envelope pattern:

```
Interface --> Normalizer --> NormalizedEnvelope --> Policy Engine
```

### Why This Matters

The policy engine runs on envelopes, not on raw interface events. This means adding a new interface requires only a new normalizer and a new entry in the `Channel` enum. Zero changes to access control, deduplication, or identity resolution.

Cross-interface identity resolution maps all user handles (Google OAuth email, Slack user ID, chat handle, email address) to canonical actor IDs. A person who reads a document in Onyx and comments on it via email is recognized as the same actor.

See [multi-interface-knowledge-layer.md](multi-interface-knowledge-layer.md) for the full envelope schema, adapter pattern, and identity resolution design.

---

## Governance Layer

We built the governance layer alongside the Onyx deployment, not after it. It is Python, Pydantic v2, and flat files. No additional database.

### What It Covers

| Concern | Implementation |
|---------|---------------|
| Typed schemas | Pydantic v2 models for all knowledge artifact types |
| Access control | RBAC: viewer / editor / admin per domain |
| Approval workflow | Authority records always require approval; multi-domain and sensitive content also require approval |
| Audit trail | Append-only YAML event files |
| Revision tracking | Point-in-time snapshots of projected artifacts |
| Deduplication | SHA-256 fingerprint registry |
| Content intake | Staging + triage pipeline with human-in-loop review |

### Why File-Backed

All governance state lives in YAML and JSONL files on the filesystem. No PostgreSQL, no Redis, no additional infrastructure beyond what Onyx itself requires.

This was deliberate. The governance layer must be:

1. **Auditable.** `git log` shows who changed what and when.
2. **Portable.** Copy the directory and you have the full state.
3. **Debuggable.** Open a YAML file in a text editor. No query language required.

The Pydantic v2 schemas caught real bugs during development -- type mismatches in the adapter code that would have been silent errors with untyped dicts. The pure-function policy engine (no I/O, no side effects) is testable without mock objects. These are not theoretical benefits; they saved hours during integration.

---

## Infrastructure and Cost

### The VM

| Spec | Value |
|------|-------|
| Instance type | GCP e2-standard-4 |
| vCPUs | 4 |
| RAM | 16 GB |
| Disk | 200 GB SSD |
| Region | Nearest to the team |
| Monthly cost | ~$131 |

Onyx is memory-hungry. The Vespa vector index, the embedding model server, and the background workers all want RAM. We tried `e2-standard-2` (8 GB) first and it was not usable -- Vespa alone can consume 6+ GB under indexing load. The `e2-standard-4` is right-sized for a corpus of several hundred documents with a small team.

### Infrastructure as Code

Three shell scripts handle the full deployment:

1. **Provision VM** -- create the GCP instance, configure firewall (HTTP/HTTPS + Onyx port), attach disk.
2. **Setup Onyx** -- clone the Onyx repo, write the `.env` file, run `docker compose up -d`.
3. **Prepare corpus** -- run the projection pipeline locally, upload the output to Onyx via the file upload connector.

This is intentionally simple. We considered Terraform and decided it was over-engineering for a single VM. The three scripts are idempotent and take less than 10 minutes to run from scratch.

---

## What Worked

1. **Zero-fork strategy.** We've pulled upstream updates three times with zero conflicts. The `.env` + admin panel customization surface is sufficient for our needs.

2. **Vault projection with sensitivity filter.** Clean separation between the private vault and the team-visible corpus. The vault owner sleeps well knowing that `confidential` and `restricted` files are structurally excluded, not just hidden.

3. **File-backed governance.** No extra infrastructure, git-versioned audit trail, human-readable state. When something looks wrong, you open a YAML file and read it.

4. **Pydantic v2 schemas.** Type safety caught real bugs in the adapter code -- a `str` where an `ArtifactType` enum was expected, a missing field in a projection record. These would have been runtime errors in production.

5. **Pure-function policy engine.** Every policy check is a function that takes data in and returns a decision. No database calls, no network requests, no mock objects in tests. This is the most testable code in the system.

6. **Multi-channel envelope pattern.** Adding a new interface is writing a normalizer function. The fourth adapter (email) took 20 minutes to implement.

7. **Custom agent prompts in version control.** Even though copying them to the admin panel is manual, having the prompt history in git has been valuable for iterating on agent behavior.

8. **GCP e2-standard-4 sizing.** After one false start with a smaller instance, the 4-vCPU / 16-GB configuration runs Onyx comfortably with room for corpus growth.

---

## What We'd Do Differently

1. **Automate corpus refresh from day one.** Manual upload is the number-one operational burden. We run the projection pipeline, produce a staged directory, then upload it through the Onyx UI. This should be a cron job that calls the Onyx API. We knew this on day one and still didn't build it. Build it on day one.

2. **Script Onyx admin setup via REST API.** Creating agents, document sets, and connectors through the admin panel is manual and unreproducible. Onyx has a REST API for most of these operations. A setup script that reads agent definitions from the repo and provisions them via API would make deployment fully reproducible.

3. **Start with the Slack connector early.** Team adoption followed the path of least resistance, which was Slack. We added the Slack connector late and wished we'd started there. People who won't open a new web app will type a question in a Slack channel.

4. **Build a health check script.** A script that verifies: all Docker containers are running, Vespa index is fresh (last indexed timestamp within threshold), LLM connectivity is working (send a test prompt), and disk space is adequate. We built this reactively after an outage. It should exist before the first user logs in.

5. **Use Onyx's built-in web connector earlier.** For public reference materials (standards documents, regulatory frameworks, published reports), the web connector eliminates the need to download, convert, and upload manually. We discovered this capability late.

---

## Gaps and Remaining Work

| Gap | Priority | Notes |
|-----|----------|-------|
| Automated corpus sync | High | Cron job + Onyx API to replace manual upload |
| Onyx admin API setup script | High | Reproducible deployment: agents, document sets, connectors from code |
| Slack connector | Medium | Team adoption through familiar tools |
| Google Drive connector | Medium | For shared team documents outside the vault |
| Langfuse observability | Medium | LLM usage tracking, cost monitoring, query quality |
| Multilingual query expansion | Low | English + local language queries against mixed-language corpus |
| Custom domain with TLS | Low | Currently accessed via IP; needs proper domain + certificate |

---

## Lessons on the Pattern

### Projection Is Not Replication

The projection pipeline is not a sync tool. It is a one-way sensitivity gate. Content flows from the vault through the filter and into Onyx. Nothing flows back. Team feedback (comments, corrections, requests) arrives through the multi-interface pipeline as events, and the vault owner decides what to incorporate.

This asymmetry is the point. The vault is the authoritative source. Onyx is a read-only projection of a subset. If they diverge, the vault wins.

### The Admin Panel Is the Product

In a zero-fork Onyx deployment, the admin panel is where your product lives. Agent personas, document sets, connector configuration, user access -- all configured through the browser UI. This is simultaneously the strength (no code to maintain) and the weakness (no version control, no scripting, no reproducibility) of the approach.

The mitigation: store everything that *goes into* the admin panel as files in a git repository. Agent prompts as markdown. Document set definitions as YAML. Connector configurations as notes. Then the admin panel is a deployment target, not the source of truth.

### Right-Size the VM, Then Stop Thinking About It

We spent too long trying to minimize infrastructure cost. The difference between `e2-standard-2` ($66/month) and `e2-standard-4` ($131/month) is $65/month. The time we spent debugging OOM kills and slow indexing on the smaller instance cost more than a year of the price difference. Pick the right size, pay the bill, and focus on the product.

---

*The pattern of projecting personal knowledge into team-facing search is older than any of the tools we used. Every curator, every librarian, every editor has done this: maintain a larger collection than any visitor sees, choose what to surface, and take responsibility for what's excluded. The vault is the stacks. Onyx is the reading room. The projection pipeline is the catalog. The only thing that's new is that the catalog runs in Python and the reading room understands natural language. What matters hasn't changed: the source of truth stays with the person who tends it.*