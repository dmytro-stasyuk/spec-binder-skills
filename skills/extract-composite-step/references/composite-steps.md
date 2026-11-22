# Composite steps

How SpecBinder's composite steps work, and how to put imperative steps under a new composite step without changing
what the test does. Shared by `extract-composite-step` and `review-spec`.

## What a composite step is

With `@Gherkin2JUnitOptions(enableCompositeSteps = true)` on a spec's marker class, or on a class above it, a step
with any keyword but `*` that is directly followed by `*` steps is a **composite step**, and the `*` steps are its
**sub-steps**. The option is inherited and off by default; without it, `*` steps are ordinary
steps of their own. [java-wiring.md](../../automate-spec/references/java-wiring.md) says how a spec's marker class
is found and which options apply to it.

```gherkin
Given the cart holds "2" of "Notebook"
* search the catalogue for "<$p2>"
* add "<$p1>" of the first result to the cart
```

- Composite steps work in Scenarios, Scenario Outlines and Backgrounds. A sub-step can't have sub-steps of its own.
- The generated class gets a **composite method** for the parent step, which runs its sub-steps. It needs no
  implementation.
- A use of the parent's wording without sub-steps is an ordinary step. Its step method is an overload of the
  composite method, and neither calls the other.
- Each sub-step gets the step method it would get written as a step of its own. The keyword is dropped from the
  method's name, and a reference stands where the value was, so `* search the catalogue for "<$p2>"` calls the same
  `searchTheCatalogueFor$p1` as `When search the catalogue for "Notebook"`. A value keeps the type it has on the
  parent step: `"2"` reaches the sub-step as an `Integer`.
- A parent step takes at most five values, its data table or doc string counting as one. A reference that matches
  nothing fails the build, listing the references there are.

A sub-step refers to its parent step's values like this:

| In a sub-step | Stands for |
|---|---|
| `"<$p1>"` … `"<$p5>"` | the parent step's first to fifth quoted value |
| `"ORD-<$p1>"` | a value made of the parent's value and other text |
| `"<$name>"`, `"<$name[2]>"` | the `name` cell of the parent's data table: of its only data row, or of data row 2, counting from 1 below the header row. In a vertical table: the value next to `name`, in its only value column or in value column 2, counting from 1 after the name column |
| `"<$dataTable>"` at the end of the text | the parent's whole data table, taken as the sub-step's own |
| `"<$docString>"` at the end of the text | the parent's doc string, taken as the sub-step's own |

In a step's text a reference is written in double quotes; outside them it is plain text. In a sub-step's own data
table, a reference fills a cell or part of one, without quotes (`| qty | <$p2> |`, `| ORD-<$p1> |`), but never a
header cell or a vertical table's name cell.

## Which steps go under one parent

A step is **imperative** when it says how the system is driven or inspected rather than what is true in business
terms, through whatever interface: a screen (clicking, typing into a field, opening a page, what a page shows), an
HTTP request or response, a command line, a database row, a queue message, a file. A check in the project's review
guidelines that flags such steps refines this: follow its checks and guidance where it says more. In the starter
guidelines these are `declarative-steps`, for a procedure walked through step by step, and `implementation-detail`,
for UI mechanics or a technical mechanism in a step.

A step is imperative too when it is one of several moves that together make up one action, even when its words are
business words: `confirm the delivery address` and `pay with the saved card` are moves of a checkout. Such a move is
usually written as an instruction to whoever runs the Example, verb first or with "I": `open the cart`, `I confirm
the order`. The fact is the action the moves make up (`the customer checks out`), not each move. A business-worded
step that stands alone, such as the only `When` of an Example, states a fact.

Put imperative steps under one parent step per business fact:

- **One fact.** A group is a run of steps next to each other that together set up one fact, carry out one action, or
  check one outcome: "Alice is signed in", "the customer checks out", "the order is placed". A run that does two
  becomes two groups. A single imperative step can be a group of its own.
- **One role.** A group never crosses from `Given` to `When`, or from `When` to `Then`. A check that only confirms an
  action worked, such as `And I should see "Welcome, Alice"` after signing in, stays in that action's group: it is
  part of how the action is done. Imperative checks in the `Then` part become a `Then` composite step of their own.
- **What stays.** Steps that already state a business fact stay as they are, and so do composite steps and their
  sub-steps. A group stops at them.

## Nesting steps without changing the test

Putting steps under a new composite step is a pure refactoring: each sub-step, its references filled in from the
parent, reads as the step it was, so the test still calls the same step methods, in the same order, with the same
values. No step method is added, renamed or changed. [Checking the sub-steps](#checking-the-sub-steps) says how to
make sure of it.

### The sub-steps

- Each step the parent replaces becomes a sub-step: `* ` and then its text as written, without its keyword. Nothing
  else about it changes: not its wording, nor its place among the others, its own data table or doc string, or the
  comments between the steps.
- A value of it that the parent now states becomes a reference to the parent's value. When the parent states part of
  a value, the rest stays around the reference: `"ORD-<$p1>"`. An Examples placeholder written without quotes
  (`<code>`) becomes a quoted reference (`"<$p1>"`), which keeps the same step method.
- A cell of its data table that the parent now states becomes a reference.
- When its whole data table or doc string moves up to the parent, `"<$dataTable>"` or `"<$docString>"` goes at the
  end of its text, after a space: `* add the products to the cart: "<$dataTable>"`.
- Sub-steps take the parent step's indentation. Their own data tables and doc strings keep theirs, relative to the
  sub-step.

### The parent step

- **Where.** In place of the first step it replaces, at its indentation, followed by its sub-steps.
- **Keyword.** The one the first step it replaces is written with: `Given`, `When`, `Then`, `And` or `But`. When that
  step is written with `*`, use the keyword of its role, or `And` after a step of the same role.
- **Wording.** One business fact, in the spec's own words: those of its story, its Rule titles and its other steps.
  For a `Given`, what is true (`"alice@example.com" is signed in`); for a `When`, what happens (`the customer checks
  out`); for a `Then`, the outcome (`the order is placed`). No screens, buttons, fields, endpoints, commands or
  database tables. A parent with a data table or doc string leads into it, ending with a colon, as part of the fact
  it states: `Given the cart holds:` over the products and their quantities, `Given the order has the gift note:`
  over the note. A table that moves up whole keeps its header row, since the sub-step takes it as its own. When this
  spec or another spec file in its folder has a step stating the same fact, reuse its wording word for word: where it
  is used without sub-steps, it keeps its own step method. A value has one type across every use of a wording, so
  when those uses hold a value of another type than the sub-steps take, such as `"A-7"` where they take
  a number, word the parent another way: the sub-step's method would change type.
- **Values.** By default, the parent states the values the Example depends on, and no others:
  - Those a later step, the Example's title, its Rule or the spec relies on, and those that make this Example differ
    from the others under its Rule, or in the spec when it has no Rules. An Examples placeholder always counts.
  - Labels stay in the sub-steps: the names of fields, buttons, links, pages, menus, endpoints and columns. So does
    data the outcome doesn't depend on, such as the password in an Example about something else.
  - Every value the parent states, quoted or in its data table, reaches at least one sub-step through a reference,
    and every sub-step that has it, in its text or in its own table: under `Given the cart holds:`, with Notebook in
    its first row, `* the first cart line should be for "<$product[1]>"`. A sub-step whose text only happens to
    match a value, such as a page number `"2"` next to a quantity of 2, keeps its own. Never state a value that no
    sub-step receives: changing it would change nothing the test does. When the fact's subject has no matching value
    in the sub-steps, such as "Alice" where the sub-steps type `alice@example.com`, write it without quotes
    (`Given Alice is signed in`), so that it isn't a value, or state the value the sub-steps have.
  - Copy each value exactly as the sub-steps have it, so it keeps its type: `"2"` stays `"2"`, never `"2.0"`.
  - **The same steps in several places.** When the same steps (the same step methods, in the same order) go under a
    parent in more than one of the places given, those places share one parent: worded the same, stating every value
    that any of their Examples depends on, each place with its own values. An Example about a gift note gets
    `Given the cart holds "1" of "Backpack"`, because another Example's total depends on the quantity and the
    product. A place where that wording isn't true of it, such as a failed sign-in among sign-ins, gets a parent of
    its own.
  - With **all parameters** (`extract-composite-step`'s `--move-all-parameters-up`), the parent states every quoted
    value of the steps it replaces, labels included, and takes over every data table and doc string they have.
- **Data tables, row by row.** The rows of a sub-step's table that the Example depends on move up: as quoted values
  when there are few, or into the parent's own table. The sub-step keeps its table, with those cells as references
  to the parent's. The whole table moves up, leaving `"<$dataTable>"` on the sub-step, only when every row matters,
  or with all parameters. A doc string can't be split: it moves up whole, leaving `"<$docString>"`, when the Example
  depends on it, and otherwise stays.
- **More than five values.** Put every value the parent states in one vertical data table on the parent, in place of
  quoted values, and word the parent to introduce it (`Given Alice has an account with:`). Name each value after
  what it is, often the label it was typed into, in one word, never `p1` to `p5`; the sub-steps refer to it as
  `"<$email>"`. Vertical tables need SpecBinder 2026.52.0 or later. With an older SpecBinder, write the table with a
  header row and one row of values: `"<$email>"` refers to its cells the same way. By default, needing more than
  five values usually means the steps set up more than one fact: consider two groups first.
- **One table or doc string.** A step has at most one data table or one doc string. When more than one would move up,
  put the rows of those that hold one record each (one row of values, or one value column) together in the parent's
  vertical table under distinct names, each sub-step's table referring to them cell by cell. A doc string, or a
  table of several records, that can't join them stays on its sub-step, as it is.

Before and after, by default:

```gherkin
Given I open the "sign-in" page
And I enter "alice@example.com" into "Email"
And I enter "secret" into "Password"
And I click "Sign in"
```

```gherkin
Given "alice@example.com" is signed in
* I open the "sign-in" page
* I enter "<$p1>" into "Email"
* I enter "secret" into "Password"
* I click "Sign in"
```

With all parameters, the parent states six values, so they go in a vertical table:

```gherkin
Given Alice is signed in with these details:
  ! page          | sign-in           |
  | email         | alice@example.com |
  | emailField    | Email             |
  | password      | secret            |
  | passwordField | Password          |
  | button        | Sign in           |
* I open the "<$page>" page
* I enter "<$email>" into "<$emailField>"
* I enter "<$password>" into "<$passwordField>"
* I click "<$button>"
```

### Checking the sub-steps

Before writing, fill each sub-step's references back in from its parent:

- Each reference takes the value it stands for: `"<$p1>"` the parent's first quoted value, `"<$email>"` its `email`
  cell, and `"ORD-<$p1>"` becomes `"ORD-"` followed by that value. The references in a sub-step's own data table are
  filled in the same way.
- A sub-step ending in `"<$dataTable>"` or `"<$docString>"` drops it and takes the parent's data table or doc string
  back under it.

Each sub-step must then read exactly as the step it was, without its keyword, but for an Examples placeholder
written without quotes, which comes back quoted (`<code>` as `"<code>"`). Every step the parent replaces comes back
once, in its place, and nothing else in the Example changes. A sub-step that doesn't has a reference to the wrong
value, or a value copied wrong: `search the catalogue for "2"` where the step was
`search the catalogue for "Notebook"`, or a doc string that has lost a line. Correct it, and check again.
