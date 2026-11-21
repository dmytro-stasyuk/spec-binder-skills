---
name: separate-out-specs
description: Move the top-level blocks of a SpecBinder spec file (.specb / .feature) that holds several Abilities or Features, or of every such file in the files and folders given, into spec files of their own — each moved block keeps its content, cross-links are rewritten to name the new files, header comments follow the block they concern, and in a Java project the step methods are sorted between the test classes and the build is fixed until it compiles. Use when asked to separate out the specs held in one file or in the files of a folder, give each spec its own file, move a spec's Abilities into their own files, or split a spec file into several.
argument-hint: <paths of .specb or .feature files, or of folders> [--only-lines=<lines> | titles of the blocks to move, for one file; default every block after the first]
---

# Separating out the specs of a file

A SpecBinder spec file can hold several top-level blocks, one after another: `Ability:`, `Feature:`,
`Business Need:` or `Narrative:`, or their equivalent in the language a `# language:` header sets. You move
blocks out of such a file into spec files of their own, and in a Java project their test code moves with them.
Below, "block" means one of these top-level blocks with everything under it.

Dividing one block into several is a different step, done before this one: `move-under-new-spec` moves the Rules
and Examples the author picks under a new block, and a review's split finding divides a block too. Given a single
file with a single top-level block, stop and say so.

## Several files, or a folder

The request names one spec file, or several files and folders; a folder stands for every spec file beneath it,
in its subfolders too. The IDE's Separate Out Specs into Their Own Files sends the files and folders selected in
the Project view. With more than one file to look at:

1. Find the spec files among them, and beneath the folders, that hold more than one top-level block. Leave the
   others out: there is nothing to separate out of them. If none holds more than one, say so and stop.
2. List those files, each with the titles of its blocks, so the author sees what will move, and carry on without
   waiting for a reply. List and work through them in the order the IDE's Project view lists them: the files and
   folders as the request gives them, and beneath a folder its own spec files first, alphabetically, then each of
   its folders, alphabetically, the same way down.
3. Go through sections 1 to 3 for each file, moving every block after the first: `--only-lines` and block titles
   choose blocks in a single file only, so with several, say you are leaving them aside. Then, in section 4, sort
   the step methods of every file before rebuilding once for them all.
4. Report on them all together, file by file.

## 1. Choose the blocks

Read the whole file. Move the blocks the request chooses:

- With `--only-lines=<lines>`, such as `--only-lines=7` or `--only-lines=6-12`, the blocks those lines of the file
  are on, a block's lines running from the tags just above its keyword line to its last line; several lines or
  ranges are separated by commas. The IDE's Separate Out Specs into Their Own Files sends the lines selected in the
  editor. If the lines run past the end of the file, or are on no block (blank lines between blocks, say), ask:
  the file may have changed since they were taken.
- Named by title in the request: those blocks. If a name matches no block's title, or matches several, ask.
- Otherwise, every block after the first.

Any block can move, the first one included, but one always stays in the original file, which keeps its name and
its test class for the blocks left in it. When the request chooses every block, the first stays.

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

Give each new file's path and the block it holds, grouped by the file it came from, the links you rewrote, the header comments that moved and any
you couldn't place, the step methods that moved or became shared, and whether everything compiles. Offer to run
the test classes.

When the original's first block moved out, its file name was probably chosen for that block. If the name no
longer fits the block now at the top of the file, say so and suggest a name after that block's title, for the
author to give it with the IDE's Rename, which renames the spec's test classes with it. Don't rename it yourself.
