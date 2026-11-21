## `duplication` — each thing said once

Kind: issue

- Two places that say the same thing make the reader wonder how they differ, and drift apart when one of them
  changes. Compare like with like: the Examples under one Rule and across Rules, the Rules across the spec, and the
  blocks of the file. Two Examples show the same case when they set up the same context and trigger the same
  event, whatever their titles; values that differ only incidentally don't make them different cases.
- Flag: two Examples that show the same case, with the same outcome. Its fix removes the later one, moving any
  outcome only it checks into the other.
- Flag: two Rules that state the same acceptance criterion in different words, or one whose criterion is a case of
  the other's. Its fix merges them into one Rule, titled with the criterion, keeping every Example of both.
- Flag: two blocks of the spec file that cover the same aspect. Its fix merges them into one block, keeping every
  Rule and Example of both.
- Flag (question): a spec covering an aspect that another spec file of the project already covers: name that file,
  for the author to decide which one it belongs in. Its fix leaves the aspect to the other spec: it removes from
  this spec the Rules and Examples the other already shows, and adds to this spec's description an
  `Out of scope: … — see …` line pointing at it. The message names any Rule or Example the fix keeps, for the
  author to move there, and says the file can go when it keeps none.
- Flag (question): a Rule whose acceptance criterion a Rule of another spec file already states: name that file and
  its Rule, for the author to decide which spec it belongs in. Its fix leaves the criterion to the other spec: it
  removes this Rule with its Examples, but keeps those that show a case none of the other Rule's Examples shows,
  under the Rule retitled, and its description cut, to what only they show. The message names them, for the author
  to move there instead.
- Flag (question): an Example that shows a case an Example of another spec file already shows: name that file and
  its Example, for the author to decide which spec it belongs in. Its fix leaves what both check to the other spec:
  it removes from this Example the outcomes the other Example checks too, turning the `And` that follows into a
  `Then` when it was the first, and removes the whole Example when it checks nothing else.
- A fix changes only this spec, so a fix about another spec file leaves the repetition there. Its message names
  the other answer too: keeping it here and taking it out of the other spec instead. When the whole aspect is
  covered there, report that, not its Rules and Examples one by one.
- Flag (suggestion): Rule description lines that say again what the Rule's title states, retell what its Examples
  show, or add nothing a reader needs, such as a reason the title already makes plain: they only add to what the
  reader has to read. Lines that add what neither the title nor the Examples carry, such as what a term means, a
  reason that isn't obvious or a pointer to another spec, stay. Its fix removes the other lines, and the whole
  description when none stay.
- Flag (suggestion): Example description lines that only retell what its steps show, as a description written
  before the steps does once they are added. Lines that add what neither the title nor the steps carry stay. Its
  fix removes the other lines, and the whole description when none stay.
- Flag (suggestion): a `Then` that checks an outcome other Examples of the spec already show, under the Rule
  whose criterion it is: it repeats them and blurs what this Example is about. Its fix removes it, turning the
  `And` that follows into a `Then` when it was the first.
- Flag (suggestion): the same `Given` steps repeated word for word at the start of every Example, where a
  `Background:` at Ability or Rule level would hold them once. A step whose value varies between Examples, or that
  is the condition the Rule is about, stays inline. Its fix removes those steps from each Example, turning the
  `And` that then starts it into a `Given`, and puts them in a `Background:` after the description of the Rule
  whose Examples repeat them, or of the Ability when every Example of the spec does. A `Background:` already
  there gets them after its own steps.
- Flag (suggestion): three or more Examples under one Rule whose steps are the same but for their values, and whose
  titles only restate those values: they read as one `Scenario Outline` with a row for each. Its fix rewrites the
  first of them as that outline, titled with what they have in common, with a `<name>` placeholder for each value
  that varies and an `Examples:` table holding each Example's values as a row, in their order, and removes the
  others.
