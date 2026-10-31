## `single-behaviour` — one behaviour per Example

Kind: split

- Flag: an Example with several `When` → `Then` rounds that tests more than one behaviour.
- Flag: a `Then` that checks an outcome unrelated to the acceptance criterion the Example illustrates — its
  Rule's, or the spec's aspect for an Example outside any Rule — and that no other Example of the spec shows: it
  becomes an Example of its own. Several outcomes of the same criterion are fine.
- Flag: a `Scenario Outline` whose `Examples:` rows show different behaviours, such as accepted and rejected
  inputs, so its steps have to stay vague enough to cover both.
- Flag: an Example whose `When` doesn't exercise the capability the spec is about: it is setup that belongs in
  a `Given`, or an event that belongs in another spec.

## `single-behaviour` — one acceptance criterion per Rule

Kind: split

- Flag: a Rule whose title or description covers more than one acceptance criterion, such as a main behaviour
  plus its boundary, an "and also …", or a verb like "cycles" that hides a step and a wrap-around. Each criterion
  is a narrow Rule of its own.
- Flag (issue): an Example under a Rule that illustrates the acceptance criterion of another Rule of the
  spec: it belongs under that Rule.
- Flag: an Example under a Rule that shows an acceptance criterion no Rule of the spec covers yet: it starts
  a Rule of its own.
- Flag (issue): an Example whose `When` triggers the event another Rule's Examples share, not its own
  Rule's: it belongs under that Rule.
- Flag: Examples under one Rule that trigger different events, where no other Rule of the spec has Examples
  triggering one of them. A Rule's Examples share their `When` and differ in their `Given`s; another event
  is usually another criterion, a Rule of its own. A `When` that varies only in its data is the same event.
- Flag: a Rule with more than about five Examples that vary in more than one way, such as in their input and in
  their starting state. Each way they vary is usually a criterion of its own. Many Examples that vary one way,
  such as values across a range, are fine.

## `single-behaviour` — one aspect per spec

Kind: split

- A spec's scope is what its header says: the activity its `Ability:` (or `Feature:`) title names, and what its
  `I want` asks for. Read every Rule against them, and ask which question it answers: what the user sees in each
  case, how results combine across the parts that hold them, when what is shown changes, where the data comes
  from, what happens when something is added or missing. Rules that answer different questions are different
  aspects, even when each fits loosely under the title.
- Flag: a spec file whose Rules fall into separate aspects a reader would look for in different places (opening
  versus searching versus navigating, say), or whose Ability title has to stay vague to cover them all. Each
  aspect is a spec of its own.
- Flag: Rules whose acceptance criteria the spec's title and `I want` don't ask for, such as a story about the
  price each item shows holding Rules on how the cart totals those prices: they form a spec of their own, with
  its own title and story.
- Flag: a spec with more than about five Rules, or with a Rule of more than about five Examples, whose Rules or
  Examples answer more than one of those questions. Split it by the question each Rule answers, each aspect a spec
  of its own.
- Flag: a spec with more than about five Rules, or about twenty Examples in all, whose Rules all answer the same
  question but fall into groups that each cover a narrower part of its aspect, such as one context or one
  sub-activity a reader would look for by name: it is too long to scan. Each group is a spec of its own, with an
  `Ability:` title naming that narrower part and an `I want` for it. A long spec whose Rules don't group this way
  is fine.
- Flag: a spec without Rules in which at least two acceptance criteria each have several Examples: each
  criterion is a Rule of its own.
