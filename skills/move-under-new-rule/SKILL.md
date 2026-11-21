---
name: move-under-new-rule
description: Move the Examples (Scenarios) of a SpecBinder spec file (.specb / .feature) that the given lines or titles pick out under a new Rule of their own in the same spec, titled with the acceptance criterion they show and placed after the Rule it is closest to — the Rule they leave is narrowed where it no longer fits what is left, and each Rule's Background is trimmed to what its own Examples need. Optionally checks it by running the spec's tests, or publishes the change as a review finding for the author to apply. Use when asked to move some Examples or Scenarios of a spec under a new Rule, give them a Rule of their own, or separate them from the rest of their Rule. To move Rules and Examples under a new spec of their own instead, use `move-under-new-spec`.
argument-hint: <path of a .specb or .feature file> [--only-lines=<lines> | titles of the Examples to move] [--verify] [--as-review-finding]
---

# Moving Examples under a new Rule

You take the Examples the author picks and give them a Rule of their own, in the same spec: a new Rule stating the
acceptance criterion they show. An Example's "original Rule" is the Rule it comes from; an Example at the spec's
own level, outside any Rule, has none.

Keywords in this skill stand for whatever the spec uses: `Ability:` for any top-level keyword (`Feature`,
`Ability`, `Business Need`, `Narrative`), `Example:` for `Scenario` too and for an outline, and each keyword in
the spec's own language when a `# language:` header sets one. `Given`, `When` and `Then` name a step's role: a
step written with `And`, `But` or `*` takes the role of the step before it.

To move Rules and Examples under a new spec of their own, in the same file, use `move-under-new-spec` instead.

## Options

- `--only-lines=<lines>`: the lines of the spec file that pick the Examples to move, such as `--only-lines=7` or
  `--only-lines=12-30`, counting from 1; several lines or ranges are separated by commas. The IDE's Move Under a New
  Rule sends the lines selected in the editor, or the caret's line with nothing selected. A request can instead name
  the Examples by title ("move the Example 'with the option off' under a new Rule"). Without either, ask what to
  move.
- `--verify`: check that nothing broke by running the spec's tests before and after, as
  [section 5](#5-verify-with---verify) says. Off by default.
- `--as-review-finding`: publish the change to the spec's AI Review tab in IntelliJ IDEA as a finding, for the
  author to apply or dismiss, instead of writing it to the file, as
  [section 6](#6-as-a-review-finding-with---as-review-finding) says. `--verify` doesn't apply then: nothing
  changes until the author applies it. Say so in the report when the request has it.

## 1. Pick the Examples

Work on one spec file. If the request names several, ask which one. Read the whole file.

With `--only-lines`:
- A line on an Example picks that Example, its lines running from the tags and comments just above its keyword
  line to its last line.
- Lines on anything else pick nothing: a Rule's own lines (its title, description or `Background:`), a block's
  header, the blank lines between parts. A selection that runs over a Rule's title on its way to the Examples
  below it picks just those Examples.
- If the lines run past the end of the file, or pick no Example, ask: the file may have changed since they were
  taken.

By title: a title picks the Example it names. If it names a Rule instead, matches no Example, or matches several,
ask.

The Examples must come from one top-level block, since a Rule lives in one. If they come from several, say so and
stop.

When the picked Examples are every Example of an original Rule, judge whether they show its criterion:
- when they do, they already have a Rule of their own: say so and stop. If Examples of other Rules are picked
  with them, ask instead whether those belong under that Rule;
- when they don't, they move, and the original Rule stays behind with its title and description and no Examples.
  By default SpecBinder turns a Rule without Examples into a failing test tagged `new`, so it shows as work to do.
  Name it in the report as needing an Example.

## 2. Write the new Rule

Give it:
- a title stating the acceptance criterion the picked Examples show, a statement with a subject and a verb,
  worded and laid out as the spec's other Rules are, such as a blank line after a title without a description;
- a description holding only what the title and the Examples don't carry: the original Rules' description lines
  the moved Examples need, such as a term they use, copied unchanged, and what the criterion needs that neither
  shows, such as a reason that isn't obvious. Otherwise, no description;
- a copy of the original Rule's `Background:` when the moved Examples rely on it, trimmed as section 3 says, its
  name, if it has one, narrowed to the steps left. When they come from several Rules whose `Background:`s differ,
  give it only the steps all of them rely on. If that leaves an Example without a step only it needs, say so and
  ask, rather than writing the step into the Example;
- the picked Examples, unchanged, each with its tags and the comments just above it, in the order they had in the
  file. An Example from the spec's own level moves one level deeper, two spaces, with every line of it, doc
  strings and tables included; the others keep their indentation.

Place it after the Rule whose criterion is closest to the new Rule's, as you judge it: the Rule whose Examples
share its situation and setup, or else its outcome. Say in the report where you put it and why. When no Rule is
clearly closest, put it right after the original Rule, or after the last of them. It never goes above an Example
at the spec's own level: Gherkin needs those to come before every Rule. Keep one blank line between it and the
parts around it.

## 3. Narrow the original Rules and trim the Backgrounds

**An original Rule's title and description** are narrowed to what is left when they name or imply what moved out,
such as a description that also speaks of the moved Examples' case, or an "only when …" whose other side they
show. A description line goes only when the moved Examples
needed it and nothing left under the Rule does. A line that nothing needed before the move stays. When you can't
tell whether a line is needed, keep it. A Rule left without Examples (section 1) keeps its title and description
as they are.

**Each Rule's `Background:`**, the new Rule's and the original Rules', keeps only the steps its own Examples rely
on:
- Trim a step only when it sets up something no step of those Examples touches or depends on, such as a file
  none of them opens. Keep it whenever in doubt, such as a setting or a plugin being available, which an
  Example's `When` may need without naming it.
- Remove the `Background:` when none of its steps are left.
- List every step you remove, Rule by Rule, in the report: no test run checks them unless `--verify` is given.

The spec's own, Ability-level `Background:` stays as it is: the moved Examples are still in the spec.

## 4. Write the file

With `--verify`, run the tests first, as [section 5](#5-verify-with---verify) says. With `--as-review-finding`,
publish the change as [section 6](#6-as-a-review-finding-with---as-review-finding) says instead of writing it.

Remove each moved Example with the blank line before it, so the parts left are separated by one blank line as
before, and write the new Rule where section 2 places it. The spec's step methods stay where they are, since its
steps are unchanged. The generated test classes and methods are named by position (`Rule_2`,
`rule_2_scenario_1`), so the new Rule renumbers every Rule after it, and each moved Example renumbers the
Examples after it in its Rule.

## 5. Verify, with `--verify`

Only for Java projects with JUnit 5 tests. Otherwise say the run is the author's, and skip this section.
[java-wiring.md](../automate-spec/references/java-wiring.md) says how a spec's test class is found and how to
build and run it; read it first. If the spec has no test class yet, it isn't wired: say so and skip the run.

1. **Before writing the file**, rebuild and run the spec's test class, and note which Examples pass, by their
   titles, with their Rule's title where an Example's title repeats: the test methods are named by position, and
   the move renumbers them. Match each Example to its result after the move by where you moved it, since a
   narrowed Rule title changes too.
2. **After writing it**, rebuild and run the test class again.
3. **An Example that passed before and fails now** most likely lost a `Background:` step it relied on. Put the
   step back in its Rule's `Background:`, and run again until those Examples pass. Name every step you put back in
   the report, and drop it from the list of removed steps. If no removed step explains a failure, name the
   Example and its failure in the report.
4. **An Example that failed before** is left as it is. Name it in the report.

## 6. As a review finding, with `--as-review-finding`

Publish the whole change as one finding, for the author to apply or dismiss in the IDE. The spec file stays as it
is. This needs the SpecBinder plugin's tools on the IDE's MCP server (`specbinder_get_review_scope`,
`specbinder_get_spec_review` and `specbinder_publish_spec_review`), used as the `review-spec` skill uses them.
Without them, write the change to the file instead, and say so.

1. **Check the review.** Call `specbinder_get_review_scope` and `specbinder_get_spec_review` for the spec.
   Publishing replaces the spec's current review. If `split` is among the kinds to skip, or the spec still has open
   findings, ask the author once, saying which. They can tick `split` in the IDE, or let this finding replace the
   open ones, or both, and you publish; or you write the change to the file instead.
2. **Read the spec** through the IDE's file tool, not from disk: the finding names lines as the editor has them.
3. **Publish** one finding, built as `review-spec` builds a `split` finding: see step 5 of its
   [SKILL.md](../review-spec/SKILL.md), and write the message as its "Writing for the author" says. Give it:
   - `line` and `quote`: the original Rule's line and title, when every moved Example comes from that Rule and the
     new Rule goes right after it; otherwise the spec's `Ability:` line and title, since only a finding there can
     change lines anywhere in the block;
   - `kind` `split`, `category` `move-under-new-rule`, and no `check`, since no guideline raised it;
   - a fix of whole-lines edits, one per place the change touches: each moved Example removed, each original
     Rule's title or description narrowed, each `Background:` trimmed, and the new Rule inserted after the last
     non-blank line of the Rule it follows, that line kept. When that Rule's last Example moves out, one edit
     removes it and inserts the new Rule in its place. The edits must not overlap;
   - a message saying which Examples move under a new Rule and the criterion they show that their Rule doesn't,
     and then, in a paragraph of its own, what the fix does: the new Rule's title, where it goes, what the original
     Rules lose, any Rule left without Examples, and the `Background:` steps it removes.
4. **Correct and publish again** if the tool turns the finding down.

## 7. Report

Give:
- the new Rule's title, where it sits and why there, and the Examples it holds, by title;
- any Rule left without Examples, which needs one;
- what changed in the original Rules' titles and descriptions, and the description lines narrowed, copied or
  removed;
- every `Background:` step removed, or that none were, and any `Background:` copied whole into the new Rule, which
  now appears twice;
- with `--verify`, the Examples' results and any step put back; without it, that no tests were run;
- with `--as-review-finding`, the finding's number, and that the author applies it in the AI Review tab.
