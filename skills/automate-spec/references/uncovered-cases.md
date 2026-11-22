# Adding uncovered cases

An uncovered case is a case the spec doesn't show, met while you work:

- behaviour the production code already has that no Example of the spec shows, such as a branch that turns down
  a negative quantity or an error for an unknown product;
- with `--implement`, a choice the spec leaves open, where the code has to do something but no Example says
  what, such as adding a quantity of zero.

Bring back only what you came across. Don't search the spec for missing edge cases: that is what a review's gap
findings are for (`review-spec`). With `--only-lines`, bring back only the cases of the selected part.

Only the Examples that weren't passing when the run started lead to uncovered cases. An Example that already
passed needs nothing from this run, so its code isn't looked at for them; when every selected Example already
passed, the run stops before looking for any (step 3 of the skill).

## Where each case goes

- **Under a Rule**, as an Example, when it is a case of that Rule's acceptance criterion: after the Example it
  varies or contrasts with, or else at the end of that Rule. In a spec without Rules, in the spec itself.
- **As a new Rule**, when no Rule's criterion covers it: after the Rule it relates to most, or else at the end of
  the spec.

Lay each one out like the rest of the spec: its indentation, and a blank line before it.

## Without steps (the default)

An Example gets a title and a description, and no steps. A Rule gets a title and a description, and no
Examples. The description holds what the steps would otherwise show, so the author can tell what the case
involves without them:

- what the case is: the situation, and what the user or the system does in it;
- what the system does in it now, as the code shows;
- what it should do instead, as a question, naming the outcomes the author could choose.

```gherkin
    Example: adding zero of a product
      the shopper asks to add a product with a quantity of 0.
      today the cart adds a line for it, with a quantity of 0 and a total of £0.00.
      should the cart keep that line, ignore the request, or turn it down with a message?
```

Write in the spec's own words, about what the user sees, not about the code: no class, method or field names.
Keep each line under 140 characters, and don't start one with a capitalised Gherkin keyword such as `When`,
`Then` or `And`, which the editor would read as a step. A Rule's title states its criterion, as far as the code
shows it, and its description says what is still open.

SpecBinder makes an Example without steps, or a Rule without Examples, fail as empty (or skip, as the spec's
`emptyScenarioBehavior` and `emptyRuleBehavior` say), so the case shows in every test run until the author writes
it. Once they add its steps, a review flags the description lines that only retell them, with a fix that removes
them.

## With steps (`complete-with-scenario-steps`)

Each case is a full Example: a title naming the situation and the `Given`, `When` and `Then` it needs, reusing
the spec's and the project's step phrasings word for word where one fits. A new Rule has a title stating its
criterion and an Example of its main case. It needs no long description.

Write a data table normally, its names in a header row. When a line of it would run past 140 characters and the
table turned around would fit, write it vertically instead: its first row opens with `!` in place of the first
`|`, the names run down the first column and each further column is one record. An `Examples:` table is always
written normally.

The `Then` states what the system does now. Where that is a crash or plainly unintended, such as an error thrown
from deep in the code, state the outcome you would propose instead, and say in the report that it is a proposal.

## In the spec file (the default)

Add the cases to the spec file yourself once the selected Examples are done: with `--implement`, after the last
Example and before the refactoring pass. Add them as
[changing-a-spec-file.md](../../review-spec/references/changing-a-spec-file.md) says: in the IDE's editor when
you can, all in one call, so the Rules and Examples the author folded stay folded. Nothing marks them as yours:
list them in the report by title. Then rebuild, so the generated class includes them.

Wire the ones with steps as step 5 of the skill says, then compile and run them. An Example whose `Then` states
what the system does now should pass; if it doesn't, fix the wiring. An Example with a proposed outcome fails as
"the system behaves differently from the spec": don't build what it proposes, even with `--implement`. Whether to
keep that outcome is the author's call.

## As gap review findings (`as-gap-review-findings`)

Publish each case to the spec's AI Review tab in IntelliJ IDEA as a `gap` finding, for the author to apply or
dismiss. The spec file stays as it is. This needs the SpecBinder plugin's tools on the IDE's MCP server
(`specbinder_get_review_scope`, `specbinder_get_spec_review` and `specbinder_publish_spec_review`). Without them,
add the cases to the spec file instead, and say so.

1. **Check the review.** Call `specbinder_get_review_scope` and `specbinder_get_spec_review` for the spec.
   Publishing replaces the spec's current review. If `gap` is among the kinds to skip, or the spec still has open
   findings, ask the author once, saying which. They can tick `gap` in the IDE, or let these findings replace the open
   ones, or both, and you publish; or you add the cases to the spec file instead.
2. **Read the spec** through the IDE's file tool, not from disk: the findings name lines as the editor has them.
3. **Publish** one finding per case, built as `review-spec` builds a gap finding: see step 5 of its
   [SKILL.md](../../review-spec/SKILL.md), and write each message as its "Writing for the author" says. Give each:
   - `kind` `gap`, `category` `uncovered-behaviour`, and no `check`, since no guideline raised it;
   - a fix that inserts the case where it goes, as above, without steps or with them;
   - a message saying what the case is and what the system does in it now, and, for a case without steps,
     asking what it should do.
4. **Correct and publish again** if the tool turns down any finding, with the whole set, since publishing replaces
   the review. A finding turned down because the author dismissed one like it there goes in the report instead.
