# Moving and renaming specs

The wiring basics are in [java-wiring.md](../../automate-spec/references/java-wiring.md). This file covers what
changes when a spec moves to another package or gets another name, and how to carry that out.

## Facts that drive every move

- **The generated class is named after the spec file.** `Payment.specb` generates `PaymentScenarios` (the suffix
  is `classSuffixIfAbstract` in `@Gherkin2JUnitOptions`, `Scenarios` by default). Renaming the file renames the
  generated class. Nothing inside the spec affects the name, so a move or rename never edits the spec.
- **The generated class lands in the spec's own package** when a bare or glob marker found the spec, and it
  extends that marker. The concrete test class (`PaymentTest extends PaymentScenarios`) therefore belongs in
  the spec's package, and moves with it. With an explicit-path marker, check in the generated sources where the
  class lands before and after the move.
- **What a marker covers depends on its annotation:**
  - a bare `@Gherkin2JUnit` covers the spec files in the marker's own package only, not subpackages;
  - a glob such as `@Gherkin2JUnit("./**/*.specb")` covers the marker's package and every package below it
    (`./` is the marker's package);
  - an explicit path names one spec file, and has to be edited when that file moves or is renamed.

  A spec that no marker covers generates nothing, and its test class stops compiling.
- **The marker decides inherited steps and options.** Step methods on the marker, its superclasses and its
  interfaces are not declared abstract in the generated class, and `@Gherkin2JUnitOptions` on the marker or
  its superclasses shape the generated code. A spec that moves under a different marker gets that marker's
  steps and options. If they differ, the test class gains or loses abstract steps, or the generated
  signatures change.
- **Concrete mode** (`@Gherkin2JUnitOptions(shouldBeAbstract = false)`): the generated class is itself the test
  (`<Name>Test`), its steps are implemented on the marker, and there is no concrete class to move. Such a spec
  stays with its marker: move both, or leave it in place.

## Choosing names

- Keep a spec's name when it moves. Rename it only to fix a name that doesn't follow the project's naming pattern
  (usually the Ability title in PascalCase), or when the destination already has a file of that name.
- Specs with the same name in different packages are fine: their generated and test classes live in different
  packages too.
- Never overwrite a file: when the destination folder already has a spec of that name, or its package a class
  of the test class's name, choose another name that says what distinguishes the spec, and list the rename in
  the plan.

## Procedure for each spec

Use the IDE's tools when its MCP server is connected: a rename refactoring tool (its name ends with
`rename_refactoring`) renames a class and updates its `extends`, imports and suite entries in one go; use a move
refactoring tool too if the IDE offers one. Otherwise use `git mv` (a plain move outside git) and edit the
references yourself, finding them with the IDE's search tools or `grep`. Read a file before editing it.

1. **Cover the destination.** Make sure a marker will cover the spec in its new package, with the same steps and
   options as before:
   - an existing marker whose glob reaches the new package, with the same superclass, interfaces and options;
   - else widen an existing marker's glob (`./**/*.specb`), if that doesn't pull in specs it shouldn't;
   - else add a marker in the new package, following the project's pattern: the same annotation, superclass,
     interfaces and options as the old marker. If the old marker implements steps itself, move those steps to a
     superclass or interface both markers share, rather than copying them, and list that in the plan.
   - an explicit-path marker: edit its path.
2. **Move the spec** to the new path, and rename it if the plan says so. Don't change its content.
3. **Move the concrete test class** to the same package. Update its `package` line. If the spec was renamed,
   rename the class and its `extends`: `class PaymentRefundTest extends PaymentRefundScenarios`. Fix its
   imports: classes it used from the old package without an import now need one, and package-private members
   it relied on must move with it or be made accessible.
4. **Move spec-specific files** with it: helpers only this test uses, and test data it loads relative to its
   own package. Shared test code moves separately, by who uses it, as the plan says: into a capability's package
   when only that capability's specs use it, or into the shared package at the root of the spec tree. Update the
   imports of everything that uses it.
5. **Update references** to the test class and the spec path:
   - JUnit suites: `@SelectClasses` entries, `@SelectPackages`, `@IncludeClassNamePatterns`,
     `@IncludePackages` and their exclude forms;
   - build settings that name tests or packages: Maven Surefire or Failsafe `<includes>`/`<excludes>`, Gradle
     `test { filter { … } }`;
   - saved run configurations and launch files that name a class or package (IntelliJ IDEA's `.run/*.run.xml`
     and `.idea/runConfigurations/`, VS Code's `.vscode/launch.json`, Eclipse `.launch` files), and CI scripts
     and workflows (`.github/workflows/*.yml`, `Jenkinsfile`, …);
   - documents in the repository that link to the spec's path.
6. **Clean up** after all moves: delete markers that no longer cover any spec, unless other classes extend or
   use them, and remove folders left empty.

## Building and checking

- Rebuild in full so the generated classes are regenerated and stale ones are removed:
  `mvn clean test-compile` / `./gradlew clean compileTestJava` (use the project's wrapper when there is one).
  Generated sources are under `target/generated-test-sources/` (Maven) or
  `build/generated/sources/annotationProcessor/java/test/` (Gradle).
- Each spec now has exactly one generated `*Scenarios` class, in its new package and under its new name.
- Run the moved tests, the changed suites, and every test that uses a shared class, helper, test-data file or
  resource that moved (`mvn test -Dtest=…` / `./gradlew test --tests …`). A test that didn't move can still
  break when a file it loads by path, or a class it uses, does.
- Check what ran against the specs, not against an earlier run: every scenario of every moved spec runs exactly
  once, a Scenario Outline once per row of its `Examples:` tables. A scenario with no test means its spec is
  covered by no marker or suite; one that ran twice means two markers' globs both cover its spec.
- Search the whole project, outside build output and `.git`, for each moved test class's old fully qualified
  name and each old package name. Every hit is a reference the moves missed: fix it. The search finds them
  whatever wrote them, IDE run configurations, CI files and build filters alike. A run configuration in a
  git-ignored folder such as `.idea/` is fixed only in this checkout: when you fix one, say so in the report,
  as other people's copies still name the old class.
- Tell from its error whether the moves caused a failure. A move changes only names and places, so its failures
  are of these kinds:
  - a class, package, file or resource that can't be found (`ClassNotFoundException`,
    `NoClassDefFoundError`, `FileNotFoundException`, a resource that loads as `null`);
  - an access error on a package-private class or member that now sits in another package;
  - an error naming a spec path, such as a `@SourceFilePath` or a marker glob that still names the old place;
  - a scenario whose test wasn't generated, as above.

  Fix those and run again until none is left. Any other failure, such as an assertion about behaviour, can't come
  from moving files when no spec and no production code changed: leave it, and list it in the report with its
  error, for the user to say whether it was failing before.
- Don't trust git's rename arrows in `git status`: with several near-identical test classes, git pairs them by
  content and may show the wrong pairs. Check the files themselves.
