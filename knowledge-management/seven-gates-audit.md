# The Seven Gates: Pipeline Completeness Audit

A framework for auditing any knowledge management pipeline for structural
completeness. Adapted from Richard Kieckhefer's analysis of medieval learned
magic as a rational technology with seven structural requirements ("The Specific
Rationality of Medieval Magic," American Historical Review, 1994).

## Origin

Kieckhefer argued that medieval learned magic was not irrational but a systematic
technology with its own operating logic. He identified seven stages that map onto
Aristotelian four-cause theory. The insight: these seven stages aren't historically
specific -- they're structural requirements for any system where an operator works
through symbolic manipulation of an opaque system.

A knowledge management system is exactly such a system:
- The mechanisms by which knowledge compounds are opaque (which connections will matter?)
- The inputs are symbolic (markdown, frontmatter, wikilinks)
- The outputs are powerful but unpredictable (what will the system surface when you need it?)
- The operator works through indirect manipulation (schemas, tags, links -- not direct neural wiring)

## The Seven Gates

### Gate 1: Intent (vis imaginativa)

**Medieval**: The magician must hold a clear image of the desired outcome in
directed imagination before beginning.

**Knowledge system**: Every file must have a clear `type` and `description`.
Why are we capturing this? What will a future session search for?

**Health check**: Count files missing `type` or `description` fields.
Zero tolerance -- every file without intent is a file that can't be found.

**Failure mode**: Files you can't find because nobody recorded what they are
or why they matter.

### Gate 2: Precision (exact language)

**Medieval**: The vowel points in divine names must be exact. One wrong syllable
and the invocation fails or misfires.

**Knowledge system**: Frontmatter must match the schema exactly. The spec says
`type: concept` -- not `type: Concept`, not `category: concept`, not `kind: concept`.

**Health check**: Validate frontmatter against the schema. Count deviations.

**Failure mode**: Schema drift. The spec says one thing, the files do another.
Agents trained on the spec produce inconsistent output. Search and filtering break.

### Gate 3: Materia (source provenance)

**Medieval**: Agrippa catalogued thousands of material correspondences -- saffron
to the Sun, silver to the Moon, copper to Venus. The right materials produce the
right effects. The wrong materials produce nothing or worse.

**Knowledge system**: Every file tracks where it came from. `source`, `source_url`,
`source_date`, `agent` fields record provenance.

**Health check**: Count files missing source fields.

**Failure mode**: Orphaned claims with no way to verify, update, or trace back
to origin. When a source becomes unreliable, you can't find everything derived
from it.

### Gate 4: Containment (the Solomonic circle)

**Medieval**: The circle is the precondition, not decoration. More time is spent
inscribing the circle than performing the ritual. Break the circle and the
operator is consumed.

**Knowledge system**: Temporal content stays out of durable spaces. Private content
stays out of synced partitions. Operational metadata stays out of knowledge files.

**Health check**: Audit for boundary violations:
- Files with `type: meeting-extract` or `type: meeting-prep` in atlas/ (temporal in durable)
- Triage reports mixed into atlas/ (operational in knowledge)
- Personal contacts in synced partitions (private in public)
- Intake queue older than 90 days (stagnation = undifferentiated mass)

**Failure mode**: Search pollution. Meeting preps appearing in concept searches.
Agent confusion between meta-work and real knowledge. Privacy leaks across
sync boundaries.

**This is the most critical gate.** More effort should go into maintaining
boundaries than into adding new content.

### Gate 5: Currency (temporal alignment)

**Medieval**: The ritual must be performed at the correct planetary hour. Timing
matters because the forces being invoked have their own rhythms.

**Knowledge system**: Knowledge has a shelf life. A person's role changes. An
organization pivots. A technology assessment becomes obsolete. Without tracking
freshness, the system silently fills with outdated information that looks
authoritative.

**Health check**: For time-sensitive content types (people, organizations,
technology assessments), check for `last_verified` field. Flag when absent
or older than 180 days.

```yaml
last_verified: 2026-02-26  # when this information was last confirmed accurate
```

**Failure mode**: Stale people profiles with wrong roles. Obsolete org descriptions.
Technology assessments that describe last year's landscape. The system looks
authoritative but is silently wrong.

### Gate 6: Cost (energy sacrifice)

**Medieval**: The magician fasted for three days, prayed, purified. The toll was
real and paid by the operator's own body.

**Knowledge system**: Agent compute, human attention, and source labor are real
costs. A system that consumes heavily but refines little is extracting without
transforming.

**Health check**: Report the intake-to-atlas ratio. How much raw material is being
captured vs. how much is being refined into permanent knowledge?

**Failure mode**: Intake queue growing without bound. Agents extracting content
from every meeting and bookmark but nobody triaging, connecting, or promoting.
The system becomes a dump, not a brain.

### Gate 7: Verification (empirical verification)

**Medieval**: Did the spirit appear? Did it obey the constraints? Did the desired
outcome occur?

**Knowledge system**: Does the system actually work as a knowledge system? Are
connections being made? Are orphans being linked? Are broken references being fixed?

**Health check**: Orphan count, dead-end count, broken link count, pending
observation count. Run continuously, not just at setup time.

**Failure mode**: The system rotting invisibly. Files accumulating but not
connecting. Links breaking but nobody noticing. The knowledge graph becoming a
knowledge graveyard.

## Running the Audit

For each gate, report one of:

| Status | Meaning |
|--------|---------|
| **PASS** | No issues or trivial count |
| **WARN** | Minor issues that should be addressed |
| **FAIL** | Structural problem that needs immediate attention |

The seven gates should be checked during every health audit -- not just at
system setup, but continuously. Knowledge systems decay by default. The audit
is what prevents decay.

## Implementation Notes

The gates are ordered by dependency:
1. Intent and Precision (gates 1-2) make content findable
2. Materia (gate 3) makes content traceable
3. Containment (gate 4) keeps the system coherent
4. Currency (gate 5) keeps the system current
5. Cost (gate 6) keeps the system honest
6. Verification (gate 7) keeps the system alive

If you're implementing incrementally, start with gates 1, 2, and 4.
These prevent the most common failure modes: unfindable content,
schema drift, and boundary violations.

## Sources

- Kieckhefer, R. (1994). "The Specific Rationality of Medieval Magic."
  *American Historical Review* 99.3: 813-836.
- nraford7. "Mechanics of Magic." https://nraford7.github.io/mechanics-of-magic/
