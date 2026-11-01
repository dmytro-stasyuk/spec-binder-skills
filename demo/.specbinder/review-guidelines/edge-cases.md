## `edge-cases` — each Rule covers the cases that break it

Kind: gap

- Flag: a Rule with a real boundary (match or no match, on or off, valid or rejected) but no counter-example
  beside its main case.
- Flag: boundaries — a Rule with a limit, threshold or range but no Example at, just below or just above it.
- Flag: empty and missing input — no Example for nothing entered, an empty list, or a required value left out.
- Flag: errors and failures — no Example for invalid input, or for a service or resource the user relies on
  being unavailable, and what the user sees then.
- Flag: permissions — no Example for someone who may not do the thing, or may see only part of it.
- Flag: state and ordering — no Example for doing it twice, undoing it, or doing it when it was already done or
  doesn't apply yet.
- Flag: concurrency — no Example for two people changing the same thing at once, where both can.
- Flag: time — no Example for an expiry, a deadline, a time zone or a date rollover the Rule depends on.
- Flag: multiplicity — no Example where one input affects several targets, when it can.
- Go through the Rules one at a time; a spec without Rules counts as one Rule. For each Rule, ask of every check
  here whether the Rule's behaviour can meet that case, and whether an Example shows what happens then.
- Report a gap only when the case can really happen to the Rule's behaviour and its Examples don't make the
  outcome obvious, a reader would want to know the answer or the author would have to decide it, and no other
  Rule covers it and the spec doesn't say it is out of scope.
- Most Rules have no gap for most of these checks: don't report one just because it is listed.
- Don't ask for degenerate cases that no realistic change to the behaviour could break.
