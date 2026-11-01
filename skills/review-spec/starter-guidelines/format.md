## `format` — SpecBinder's layout and wording conventions

Kind: polish

- Flag: an `Ability:` title that is a sentence or a description rather than a short verb phrase of about one to
  three words naming the capability, starting lowercase ("renaming a step", "paying by card").
- Flag (suggestion): an Ability without an `As a … / I want … / So that …` user story, or a story whose parts are
  not each on their own line with a capitalised connector (`So that`, never `so that`).
- Flag: description lines that run straight on from the user story, with no blank line between them.
- Flag (suggestion): an Ability description written as flowing prose. It holds only context the Rules don't
  carry (terms, a legend, `Out of scope: … — see <OtherSpec>` pointers), as short `Label:` lines and `-` items.
- Flag (suggestion): a description line that states testable behaviour, which belongs in a Rule, or that repeats
  a Rule of this spec or another one.
- Flag (polish): a Rule whose title and description together take more than three lines.
- Flag: a line longer than 140 characters.
- Flag: a description line (under an Ability, Rule, Example or Background) that starts with a capitalised
  Gherkin keyword such as `Given`, `When`, `Then`, `And`, `But`, `Rule` or `Example`, which the editor
  mistakes for a real keyword.
- Flag: indentation other than two spaces per level: `Ability:` at column 0, its story, description,
  `Background:`, `Rule:`s and Ability-level `Example:`s at 2, a Rule's description and Examples at 4, and each
  Example's steps two further in. Doc-string fences and table rows line up with their step's text.
- Flag: a missing blank line before a `Rule:`, `Example:` or `Background:` (titles-only drafts included), or blank
  lines between the steps of one Example.
- Flag (suggestion): an Example without at least one `Given`, one `When` and one `Then`. When nothing needs setting
  up, a `Given` names the baseline ("Given the cart is empty") rather than the Example opening with `When`.
- Flag: a repeated `Given`, `When` or `Then` where `And` or `But` would continue the previous step.
- Flag: a lowercase "i" for the pronoun in step text.
- Flag (suggestion): a step that describes its data instead of showing it ("Given a valid order", "Given the
  cart holds three items"). A single value is quoted, a set of like rows is a data table, and structured or
  multi-line content is a doc string.
- Flag (suggestion): a `Then` whose expected value is part of the prose ("Then the toolbar is hidden") rather
  than quoted or in a doc string (`Then the toolbar is "hidden"`). A quoted expected value is what the IDE's
  Accept actual rewrites when the step fails.
- Flag: a quoted identifier (`the "discount" setting is "on"`). Only the value that varies is quoted, and the
  subject stays as plain words.
- Flag (suggestion): a step that joins several concerns, such as setting up content and a state at once, where
  separate single-concern steps already used in the project would compose.
- Flag: a nested doc-string fence inside a doc string that does not escape all three quotes (`\"\"\"`).
- Flag: data tables and `Examples:` tables whose pipes don't line up in columns.
- Flag: a `Background:` with a title or description that only restates its steps.
- Flag (suggestion): visual or spatial state (editor content, a cursor position, a rendered table or panel)
  named by a word or a coordinate where a doc string showing it would let the reader see it, in setup and
  assertions alike.
- Flag (suggestion): a counter-example or variation placed before the main case it varies, under the same Rule or
  Ability.
