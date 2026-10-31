---
name: split-spec
description: Move the top-level blocks of a SpecBinder spec file (.specb / .feature) that holds several Abilities or Features into spec files of their own — each moved block keeps its content, cross-links are rewritten to name the new files, header comments follow the block they concern, and in a Java project the step methods are sorted between the test classes and the build is fixed until it compiles. Use when asked to split a spec file, move a spec's Abilities into their own files, or separate the specs held in one file.
argument-hint: <path to .specb or .feature file> [titles of the blocks to move; default every block after the first]
---

# Splitting a spec file

A SpecBinder spec file can hold several top-level blocks, one after another: `Ability:`, `Feature:`,
`Business Need:` or `Narrative:`, or their equivalent in the language a `# language:` header sets. You move
blocks out of such a file into spec files of their own, and in a Java project their test code moves with them.
Below, "block" means one of these top-level blocks with everything under it.

Dividing one block into several is a separate step, done before this one (a review's split finding does it, for
example). Given a file with a single top-level block, stop and say so.

## 1. Choose the blocks

Read the whole file. Move the blocks the request names by title; when it names none, move every block after the
first. The first block always stays in the original file, which keeps its name and its test class. If a name in
the request matches no block's title, or matches several, ask.

## 2. Name each new file

- Put it in the same folder as the original, with the same extension.
- Name it after the block's title in PascalCase.
- Never overwrite a file: if the folder already has one of that name, choose another name.
- Prefer a name no other spec file, `<Name>Scenarios` or `<Name>Test` class in the project uses: tools that
  identify tests by simple name can mix up two that share one. If you change the name for either reason, tell the
  author.

## 3. Write the files

Each new file holds:
- the original's `# language:` header, if it has one, first: without it the block's keywords don't parse;
- the header comments that concern the moved block (see below);
- the block, unchanged, with its tags, story, description, `Background:`, Rules and Examples.

Then fix what refers across files:
- **Links.** An `Out of scope: … — see <title>` line that points at a block now in another file names that file
  instead: `— see ReplaceText`. Do this in the moved blocks and in those left behind. A link to a block still in
  the same file keeps its title. If the specs already word such lines differently, follow them.
- **Header comments.** A comment above the original's first block, such as `# ADR: docs/ADRs/…`, is copied into
  each new file whose block it concerns, and removed from the original only when nothing left there concerns
  it. When you can't tell what a comment concerns, leave it in the original and name it in the report.

Remove the moved blocks from the original, with the blank lines between them, so the blocks left are separated
by one blank line as before.

## 4. Move the code

Only for Java projects with JUnit 5 tests. Otherwise, stop after the files and tell the author the test code is
theirs to move.

How specs, markers, generated classes and test classes fit together, in both generation modes, is in
[java-wiring.md](../automate-spec/references/java-wiring.md): read it first. Never edit a generated class.

1. Check that the marker's `@Gherkin2JUnit` value covers each new file. If it lists files by name, add them.
2. Work out the mode, as java-wiring.md says. In concrete mode the steps live on the marker, so nothing moves:
   go to step 5.
3. In abstract mode, find the original's test class: the class that extends `<OriginalName>Scenarios`. If it
   has none, the spec isn't wired yet: create no test classes for the new files either, and go to step 5. If it
   has one, create `<NewName>Test extends <NewName>Scenarios` for each new file in the same package, set up like
   the original: the same annotations, base classes, interfaces, fields and setup methods it needs.
4. Sort the original's step methods by which blocks' steps call them:
   - only one moved block: move them to that block's test class, with the private helpers and fields only they
     use;
   - only the blocks left behind: leave them;
   - blocks in more than one file: put them where the project already keeps shared steps, as
     [java-wiring.md](../automate-spec/references/java-wiring.md#where-step-implementations-live) says. Follow
     its existing pattern; if it has none, use the marker class. Remove them from every test class, so they are
     inherited from there.
5. Rebuild, so every generated class is written again. Run the project's build from the command line, such as
   `mvn test-compile` or `./gradlew compileTestJava`, through its wrapper if it has one. If the user explicitly
   asks you to use the IDE, use its build tool instead.
6. Fix what the compiler reports until every test class compiles. An `@Override` that no longer overrides
   anything is a step that is now inherited: delete it. A missing method is a step that didn't move.

## 5. Report

Give each new file's path and the block it holds, the links you rewrote, the header comments that moved and any
you couldn't place, the step methods that moved or became shared, and whether everything compiles. Offer to run
the test classes.
