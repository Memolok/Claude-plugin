# Need-versus-Verdict drift

Content that only became true *because of* deliberation gets written back into the Need, which is
supposed to describe what was true *before* deliberation resolved anything.

## The test

> Would this Need statement have been askable, word for word, before any alternative was discussed?
> If a mechanism, technique, or scope boundary in the Need only became known through exploring
> alternatives, it belongs in `alternatives`, `deliberationFacts`, or `verdict` — regardless of who
> first said the words.

## A real correction arc

The conflation below survived a first correction and had to be caught three times before the Need was
honestly fuzzy.

**Attempt 1 — mechanism smuggled in.** After the user confirmed a specific synchronization mechanism
during belly discussion, the Need was drafted as:

> *"The two anchors must keep their handle directions synchronized while each retains an
> independently free handle length."*

Wrong: "synchronized" is the mechanism deliberation had just picked — a Verdict — not a constraint
that predated it.

**Attempt 2 — same failure, new content.** After the user widened scope:

> *"…any point on any drivable curve can independently be set to one of three states — cusp, smooth,
> or symmetric."*

Wrong the same way. "Three states" is a resolved alternative. This is a *recurrence* of Attempt 1, not
a new mistake — the first correction did not transfer.

**Attempt 3 — relapse inside the correction.** Trying to fix Attempt 2:

> *"…must give the practitioner independent, per-point control over each point's continuity
> behaviour… sufficient to let the practitioner deliberately choose, point by point, between
> continuous and intentionally-broken behaviour."*

Still wrong: "per-point" granularity was itself arrived at through deliberation — the alternative
could have been per-terminal-point, or per-curve. The correction restated a narrower version of the
error.

**Attempt 4 — honestly fuzzy.**

> *"…the practitioner needs creative control over the shape's contours — not merely whatever
> continuity behaviour gets automatically enforced."*

Everything specific from the first three attempts — the synchronization mechanism, the three-state
model, per-point granularity — moved into `alternatives`, `deliberationFacts`, and `verdict`, in the
order it was actually discovered.

## Fuzzy is not unfalsifiable

Attempt 4 is still falsifiable: a later observation could confirm or refute whether the practitioner
actually got creative control over the contours, versus being stuck with auto-enforced continuity.

De-sharpening a Need to remove a smuggled-in mechanism does not make it vague in the bad sense
(*"we need better performance"*). It stops pre-committing to *which* mechanism delivers the testable
outcome.

## Why one correction is not enough

The pull is structural. The same turn format that correctly produces a specific recommended answer for
a belly or waist question gets applied unchanged to Need-sharpening questions — where a confident,
specific "recommended Need" is close to a contradiction in terms.

If you relapse after a first correction, do not reword the same sentence. Re-derive the Need from
scratch by asking *"what did we actually know before we explored anything?"*

## Where the specificity goes

It relocates, it does not disappear:

```json
{
  "hasNeed": {
    "description": {
      "markdown": "The practitioner needs creative control over the shape's contours — not merely whatever continuity behaviour gets automatically enforced.",
      "lang": "en"
    }
  },
  "alternatives": [
    {
      "id": "alt-per-point-3state",
      "label": "Per-point 3-state toggle",
      "description": {
        "markdown": "Cusp / smooth / symmetric, settable independently per point.",
        "lang": "en"
      }
    }
  ],
  "verdict": {
    "description": {
      "markdown": "Adopt the per-point 3-state toggle; the symmetric state also covers the seam-mirroring case raised earlier.",
      "lang": "en"
    }
  }
}
```

This is not "whoever spoke the words owns the field". Both the Need and the Verdict above were reached
jointly across the session. The discipline is about which region a clause belongs in, not about whose
turn it was to speak.
