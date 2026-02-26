# Conventions Are the Safety Manual

Why your schemas, routing trees, and processes aren't overhead -- they're the
safety system for operating an opaque knowledge graph.

## The Insight

From "Mechanics of Magic" (nraford7), drawing on Kieckhefer (1994):

> "The grimoire was not mysticism dressed as procedure. It was procedure dressed
> as mysticism. The only language available for 'here is how you safely operate
> a system whose mechanisms are hidden from the operator' was the language of
> the sacred. You discarded the language. You kept the system. You lost the
> safety manual."

Applied to knowledge systems: the frontmatter schemas, the routing decision tree,
the triage process, the observation mechanism -- these aren't bureaucratic overhead.
They are the safety system for operating a system whose internal mechanisms are
opaque to the operator.

## Why Knowledge Systems Are Opaque

You can't predict:
- Which connections will matter in six months
- Which files will be found by which queries
- How a misrouted file will pollute future searches
- When a stale claim will be mistaken for current truth

The mechanisms by which knowledge compounds (or decays) are hidden from you.
You work through indirect manipulation: schemas, tags, links, descriptions.
You don't directly wire the neural connections.

This is structurally identical to the problem medieval magicians faced:
operating a system whose mechanisms are hidden, through symbolic manipulation,
with powerful but unpredictable outputs.

## The Safety Manual in Practice

| Convention | What It Prevents |
|-----------|-----------------|
| Frontmatter schema | Files that can't be found, filtered, or audited |
| Routing decision tree | Content in the wrong place polluting search |
| Triage process | Intake queue growing without bound (queue blindness) |
| Observation mechanism | System friction going unnoticed until catastrophic |
| Containment boundaries | Private content leaking, temporal polluting durable |
| Description field | The 2,400th file being as findable as the 1st |
| Reweave pass | Knowledge accumulating without connecting |

## The Consequence of Skipping

When an agent skips a convention ("I'll just dump this in atlas/"), the immediate
cost is zero. The file is there. It looks fine.

The deferred cost is:
- One more file that doesn't match the schema (search breaks incrementally)
- One more misrouted file (containment erodes)
- One more description-less file (discovery degrades)
- One more connection unmade (knowledge doesn't compound)

These compound. By the time you notice, the system has decayed past the point
where incremental fixes help. You need a full audit.

**The ritual apparatus is the safety apparatus. Treat it accordingly.**

## For Agent Developers

If you're building agents that operate on knowledge systems:

1. **Make conventions machine-readable** (YAML schemas, not prose guidelines)
2. **Validate on write** (check frontmatter before saving, not during audits)
3. **Surface violations immediately** (agent friction signals, not silent failures)
4. **Never skip conventions for speed** (the time saved now is debt paid with interest later)
