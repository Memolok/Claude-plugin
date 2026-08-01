# Sorting the intake

## The test

Ask of whatever the user brought: **could some future observation settle whether this was achieved?**

- Yes, and it names no mechanism → **head Claim**, expert branch.
- No, it describes a situation, a wish, an opportunity, or an open question → **bait**, matter branch.
- No, it names a solution → the need has been amputated. Recover it before routing.

Then two further exits, because not everything a user says is a decision at all:

- It is true regardless of any choice → **World Fact**.
- It is something that happened after a decision already sealed → **wake**.

## Why this is worth getting right

`promptedBy` is set at mint and can never be patched. A record minted on the expert path can never be
given matter provenance afterwards; the raiser's own words are simply not on the ledger.

The reverse error is now recoverable — a matter registered by mistake can be found with
`list_matters` and closed honestly through Path B. That asymmetry is a reason to ask rather than
guess, not a licence to register matters speculatively.

## Worked examples

### 1. "Login takes forever, users are furious."

**Bait.** A symptom, in someone's words, with no target. Register verbatim. Do not translate it into
*"auth latency must be under 200ms"* at intake — that target is the *output* of sharpening, and
inventing it now destroys the raw signal while smuggling in a number nobody agreed to.

### 2. "P99 auth latency must stay under 50ms."

**Head Claim.** A future measurement settles it either way, and it names no mechanism. Expert branch,
`create_MDR` at Deliberating.

### 3. "We should switch to Postgres."

**Neither.** This is a Verdict with the Need cut off. Nothing here is falsifiable, because "switching
to Postgres" is something you *do*, not something that turns out to be true or false.

Recover the need: *"what would switching fix?"* The answer routes it:

- *"Our joins are getting unmanageable in Dynamo"* → likely a Claim about query expressiveness.
- *"The team keeps hitting bugs nobody can debug"* → bait.

Then Postgres becomes what it always was — an **alternative**, recorded in the belly with the reasoning
that favoured it.

### 4. "Support filed three tickets this week about checkout errors and I want to fix it."

**Bait**, and the tickets are the reporters' words. Register the matter from the ticket content,
not from the user's summary of it. "I want to fix it" states intent to decide, which is why you carry
straight on into sharpening — but it does not make the intake a Claim.

### 5. "I need to pick a language for the stats pipeline; it must have a mature stats library ecosystem."

**Head Claim.** Falsifiable — a later observation can confirm or refute whether the ecosystem held up —
and mechanism-neutral, since it does not say *which* language. Expert branch.

Note what would have made this the wrong call: *"it must use Python for pandas and numpy"* names the
answer, and would need backing out per example 3.

### 6. "The build is flaky." — said by the only engineer on the project

**Bait.** Still a symptom, still no target. Being solo does not promote it.

This is the most commonly missed case, because with no second person around it is tempting to treat
your own observation as expert framing and mint directly. The `memolok-method` rule applies: with no
separate human in the Expert #1 seat you inherit it by name only, and the head Claim is still
co-discovered with the user.

### 7. "Legal says we must retain records for seven years."

**World Fact.** True whether or not anyone decides anything about it, so it is a premise rather than a
decision. Hand off to `manage-almanac`.

The decision it *provokes* — what to change so retention is met — is a separate fish, and it should
cite this fact through `hasContext` while still staged.

### 8. "We decided last quarter to use Kafka and it's not working out."

**A wake**, on a record already on the ledger. Hand off to `record-outcome` to register the observed
outcome against the expected outcome it tests.

Then it may well become fresh bait: a `Violated` result is exactly the kind of signal that starts the
next matter. What it is *not* is a reason to edit the original record — that record was honest at
its t₀, and rewriting the tail to match what happened is ledger fraud.

### 9. "Just noting that the CSV export mangles unicode — don't do anything about it now."

**Bait, parked.** The user has explicitly declined to decide. Use `save-matter`: register verbatim,
confirm, stop. Do not analyze it, do not sharpen it, and do not offer to open a decision.

It stays at `MatterReceived` and shows up later in
`list_matters(status="MatterReceived")`.

## Common failures

**Paraphrasing bait into a Claim and expert-minting it.** The dominant one. It destroys the raw signal
*and* usually over-sharpens the Need with a target that only exists because you invented it. Two rule
violations in one call, and neither is reversible.

**Registering a matter for a fully-formed decision.** The user arrived with a Claim and their
reasoning; wrapping it in a fabricated raw input puts words in someone's mouth. Recoverable
through Path B, but it pollutes the inbox.

**Routing on the opening verb.** "We should…", "I need…", "can you fix…" say nothing reliable about
which side of the fork the content sits on. Route on falsifiability, not phrasing.

**Treating the fork as a menu to read aloud.** Sort it yourself from what they said. Ask only when it
is genuinely ambiguous, and ask one question — not five.
