# Changelog

All notable changes to the SpecBinder skills plugin for coding agents will be documented in this file.
The format is based on [Keep a Changelog](https://keepachangelog.com),
and this project adheres to [Semantic Versioning](https://semver.org).

## Unreleased

### Added

- Each review finding now names the guideline check that flagged it, so in IntelliJ IDEA its card in the AI Review
  tab links to that check: hover to read it, click to open the guideline file on it and tune it. Needs
  SpecBinder IntelliJ plugin 2026.20.0 or later.
- The plugin can also be installed in Codex, from the same repository, with `codex plugin marketplace add` and
  `codex plugin add`. Its skills no longer assume Claude Code.
- New `automate-spec` skill for Java projects wires a spec to a JUnit 5 test and writes its steps against the
  production code.
- New `restructure-spec-folders` skill restructures the spec folders, with their test code, so they reveal what
  the system does: by business capability by default, or by a principle you give, or as a tidy-up of the current
  tree.
- When the gap kind is ticked, a review also looks for missing edge cases in each Rule and offers the missing
  Examples as fixes. Projects can add their own edge cases in their guidelines.
- New `split-spec` skill moves the top-level Features of a spec file that holds several into spec files of
  their own, rewriting the links between them, and in a Java project creates their test classes and sorts the
  step methods between them, rebuilding until everything compiles.
- A split finding's fix now leaves the spec fully split: an Example becomes complete Examples, a Rule becomes
  narrow Rules with its Examples shared out between them, and a spec covering several aspects becomes several
  Features in the same file, each with its own title, story and Rules.
- When you disagree with a finding, the agent offers to add your reason to the project's review guidelines.
- A review doesn't run while a section of your guidelines has no `Kind:` line, or a kind isn't one of the
  eight. The agent says where it is and offers to correct it.
- The starter guidelines include SpecBinder's layout and wording conventions, under a `format` category, mostly
  as polish.
- The starter guidelines also flag the Examples of one Rule that trigger different events, an Example whose
  `When` isn't the capability the spec is about, a `Given` the outcome doesn't depend on, a Rule whose many
  Examples vary in more than one way, and sibling Example titles that don't tell the Examples apart.
- **⚠️ BREAKING:** Each finding now also gets a card in the spec's AI Review tab in IntelliJ IDEA, with its
  proposed fix as a diff and buttons to apply or dismiss it. The tab is on unless you turn it off under
  Settings › SpecBinder › Split view. Needs SpecBinder IntelliJ plugin 2026.20.0 or later.
- Every finding is one of eight kinds: blocker, issue, split, gap, suggestion, polish, question or praise. Each
  kind has its own look, count and Problems view group in IntelliJ IDEA. The starter guidelines give each category a
  kind and a few checks their own, and each finding has the kind of the check it comes from. A project's
  guidelines can set a category's kind with a `Kind:` line under its heading, or a single check's as
  `- Flag (polish): ...`.
- You choose which kinds each spec's review reports, from the spec's toolbar or its AI Review tab in IntelliJ
  IDEA. The agent reports only the ticked ones, and unticking a kind sets its current findings aside until you
  tick it again. Naming kinds in your request overrides the ticks for that review. Findings are numbered top to
  bottom as they appear in the spec.
- A review asked for with IntelliJ IDEA's Review This Spec button checks that the Claude Code session is connected
  to that same IDE. If it's connected to another one, such as a second IDE window running a sandbox, the agent
  reviews nothing and tells you how to reconnect it, rather than publishing findings to an IDE you aren't looking
  at.
- A finding you dismiss in IntelliJ IDEA stays out of later reviews of the spec until you edit the text it was
  about or restore it, however many times the spec is reviewed again.
- A finding whose fix reaches beyond its quoted text, such as splitting an example in two, can now come with a
  rewrite of whole lines, which you can review as a diff and apply in one step. If you edit those lines while
  the agent is still reviewing, the finding is left out, rather than offering a rewrite that would undo your
  edit.
- Project review guidelines can be split across several `.md` files in a `.specbinder/review-guidelines/`
  directory, subdirectories included, alongside or instead of `.specbinder/review-guidelines.md`. Files are read
  in path order, and where two disagree, the later one wins.
- You can switch a guideline off by commenting it out with IntelliJ IDEA's line-comment shortcut
  (`[//]: # (…)`); a review ignores commented-out lines. Commenting out a check's first line switches off the
  whole check.
- A project's `.specbinder/review-guidelines.md` can define its own review categories: a ``## `name` `` heading
  followed by "Flag:" checks, in the same format as the starter guidelines. Findings for those checks show that
  category name in the spec editor.
- The starter guidelines also flag spec text that is hard to read in plain terms: an official or legal voice
  ("shall"), filler such as "successfully", double negatives, and a passive `When` that hides who acts. They flag
  a Rule title that names a topic rather than stating what holds, and sibling Example titles phrased in
  different shapes.
- A new `duplication` starter category flags a spec that says the same thing twice: two Examples of the same
  case, two Rules stating one criterion, two blocks covering one aspect, and a spec that another spec file already
  covers. Each fix removes or merges the repeat. The checks for a repeated `Then` and for repeated `Given`s now
  live in this category too, and it suggests a Scenario Outline for Examples that differ only in their values.
- The Rules, Examples and other parts of a spec that a finding's message names are links on its card in IntelliJ
  IDEA: the quoted title is the link, and a click puts the cursor at the start of that title in the spec, or in
  the other spec file it is in, however the spec has changed since the review. Needs SpecBinder IntelliJ plugin
  2026.20.0 or later.
- A missing Example the review finds is now marked on the title of the Rule it belongs under, or on the spec's
  title when the spec has no Rules, rather than on the last line of the Example it would follow, and is placed
  under the Rule whose criterion it shows. A missing case that no Rule's criterion covers comes as a new Rule,
  marked on the spec's title. Needs SpecBinder IntelliJ plugin 2026.20.0 or later.
- The starter guidelines flag Rule description lines that only repeat the Rule's title, retell what its Examples
  show or add nothing a reader needs, as a suggestion whose fix removes them. This replaces the check for a
  description that only restates the title.
- A gap finding now always comes with the missing Example or Rule as its fix, even when what should happen is
  yours to decide: its `Then` is the review's proposal, from what the system does now where the agent can tell,
  and the message names the other outcomes you could choose. Needs SpecBinder IntelliJ plugin 2026.20.0 or later.
- The starter guidelines flag a spec file whose name doesn't say what the spec is about, and propose a new name
  after the spec's title. A name that is an issue tracker's key, such as `BR-123.specb`, is left as it is, and a
  new name keeps any key the old one has. With SpecBinder IntelliJ plugin 2026.20.0 or later, applying the fix
  opens the IDE's Rename dialog on the new name, which renames the spec's test code with it; with an older
  plugin, the finding gives the new name in its message.

### Changed

- A finding's message now leads with its point, so in IntelliJ IDEA the AI Review tab can show that sentence
  first and the rest of the message under it, as detail that is easy to skip when scanning the cards.
- **⚠️ BREAKING:** Reviews no longer apply built-in guidelines. A spec is checked only against the project's own
  guidelines in `.specbinder/review-guidelines.md` and `.specbinder/review-guidelines/`, and a project without
  any gets no review until it has some. The agent offers to set them up from the starter guidelines, all of them
  or the categories you pick, copied into `.specbinder/review-guidelines/` for you to edit. A project's guideline
  files no longer change or remove default checks, since there are none: repeating a category's heading only
  matters within the project's own files. Every section of the guidelines needs a `Kind:` line. Only a heading with
  a name in backticks makes a section a category; findings from any other section show no category.
- Every finding now comes with a proposed fix you can apply, except questions and praise. When the agent can't
  propose one, the summary says why.
- In the starter guidelines, an Example that belongs under another Rule of the spec, by its acceptance criterion
  or by the event its `When` triggers, is now an issue whose fix moves it there. It is a split only when no Rule
  covers it yet, and its fix starts a Rule of its own.
- When a guideline compares the spec with the project's other specs, such as the starter check for a spec that
  covers an aspect another spec file already covers, a review now looks through the project's other spec files
  first, so it finds such a spec rather than only one it happened to know about.
- The starter guidelines also ask about a Rule or Example that another spec file of the project already has,
  naming that file and its Rule or Example for you to decide which spec it belongs in.
- Review findings are now written in plain words: each names the Rules and Examples it is about by their quoted
  title, rather than by a shorthand of its own or a line number that goes stale once an earlier fix moves
  lines, and may take up to three short sentences where one couldn't say it clearly. The summary in the chat reads
  the same way.
- A finding with a fix now also says in plain words what the fix does, such as which Rule and Examples it removes
  or where it moves an Example, so you can tell what applying it will change without working it out from the diff.
  It starts on a line of its own, below the problem, so you can find it at a glance.
- In the starter guidelines, a `Then` that only repeats what other Examples of the spec already show is now a
  suggestion whose fix removes it, rather than a split. An unrelated `Then` is a split only when no other Example
  shows it, and its fix makes it an Example of its own.
- A review now proposes splitting a spec into several in the same file more readily: it reads each Rule against
  the spec's title and user story, and flags Rules the story doesn't ask for, or a spec of more than about five
  Rules, or a Rule of more than about five Examples, that answer different questions, such as what each case shows
  and how results combine. The split also moves an Example to the aspect it shows, even when its Rule goes
  elsewhere.
- In the starter guidelines, a long spec of more than about five Rules or twenty Examples now gets a split finding
  even when all its Rules stay on the one aspect its header states, when they fall into groups that each cover a
  narrower part of it. The fix makes each group a spec of its own, with its own title and story.
- `restructure-spec-folders` now starts on more ways of asking for it: reorganise or reorganize, organise, group, sort,
  clean up, rearrange or move your specs or their folders, as well as restructure and tidy.
- `restructure-spec-folders` no longer runs your tests before it moves anything; it only checks that the project
  builds. Afterwards it checks that every scenario of every moved spec still runs, exactly once, and also runs the
  tests that use any shared code or files it moved. It finds and fixes leftover references to old class and
  package names, in run configurations and CI files too, and fixes the failures the move caused. Any other
  failure is listed for you to confirm, and the restructure is reported as done only once every test passes or
  you have.
- `restructure-spec-folders` now shows the layout it proposes as the folder tree it would become, with what each
  folder covers, which folders are new, and where each spec comes from, rather than a table or a one-line
  description. When it offers you a choice of layouts, it shows each one as a tree.
- Polish and suggestion findings are now told apart by what they change. Polish is the surface only, such as
  layout, punctuation or wording that doesn't change what the spec says, quick to apply. A suggestion changes
  what the spec says or how it is put together. In the starter guidelines, a missing `Given`, `When` or `Then`,
  one concept named differently, jargon, a passive `When` and a counter-example placed before its main case
  are now suggestions, and a Rule whose title and description run past three lines is now polish.

### Fixed

- **⚠️ BREAKING:** A finding that an Example belongs under another Rule now always comes with a fix that moves
  it there, which you can review and apply in one step. Before, such findings could arrive with no fix at all.
  Needs SpecBinder IntelliJ plugin 2026.20.0 or later.
- A fix that rewrites or removes a description no longer takes the blank line after it too, which ran the
  description or the Rule's title into the Example below once the fix was applied.

### Removed

## [0.1.0] - 2026-09-17

### Added

- The `/specbinder:review-spec` command reviews a spec file against spec-writing guidelines — written from the user's point of view, describing what the system does rather than how it is built — and shows each finding as a warning on the text it is about in the IntelliJ IDEA spec editor. From there you can ask Claude to explain or fix a finding, apply its suggested wording, or dismiss it.
- A project can adjust the review guidelines with a `.specbinder/review-guidelines.md` file. Its rules add to the defaults and take precedence where they disagree.
