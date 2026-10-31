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
  for the author to decide which one it belongs in.
- Flag (question): a Rule whose acceptance criterion a Rule of another spec file already states, or an Example
  that shows a case an Example of another spec file already shows: name that file and its Rule or Example, for the
  author to decide which spec it belongs in. When the whole aspect is covered there, report that instead.
- Flag (suggestion): Rule description lines that say again what the Rule's title states, retell what its Examples
  show, or add nothing a reader needs, such as a reason the title already makes plain: they only add to what the
  reader has to read. Lines that add what neither the title nor the Examples carry, such as what a term means, a
  reason that isn't obvious or a pointer to another spec, stay. Its fix removes the other lines, and the whole
  description when none stay.
- Flag (suggestion): a `Then` that checks an outcome other Examples of the spec already show, under the Rule
  whose criterion it is: it repeats them and blurs what this Example is about. Its fix removes it, turning the
  `And` that follows into a `Then` when it was the first.
- Flag (suggestion): the same `Given` steps repeated word for word at the start of every Example, where a
  `Background:` at Ability or Rule level would hold them once. A step whose value varies between Examples, or that
  is the condition the Rule is about, stays inline.
- Flag (suggestion): three or more Examples under one Rule whose steps are the same but for their values, and whose
  titles only restate those values: they read as one `Scenario Outline` with a row for each.
