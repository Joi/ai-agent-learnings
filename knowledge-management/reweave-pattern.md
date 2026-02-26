# The Reweave Pattern: Backward Enrichment

How knowledge compounds through connection, not accumulation.

## The Problem

Most knowledge systems are append-only. You add a file. Then another. Then another.
Each file sits beside the others without connecting. After 1,000 files, you have
a filing cabinet, not a brain.

The difference between a filing cabinet and a brain is backward connections.
When you learn something new, your brain doesn't just store it -- it rewires
existing memories to connect to the new information. The new thing changes what
you already knew.

Knowledge systems need the same mechanism.

## The Pattern

After promoting new content to the permanent knowledge base, run a **backward
pass** (reweave):

> "Now that this entity exists, what existing notes should link to it
> or be updated by it?"

### The Reweave Algorithm

1. **Identify recently promoted content** (last 30 days or since last reweave)
2. **For each new file, extract key entities, concepts, and tags**
3. **Search the existing knowledge base** for related files that don't yet link
   to the new file
4. **Check bidirectional linking**: Does A link to B? Does B link to A?
   One-directional links are incomplete connections.
5. **Check pending observations** for related structural issues
   (gaps, contradictions, unlinked clusters)
6. **Generate a prioritized action report**: high-value links to add,
   unlinked clusters to connect, observations to address

### What "High-Value Link" Means

Not all connections are equal. Prioritize:

1. **Bridge links**: Connect two previously unconnected clusters of knowledge.
   These have the highest structural value because they create new paths
   through the graph.
2. **Enrichment links**: Existing file gains new context from the promoted file.
   A person profile that now connects to a concept they discussed in a meeting.
3. **Correction links**: New information contradicts or updates old information.
   These are the most important to surface because stale knowledge looks
   authoritative.

Low-priority:
- Links between files that already share 3+ connections (diminishing returns)
- Links to files that are themselves thin or stub-quality

## Why This Works

The reweave pattern exploits a property of graph structures: **the value of a
node increases with its connections, not its content.** A short atlas file with
10 inbound links from diverse sources is more findable and more useful than a
long, detailed file with zero connections.

This means:
- **Connecting 5 existing concepts is worth more than 5 new unconnected files**
- **Enriching an existing file is usually better than creating a new one**
- **The reweave pass is not optional cleanup -- it's where knowledge actually compounds**

## Integration with the Knowledge Lifecycle

```
intake/  -->  triage  -->  promote to atlas/  -->  REWEAVE  -->  done
                                                     |
                                          "what existing notes
                                           should connect to this?"
```

The reweave step is what transforms a promotion from "adding a file" to
"enriching a knowledge system." Without it, atlas/ is just a more organized
version of intake/.

## When to Reweave

- After each batch of promotions
- During weekly reviews
- When an observation with `category: connection` surfaces unlinked clusters
- When a domain is approaching graduation threshold (15+ files)

## The Principle: Transformation, Not Creation

Agents don't create knowledge. They transform and recombine existing information.
Every "new" file is a recombination of things that already existed. Before creating
a new file, ask: "Can I enrich an existing one instead?"

This isn't a preference -- it's how knowledge systems actually work.

## Inspiration

The concept of a backward enrichment pass draws from:
- **Ars Contexta** (arscontexta.com): Their "6 Rs pipeline" includes a reweave
  phase for connecting new knowledge to existing context
- **Backpropagation** in neural networks: New information flows backward through
  the network to update existing weights
- **Spaced repetition**: The value of revisiting old knowledge with new context
