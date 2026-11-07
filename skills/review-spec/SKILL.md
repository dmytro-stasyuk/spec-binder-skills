---
name: review-spec
description: Review a SpecBinder Gherkin spec file (.specb / .feature) against the project's review guidelines, set up on request from starter guidelines, and publish the findings to the spec editor in IntelliJ IDEA, then discuss and fix them with the author. Use when asked to review, critique or check a spec or feature file, to go through its review findings, or to set up a project's review guidelines.
argument-hint: <path to .specb or .feature file> [--ide <port>]
---

# Reviewing a SpecBinder spec

You review one spec file with the author. Findings are published to the IDE, where each is highlighted on the
text it is about, by kind, and listed in the spec's AI Review tab when the author has it turned on. The author
can work through them in the IDE alone, applying a finding's fix or dismissing it, or with you in this
conversation, or both. Don't assume they will come back to discuss them.

Keywords in this skill and in the guidelines stand for whatever the spec uses: `Ability:` for any top-level
keyword (`Feature`, `Ability`, `Business Need`, `Narrative`), `Example:` for `Scenario` too and for an outline,
`Scenario Outline:` for `Scenario Template:` too, and each keyword in the spec's own language when a
`# language:` header sets one (`Regel:` for `Rule:` in German, say). `Given`, `When` and `Then` name a step's
role: a step written with `And`, `But` or `*` takes the role of the step before it. A check about which keyword
a spec uses names the keywords literally.

## Tools

The IntelliJ SpecBinder plugin adds three tools to the IDE's MCP server. Their names end with:

- `specbinder_get_review_scope`: which kinds of finding to report for this spec, as the author has ticked them
  in the IDE. Given the IDE port from the request, it also checks that this session is connected to that IDE.
- `specbinder_publish_spec_review`: publishes a review. It replaces the spec's previous review and returns the
  numbered findings, plus any finding it turned down and why.
- `specbinder_get_spec_review`: the findings still open, with their fixes. A finding closes when the author
  dismisses it, applies its fix, or edits the text it quotes. Findings whose kind the author has unticked are
  listed as set aside: don't raise them. Then it lists the findings the author dismissed, from this review or
  an earlier one, whose text is unchanged. Given finding numbers as `findings`, a comma-separated string such as
  `"2"` or `"1, 3"`, it answers for those findings only, whether open or not: each in full with its fix and its
  state (open, dismissed, fix applied, text edited, or set aside).

Reading the spec needs no tool of ours: the IDE's own file tools (`read_file`, or `get_file_text_by_path` on
an IDE that lacks it) already return a file as the editor currently has it, unsaved changes included.

If the `specbinder_*` tools are not available, stop. Tell the author that the review needs:
1. IntelliJ IDEA 2025.2 or later, with the SpecBinder plugin 2026.20.0 or later installed.
2. The MCP server enabled in Settings › Tools › MCP Server.
3. You, the agent running this review, connected to that MCP server. The **Auto-Configure** button on that page
   sets this up for the clients it lists; for any other client, add the server's address from that page to the
   client's MCP configuration. Then restart or reconnect the client so it picks up the tools.

Don't fall back to reviewing in the chat without the tools, unless the author asks you to.

## Steps

1. **Get the scope.** Call `specbinder_get_review_scope`. When the request ends with `--ide <port>` (the IDE's
   Review This Spec button adds it), pass that number as `idePort`. If the answer starts with `Error:`, this
   session is connected to a different IDE than the one the author asked from: tell the author what it says, and
   stop without reviewing. Tell them how to reconnect too: point this session's MCP configuration for the IDE's
   server at the address the answer gives, keeping the server's current name so the tool names don't change,
   then reconnect the server (or restart the session) and ask for the review again.

   Otherwise it answers `Report: …` and `Skip: …`. The kinds after `Report:` are **the kinds to report**. If the
   author named kinds in their request ("blockers and issues only"), those are the kinds to report instead, for
   this review only; say so. If it says every kind is unticked, tell the author and stop.

   Also call `specbinder_get_spec_review`. Any finding it lists under "Dismissed by the author" was rejected
   by the author: don't raise it again in this review, nor another finding of the same category on that text.
   The publish tool turns such a repeat down. A finding about something else on the same text is fine.
2. **Read the spec** through the **IDE's** file tool, not from disk: the editor's copy includes unsaved
   changes, and its line numbers are the ones the author sees and the ones the publish tool checks against.
   Count lines from 1; where the tool prefixes them (`L12: …`), the finding's `line` is that number without
   the prefix.
3. **Load the guidelines.** A review checks the spec against the project's own guidelines and nothing else:
   - `.specbinder/review-guidelines.md`;
   - every `.md` file in `.specbinder/review-guidelines/` and its subdirectories, in path order.

   If the project has none, don't review. Tell the author that reviews follow the project's guidelines and it
   has none yet, and offer to set them up: see [Setting up guidelines](#setting-up-guidelines). Review once they
   exist, unless the author would rather stop.

   Guidelines are written in sections. A section is a level-2 heading, a short summary, its `Kind:` line, which
   every section needs, and its bullets. The part of a file before its first heading is a section too,
   without a heading or summary. When the heading has a name in backticks, the section is a category, and that
   name is the `category` value of its findings. Any other section has no category, and neither do its findings.

   A line starting `[//]: #` is commented out: it is how IntelliJ IDEA's line-comment shortcut comments out a line
   of Markdown, such as `[//]: # (- Flag: …)`. Ignore it, whatever it holds: it is not a heading, a `Kind:` line,
   a check or guidance. When the commented-out line is the first line of a bullet, ignore the bullet's
   continuation lines with it.

   A bullet starting with `Flag:` is a **check**: it names what to report. A check is reported as its section's
   kind, unless it gives one of the kinds below in brackets after `Flag`. Other bullets are guidance for the
   section's checks:
   ```markdown
   ## `accessibility` — every user can complete the task

   Kind: issue

   - Flag: outcomes that are only shown by colour or only conveyed by sound.
   - Flag (polish): images in a doc string without a description.

   ## Pricing

   Kind: suggestion

   - Flag: steps that mention the staging environment.
   - Flag (issue): prices without a currency.
   ```
   The kind says what sort of remark a finding is. There are exactly eight, each shown in its own way in the IDE,
   and a guideline can't add others:
   - `blocker`: the spec is wrong or misleading as written;
   - `issue`: a real problem worth fixing;
   - `split`: something in the spec that should become two or more (a Rule, Example or step);
   - `gap`: something the spec doesn't cover yet, such as a missing example;
   - `suggestion`: it works, but what the spec says or how it is put together would be clearer changed, such as a
     title, a step, the story or where something sits;
   - `polish`: the surface only, such as layout, punctuation, spelling or wording that doesn't change what the
     spec says, safe to apply without much thought;
   - `question`: an ambiguity for the author to decide, not a defect;
   - `praise`: something done well, worth keeping.

   Load the guidelines in three passes, so that you read only the files the review needs:
   1. **Find every check's kind without reading the files.** Search all the files for three sorts of line:
      level-2 headings (`## `), `Kind:` lines, and checks (bullets starting `- Flag:` or `- Flag (<kind>):`).
      Match each at the start of its line, so that commented-out lines don't count. A section runs from its
      heading to the next heading; the lines before a file's first heading are a section of their own. A check's
      kind is the kind in its brackets, if it has any, and otherwise its section's `Kind:` line.
   2. **Stop on a missing or invalid kind.** A `Kind:` line is missing when a section has checks but no `Kind:`
      line, even if every check gives its own kind in brackets. A kind is invalid when a `Kind:` line or a
      check's brackets name anything other than the eight kinds above, such as `Kind: major` or `Flag (nit):`.
      If you find either, don't review. List each one to the author with its file and line number, say whether
      it is a missing `Kind:` line or an invalid kind, and offer to correct it. Review once there are none.
   3. **Read the files that have checks to report.** Read in full each file with at least one check whose kind
      is one of the kinds to report (step 1). Don't read the other files.

   In step 4, check the spec only against the checks whose kind is one of the kinds to report. The files you
   read can hold checks of other kinds too, even in the same section: don't check the spec against those, and
   don't report findings from them. A section's bullets that don't start with `- Flag` are guidance for its
   checks: follow them when you check the spec against those checks. Where two checks or guidance bullets
   disagree, the one in the file that comes later in the order listed above wins:
   `.specbinder/review-guidelines.md` first, then the directory's files in path order. Sections with the same
   category name in different files form one category.

   When a check you'll report compares the spec with the project's other spec files, such as one asking whether
   another spec already covers this spec's aspect, survey those files before reviewing, so that the check rests on
   what they say rather than on what you happen to know:
   - Find the project's `.specb` and `.feature` files, other than this one, leaving out build output such as
     `target/`, `build/` and `out/`.
   - Search them for their title, Rule and Example lines (`Feature:`, `Ability:`, `Business Need:`, `Rule:`,
     `Scenario:`, `Example:`, `Scenario Outline:` and `Scenario Template:` at the start of a line, after
     indentation), rather than reading each file. Read a file in full only when its titles suggest it covers the
     same ground as this spec, or some of its Rules or Examples, to see whether it does.
   - Report every finding on this spec. Another spec is something a finding names, as a link to its part (see
     [Writing for the author](#writing-for-the-author)), never a place it is reported.

   Skip the survey when no check you'll report compares specs.
4. **Review.** Check the spec against the guidelines. Report only real problems a reader would notice, not
   matters of taste. When one check matches several places in the same scenario, or the same Background, report
   it once for that scenario: quote the first place, say in the message that it repeats, and if you write a fix,
   make it a rewrite of whole lines (see step 5) from the first place to the last. A scenario elsewhere with the
   same problem gets its own finding.
5. **Publish.** Call `specbinder_publish_spec_review` with a JSON array of findings:
   ```json
   [
     {
       "line": 12,
       "quote": "I click the Save button",
       "category": "implementation-detail",
       "kind": "issue",
       "message": "Names a UI control instead of what the user wants to achieve.\n\nThe fix says what the user does instead: they save the invoice.",
       "rewrite": [{"text": "I save the invoice"}],
       "check": {
         "path": ".specbinder/review-guidelines/implementation-detail.md",
         "line": 8,
         "text": "Flag: UI mechanics where the intent is what matters (\"I click the Save button\", \"I type into the email field\", \"I"
       }
     }
   ]
   ```
   - `line` is the 1-based line number in the text you read from the IDE.
   - `quote` is copied **verbatim**, starting on that line. Keep it short: the smallest span that shows the
     problem. When the problem spans lines, the quote can run over several, with `\n` between them and each
     following line copied whole from its start, indentation included.
   - `category` is the category of the section the finding's check is in. Leave it out when that section has no
     category.
   - `kind` is required: the kind of the check the finding comes from, the check's own or else its section's.
     Never change it: the guidelines decide each finding's kind. Every finding comes from a check, so report
     nothing that no check covers.
   - `check` is the check the finding comes from, so the author can trace the finding back to it and tune it:
     `path` is its guideline file, relative to the project, `line` the 1-based line its `- Flag` bullet starts
     on, and `text` that line copied **verbatim** from `Flag` on, its kind in brackets included. Give it on every
     finding. A finding whose check is not on that line of that file is turned down.
   - `message` says what is wrong, in up to three short sentences, and then, when the finding has a fix, what the
     fix does, in one or two more. Each sentence starts with a capital letter, as it is shown on its own in the
     IDE. Put the point in its first sentence: the IDE shows that sentence as the finding's lead, and any sentences
     after it as detail under it. Always start the sentences about the fix on a paragraph of their own, after a
     blank line (`\n\n`), so the author can find them at a glance. A message with a blank line in it is cut only
     at its blank lines, its first paragraph leading, so end the lead with one too:
     `"Lead.\n\nMore about the problem.\n\nThe fix …"`. To lead with more than one sentence, put them all
     before that first blank line. Write it as [Writing for the author](#writing-for-the-author) says.
   - Every finding proposes a fix, `rewrite`: always a list of edits, each with a `text` that is the new wording.
     The exceptions are `question` and `praise` findings, which have none, and a finding about the spec file's
     name, whose fix is a `rename` (see below).
     - `[{"text": "..."}]` replaces **exactly the quoted span**, so `text` must read correctly in its place. Such
       an edit is always the fix's only one.
     - `[{"fromLine": 8, "toLine": 14, "replaces": "...", "text": "..."}]` is for a problem whose fix reaches
       beyond the quote, such as splitting a scenario: it replaces whole lines, both inclusive. `replaces` is
       those lines copied **verbatim** as you read them, joined with `\n`, so the IDE can tell if the author
       edited them while you reviewed; if they did, the finding is turned down. `text` is then the new lines,
       joined with `\n`, without a trailing line break, with the indentation they need. An empty `text` removes
       the lines, line breaks included.
     - Start and end the lines on the text the fix changes, not on the blank lines around it: rewriting or removing
       a description runs from its first line to its last, leaving the blank line before the first Example where it
       is. Take in a blank line only when the fix itself changes it, and then put back in `text` every blank line
       the spec still needs, such as the one between a Rule's description and its first Example.
     - A fix that changes places far apart, such as moving an Example to another Rule, gives one whole-lines edit
       per place, never one edit spanning everything between them. The edits must not overlap, one of them must
       take in the finding's line, and the author applies them together, as one fix.

     A fix that is long or reaches far is still a fix to give. Leave it out only when you can't tell what the
     right wording would be, and then say why in the summary. A `gap` finding always has one.
   - A `gap` finding's fix inserts what is missing: an Example under a Rule, or a Rule in the spec. Give each
     missing Example or Rule its own finding, unless a few are so closely related that the author would accept or
     reject them together, such as an Example and its counter-example. Place each where it fits best:
     - **A missing Example:** under the Rule whose acceptance criterion it shows, which is not always the Rule
       where you noticed it was missing; in a spec without Rules, in the spec itself. Put it after the Example it
       varies or contrasts with, or else at the end of that Rule or spec. Indent it like the other Examples there,
       or one level deeper than the Rule when it has none.
     - **A missing Rule:** when the missing case shows a criterion that no Rule of the spec states, propose a Rule
       for it rather than an Example under a Rule it doesn't fit. Put it after the Rule it is most closely related
       to, or else at the end of the spec. Give it a title stating its acceptance criterion and an Example of its
       main case. Indent it like the spec's other Rules, or one level deeper than the spec's first line when it
       has none.

     The finding is on the part it adds to: `line` is the line of the Rule a missing Example goes under, or the
     spec's first line (`Ability:`) for an Example in a spec without Rules and for a missing Rule, and `quote` is
     the title on that line. The insertion goes after a whole block, never inside one: after the last non-blank
     line of the Example or Rule it follows, or after the Rule line itself when that Rule has no Examples yet. The
     edit's `fromLine` and `toLine` are that line, `replaces` the line verbatim, and `text` the same line
     unchanged, then a blank line, then the new Rule or Examples, separated by blank lines. The publish tool accepts
     the edit away from the finding's line because it lies within the Rule or spec the finding is on. Give each
     Example a title naming the situation and the `Given`, `When` and `Then` it needs, reusing the project's step
     phrasings word for word where one fits.

     A gap whose outcome is the author's to decide still comes with its Example. Write the `Then` for what the
     system does now, as its production code or tests show where you can read them, or else for the outcome that
     fits the Rule's criterion and its neighbouring Examples best. Say in the message that the `Then` is a proposal
     to confirm, and name the other outcomes the author could choose instead.
   - A fix that moves an outcome or an Example to another Rule, such as for a `split` finding or for an `issue`
     that an Example illustrates another Rule's criterion, moves it to the Rule whose criterion it shows. An
     unrelated `Then` becomes a new Example there, repeating the `Given` and `When` it needs, and leaves the
     original Example; a misplaced Example moves unchanged. Place it at the end of that Rule. If no Rule fits, add
     a new Rule for it, titled with its criterion, after the Rule it relates to most, or else at the end of the
     spec. Give one edit where it leaves, rewriting or removing those lines, and one where it arrives, inserting it
     the way a `gap` fix does; the lines between them stay out of the fix.
   - A `split` finding's fix is a rewrite of whole lines that leaves the spec fully split, never half done:
     - **An Example:** rewrite the whole Example, from its title to its last step, as two or more complete
       Examples, each with its own title and the `Given`, `When` and `Then` it needs.
     - **A Scenario Outline whose rows show different behaviours:** rewrite the whole outline as one outline per
       behaviour, each with its own title, steps and rows. A behaviour left with a single row becomes an
       ordinary Example.
     - **An outcome or an Example under the wrong Rule:** move it, as above.
     - **A Rule:** rewrite the whole Rule block, from the `Rule:` line to the last line of its last Example, as
       two or more narrow Rules, each with its own title and, where it needs one, its own description. Put each
       existing Example under the Rule it shows, unchanged, and keep a Rule-level `Background:` only in the new
       Rules whose Examples use it. When the Rule had Examples, give every new Rule at least one, writing a new
       Example for a Rule none of them shows. When the Rule had no Examples yet, the new Rules have none either:
       the rewrite splits the Rule lines alone.
     - **A spec:** SpecBinder lets a spec file hold several top-level `Ability:` (or `Feature:`) blocks, so split
       the spec into two or more blocks in the same file; never create a file. Rewrite from the block's first tag
       line, or its `Ability:` line when it has no tags, to its last non-blank line, and anchor the finding on
       the `Ability:` line. Give each new block:
       - its own short title naming its aspect;
       - the original persona, with an `I want` and `So that` for its own aspect;
       - its own description, holding only the context that applies to it, and `Out of scope: … — see <title>`
         lines pointing at its sibling blocks;
       - the original's tags; in the message, name any tag that looks like it belongs to only one aspect;
       - the original's `Background:`, only when its Examples use it.

       Put each Rule and Ability-level Example, unchanged, into the block of the aspect it mainly concerns, and
       name in the message any that fit several aspects or none clearly. An Example whose Rule goes to one block
       but which shows another aspect, such as an Example of how a total combines its parts under a Rule about
       what each part shows, moves unchanged to the Rule of that other block whose criterion it shows; when no Rule
       there fits, give it a new Rule, titled with its criterion. When the original title clearly names
       one aspect, its block comes first; the others follow in the order of their first Rule in the original.
       Rules and Examples keep their original order within each block. Comments above the first block, such as
       `# ADR:` lines, stay where they are, outside the rewrite.
     - **A spec without Rules whose Examples show several criteria:** wrap its Examples in one Rule per
       criterion, each titled with its criterion, keeping every Example unchanged and in its original order
       within its Rule.
   - A finding about the spec file's name gives `rename` in place of `rewrite`: the file's new name alone, such as
     `"rename": "RefundOrder.specb"`, with no folder and keeping the file's extension. Put the finding on the spec's
     first line (`Ability:`), quoting its title, and say in the message what the name suggests the spec is about
     and what it is about instead. In IntelliJ IDEA, applying the fix opens the Rename dialog on that name, which
     renames the spec's test classes with the file. If the publish tool's description doesn't mention `rename`,
     the IDE's SpecBinder plugin predates it: publish the finding without a fix, and give the new name in the
     message.
   - Findings are numbered top to bottom as their quotes appear in the spec, whatever order you list them in.
     Refer to them by the numbers the result gives.
   - If the result turns any finding down, correct those findings and publish the **whole** review again.
     Publishing replaces the previous review.
   - A spec with nothing to report gets an empty array `[]`, so a stale review is cleared.
6. **Summarise.** List the findings in the chat, one line each, in the order of their `#number`s, which is the
   order they appear in the spec, written as [Writing for the author](#writing-for-the-author) says. Offer to fix
   or discuss any of the findings, and mention that the author can also apply or dismiss each one in the IDE.

## Writing for the author

The author reads each finding on its own, beside the spec, without the reasoning that led to it. Write it so they
understand it on the first read, in the words they would use themselves:

- **Name what you mean.** Refer to every Rule, Example or step a finding involves by its title or text, quoted as
  the spec has it: the Rule "the cart totals the price of what it holds". Where that title is not unique, add the
  Rule it sits under: the Example "with one item" under the Rule "…". Never refer to one by a label of your own
  making, such as "the totals Rule", nor by a bare line number: the IDE keeps a finding's own place up to date as
  the spec changes, but not a number written in its message, which goes stale as soon as an earlier fix moves lines.
- **Link the parts you name.** In a finding's message, make each Feature, Rule, Background, Example or Examples
  table you name a link to it: put its quoted title, quotes included, in brackets, followed by `(#L12)`, where 12
  is the line its keyword is on as you read the spec. The word naming the part stays outside the brackets:
  `the Rule ["the cart totals the price of what it holds"](#L12)`, not `[the Rule "…"](#L12)`. For a part of
  another spec file, give that file's path in the project before the `#`:
  `the Rule ["a card payment is taken in full"](features/Payments.specb#L2)`. The card shows only what is in the
  brackets, as a link that keeps leading to the part as the spec changes. Steps are not parts: name them in words
  only. The publish tool turns down a finding whose link leads to a line that doesn't start a part.
- **Say it in whole sentences.** Give each sentence a subject and a verb, and one thought. Say what is wrong first
  and, when it isn't obvious, why it matters to someone reading the spec after it. The problem in a finding about
  one place usually needs one sentence; one that involves two places, such as an Example under the wrong Rule, may
  need three.
- **Say what the fix does.** The author sees the fix as a diff, and a long one, or one that only removes lines,
  doesn't say what it is for. After the problem, in a paragraph of its own, say in plain words what the fix
  changes in the spec, so the
  author knows what the diff does before they read it: "The fix removes this Rule and its three Examples, and
  widens the title of the Rule […](#L801) to cover tags, titles and descriptions." Name what it removes, moves,
  merges, adds or rewords, and where to, but don't repeat the new wording itself, which the diff shows.
- **Use shared words.** Use the spec's words and the author's. A term from the guidelines, or one you used earlier
  in the conversation, is not one the author shares; if a finding needs it, say what it means.
- **Check before publishing.** Read each message as the author would. Could they tell which part of the spec you
  mean, what is wrong with it, and what applying the fix would do, without your notes or the diff? If not, write
  it again.

Avoid:

> Belongs under the totals Rule, not the holds criterion.

Write:

> This Example checks the cart's total, which is what the Rule ["the cart totals the price of what it holds"](#L14)
> is about. It sits under the Rule ["the cart holds what the shopper adds"](#L3), which is about what the cart
> holds.
>
> The fix moves it, unchanged, to the end of the Rule ["the cart totals the price of what it holds"](#L14).

The same goes for everything you tell the author in the chat: the summary of a review, and any discussion of its
findings. In the chat a line number may go beside a title, as the author reads it at once, while the lines are
still where it says.

## Setting up guidelines

When a project has no guidelines, or the author asks to set them up or add to them, offer the starter
guidelines in [starter-guidelines/](starter-guidelines/), one category per file. Read them all, then list every
category, with the summary from its heading and its `Kind:`, and mark the ones the project already has a section
of the same name for:

- `user-perspective`: written outside-in, from the user's side
- `implementation-detail`: what, not how
- `declarative-steps`: intent over procedure
- `structure`: Given / When / Then roles
- `single-behaviour`: one behaviour per example
- `title`: titles name the situation
- `language`: clear, consistent domain language
- `edge-cases`: each Rule covers the cases that break it
- `duplication`: each thing said once
- `format`: SpecBinder's layout and wording conventions

The author can take all of them, pick some, or write their own instead. Copy each chosen file unchanged to
`.specbinder/review-guidelines/<category>.md`. If that file already exists, or the project already has a section of
that name elsewhere, show the author the difference and ask which to keep: the starter version, which replaces
their section, or theirs, which means skipping that copy. The copies belong to the project from then on: the author
edits them to suit, and later reviews read them like any other project file. Tell the author which files you
created.

## Working through findings with the author

- When the author refers to findings by number ("fix #2", "explain #4 and #5"), call
  `specbinder_get_spec_review` with those numbers as `findings`, to see their current state. The author may have
  fixed, dismissed or set aside findings in the editor since your last call. Call it without `findings` only
  when you need the whole review, such as "go through the rest".
- When you fix findings, edit the spec file directly. The edit closes the finding once its quoted text changes.
  Don't publish a new review just to remove the findings you fixed. Leave a `rename` fix to the author to apply
  in the IDE, which renames the spec's code with it; renaming the file yourself leaves that code behind.
- Once a rename fix is applied, the spec has a new path: the tools answer for the spec only by that path, and
  name it when asked by the old one.
- Publish again only when the author asks for a fresh review, or after changes large enough that the old
  line-based findings no longer fit. Re-read the spec through the IDE first, so the line numbers are current.
- If the author disagrees with a finding, discuss it. If they're right, suggest they dismiss it from the editor:
  a dismissed finding stays out of later reviews until its text is edited. Publishing the review again without it
  removes it from this review only.
- When the author pushes back on a finding with a reason, or has dismissed several findings of the same sort
  (gone from `specbinder_get_spec_review` while their quoted text is still in the spec), offer the guideline
  that would stop them, as one exact line: "Add to `language`: 'Abbreviations shown in the product, such as
  SKU, are domain words.'?" If the author says yes:
  1. Add the guideline as a bullet under the category's heading in `.specbinder/review-guidelines/<category>.md`,
     creating the file or the heading (``## `<category>` ``) if needed. If the project keeps its guidelines
     only in `.specbinder/review-guidelines.md`, add it under the heading there instead. A new heading needs a
     `Kind:` line: ask the author which kind, suggesting the finding's. For a finding with no category, add the
     guideline to the section it came from.
  2. If the guideline makes any open finding invalid, re-read the spec through the IDE, call
     `specbinder_get_spec_review` and publish its findings again without those, since there is no tool to
     dismiss one. Tell the author which findings you removed.
