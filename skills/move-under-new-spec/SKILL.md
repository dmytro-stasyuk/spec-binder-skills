---
name: move-under-new-spec
description: Move the Rules and Examples (Scenarios) of a SpecBinder spec file (.specb / .feature) that the given lines or titles pick out under a new top-level Ability (or Feature) of their own in the same file, with a title and user story written for them — the original spec's header is narrowed where it no longer fits what is left, each spec's Background is trimmed to what its own Examples need, and both are linked by Out of scope lines. Optionally checks it by running the spec's tests, publishes the change as a review finding for the author to apply, or goes on to move the new spec into a file of its own. Use when asked to move some Rules or Examples of a spec under a new spec, a new Ability or Feature, give them a spec of their own, or separate them from the rest of their spec within the same file. To move whole specs into files of their own, use `separate-out-specs` instead; to move Examples under a new Rule of the same spec, use `move-under-new-rule`.
argument-hint: <path of a .specb or .feature file> [--only-lines=<lines> | titles of the Rules and Examples to move] [--verify] [--as-review-finding] [--own-file]
---

# Moving Rules and Examples under a new spec

You take the Rules and Examples the author picks out of a spec and give them a spec of their own: a new top-level
block in the same file, with its own title and user story. Below, "block" means one top-level `Ability:` with
everything under it, and the "original" is the block the picked parts come from.

Keywords in this skill stand for whatever the spec uses: `Ability:` for any top-level keyword (`Feature`,
`Ability`, `Business Need`, `Narrative`), `Example:` for `Scenario` too and for an outline, and each keyword in
the spec's own language when a `# language:` header sets one. `Given`, `When` and `Then` name a step's role: a
step written with `And`, `But` or `*` takes the role of the step before it.

Moving the new block into a file of its own is a later step, which `separate-out-specs` takes; see `--own-file`.
To move Examples under a new Rule of the same spec instead, use `move-under-new-rule`.

## Options

- `--only-lines=<lines>`: the lines of the spec file that pick the parts to move, such as `--only-lines=7` or
  `--only-lines=12-30`, counting from 1; several lines or ranges are separated by commas. The IDE's Move Under a
  New Spec sends the lines selected in the editor, or the caret's line with nothing selected. A request can instead
  name the Rules and Examples by title ("move the Rule 'refunds go back to the card' under a new spec"). Without
  either, ask what to move.
- `--verify`: check that nothing broke by running the spec's tests before and after, as
  [section 5](#5-verify-with---verify) says. Off by default.
- `--as-review-finding`: publish the change to the spec's AI Review tab in IntelliJ IDEA as a finding, for the
  author to apply or dismiss, instead of writing it to the file, as
  [section 6](#6-as-a-review-finding-with---as-review-finding) says. `--verify` and `--own-file` don't apply then:
  nothing changes until the author applies it. Say so in the report when the request has them.
- `--own-file`: once the new block is written, go on to move it into a spec file of its own, following
  `separate-out-specs` for that one block.

## 1. Pick the parts

Work on one spec file. If the request names several, ask which one. Read the whole file.

With `--only-lines`:
- A line on an Example picks that Example, its lines running from the tags and comments just above its keyword
  line to its last line.
- A line on a Rule's own lines before its first Example, its tags, title, description or `Background:`, picks the
  whole Rule with all its Examples. A Rule without Examples is picked by its own lines.
- Lines on anything else, such as a block's own header or the blank lines between parts, pick nothing. If the
  lines run past the end of the file, or pick nothing at all, ask: the file may have changed since they were
  taken.

By title: a title picks the Rule or Example it names, as a line on it would. If a title matches no Rule or Example,
or matches several, ask.

The parts may come from more than one block of the file. All of them go into the one new block. If they would
leave a block with no Rule or Example, say so and stop: moving everything under a new header is rewording that
block, not separating anything out.

## 2. Place the parts in the new block

Each part keeps its tags and the comments just above it, and its content unchanged but for the `Background:`
trimming section 3 describes. In the new block, the parts keep the order they had in the file.

- **A Rule picked by its own lines** moves whole: its title, description, `Background:` and every Example.
- **An Example of a spec without Rules**, or one at the block's own level beside its Rules, stays at the new
  block's own level.
- **Examples picked without their Rule's own lines** go under the Rule of the new block whose acceptance criterion
  they show: a Rule picked by its own lines, when one fits; otherwise a new Rule titled with the criterion they
  show, a statement with a subject and a verb. The new Rule keeps the original Rule's title when that still states
  it, and takes the original Rule's `Background:` when its Examples rely on it, trimmed as section 3 says. Put a
  new Rule where its first Example was in the order of the parts.
- **When those Examples are every Example of their Rule**, judge whether they show the Rule's criterion:
  - when they do, the Rule moves whole with them, as if picked by its own lines. A selection inside the one
    Example of a Rule about that same case moves the Rule too;
  - when they don't, they go under a Rule as above, and the original Rule stays behind with its title and
    description and no Examples. By default SpecBinder turns a Rule without Examples into a failing test tagged
    `new`, so it shows as work to do. Name it in the report as needing an Example.

## 3. Write the two headers

**The new block** goes right after the original, or after the last of them when the parts come from several,
separated from the blocks around it by one blank line. Comments above the file's first block, such as `# ADR:`
lines, stay where they are, even one that concerns the new block: `separate-out-specs` sorts them out if the block
later moves into a file of its own. Give it:
- the original's keyword, and its tags, all of them. Name in the report any tag that looks like it belongs to one
  of the two blocks only;
- a short title naming what the picked parts are about, worded as the titles of the specs beside it are;
- a user story for those parts, each on its own line: `As a` the role who acts in the picked Examples, which is
  the original's role when it is the same one; `I want` the capability their Rules add up to; and `So that` what
  that gives the role, without repeating the `I want`;
- after a blank line, the description lines of the original that the picked parts need, unchanged: the legends
  for pictures their steps draw, the terms their steps, titles or descriptions use, and the `Out of scope` lines
  about what borders on what they show, with the heading of the list such a line is in;
- a copy of the original's Ability-level `Background:`, when the picked Examples rely on it, trimmed as below.

**Each block points at the other** with an `Out of scope` line naming the other block by its title, since both
are in one file: as the last item of the block's `Out of scope:` list when it has one, or else as an
`Out of scope: … — see <title>` line at the end of its description. Word it as the spec's existing `Out of scope`
lines are.

**The original's title and story** are narrowed to what is left when they name what moved out, such as an
`I want` that names it. A story that what is left still serves stays as it is. When the title changes, update any
`Out of scope` line in another spec file that names the original by that title.

**The original's description** loses a line only when the moved parts needed it and nothing left in the original
does. A line that nothing needed before the move stays where it is. When you can't tell whether a line is needed,
keep it.

**Each `Background:`** of either block, at Ability or Rule level, keeps only the steps its own Examples rely on:
- Trim a step only when it sets up something no step of those Examples touches or depends on, such as a file
  none of them opens. Keep it whenever in doubt, such as a setting or a plugin being available, which an
  Example's `When` may need without naming it.
- Remove the `Background:` when none of its steps are left, and narrow its name, if it has one, to the steps left.
- List every step you remove, block by block, in the report: no test run checks them unless `--verify` is given.

## 4. Write the file

With `--verify`, run the tests first, as [section 5](#5-verify-with---verify) says. With `--as-review-finding`,
publish the change as [section 6](#6-as-a-review-finding-with---as-review-finding) says instead of writing it.

Remove each moved part from the original with the blank line before it, so the parts left are separated by one
blank line as before, and write the new block where section 3 places it. The spec's step methods stay where
they are: a file with several blocks generates one test class, whose steps all its blocks share.

With `--own-file`, go on to move the new block into a spec file of its own: follow `separate-out-specs` for the
one file, with the new block's title as the block to move. It rewrites the `Out of scope` links to name the new
file, and in a Java project moves the step methods and rebuilds.

## 5. Verify, with `--verify`

Only for Java projects with JUnit 5 tests. Otherwise say the run is the author's, and skip this section.
[java-wiring.md](../automate-spec/references/java-wiring.md) says how a spec's test class is found and how to
build and run it; read it first. If the spec has no test class yet, it isn't wired: say so and skip the run.

1. **Before writing the file**, rebuild and run the spec's test class, and note which Examples pass, by their
   titles, with their Rule's title where an Example's title repeats: the test classes and methods are named by
   position, and the new block renumbers them. Match each Example to its result after the move by where you moved
   it.
2. **After writing it**, and after `separate-out-specs` with `--own-file`, rebuild and run the test classes again,
   both of them when the new block has its own.
3. **An Example that passed before and fails now** most likely lost a `Background:` step it relied on. Put the
   step back in the `Background:` of the Example's block, or of its Rule, and run again until those Examples
   pass. Name every step you put back in the report, and drop it from the list of removed steps. If no removed
   step explains a failure, name the Example and its failure in the report.
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
2. **Check that one finding can hold it.** A fix changes lines within the block its finding is on only. When the
   parts come from more than one block, say so and ask whether to write the change to the file instead.
3. **Read the spec** through the IDE's file tool, not from disk: the finding names lines as the editor has them.
4. **Publish** one finding, built as `review-spec` builds a `split` finding: see step 5 of its
   [SKILL.md](../review-spec/SKILL.md), and write the message as its "Writing for the author" says. Give it:
   - `line` the original's `Ability:` line and `quote` its title; `kind` `split`, `category`
     `move-under-new-spec`, and no `check`, since no guideline raised it;
   - a fix of whole-lines edits, one per place the change touches: the original's header, each `Background:`
     trimmed, each part moved out, and the new block inserted after the original's last non-blank line, that line
     kept. When the original's last part moves out, one edit removes it and inserts the new block in its place.
     Start the header's edit on the `Ability:` line, so it takes in the finding's line. The edits must not
     overlap;
   - a message saying which Rules and Examples move under the new block, and what the original's header loses,
     and then, in a paragraph of its own, what the fix does: the new block's title and story, any Rule written for
     Examples that left theirs, any Rule left without Examples, and the `Background:` steps it removes.
5. **Correct and publish again** if the tool turns the finding down.

## 7. Report

Give:
- the new block's title and story, and the Rules and Examples it holds, by title;
- any Rule written for Examples that left theirs, and any Rule left without Examples, which needs one;
- what changed in the original's title, story and description;
- the description lines copied or removed, and any tag that looks like it belongs to one block only;
- every `Background:` step removed, or that none were;
- with `--verify`, the Examples' results and any step put back; without it, that no tests were run;
- with `--as-review-finding`, the finding's number, and that the author applies it in the AI Review tab.

When the original's title changed and the file was named after it, say that the name no longer fits, and suggest
one after its new title, for the author to give it with the IDE's Rename, which renames the spec's test classes
with it. Don't rename it yourself.

Without `--own-file`, offer to move the new block into a file of its own with `separate-out-specs`.
