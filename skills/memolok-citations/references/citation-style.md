# How a citation reads

**Trailing parenthetical, at the end of the clause it justifies.** `(MDR-7)`, closing the sentence or
the clause whose shape the record explains. This is the default and most of what you will write: it
reads as an aside, so the prose still parses with it deleted, and a reader who does not care about the
ledger is never made to step over it mid-sentence.

**Several entries share one parenthesis.** `(MDR-7, MDR-12)`, comma-separated, in the order the
argument uses them. Two parentheticals in a row assert two asides about two different clauses, which
is a claim about the sentence that is usually false.

**A relationship goes inside the parenthesis.** Where one record narrows, settles or replaces another
and a reader needs the pair to follow the line, say which is which in the same breath — `(MDR-12,
narrowing MDR-7)`. The bare pair says only that both bear on this, and sends a reader to open two
records to reconstruct a relationship you already had.

**An identifier may open the line where the entry is the subject.** `MDR-7:` heading a docstring or a
section banner, where what follows exists *because of* that record. That is a topic sentence rather
than a citation beside one, so it is right only where it is true — a banner over code that merely
shares the area claims a provenance the code has not got.

**Backticks mark a token, never a citation.** Write `` `MDR-7` `` when the string itself is the
subject: parsing it, printing it, telling it apart from something else. Write MDR-7 bare when you name
the decision. A backticked citation reads as an identifier to go and find in the code, and the habit
spreads — once one is in code font, the next one matches it.

**Another project's decision numbers keep that project's own notation.** A number from a different
decision-record system is never reshaped into a Memolok-looking one — `ADR-0007` stays `ADR-0007`,
padding and all — and where the surrounding text does not already make the system obvious, its name
goes in front. Reshaped to the local convention it reads as one of yours, and sends a reader to a real
entry that says something else.

Under all of them sits one placement rule: **put the citation where a reader asking "why is this line
like this" would look, and not where the line already answers for itself.** One note in a module
docstring beats the same parenthetical on six lines inside it.
