A matter has no status field. Nothing you call sets one, and there is no vocabulary of endings to
choose from. What became of a matter is read off its shape: which analyses took it up, when each
took it up, whether they concluded, and what they produced.

## How a matter comes to rest

| Reading | The shape that produces it |
| --- | --- |
| Nobody has picked it up | No analysis references it. This is what `list_matters(untaken: true)` returns |
| Investigated, nothing warranted | An analysis took it up and produced no record (Path B) |
| A decision was made about it | An analysis took it up and produced one or more records |
| Refused, externally blocked, overtaken by events | Not recordable yet — see the bottom of this file |

These are descriptions, not values. Two of them can be true of one matter at once, because an
analysis producing a record does not stop a second analysis from taking the same matter up later.

## One analysis, any number of inputs

`create_analysis` takes `motivatedBy` as a **list**, of matter, world fact and observed outcome ids
in any mix. Pass every input the reasoning actually took up:

- Three people report the same fault — one analysis over all three, not three analyses saying the
  same thing. Cloning one act of reasoning into copies describes work that did not happen.
- Two unrelated reports triaged together, surfacing three faults — one analysis, three records, and
  no claim about which record belongs to which report.
- A wake that missed its target, read against the constraint that bounds the fix — the observed
  outcome and the world fact go in together, and neither is retyped as a matter first.

**You will never be asked which record answers which input, and you must not volunteer one.** In the
triage case there is frequently no true answer: a record may exist because of something the analysis
noticed that neither raiser mentioned. The reference list answers the question that does have an
answer — what this reasoning took up.

The exception is genuine divergence. If you triage four matters and conclude *different things* about
them — three are one fault, one is user error — those are different acts of reasoning and deserve
separate analyses with separate rationales. The rule is descriptive: an analysis must describe
reasoning that occurred, which forbids both splitting one act into copies and collapsing separate
acts into one node to tidy the graph.

## Taking something up after the fact

An analysis concludes in the same call that creates it. A matter recognized later is attached with
`attach_analysis_reference`, which works fine against a concluded analysis and records the date it
happened. That is not a workaround — it is the designed path, and the timestamps do the honest work:
the reference reads as **late**, meaning the rationale visibly does not account for it.

Do not reach for `reopen_analysis` to avoid a late reference. Reopening is for the case where the
reasoning genuinely was not finished and the produced records are still pre-t₀; it is refused once
any produced record has been committed. A late attachment is not a defect to be tidied away — a late
reference with no closure yet is exactly how the ledger registers coverage nobody has verified.

If a reference turns out to be wrong — nobody actually took that matter up — retract it with
`retract_analysis_reference`. That is always allowed, including after commitment.

> **Not built yet.** Recording that a matter was **refused** (valid, and we commit to not acting),
> **externally blocked** (we know what to do and cannot yet), or **overtaken by events** (the world
> removed the subject) is modelled but has no tool. Do not invent fields for them, and do not
> approximate them with a Path B dismissal — "no decision warranted" says the investigation found
> nothing to decide, which is a different and weaker claim than refusing a problem you agree is real.
> Record the decision that closes the topic and put the refusal or the blockage in its Verdict prose;
> the matter itself will keep reading as one somebody looked at.
