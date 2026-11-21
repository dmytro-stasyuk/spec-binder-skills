# Building the feature test-first

With `--implement`, you build what the spec describes, driven by its Examples, rather than leaving steps "Not yet
drivable". Steps 1 to 4 of the skill come first; this replaces its steps 5 to 7, and says when its steps 8 to 10
come.

Write no tests but the spec's: its Examples are what drive the code. Build nothing an Example doesn't ask for. A
case the spec leaves open, such as adding a quantity of zero when no Example does, stays as the code you wrote
for the Examples happens to handle it: note it as an uncovered case, with what the code does in it now.

## Who writes the code

- By default, you do.
- With `implement-using-subagent`, a subagent does, and you check each change it makes. Start it when the loop
  starts, give it the brief in [Briefing the implementing subagent](#briefing-the-implementing-subagent), and
  keep it for the whole run, sending it each Example in turn as a follow-up message. If follow-up messages
  aren't possible, start a new subagent for each Example with the same brief and a summary of what is built so
  far. If you can't start subagents at all, tell the author and ask whether to write the code yourself instead.

You always write the steps, whoever writes the code.

## The loop

Take the selected Examples in the order of the spec file, leaving out those that already passed when the run
started (step 3 of the skill): they need nothing built. When every one of them passed, there is no loop, nor
anything after it. For each of the others:

1. **Write its steps** for real, as the bullets of step 5 of the skill say, calling the production code the way
   the step needs it, even where that code doesn't exist yet: the step says what code it wants. Write a
   Background's steps with the first Example that needs them. Steps that only later Examples use stay "Not yet
   drivable" until their turn.
2. **See it fail.** Compile and run it. A compile error for code that doesn't exist yet counts as failing. An
   Example that passes straight away needs no new code: check that its steps really exercise the code, and
   move on.
3. **Make it pass** with just enough production code, or hand the failing Example to the implementing subagent
   and check what comes back (see [Checking a subagent's changes](#checking-a-subagents-changes)).
4. **Run the spec's tests.** This Example and every earlier one must pass.
5. **Tidy up** the code this Example added or changed, such as duplication, a name, or a method or class to
   extract, keeping every test passing. The implementing subagent does this as part of its turn.

Once an Example's steps have been seen failing, change them only to fix a wiring bug, such as a wrong call,
wrong setup or leaked state, and never so the Example passes. Name every such change, with its reason, in the
report. This holds with or without a subagent.

### When the system already behaves differently

An Example can fail because the existing code already does something else, such as a discount that starts at
£100 where the spec says £50. The spec wins: change the existing code to match.

If that breaks another test, first look for a narrower change that passes both: often the change was broader
than the spec asked for, such as a threshold changed for every customer where the spec is about members. A
narrower change may only follow a difference the spec itself states, never one made up to get both tests
passing, such as treating the spec's own values as a special case. If the two really contradict, leave the other
test failing and report it: which test, what it expects, and which Example says otherwise. Don't change it.

### When an Example won't pass

After three tries at the same Example, yours or the subagent's, stop and ask the author. Say what was tried and
what is in the way, and offer the choices: a hint or a decision from them, skipping that Example, or ending the
run.

## After the last Example

1. **Add the uncovered cases to the spec** and wire the ones with steps, with `--add-uncovered-cases` and
   without `as-gap-review-findings`, as [uncovered-cases.md](uncovered-cases.md) says. Examples that state what
   the code does now then guard the refactoring.
2. **Refactor**, with `refactor-using-subagent` (see [The refactoring pass](#the-refactoring-pass)).
3. **Run every test of each module** you or a subagent changed, not only this spec's (see "Compiling and running
   one test class" in [java-wiring.md](java-wiring.md)). Treat a test this run broke as
   [above](#when-the-system-already-behaves-differently), and run the modules' tests again after any fix. The
   unselected Examples of this spec fail as not yet implemented: they aren't broken by this run.
4. **Publish the uncovered cases** as gap findings, with `as-gap-review-findings`. For cases without steps, check
   first that each description still says what the code does now, since the refactoring has run.
5. **Check your own changes**, when no subagent wrote the code: look for
   [code that fits only the spec's values](#code-that-fits-only-the-specs-values) and
   [build changes that change how tests run](#build-files), and fix what you find.
6. **Remove the step methods nothing calls**, as step 8 of the skill says, then run the modules' tests again.
7. **Report**, as step 10 of the skill and [Reporting](#reporting) say.

## The refactoring pass

With `refactor-using-subagent`, once every selected Example passes, a subagent of its own refactors the code with
fresh eyes: duplication spread across several Examples, names, methods or classes to extract, conditions to
simplify. It changes no behaviour, and production code only.

1. Take a [snapshot](#snapshots).
2. Start the subagent with the brief in [Briefing the refactoring subagent](#briefing-the-refactoring-subagent).
3. When it is done, check its changes as [Checking a subagent's changes](#checking-a-subagents-changes) says,
   throwing out any change to a build file too.
4. If a test fails, send it the failing tests once. If they still fail after its fix, throw out the whole pass:
   put back every file it changed or deleted from the snapshot, and delete every file it added. Say so in the
   report, and why.

## Checks

### Checking a subagent's changes

Take a [snapshot](#snapshots) before each hand-off. When the subagent is done:

1. **Test folders.** Compare the working tree with the snapshot, and throw out every change in a test folder:
   `src/test/`, any other test source folder such as `src/integrationTest/`, and every `.specb` or `.feature`
   file, wherever it is. Put back the files it changed or deleted there, and delete the files it added there.
2. **Build files.** Check them as [Build files](#build-files) says.
3. **Tests.** Run them yourself: this spec's, at least. Don't take the subagent's word that they pass.
4. **Code that fits only the spec's values**, as [below](#code-that-fits-only-the-specs-values). Send any you
   find back to the subagent, saying where it is.

Name each change you threw out in the report.

### Code that fits only the spec's values

Read each production change for code that passes the Examples without doing what their Rules say: a branch on a
value the spec uses (`if (quantity == 3) return "£9.00"`), an expected result returned as a constant, a lookup
keyed by the spec's own data, or a check for whether the code runs under a test.

### Build files

A change to a build file (`pom.xml`, `build.gradle` or `build.gradle.kts`, `settings.gradle` or
`settings.gradle.kts`, `gradle/`, `.mvn/`) may add a dependency the production code needs. Read each one, and
throw out anything in it that changes how tests are found, compiled or run: the test plugin's settings (Surefire,
Failsafe, Gradle's `test` task), test includes, excludes and skip flags, test dependencies, the source folders,
and the annotation processor and SpecBinder's options. Name what you threw out in the report.

### Snapshots

A snapshot records the whole working tree, so you can see what a subagent changed and put files back, without
touching the author's own uncommitted changes, what they have staged, or their branch. In a git repository, write
the snapshot as a tree from a temporary index, kept in a temporary folder outside the project:

```
GIT_INDEX_FILE=<temporary folder>/snapshot-index git add -A
GIT_INDEX_FILE=<temporary folder>/snapshot-index git write-tree
```

The id it prints is the snapshot. To see what changed since, write a second snapshot the same way, with a new
index file, and compare the two with `git diff --name-status <before> <after>`. To put files back, use
`git restore --source=<before> --worktree -- <paths>`, which leaves the staging area as it is, and delete the
files the comparison lists as added. In a project outside git, copy the source folders and build files to the
temporary folder instead.

## Briefing the implementing subagent

In the first message, give it:

- the spec's path, the conventions you learnt in step 4 of the skill (how the project's code is laid out, the
  helpers and fixtures its tests use), and that it may read anything in the project;
- the Example to make pass now, by its title, with the output of its failing test;
- the rules:
  - write anywhere except test folders: `src/test/`, other test source folders, and `.specb` and `.feature`
    files. Never change a test, a step or the spec;
  - a build file may gain a dependency the production code needs, and nothing that changes how tests are found,
    compiled or run;
  - write just enough code for the failing Example, then tidy up what it added, keeping the earlier Examples
    passing. Build nothing for a case no Example asks for;
  - make the code do what the Example's Rule says, not fit the Example's values;
  - where existing behaviour has to change for the Example, change it: the spec wins. Make the change no broader
    than a difference the spec states;
  - report back what it changed and why, any other test it saw break, and every case the spec leaves open that
    it met, with what the code does in it now;
  - if it can't make the Example pass, say what is in the way rather than working around the tests.

For each later Example, send its title and the output of its failing test.

## Briefing the refactoring subagent

Give it:

- the spec's path, the conventions you learnt in step 4 of the skill, and the files this run created or changed;
- the task: refactor the production code this run wrote or changed, and existing code that duplicates it or is
  tangled up with it, without changing what any of it does;
- the rules: production code only, never a test, a step, the spec or a build file. Every test passes before and
  after: give it the commands for this spec's tests and for the changed modules' tests;
- to report back what it changed and why.

## Reporting

With `--implement`, the report also has:

- what you built, Example by Example, and each existing behaviour you changed to match the spec;
- each other test that contradicts the spec, left failing, with what it expects and which Example says otherwise;
- each step you changed after seeing it fail, and why;
- each change you threw out, from a subagent or a build file;
- the refactoring pass, when there was one: what it changed, or that it was thrown out, and why.
