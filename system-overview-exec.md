# AI-Powered Knowledge Management: A Working System

This document describes a personal AI knowledge management system that has been in daily production use since early 2026. It captures, organizes, connects, and retrieves knowledge across meetings, research, conversations, and bookmarks.

---

## The problem

Knowledge work produces a lot of knowledge that goes nowhere. You read something interesting in a meeting, bookmark an article on your phone, have an insight during a conversation -- and three weeks later, you cannot find any of it. It is not that you lack information. You lack architecture.

AI tools make this worse before they make it better. ChatGPT can answer questions, but it does not remember what you read last month. Meeting transcription tools produce transcripts that nobody reads. Bookmarks pile up. The gap between "I encountered something valuable" and "I can find and use it when I need it" keeps widening.

This system closes that gap.

## What it is

An Obsidian vault -- a folder of plain markdown files -- with AI agents that help capture, classify, connect, and retrieve knowledge. Currently about 5,700 files, 15,000 wiki-links between them, and 4,300 files indexed for semantic search. Around 930 people profiles, meeting transcripts in Japanese and English, research notes, concept maps, organizational profiles, and several deep-knowledge domains.

The key idea: AI agents read and write the same plain text files that humans do. There is no separate AI database. Everything is human-readable, human-editable, and survives any tool change.

## How it works: the knowledge pipeline

Knowledge flows through three tiers, from raw to refined.

### Tier 1: Intake

Everything enters here first. A bookmark sent via WhatsApp or Telegram. A concept extracted from a meeting. A research note from a paper. Each gets a small structured header (type, source, date, a one-line description) and lands in a flat inbox folder.

Think of intake as a well-organized inbox. Items are tagged draft and unsorted. The system knows they exist but has not yet decided what they are worth.

### Tier 2: Atlas

Promoted, permanent knowledge. Organized into four categories:

| Category | What it holds | Example |
|----------|--------------|---------|
| Concepts | Ideas, theories, technologies | Oracle data integrity, techno-populism |
| People | Public-facing profiles | Researchers, collaborators, officials |
| Organizations | Institutions and companies | W3C, Digital Garage |
| References | Source material, articles, papers | Academic papers, policy documents |

Every atlas file has a `description` field -- a one-line summary that lets AI agents scan thousands of files without opening them. This is the single most important design decision in the system. It turns brute-force search into intelligent filtering.

### Tier 3: Domains

When a topic accumulates enough depth -- roughly 15 or more files, recurring vocabulary, distinct structure -- it graduates from atlas into its own curated space. The first domain to graduate was chanoyu (Japanese tea ceremony), which now has its own concept hierarchy, source texts, and conventions.

Other domains in progress: AI-assisted development practices, and a book project.

### The pipeline

```
Capture → Classify → Triage → Promote → Connect → Graduate
```

The critical step is **triage**. Not everything gets promoted. An AI agent reviews the intake queue, cross-references what already exists in the vault, and recommends: promote, merge with an existing file, hold for enrichment, or discard. This is where quality control happens. The system has taste.

After promotion, a **reweave** pass runs backward: now that this new entity exists, what older notes should link to it? This is how knowledge compounds. New information enriches old notes, not just sits beside them.

## The AI agents

Multiple specialized agents handle different parts of the workflow. None of them are particularly complex individually. The value is in how they compose.

**Curator agent** (always-on via WhatsApp or Telegram). Send it a URL or a research question from your phone. It extracts structured knowledge -- title, summary, key concepts, related entities -- and files it into intake. This is the most-used agent. It turns "that was interesting" into durable, findable knowledge in about 30 seconds.

**Meeting extractor**. After meetings are transcribed (by Granola, a macOS app), a pipeline translates Japanese to English when needed, summarizes the discussion, and extracts notable concepts, people, and organizations into the knowledge system.

**Triage agent**. Reviews the intake queue and generates recommendations. Runs weekly or on demand.

**Knowledge health agents**. Monitor for orphaned files (no links pointing to them), broken links, stale content, and structural problems. The vault currently tracks these metrics automatically.

**GTD (Getting Things Done)**. Personal productivity layer: syncs tasks from Apple Reminders, generates a daily dashboard, manages email follow-ups, tracks development tasks across git repositories.

**Research agents**. Academic paper search across Semantic Scholar, arXiv, PubMed. People and company lookup. Web research.

**Talk builder**. Takes a topic and builds a full presentation: researches across the knowledge base, generates an outline, suggests structure, and outputs a document ready for slide generation.

## The daily workflow

**Morning**: A routine runs automatically -- syncs tasks, generates a dashboard with focus items, checks repository status, surfaces what needs attention today. Takes about three seconds.

**Throughout the day**: Bookmark interesting things via WhatsApp or Telegram. They auto-file into the knowledge system. No friction, no context switching. If you are on an iPad, this works the same way -- send a message, knowledge gets captured.

**Meetings**: Transcribed, translated when needed, summarized. Key concepts and people extracted and cross-referenced against existing knowledge. "Who was that person we met with last month who works on X?" becomes a searchable question.

**Weekly review**: Audit knowledge health. Process accumulated intake. Review stale contacts. Promote what deserves promotion. Discard what does not.

**When building a talk or writing**: The system searches semantically across all accumulated knowledge. Not just keyword matching, but meaning. "What do I know about decentralized governance?" returns relevant notes even if they never use those exact words.

## Deep dives: rapid learning on unfamiliar topics

The daily workflow handles the steady state. But sometimes you need to get smart on something fast -- a major infrastructure project, a new technology area, an unfamiliar policy domain. The system supports a more intensive mode for this.

The process looks like this: gather meeting materials, presentations, and briefing documents into the knowledge system. The AI agents research the topic online -- pulling in background, context, key players, technical details. From all of this, the system produces a structured summary. You review it, give feedback, and the summary gets refined. Meeting notes from related discussions get integrated as they happen, so the picture keeps updating.

For producing output -- memos, briefings, presentation decks -- a collaborative editing mode (Claude cowork) lets you work with AI side by side in a document. You write, the AI suggests, you edit, it restructures. The result is polished output produced faster than either human or AI could manage alone.

The key insight is that this is not just research assistance. It is a feedback loop: gather, synthesize, present, get feedback, refine, gather more. Each cycle makes you meaningfully smarter on the topic. What would normally take weeks of reading and meetings to absorb can be compressed into days.

This works as a solo workflow, but it also works collaboratively. Multiple people can feed materials into the same knowledge base, review the same evolving summaries, and contribute feedback. We are currently using this pattern on a medtech project with a small team, and the shared knowledge base means everyone stays current without status meetings.

## The tool stack

| Layer | Tool | What it does |
|-------|------|-------------|
| Knowledge substrate | Obsidian | Markdown editor with wiki-links. The "database" is just files and folders. Free, works on iPad. |
| Sync | Syncthing | Peer-to-peer file sync across devices. No cloud dependency. Open source. |
| AI framework | Amplifier (Microsoft) | Orchestrates AI agents, tools, and workflows. Open source. |
| LLM providers | Claude, GPT, Gemini | Multiple AI models, auto-routed by task type. |
| Search | QMD | Three-mode search: keyword, semantic (meaning), and deep (hybrid). |
| Cloud sandboxes | Sprites (sprites.dev) | Tiny Linux VMs for running extraction services safely. |
| Task management | Apple Reminders + Beads | Personal tasks in Reminders, dev tasks in Beads (git-integrated). |
| Meeting transcription | Granola + muesli | Granola records meetings, muesli syncs/translates/indexes. |
| Always-on AI | amind.ai (Ethoswarm) | Persistent AI agent reachable via WhatsApp or Telegram. Always capturing. |
| Slides | Gamma.app | AI-assisted slide generation from structured talk documents. |
| People intelligence | Nyne.ai | Enrich contact profiles with public information. |

Most of these are either free, open source, or pay-per-use. The system is not one monolithic product. It is a composition of focused tools.

## Why this approach

**Files, not databases.** Everything is plain markdown. You can read it in any text editor, on any device, forever. No vendor lock-in. If Obsidian disappeared tomorrow, the knowledge would still be there -- readable by humans and AI alike.

**Local-first.** Your knowledge lives on your devices. Syncthing syncs peer-to-peer with no cloud intermediary. For sensitive institutional knowledge, this matters. Nothing transits a foreign server unless you choose to send it to an AI model for processing.

**AI-native but human-readable.** Agents read and write the same files you do. There is no hidden AI database that drifts out of sync with what you see. When an agent adds a link between two concepts, you see it in your notes. When you edit a note, the agent sees your edit.

**Incremental.** This system started as just Obsidian with some notes. The AI agents were added one at a time as needs became clear. You do not need to build everything at once. Start with capture and search, add automation as patterns emerge.

**Quality over quantity.** The triage pipeline means not everything gets promoted. Intake is disposable. Atlas is curated. This distinction is the difference between a knowledge system and a pile of files.

## What it takes to get started

The minimum viable version:

- **Obsidian** -- free, runs on iPad, Mac, Windows, Linux, Android
- **Syncthing** -- free, open source, handles multi-device sync without cloud
- **Amplifier** -- free, open source (Microsoft), orchestrates AI agents
- **API keys** for AI providers -- pay-per-use, typically $30-100/month for heavy daily use

Initial setup takes a few hours for the basic system. The agents and automation layer can be added incrementally over weeks. The system grows with you -- it does not require a big-bang deployment.

---

*Last updated: March 2026*
