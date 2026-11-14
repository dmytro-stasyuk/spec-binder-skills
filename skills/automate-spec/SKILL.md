---
name: automate-spec
description: Turn a SpecBinder spec file (.specb / .feature) into a runnable JUnit 5 test in a Java project — wire it to the generated test class, write step methods that exercise the production code the way the project's existing specs and tests do, reuse the project's existing steps and helpers, then compile, run and report. On request, also build the feature test-first, one Example at a time, and add the cases the spec doesn't cover to it. Use when asked to automate, wire, implement the steps of, or make runnable a spec, to fix its missing or failing step methods, or to build the feature a spec describes.
argument-hint: <path to .specb or .feature file> [--only-lines=<lines>] [--implement[=implement-using-subagent,refactor-using-subagent]] [--add-uncovered-cases[=as-gap-review-findings,complete-with-scenario-steps]]
---

# Automating a SpecBinder spec

You make one spec runnable: the test class behind it and real step bodies that exercise the production code the
way the project's existing specs and tests do, then you compile and run it and report. Unless the request asks
you to build the feature too, you don't: where the production code a step needs doesn't exist yet, the step
fails with a message saying what's missing.

Java only (JUnit 5, Maven or Gradle). How SpecBinder generates code, how step methods are named and where they
belong is in [java-wiring.md](references/java-wiring.md). Read it before step 2.

## Options

A request can add these, as flags or in plain words:

- `--only-lines=<lines>`: work only on the Examples and Rules those lines of the spec file are on, such as
  `--only-lines=7` or `--only-lines=12-30`; several lines or ranges are separated by commas. The IDE's Automate This
  Spec button sends the lines selected in the editor. A request can also name the Examples or Rules instead
  ("automate the Rule 'members get 10% off'"). Without either, the whole spec.
- `--implement`: also build the feature, test-first, one Example at a time, as
  [implementing.md](references/implementing.md) says. Its values, separated by commas, in any order:
  - `implement-using-subagent`: a subagent writes the production code, and you check what it changes;
  - `refactor-using-subagent`: once the feature is built, another subagent refactors the code, and you check it.
- `--add-uncovered-cases`: add the cases the spec doesn't cover, met while you work, to the spec as Examples or
  Rules with a title and a description but no steps, as [uncovered-cases.md](references/uncovered-cases.md) says.
  Its values:
  - `as-gap-review-findings`: publish them to the spec's AI Review tab in the IDE as gap findings instead, for the
    author to apply or dismiss;
  - `complete-with-scenario-steps`: give them full `Given`, `When` and `Then` steps, and wire them.

Change production code only when the request has `--implement` or plainly asks you to build the feature as well
("and build the feature"). "Implement the steps", "wire it" or "make it runnable" asks for the steps alone.

## Steps

1. **Find the spec.** Use the path in the request. Otherwise use the spec open in the IDE, if you are
   connected to it. Otherwise ask. Read the spec from disk: the generated class you wire against is built from
   the file on disk. If its comments point to a design document or ADR, read that too, for any decisions that
   affect how the spec is implemented. The **selected Examples** are those named in the request, or those the
   lines of `--only-lines` are on; without either, every Example of the spec. A line selects:
   - the Example it is on, an Example's lines running from the tags just above its keyword line to its last line;
   - the whole Rule, when it is one of the Rule's own lines before its first Example: its title, description or
     Background;
   - every Example, when it is one of the spec's own lines before its first Rule or Example.

   If the lines run past the end of the spec, or select no Example (blank lines between Examples, say), ask: the
   spec may have changed since they were taken.
2. **Find the wiring.** Follow "Finding the project's existing wiring pattern" in
   [java-wiring.md](references/java-wiring.md): which marker covers this spec, the base classes and step
   interfaces in use, and which of this spec's steps are already implemented somewhere. Work out the spec's
   **mode** from `@Gherkin2JUnitOptions(shouldBeAbstract = …)` on its marker or above: abstract unless it says
   `false` (see "The classes, in either mode"). In abstract mode, also check whether a `<Name>Test` already
   exists. Create a marker only if none covers the spec. In a project with no wired specs yet, use the
   simple layout and say so in your report. If SpecBinder isn't in the build at all, stop and tell the user
   what to add.
3. **Generate.** Look for the generated class in the generated test sources: `<Name>Scenarios` in abstract
   mode, `<Name>Test` in concrete mode (where they land is in [java-wiring.md](references/java-wiring.md)). If it
   is there and newer than the spec file, read it as it is. Otherwise, because it is missing or the spec has
   changed since, compile the test sources so the processor writes it, then read it. Your to-do list is its
   abstract methods and the compile errors in `<Name>Test` in abstract mode, or its stub methods ("Step is not
   yet implemented") in concrete mode. Copy signatures from it; never guess them.

   Everything this run does is driven by Examples that fail. When the to-do list is empty, run the selected
   Examples' tests now, as step 7 says. If every one passes, there is nothing to drive the run: report that they
   already pass and stop, without looking for uncovered cases, building or refactoring, whatever the options.
   Otherwise go on, and leave out the Examples that already pass: they need nothing, and drive no search for
   uncovered cases.
4. **Learn the project's conventions.** For each step, find the production code it describes and how the
   project's existing specs and tests exercise that kind of code: at what level (a class, a service, an HTTP
   endpoint, the UI), and with which fixtures, test contexts, clients, page objects and builders. Follow the
   same conventions and reuse those helpers, preferring the specs and tests nearest this one: its own package,
   then sibling packages, then further out. If nothing in the project shows a convention for this kind of
   code, ask the author what the steps should exercise before writing them.

   As you read the production code that the selected Examples not already passing drive, note each **uncovered
   case**: behaviour it has that no Example of the spec shows, such as a branch that turns down a negative
   quantity or an error for an unknown product. Note only what you come across; don't go looking for edge cases.
5. **Write the steps.** With `--implement`, steps 5 to 9 go as [implementing.md](references/implementing.md)
   says instead: it builds the feature one Example at a time, writing each one's steps as the bullets below say,
   and then adds the uncovered cases and reports in its own order. Otherwise, in abstract mode, create
   or complete `<Name>Test`. In concrete mode, add the steps to the marker (see "Where step implementations
   live"), and never write a `<Name>Test`: the generator writes that class. For each step the selected Examples
   use:
   - already implemented up the hierarchy: leave it;
   - shared with other specs, worded the same: implement it once in the project's shared place, not here;
   - otherwise: implement it on `<Name>Test` in abstract mode, or on the marker in concrete mode, following
     "Writing step bodies" and "Assertions" in [java-wiring.md](references/java-wiring.md);
   - needs production code that doesn't exist yet: throw
     `UnsupportedOperationException("Not yet drivable: …")` naming what is missing, and note it for the report.

   In abstract mode, a step that only unselected Examples use still needs a method for `<Name>Test` to compile.
   If it has none, give it the body SpecBinder's own stub would have (see "The classes, in either mode"):
   `Assertions.fail("Step is not yet implemented")`, or `Assumptions.assumeTrue(false, "Step is not yet
   implemented")` when the spec's `unimplementedStepBehavior` is `ABORT`. In concrete mode the generator already
   writes these stubs.

   In abstract mode, if a sibling test already has the same `Given` or `When` step with the same body, hoist it
   as "Where step implementations live" describes. Don't edit the spec, except to add uncovered cases in step 8,
   and don't change production code.
6. **Compile.** Build the test sources with Maven or Gradle until there are no errors (see "Compiling and running
   one test class" and "Reading compile errors"). In concrete mode a spec can compile with steps still missing,
   so also check that the regenerated `<Name>Test` has no stubs left for the selected Examples' steps.
7. **Run** just `<Name>Test`, or only the selected Examples' tests when not every Example is selected (see
   "Compiling and running one test class"). Read every failure and sort it as a wiring bug, missing production
   code, or a difference between the spec and the system (see "Reading test results"). Fix wiring bugs, and in
   concrete mode write any step still reported as "Step is not yet implemented", then run again until only the
   other two are left. Never stop at "it compiles".
8. **Add the uncovered cases**, with `--add-uncovered-cases`, as [uncovered-cases.md](references/uncovered-cases.md)
   says. Without it, they go in the report only.
9. **Report**, briefly:
   - the files you created or changed, and the layout you used (and that it is the simple layout, for a
     project's first spec);
   - the result per selected Example: passing, failing because something is missing, or failing because the
     system behaves differently from the spec (with expected and actual);
   - when not every Example was selected, which ones were left out of this run;
   - **Missing production code:** one line per missing piece (such as a method, an endpoint or a UI control),
     with the steps that need it;
   - **Uncovered cases:** one line per case, and where it went: added to the spec, published to the IDE, or only
     listed here;
   - steps you hoisted into a shared place, and any spec wording you think should change (don't change it);
   - with `--implement`, also what [implementing.md](references/implementing.md#reporting) lists.
