# Containment: Why Boundaries Matter More Than Content

The most important property of a knowledge system is not what's inside it --
it's where the boundaries are and whether they hold.

## The Solomonic Circle

In medieval learned magic, the operator inscribed a circle before beginning the
ritual. The circle was the precondition, not decoration. More time was spent on
containment than on the invocation itself. Break the circle and the operator is
consumed.

Applied to knowledge systems: the boundaries between temporal and durable,
private and public, operational and knowledge, intake and permanent -- these
are structural requirements, not suggestions. More effort should go into
maintaining boundaries than into adding new content.

## Containment Boundaries in Practice

### Temporal vs. Durable

| Space | Contains | Lifecycle |
|-------|----------|-----------|
| intake/ | Draft content, meeting extracts, raw captures | Disposable after triage |
| atlas/ | Permanent entities (concepts, people, orgs) | Persists indefinitely |
| domains/ | Deep knowledge areas with own structure | Persists and evolves |
| _review/ | Operational metadata (triage reports) | Ephemeral |

**The violation**: A meeting-prep file promoted to atlas/concepts/. It looks like
a concept but it's temporal content. Future agents searching for concepts get
polluted results. One misrouted file is harmless. Twenty of them degrade every
concept search.

### Private vs. Public (Sync Boundaries)

Different machines get different subsets of the vault via Syncthing:

- Primary workstation: full vault
- Agent server: non-private partition only (jibrain/)
- Cloud extraction service: text-only subset

`.stignore` files enforce these boundaries at the infrastructure level.
Private content physically cannot reach machines that shouldn't have it.

### Queue vs. Archive

intake/ is a queue, not a dump. A queue has expected throughput: things enter,
get processed, and leave. When intake/ files older than 90 days still sit
unprocessed, the queue has stagnated. This is a containment failure -- the
boundary between "needs processing" and "has been processed" has dissolved
into an undifferentiated mass.

## The "Broken Circle" Failure Modes

| Violation | Example | Consequence |
|-----------|---------|------------|
| Temporal in durable | Meeting prep in atlas/concepts/ | Pollutes concept search |
| Operational in knowledge | Triage reports in atlas/ | Agents confuse meta-work with real knowledge |
| Private in public | Personal contacts in synced partition | Privacy leak across devices |
| Queue stagnation | 50+ intake files unprocessed for 90+ days | New knowledge stops being processed |

## The Principle

**The circle must hold or the operator is consumed** -- not by supernatural forces,
but by information entropy. Small boundary violations compound silently until the
system is too degraded to trust.

Check containment during every health audit. Fix violations before adding new content.
A clean system with 500 files outperforms a polluted system with 5,000.

## For Agent Developers

Containment is the easiest thing to skip and the hardest thing to recover from.

1. **Validate placement on write** (is this the right directory?)
2. **Audit boundaries in health checks** (has anything leaked?)
3. **Fix violations immediately** (don't defer -- they compound)
4. **Sync boundaries are non-negotiable** (different machines, different access)
