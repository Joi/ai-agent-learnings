# Jibot Unified Architecture: Tiered Access Design

**Status**: LIVING DOCUMENT — Working design, iterating until stable  
**Date**: 2026-02-16 (ongoing)  
**Author**: Joi + Amplifier

> **⚠️ STALENESS WARNING (2026-03-17)**: Model references (gpt-5.2-pro) are outdated — current stack uses claude-opus-4-6 / gpt-5.4 / gemini-3.1-pro. OpenClaw routing configuration and sandbox agent details may have drifted from production. Verify against agent-mac before relying on specifics. Core four-tier security model remains the active design.

> **Note**: This document is actively being refined. Joi is remote from agent-mac;
> changes here will be tested when back on local network.
>
> **Canonical architecture source**: This file supersedes
> `design/mvp-architecture.md` and `design/jibot-migration-design.md`.

---

## Vision

A single "jibot" identity that manifests at different capability levels depending on who's talking to it. Joi gets a fully empowered personal agent. Real assistants get calendar, email, meeting, and document access. Trusted contacts get knowledge lookup and task submission. Everyone else gets friendly Q&A.

All of this runs on agent-mac (Mac Mini M4 Pro, 64GB), with jibot-3's community bot features (Slack, Discord, MUD) running alongside OpenClaw as a separate, focused process.

---

## Jibotmac Dedicated Accounts

**Key insight**: agent-mac has its own dedicated identities, separate from Joi's personal accounts. This significantly reduces risk and enables a more permissive security model.

| Service | Account | Purpose |
|---------|---------|---------|
| **Phone** | Dedicated SIM | WhatsApp, Signal, Telegram for jibot |
| **Apple ID** | jibot@example.com | iCloud, Apple Reminders, iMessage |
| **Google** | jibot@example.com | Gmail, Calendar, Drive |

### Security Implications

Because these are **dedicated jibot accounts** (not Joi's personal accounts):

1. **Email exposure is bounded** — If jibot@example.com Gmail is compromised, Joi's personal email (owner@example.com) is unaffected
2. **Calendar is jibot's view** — Events are copied/shared to jibot's calendar, not direct access to Joi's primary calendar
3. **Reminders are local to agent-mac** — Apple Reminders on agent-mac, synced via jibot@example.com iCloud
4. **No OAuth to Joi's accounts** — The agent never holds tokens for owner@example.com directly

**This means we can loosen sandbox restrictions** for owner and assistant tiers without risking Joi's primary accounts.

---

## Current Limitations & Workarounds

### Gmail Access from Sandbox (RESOLVED)

**Status**: WORKING as of 2026-02-17. gog CLI v0.11.0 is installed in the sandbox workspace.

**How it works**: The gog-linux binary lives at /workspace/.bin/gog-linux (21MB static build). A wrapper script at /workspace/.bin/gog auto-detects the Docker environment and routes to the Linux binary with correct SSL certs and config paths.

**Usage in sandbox-agent**:
```bash
gog gmail list -a jibot@example.com "in:inbox is:unread" --max 20
gog gmail search -a jibot@example.com "newer_than:7d" --max 10
gog calendar events primary --from 2026-02-18T00:00:00Z --to 2026-02-19T00:00:00Z
```

**Known issue**: After gateway restarts or OpenClaw updates, duplicate sandbox containers can cause exec routing to fall back to the host (symptom: zsh:1: no such file or directory). Fix by removing stale containers -- see Agent Runtime Names section above.

### Apple Reminders Access

**Status**: Works via `remindctl` skill (runs on host, not sandboxed)

### Google Calendar Access

**Status**: Works via `gog calendar` if gog is available. Same sandbox limitation as Gmail.

---

## Proposed Security Model (Loosened)

Given dedicated accounts, here's a more permissive model for owner/assistant tiers:

### Owner Tier (Joi) — FULL ACCESS

| Capability | Current | Proposed | Rationale |
|------------|---------|----------|-----------|
| Sandbox | Off | Off | No change — owner needs host access |
| Gmail (jibot@) | Blocked | **ALLOW** | Dedicated account, acceptable risk |
| Calendar (jibot@) | Blocked | **ALLOW** | Dedicated account |
| Shell exec | Allowed | Allowed | No change |
| Home automation | Allowed | Allowed | No change |

### Assistant Tier — EXPANDED ACCESS

| Capability | Current | Proposed | Rationale |
|------------|---------|----------|-----------|
| Sandbox | On (rw workspace) | On (rw workspace) | Keep sandboxed |
| Gmail (jibot@) | Blocked | **ALLOW READ** | Can read inbox, draft replies |
| Gmail send | Blocked | **ALLOW with review** | Drafts shown before send |
| Calendar | CRUD | CRUD | No change |
| Shell exec | Blocked | Blocked | Keep restricted |
| File write | jibot-docs only | jibot-docs only | No change |

### What This Enables

**For Owner (Joi via WhatsApp):**
- "What's in my inbox?" → jibot reads jibot@example.com Gmail
- "Reply to the MIT email saying I'll attend" → jibot drafts and sends from jibot@example.com
- "Add this to my calendar" → jibot creates event on jibot@example.com calendar

**For Assistants:**
- "Check if there's anything urgent in the inbox" → Can read jibot@example.com
- "Draft a reply to the venue about the event date" → Creates draft, shows for approval
- "What meetings does Joi have this week?" → Reads jibot@example.com calendar

---



## Agent Runtime Names

The "main" OpenClaw agent manifests in two distinct runtime contexts:

| Name | Runtime | Channels | Sandbox | Key Traits |
|------|---------|----------|---------|------------|
| **sandbox-agent** | Docker container (openclaw-sbx-agent-main-main-*) | Signal, Telegram, WhatsApp, iMessage | Yes -- all exec runs inside Docker | Uses bare gog (in PATH) for Google APIs. Filesystem scoped to /workspace. |
| **voice-agent** | macOS host (native CLI) | Jibot Voice, openclaw chat | No -- full host access | Uses /opt/homebrew/bin/gog (or bare gog). Full filesystem and macOS APIs (AppleScript, remindctl, osascript). |

Both agents share the same workspace (~/.openclaw/workspace), SOUL.md, MEMORY.md, and skills. The difference is **where commands execute**:

- **sandbox-agent**: All exec tool calls run inside the Docker container. Paths like /workspace/... are valid; host paths like ~/.openclaw/... are not. The container default shell is dash (not bash/zsh). Bare gog is in PATH and auto-detects Docker vs macOS, using the correct binary.
- **voice-agent**: Commands run directly on macOS. Has access to Homebrew tools, AppleScript, native Mac APIs, and the full filesystem.

### Troubleshooting sandbox-agent

Common failure modes:

| Symptom | Cause | Fix |
|---------|-------|-----|
| zsh:1: no such file or directory: /workspace/.bin/gog | Command ran on host instead of Docker (sandbox routing failure) | Restart gateway: openclaw daemon restart. Check for duplicate containers: openclaw sandbox list. Remove stale ones. |
| sh: 1: source: not found | Using source instead of . in dash shell | Use . /path/to/script.sh not source |
| gog: not found | PATH not set | Verify openclaw.json has /workspace/.bin in docker.env.PATH. On host, ensure /opt/homebrew/bin is in PATH. |
| Duplicate sandbox containers | Gateway restart created new container alongside old one | openclaw sandbox recreate --all or manually remove stale containers |

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│                         agent-mac (Mac Mini M4 Pro)                 │
│                                                                     │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │                    OpenClaw Gateway (:18789)                  │  │
│  │                                                              │  │
│  │  Channels: WhatsApp │ Signal │ Telegram │ iMessage            │  │
│  │                                                              │  │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐       │  │
│  │  │  owner   │ │ assistant│ │ trusted  │ │ general  │       │  │
│  │  │  agent   │ │  agent   │ │  agent   │ │  agent   │       │  │
│  │  │          │ │          │ │          │ │          │       │  │
│  │  │ FULL     │ │ Calendar │ │ Knowledge│ │ Q&A only │       │  │
│  │  │ ACCESS   │ │ Email    │ │ Calendar │ │ No tools │       │  │
│  │  │          │ │ Meetings │ │ avail.   │ │          │       │  │
│  │  │          │ │ Documents│ │ Task     │ │          │       │  │
│  │  │          │ │ Tasks    │ │ submit   │ │          │       │  │
│  │  └──────────┘ └──────────┘ └──────────┘ └──────────┘       │  │
│  │       ▲              ▲            ▲            ▲             │  │
│  │       │              │            │            │             │  │
│  │  ┌────┴──────────────┴────────────┴────────────┴────────┐   │  │
│  │  │              Channel Routing (bindings)               │   │  │
│  │  │  Joi's number ──────────────────────► owner           │   │  │
│  │  │  Assistant numbers ─────────────────► assistant       │   │  │
│  │  │  Trusted contact numbers ───────────► trusted         │   │  │
│  │  │  Signal group "Staff" ──────────────► assistant       │   │  │
│  │  │  Everyone else ─────────────────────► general         │   │  │
│  │  └──────────────────────────────────────────────────────┘   │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                     │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │              Jibot-3 (Community Bot)                          │  │
│  │              Slack (:3000) │ Discord │ MUD API (:3001)       │  │
│  │              People facts │ Knowledge lookup │ Vibecheck     │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                     │
│  ┌────────────────────────────────────────┐                        │
│  │         Shared Resources               │                        │
│  │  ~/jibot-docs/  (shareable knowledge)  │  ← See switchboard-    │
│  │  Apple Reminders  (tasks)              │    sharing.md for      │
│  │  Google Calendar  (schedule)           │    security model      │
│  └────────────────────────────────────────┘                        │
│                                                                     │
│  NOTE: ~/switchboard/ stays on Joi's laptop only.                   │
│  Shareable subsets are exported to jibot-docs/ via the              │
│  "chanoyu pattern" - see design/switchboard-sharing.md              │
└─────────────────────────────────────────────────────────────────────┘
```

---

## The Four Tiers

### Tier 1: Owner (Joi)

**Who**: Joi only
**Channels**: All (WhatsApp DM, Signal DM, Telegram DM, iMessage)
**Agent**: `owner`

| Capability | Details |
|------------|---------|
| Full tool access | exec, read, write, edit, process, all skills |
| Sandbox | Off (host access) |
| Calendar | Full CRUD |
| Email | Read, compose, send via himalaya |
| Tasks/Reminders | Full CRUD via apple-reminders |
| Knowledge | Full Obsidian vault access |
| Home automation | LogicMachine, cameras |
| Research | Nyne, academic search, web |
| Image generation | nano-banana-pro, imagen |
| Messaging | Send iMessages, WhatsApp, Slack posts |
| Code/Dev | GitHub, coding-agent, exec |
| Financial | Read-only (SOUL.md restriction) |

**SOUL.md**: Current SOUL.md with financial guardrails.

### Tier 2: Assistant

**Who**: Real human assistants (named, trusted, employed)
**Channels**: Signal DM, Signal group "Staff", WhatsApp DM
**Agent**: `assistant`

| Capability | Details |
|------------|---------|
| Calendar | Full CRUD — create, move, cancel meetings |
| Email | Read inbox, draft replies, send on behalf of Joi |
| Meeting arrangement | Availability lookup, schedule coordination |
| Documents | Read/write to jibot-docs and shared folders |
| Tasks | Create and manage reminders for Joi |
| Knowledge | Read Obsidian vault (people, orgs, concepts) |
| Travel/Logistics | Booking research, itinerary management |

**Cannot do**: Execute shell commands, install software, access financial data, modify system config, send messages as Joi without drafting first, access home automation.

**SOUL.md (assistant)**:
```
You are jibot, acting as Joi Ito's assistant-facing helper.
You are speaking with one of Joi's trusted human assistants.

ALLOWED:
- Calendar: full CRUD, create/move/cancel events
- Email: read, draft, send (clearly marked as from assistant)
- Documents: read/write in jibot-docs and shared folders
- Tasks: create reminders, check status, update
- Knowledge: search vault for people, orgs, topics
- Meeting prep: research attendees, compile briefings

BOUNDARIES:
- Never share financial data or portfolio information
- Never execute system commands or install software
- Never modify jibot's own configuration
- Never forward Joi's private DM history
- Never send messages that appear to come directly from Joi
  without explicitly marking them as drafted by assistant
- When uncertain about authority, ask the assistant to confirm
  with Joi directly
```

### Tier 3: Trusted Contact

**Who**: Board members, close collaborators, frequent contacts
**Channels**: Signal DM, Telegram DM
**Agent**: `trusted`

| Capability | Details |
|------------|---------|
| Knowledge | Read Obsidian vault concepts, orgs (not private notes) |
| Calendar availability | "Is Joi free on Tuesday?" (read-only, no details) |
| Task submission | "Remind Joi to call me about X" (creates reminder) |
| Q&A | Answer questions using vault knowledge |
| Document sharing | Access documents in jibot-docs/shared/ (read-only) |

**Cannot do**: Read email, see meeting details, modify calendar, access private notes, execute any tools, see financial data.

**SOUL.md (trusted)**:
```
You are jibot, Joi Ito's assistant bot.
You are speaking with a trusted contact.

ALLOWED:
- Answer questions using your knowledge base
- Check Joi's calendar AVAILABILITY (free/busy only, no details)
- Create reminders for Joi ("Contact X asked about Y")
- Share publicly-available information about Joi's work
- Access shared documents in the shared/ folder

BOUNDARIES:
- Never reveal meeting details (attendees, topics, locations)
- Never share email content or private notes
- Never modify calendar, email, or documents
- Never share financial, health, or personal data
- Never execute system commands
- Never reveal information about other contacts or conversations
- If asked something beyond your scope, say:
  "I'll pass that along to Joi" and create a reminder
```

### Tier 4: General

**Who**: Anyone who passes pairing (or is in an open group)
**Channels**: Signal (pairing), Telegram (pairing)
**Agent**: `general`

| Capability | Details |
|------------|---------|
| Q&A | General knowledge, public info about Joi's work |
| Routing | "I'll pass that along to Joi" → creates reminder |

**Cannot do**: Anything else. No tools, no file access, no calendar, no email.

**SOUL.md (general)**:
```
You are jibot, a friendly AI assistant.
You can answer general questions and take messages for Joi Ito.

BOUNDARIES:
- You have no access to Joi's calendar, email, or files
- You cannot look up information or run any tools
- You cannot share any private information
- If someone needs to reach Joi, offer to take a message
- Keep responses brief and helpful
- Never pretend to have capabilities you don't have
```

---

## Signal Group Architecture

OpenClaw natively supports Signal groups. This unlocks a powerful collaboration pattern:

### Staff Signal Group

A Signal group with Joi + assistants + jibot. Everyone in the group can:
- Ask jibot to check calendar
- Ask jibot to draft emails
- Ask jibot to look up contacts
- Ask jibot to prepare meeting briefs
- Share documents via jibot-docs

The group routes to the `assistant` agent, so it has assistant-level access. Joi gets the same assistant-level access in the group (not owner-level), which is appropriate for shared context.

```json
"channels": {
  "signal": {
    "groups": {
      "<staff-group-id>": {
        "agentId": "assistant",
        "requireMention": false,
        "tools": { "allow": ["read", "exec", "group:sessions", "group:messaging"] }
      }
    }
  }
}
```

### Configuration Notes

- Signal groups are session-isolated: staff group conversations don't leak into Joi's DM sessions
- `requireMention: false` means jibot responds to all messages in the group (can change to true for mention-only)
- Per-group tool overrides possible via `toolsBySender` if Joi needs elevated access within the group
- Signal groups support reactions, media, and typing indicators

---

## OpenClaw Routing Configuration

```json
{
  "agents": {
    "defaults": {
      "model": { "primary": "openai/gpt-5.2-pro" }
    },
    "list": [
      {
        "id": "owner",
        "sandbox": { "mode": "off" },
        "tools": { "profile": "full" }
      },
      {
        "id": "assistant",
        "sandbox": { "mode": "all", "workspaceAccess": "rw" },
        "tools": {
          "profile": "messaging",
          "allow": ["read", "write", "exec", "group:sessions", "group:messaging"],
          "deny": ["browser", "canvas", "nodes", "gateway", "cron"]
        }
      },
      {
        "id": "trusted",
        "sandbox": { "mode": "all", "workspaceAccess": "ro" },
        "tools": {
          "allow": ["read", "group:sessions", "session_status"],
          "deny": ["exec", "write", "edit", "apply_patch", "browser", "cron", "gateway"]
        }
      },
      {
        "id": "general",
        "sandbox": { "mode": "all", "workspaceAccess": "none" },
        "tools": {
          "allow": ["group:sessions", "session_status"],
          "deny": ["exec", "read", "write", "edit", "apply_patch",
                   "browser", "canvas", "nodes", "cron", "gateway"]
        }
      }
    ]
  },

  "bindings": [
    { "agentId": "owner",     "match": { "channel": "signal",   "peer": { "kind": "direct", "id": "+81XXXXXXXXXX" }}},
    { "agentId": "owner",     "match": { "channel": "whatsapp", "peer": { "kind": "direct", "id": "+81XXXXXXXXXX" }}},
    { "agentId": "owner",     "match": { "channel": "telegram", "peer": { "kind": "direct", "id": "JOIS_TELEGRAM_ID" }}},
    { "agentId": "owner",     "match": { "channel": "imessage", "peer": { "kind": "direct", "id": "owner@example.com" }}},

    { "agentId": "assistant", "match": { "channel": "signal",   "peer": { "kind": "direct", "id": "+81ASSISTANT1" }}},
    { "agentId": "assistant", "match": { "channel": "signal",   "peer": { "kind": "direct", "id": "+81ASSISTANT2" }}},
    { "agentId": "assistant", "match": { "channel": "signal",   "peer": { "kind": "group",  "id": "<staff-group-id>" }}},

    { "agentId": "trusted",   "match": { "channel": "signal",   "peer": { "kind": "direct", "id": "+1TRUSTED1" }}},
    { "agentId": "trusted",   "match": { "channel": "signal",   "peer": { "kind": "direct", "id": "+1TRUSTED2" }}},

    { "agentId": "general",   "match": { "channel": "signal" }},
    { "agentId": "general",   "match": { "channel": "telegram" }}
  ]
}
```

---

## Shared Personality Core

All four agents share the same voice but different capability scopes.

```
~/.openclaw/workspace/personality/
├── CORE.md         # "I am jibot. Calm, direct, quietly confident."
│                   # Shared knowledge, values, communication style
│                   # References to Joi's background and interests
│
├── OWNER.md        # Full capabilities, proactive behavior
│                   # Heartbeat, memory management, initiative
│
├── ASSISTANT.md    # Helper role, calendar/email/doc focus
│                   # Clear about what it can and can't do
│                   # Drafts rather than sends directly
│
├── TRUSTED.md      # Friendly but bounded
│                   # "I'll pass that along" pattern
│                   # Knowledge sharing within limits
│
└── GENERAL.md      # Minimal, polite, message-taking
```

Each agent's IDENTITY.md references CORE.md + its tier-specific file.
The personality feels consistent regardless of who's talking to jibot.

---

## Jibot-3 Integration (Option C+)

Jibot-3 continues running for Slack/Discord/MUD but slimmed down:

### Keep in Jibot-3
- People facts system (Slack-native, cross-workspace)
- Switchboard knowledge lookup (already works)
- MUD NPC bridge (unique, no OpenClaw equivalent)
- Discord vibecheck URL collection
- Slack mentions monitoring
- Permission system (owner/admin/guest)

### Move to OpenClaw
- Apple Reminders (OpenClaw's remindctl is better)
- Google Calendar (OpenClaw's gog skill is better)
- LLM intent parsing (OpenClaw is natively LLM-first)
- Health data (build OpenClaw skill)
- Daily digest (OpenClaw heartbeat system)
- Amplifier bridge (unnecessary on agent-mac)

### Bridge Between Them
- Jibot-3 Express API (:3001) for OpenClaw → Slack posting
- OpenClaw gateway API (:18789) for Jibot-3 → OpenClaw tasks
- Shared knowledge via jibot-docs repo (NOT direct ~/switchboard access)
- Jibot-3 accesses ~/switchboard/jibot/ locally on Joi's laptop for people facts
- See [Switchboard Sharing Strategy](../design/switchboard-sharing.md) for security model

---

## Document Security & the Airgap Strategy

> **IMPORTANT**: See [Switchboard Sharing Strategy](../design/switchboard-sharing.md) for the
> detailed pattern on how to selectively export content from ~/switchboard to jibot-docs.
> The full ~/switchboard directory should NEVER be shared with agent-mac directly.

### Three Document Tiers

| Tier | Location | Access | Examples |
|------|----------|--------|---------|
| **Private** | ~/private-dotfiles (encrypted) | Joi only, age-encrypted | API keys, financial data, security configs |
| **Internal** | ~/switchboard (Joi's laptop only) | Owner agent only | Daily notes, private people notes, health data |
| **Shareable** | ~/jibot-docs/ (synced to agent-mac) | Owner + Assistant + Trusted | Chanoyu, public concepts, org profiles |

### Airgap Strategy (airgap-mac)

The airgapped machine (airgap-mac) can serve as a document staging area:

```
agent-mac (full access)
    │
    │  Automated sync of "ok if hacked" documents
    │  via cron + rsync over local network
    ▼
airgap-mac (airgapped from internet)
    │
    │  Documents available for:
    │  - Physical access viewing
    │  - Local network sharing
    │  - Backup/archive
    ▼
```

**What goes to airgap-mac:**
- Published articles and public documents
- Shared meeting agendas (non-confidential)
- General reference material
- Backup copies of non-sensitive vault content

**What stays on agent-mac only:**
- Email content and drafts
- Private calendar details
- Financial data
- API keys and credentials
- Private people notes
- SOUL.md and security configs

---

## Migration Phases

### Phase 1: Move Jibot-3 to Jibotmac (1-2 hours)
- Clone repo, install deps, set up launchd
- Test Slack connectivity
- Run alongside OpenClaw (no conflicts: ports 3000/3001 vs 18789)

### Phase 2: Configure Multi-Agent Tiers (2-3 hours)
- Create 4 agent workspaces with SOUL.md for each
- Set up routing bindings in openclaw.json
- Create Signal staff group and configure routing
- Enable per-channel-peer session isolation
- Test each tier with real phone numbers

### Phase 3: Slim Down Jibot-3 (1-2 hours)
- Remove personal assistant features (calendar, reminders, health, digest)
- Remove amplifier bridge
- Remove unused LLM parser
- Test remaining community features

### Phase 4: Onboard Assistants (1 hour per person)
- Add assistant phone numbers to bindings
- Walk through capabilities and limitations
- Create Signal staff group
- Test end-to-end: "Schedule a meeting with X on Tuesday"

### Phase 5: Document Staging to Jibotmini (Optional, 1-2 hours)
- Set up rsync cron for shareable documents
- Configure airgap-mac access controls
- Test document availability

---

## Key Technical Decisions Needed

1. **Which assistants get access first?** Need their Signal phone numbers.

2. **Signal staff group or individual DMs?** Groups are more collaborative but less private. Recommendation: Both — group for coordination, DMs for sensitive requests.

3. **Should trusted contacts use Signal or Telegram?** Signal is more secure. Telegram is more widely used. Could support both with same routing rules.

4. **Model choice per tier?** Owner gets GPT-5.1-codex (400k context). Could use a cheaper/faster model for general tier to reduce costs.

5. **Pairing vs explicit allowlist?** Pairing is more flexible (new contacts self-onboard with approval). Allowlist is more controlled. Recommendation: Pairing for general tier, explicit allowlist for assistant/trusted.

6. **Notification routing?** When a trusted contact submits a task, should Joi get an iMessage? A Signal DM? A Slack notification?

---

## Open Questions

- Can OpenClaw skills be restricted per-agent? (Skills live in workspace; each agent workspace can have different skills installed — needs testing)
- How does memory isolation work across agents? (Each agent has its own session storage, but do they share the SQLite vector memory?)
- Can the assistant agent send emails on Joi's behalf, or only draft them? (himalaya skill capabilities need verification)
- What happens when an assistant creates a calendar event — does it appear on Joi's calendar directly? (gog skill uses Joi's Google auth — yes, it would)
- Rate limiting per tier? (Prevent general-tier abuse)

---

## Risk Assessment

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| Session context leakage between tiers | Low | High | per-channel-peer session isolation |
| Assistant oversteps capabilities | Medium | Medium | SOUL.md boundaries + tool restrictions |
| General tier abuse (spam/manipulation) | Medium | Low | Pairing gate + rate limiting |
| Jibot-3 ↔ OpenClaw state conflicts | Low | Medium | Separate data paths, shared read-only |
| Signal-cli instability | Medium | Medium | Heartbeat monitoring, auto-restart |
| Model costs from multiple agents | Low | Low | Same model, separate sessions only |

---

## Next Steps

1. **Review this document** — Mark up with comments, questions, changes
2. **Decide on tier membership** — Who goes in which tier?
3. **Phase 1 execution** — Move jibot-3 to agent-mac (low risk, immediate)
4. **Phase 2 prototype** — Set up one assistant agent, test with a real number
5. **Iterate** — Adjust SOUL.md boundaries based on real usage

---

*This document lives in jibot-docs/inbox/ for mobile review.
Move to jibot-docs/design/ once approved.*
