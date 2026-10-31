---
name: automate-spec
description: Turn a SpecBinder spec file (.specb / .feature) into a runnable JUnit 5 test in a Java project — wire it to the generated test class, write step methods that exercise the production code the way the project's existing specs and tests do, reuse the project's existing steps and helpers, then compile, run and report. Use when asked to automate, wire, implement the steps of, or make runnable a spec, or to fix its missing or failing step methods.
argument-hint: <path to .specb or .feature file>
---

# Automating a SpecBinder spec

You make one spec runnable: the test class behind it and real step bodies that exercise the production code the
way the project's existing specs and tests do, then you compile and run it and report. You don't build the
feature: where the production code a step needs doesn't exist yet, the step fails with a message saying what's
missing.

Java only (JUnit 5, Maven or Gradle). How SpecBinder generates code, how step methods are named and where they
belong is in [java-wiring.md](references/java-wiring.md). Read it before step 2.

## Steps

1. **Find the spec.** Use the path in the request. Otherwise use the spec open in the IDE, if you are
   connected to it. Otherwise ask. Read the spec from disk: the generated class you wire against is built from
   the file on disk. If its comments point to a design document or ADR, read that too, for any decisions that
   affect how the spec is implemented.
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
4. **Learn the project's conventions.** For each step, find the production code it describes and how the
   project's existing specs and tests exercise that kind of code: at what level (a class, a service, an HTTP
   endpoint, the UI), and with which fixtures, test contexts, clients, page objects and builders. Follow the
   same conventions and reuse those helpers, preferring the specs and tests nearest this one: its own package,
   then sibling packages, then further out. If nothing in the project shows a convention for this kind of
   code, ask the author what the steps should exercise before writing them.
5. **Write the steps.** In abstract mode, create or complete `<Name>Test`. In concrete mode, add the steps to
   the marker (see "Where step implementations live"), and never write a `<Name>Test`: the generator writes
   that class. For each step:
   - already implemented up the hierarchy: leave it;
   - shared with other specs, worded the same: implement it once in the project's shared place, not here;
   - otherwise: implement it on `<Name>Test` in abstract mode, or on the marker in concrete mode, following
     "Writing step bodies" and "Assertions" in [java-wiring.md](references/java-wiring.md);
   - needs production code that doesn't exist yet: throw
     `UnsupportedOperationException("Not yet drivable: …")` naming what is missing, and note it for the report.

   In abstract mode, if a sibling test already has the same `Given` or `When` step with the same body, hoist it
   as described there. Don't edit the spec, and don't change production code.
6. **Compile.** Build the test sources with Maven or Gradle until there are no errors (see "Compiling and running
   one test class" and "Reading compile errors"). In concrete mode a spec can compile with steps still missing,
   so also check that the regenerated `<Name>Test` has no stubs left.
7. **Run** just `<Name>Test` (see "Compiling and running one test class"). Read every failure and sort it as a
   wiring bug, missing production code, or a difference between the spec and the system (see "Reading test
   results"). Fix wiring bugs, and in concrete mode write any step still reported as "Step is not yet
   implemented", then run again until only the other two are left. Never stop at "it compiles".
8. **Report**, briefly:
   - the files you created or changed, and the layout you used (and that it is the simple layout, for a
     project's first spec);
   - the result per Example: passing, failing because something is missing, or failing because the system
     behaves differently from the spec (with expected and actual);
   - **Missing production code:** one line per missing piece (such as a method, an endpoint or a UI control),
     with the steps that need it;
   - steps you hoisted into a shared place, and any spec wording you think should change (don't change it).
