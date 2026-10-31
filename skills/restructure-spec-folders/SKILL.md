---
name: restructure-spec-folders
description: Reorganise a project's SpecBinder spec files (.specb / .feature) and their test code so the folder tree reveals what the system does — by default restructured by business capability and use case (Screaming Architecture), never by technical layer or test type; or by another principle the user names, such as "by persona"; or just repaired (misfiled specs, one-spec folders, oversized folders, inconsistent names). Moves each spec with its test class, places shared test code by who uses it, fixes packages, imports, @Gherkin2JUnit globs and suites, then rebuilds and runs the tests. Java projects only. Use when asked to restructure, reorganise or reorganize, organise or organize, group or regroup, sort, tidy, clean up, rearrange or move specs or their folders into a better layout, or asked how the spec folders should be laid out.
argument-hint: "[principle, e.g. \"by persona\", or \"repair only\"] [folder or package to limit it to]"
---

# Restructuring the spec set

You reorganise a project's spec files and their tests: you propose a move plan, show it once, then carry it out
and prove the build and tests still pass. A spec never moves alone. Its concrete test class, and anything that
belongs to that spec only, moves with it, and everything that refers to them is updated. You move files; you
never change a spec's content.

This skill is for Java projects (JUnit 5, Maven or Gradle). It reorganises specs that already exist; it doesn't
place a new spec as it is written.

How specs, markers, generated `*Scenarios` classes and concrete test classes fit together is in
[java-wiring.md](../automate-spec/references/java-wiring.md). The facts that matter when moving things, and
the procedure for each move, are in [moving-specs.md](references/moving-specs.md). Read both before step 4.

## Showing a layout

Whenever you show the user a folder layout, for them to agree the capabilities or to confirm the plan, draw it
as the folder tree it would become, in a code block, so they can see the result rather than piece it together
from a description:

```
payments/
├── refunds/                             giving money back to a customer
│   ├── partial/                         new: refunding part of an order
│   │   ├── RefundOneItem.specb          ← refunds/
│   │   └── RefundShipping.specb         ← RefundDelivery.specb
│   └── RefundWholeOrder.specb           (stays)
├── card/                                new: paying by card
│   └── PayByCard.specb                  ← checkout/
└── invoices/                            billing a business customer (unchanged, 4 specs)
```

- Annotate each folder's line with what it covers, and mark the folders the layout adds as `new`.
- List each spec under its target folder, with where it comes from (`← old/folder/`), `(stays)`, or its old
  name when it is renamed (`← OldName.specb`). Leave out each spec's test class and its own files, since they
  move with it, but show shared test code or files that move, under their new folder.
- Collapse a folder whose specs don't move into one line with its spec count, as `invoices/` is above.
- Below the tree, list what the tree doesn't show: markers added, changed or deleted, suite and build-setting
  edits, and the folders removed.

When you offer the user a choice between layouts, show each one as its own tree, or at least the part where
they differ, before you ask. A one-line description of each option isn't enough for them to choose.

## Modes

- **By capability (default).** Restructure the spec tree so that it says what the system does, as Robert C.
  Martin's "Screaming Architecture" asks of a codebase: looking at the top-level folders, a reader should see
  the system's capabilities (`checkout/`, `refunds/`, `catalogue/`), not how it is built or tested (`api/`,
  `ui/`, `web/`, `integration/`, `unit/`, `controllers/`). Delivery mechanisms are not a reason to separate
  specs: a spec that checks checkout through the web and one that checks it through the API both belong in
  `checkout/`.
  - **Finding the capabilities.** Derive them from the specs themselves: the business capability each spec's
    Ability, story and Rules describe, named in the domain's own words. Where the production code's packages
    already reveal the domain (`com.acme.billing`, `com.acme.catalogue`), reuse their names so specs and code
    line up; where the code is organised by layer, it doesn't help.
  - **Shape.** The tree goes as deep as the domain does. A capability that breaks down into distinct
    sub-capabilities gets a subfolder for each (`payments/refunds/`, `payments/refunds/partial/`), and so on at
    any depth; a capability that doesn't stays flat, with its specs directly in its folder, one spec per use
    case or aspect. A folder that has grown hard to scan is a sign it hides sub-capabilities worth separating, but
    split by what the domain distinguishes, never just by count. Avoid folders that would hold a
    single spec, and chains of folders that each hold only one subfolder. No level has folders by layer or
    test type.
  - **Agreeing them first.** Before planning any move, show the capabilities as a folder tree, as "Showing a
    layout" says, and ask the user to confirm, rename, merge, split or nest them. Plan from the agreed tree.
  - **Specs that span capabilities.** A spec whose Rules cover more than one capability moves unchanged:
    - when those capabilities share a parent below the top level (`payments/refunds/` and `payments/card/`),
      it goes in that parent (`payments/`), which covers the whole spec;
    - otherwise it goes with the capability it mainly concerns.

    Either way it is named in the report as a candidate for splitting: a review's split finding divides it into
    blocks in the same file, then `split-spec` moves them into files of their own. Rerun this skill afterwards
    to file them.
- **By another principle.** When the user names one ("by persona", "by product area"), design the tree by that
  principle instead. For "by persona", group by the `As a …` role of each spec's story, merging similar roles;
  specs with no story go in a folder of their own, and you say so. Agree the folders first, as above.
- **Repair only.** When the user asks only to tidy, keep the current structure and fix what is wrong with it:
  - **misfiled specs**: a spec whose subject belongs with another folder's specs;
  - **folders with one spec**: merge it into the folder it belongs with, unless the folder is clearly the start
    of an area that has its own marker or setup;
  - **oversized folders**: so many specs with clearly different subjects that the folder is hard to scan; split
    them into subfolders by subject;
  - **inconsistent names**: file names that don't follow the project's prevailing pattern (usually the Ability
    title in PascalCase), mixed casing, stray suffixes, folder names in mixed styles.

  Leave alone what works. Don't move a spec only because another place is equally good.

## Scope

By default the skill works on every spec in the project. A folder (`src/test/java/com/acme/shop`) or a package
(`com.acme.shop`) in the request limits any mode to the specs under it; a package means the folder that holds it
in the test sources, and if several do (in several modules), ask which. Within a limited run:
- the new layout is built inside that folder: its capabilities become its subfolders, and nothing moves out of
  it or into it;
- shared test code moves only when every class that uses it is inside the scope, and the shared package for
  code used by several of its capabilities sits at the scope folder's root; code that specs outside the scope
  also use stays where it is;
- the inventory and the final run cover the specs in scope, plus any suite or marker they share with specs
  outside it, so a change to a shared marker or suite is checked against everything it covers.

## Shared test code

Besides each spec's own test class, a restructure moves the test code specs share, by who uses it:
- code used only by one capability's specs (step interfaces, base classes, fixtures, helpers) moves into that
  capability's package;
- code used by specs of several capabilities goes in one shared package at the root of the spec tree: the one
  the project already has for this, or else `shared`;
- markers follow the project's existing pattern: one marker per capability package, or one glob marker at the
  root covering every capability below it.

## Steps

1. **Take inventory.** Find every spec file (`*.specb`, `*.feature`) in scope, outside build output, and for
   each one: its concrete test class (the class that `extends <Name>Scenarios`), the marker that generates it (an
   `@Gherkin2JUnit` class whose path or glob covers the spec), the suites and build settings that name it, and
   files used only by its test (helpers, test data). Read each spec's Ability title, story and Rules. Note the
   project's naming pattern, its wiring pattern (one marker per package, or glob markers covering
   subpackages), its shared test code and who uses it, and how its production code is packaged.
2. **Check the build.** Build the project, with Maven or Gradle from the command line, or with the IDE's build
   tools if the user explicitly asks for that (then use them for step 7 too). Don't run any tests yet. If the
   project doesn't compile, stop and tell the user: the move can't be verified on a broken build, and its
   compile errors would be mixed up with any the move causes.
3. **Agree the tree.** In the by-capability and by-principle modes, show the capability tree (or the
   principle's folders) and agree it with the user, as the mode says. Skip this in repair mode.
4. **Plan.** Work out the target layout, then list:
   - each move as `old path → new path`, with the new name where it changes;
   - the test classes and spec-specific files that move with each spec;
   - the shared test code that moves, and where to;
   - markers to add, change or delete, and suite and build-setting edits;
   - the specs that span capabilities, with the folder chosen for each and why;
   - for repair mode, the reason for each move in a few words.

   New package names are lowercase, valid Java identifiers, in the project's style. Names follow "Choosing
   names" in [moving-specs.md](references/moving-specs.md).
5. **Show the plan once.** Present it as the target folder tree, as "Showing a layout" says, with every spec
   under the folder it ends up in, then the totals (specs moved, renamed, folders added and removed). Ask the
   user to confirm or adjust it. Apply their adjustments without showing the whole plan again. If the plan is
   empty, say the structure is fine and stop.
6. **Carry it out.** Follow [moving-specs.md](references/moving-specs.md) for every move: spec, test class and
   spec-specific files together; shared test code; packages and imports; markers and their globs; suites. Use
   the IDE's refactoring tools when they are connected, since they update references for you. Otherwise move
   files with `git mv` (plain moves outside git) and edit the references. Never change a spec's content.
7. **Rebuild and run.** Follow "Building and checking" in [moving-specs.md](references/moving-specs.md):
   - Do a full rebuild, not an incremental one, so every `*Scenarios` class regenerates from the specs' new
     locations and names and stale ones go. Fix every compile error the moves caused.
   - Run the moved tests, the suites you changed, and the tests that use any shared test code or file you moved.
   - Check that every scenario of every moved spec ran exactly once: one test per scenario, and one per row of a
     Scenario Outline's `Examples:` table. The specs' content didn't change, so they say what should run. A
     missing test means no marker or suite covers its spec any more; one that ran twice means two markers'
     globs overlap.
   - Search the project for the old names of the classes and packages you moved, so nothing still points at
     them.
   - Fix every failure the moves caused, then run again, until none is left. Report any other failure without
     fixing it.
8. **Report.** Summarise what moved and was renamed, the shared code that moved, what else changed (markers,
   suites, build settings, run configurations), and the build and test results. List each failing test that the
   move didn't cause, with its error, and ask the user whether it was failing already: the restructure is done
   only when every test passes, or the user confirms the rest were failing before it. List the specs that span
   capabilities, as candidates for splitting, and anything you left alone and why. Don't commit.
