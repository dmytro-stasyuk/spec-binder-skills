# Wiring a SpecBinder spec to Java (JUnit 5)

How a spec file becomes a runnable JUnit 5 test, and how to write the code behind it. Shared by
`automate-spec`, `split-spec` and `restructure-spec-folders`.

When this file and the project disagree, the project wins: read the generated code and copy what it does.

## The classes, in either mode

SpecBinder generates in one of two modes, set by `@Gherkin2JUnitOptions(shouldBeAbstract = …)` on the marker or
any class above it. The option is inherited, and different markers in one project can set it differently, so
work out the mode for each spec. Without the option, the mode is abstract.

**Abstract mode** (the default):

```
<Name>Feature     you write it    a marker annotated @Gherkin2JUnit, pointing at one or more spec files
   ^
<Name>Scenarios   generated       abstract; extends the marker; one @Test per Example, one @Nested class
   ^                              per Rule, one public abstract method per distinct step
<Name>Test        you write it    extends <Name>Scenarios and implements the step methods; JUnit runs this
```

An unimplemented step is a compile error in `<Name>Test` ("is not abstract and does not override abstract
method …"). That is the point: the compiler's error list is the to-do list.

**Concrete mode** (`shouldBeAbstract = false`):

```
<Name>Feature     you write it    the marker; the step methods go here (or above it)
   ^
<Name>Test        generated       concrete; extends the marker; one @Test per Example, one @Nested class per
                                  Rule, and a stub for each step not yet implemented above it; JUnit runs this
```

Never write a `<Name>Test` in concrete mode: the generator writes a class of that name, and yours would clash
with it. What an unimplemented step does depends on `unimplementedStepBehavior`:
- `FAIL` (the default): its stub calls `Assertions.fail("Step is not yet implemented")`, so the class compiles
  and each Example using the step fails with that message.
- `ABORT`: its stub calls `Assumptions.assumeTrue(false, "Step is not yet implemented")`, so those Examples are
  reported as aborted (skipped), not failed.
- `COMPILATION_ERROR`: its stub's body is the bare text `Step is not yet implemented`, so the generated class
  doesn't compile until the step is implemented.

In both modes:
- SpecBinder is an annotation processor. It runs during `javac` (test compilation), so the generated class
  exists only after the test sources compile, and is regenerated on every build. Never edit it.
- Generated sources land in `target/generated-test-sources/test-annotations/` (Maven) or
  `build/generated/sources/annotationProcessor/java/test/` (Gradle).
- The generated class's name is the spec file's name plus a suffix: `classSuffixIfAbstract` (default
  `Scenarios`) or `classSuffixIfConcrete` (default `Test`).

## Markers and which specs they cover

- `@Gherkin2JUnit` with no value: every `.specb` and `.feature` file in the marker's own package. It does not
  look in subpackages.
- `@Gherkin2JUnit("./**/*.specb")`: a glob, resolved against the marker's package. `**` reaches subpackages,
  so one marker can cover a whole tree of specs.
- `@Gherkin2JUnit("specs/cart.feature")`: one file by path, for example under `src/test/resources`.
- The generated class is named after the spec **file** (`cart-items.specb` → `CartItemsScenarios` in abstract
  mode) and is put in the **spec file's package**, which is not necessarily the marker's. It extends the
  marker. In abstract mode, put `<Name>Test` in that same package.
- Specs usually sit next to their marker in `src/test/java`. With Maven that needs a `<testResources>` entry
  including `src/test/java` with `**/*.specb` and `**/*.feature`; if specs there are not being picked up,
  check the `pom.xml` for it.
- `@Gherkin2JUnitOptions` on a marker or any class above it configures generation and is inherited; a child
  overrides only the options it sets. Options that change the code you write: `shouldBeAbstract`,
  `dataTableParameterType`, `useStepKeywordInStepMethodName`, `emptyScenarioBehavior`, `emptyRuleBehavior`.

## What the generated class contains

- **Ability** (= Feature): the class. Its `@DisplayName` is the title; the description goes into its Javadoc.
- **Example** (= Scenario): a `@Test` method `scenario_1`, `scenario_2`, … in file order.
- **Rule**: a non-static `@Nested` inner class `Rule_1`, `Rule_2`, …; its Examples are
  `rule_1_scenario_1`, …. Being inner classes, they call the same step methods and see the same fields as the
  outer test instance.
- **Background**: a `@BeforeEach` method calling the steps: `background(TestInfo)` at Ability level,
  `ruleBackground(TestInfo)` inside a Rule's class. The Ability's runs first, then the Rule's.
- **Scenario Outline** with `Examples:`: one `@ParameterizedTest` with a `@CsvSource` holding the table. Each
  `<placeholder>` in a step becomes a step argument, typed from its column (a column falls back to `String` if
  any value doesn't fit the inferred type).
- **Empty Example or Rule** (title only): a test that fails (`emptyScenarioBehavior` /
  `emptyRuleBehavior`: `FAIL` by default, or `ABORT`, or `COMPILATION_ERROR`), tagged `new`.
- **Tags**: `@Tag` on the class, Rule class or test method.
- **Comments** (`#` lines): ignored by the generator.

JUnit creates a fresh test instance for every Example (and every outline row), so instance fields are the
scenario's state and start clean each time. They live on `<Name>Test` in abstract mode, and on the marker (or a
class above it) in concrete mode.

## Step methods

The generated class declares each distinct step once: as `public abstract void …` in abstract mode, or as a
`public void …` stub in concrete mode.

**Names.** Derived from the step's text:
- The keyword (`Given`, `When`, `Then`, `And`, `But`, `*`) is dropped, so the same text under different
  keywords is one method. (`useStepKeywordInStepMethodName = true` keeps it as a prefix: `givenMyCart…`.)
- Words are split on whitespace. The first word is all lower case; every later word gets a capital first
  letter and the rest of it in lower case (`the JSON report` → `theJsonReport`).
- Characters that can't be in a Java identifier are dropped. Inside a word, punctuation other than an
  apostrophe starts a new capital (`signed-in` → `SignedIn`, `user's` → `Users`).
- Each `"quoted value"` (and each `<placeholder>` in an outline) is replaced by `$p1`, `$p2`, … in place:
  `I see the "Free shipping" banner` → `iSeeThe$p1Banner`.

**Parameters.** One per quoted value or placeholder, in order, named `p1`, `p2`, … (you may rename them in
your override). Types are inferred from the value: `true`/`false` → `Boolean`, whole numbers → `Integer`
(`Long` if too large), decimals → `Double`, a single character → `Character`, anything else → `String`. So
`"1"` and `"1.5"` in two uses of one step can clash; the compiler will say so.

**Doc string.** A step followed by `"""` gets one extra trailing `String docString` parameter. The text arrives
verbatim (indentation kept) and ends with a newline. A content type after the opening `"""` (`json`, `svg`, …)
is for the IDE only; the value is still a `String`.

**Data table.** Depends on `dataTableParameterType`:
- `LIST_OF_OBJECT_PARAMS` (default): a `List<XParam>`, where `X` is the step's last word capitalised
  (`my cart contains the following products:` → `List<ProductsParam> products`). `XParam` is a static class
  nested in the generated class, with one typed field and accessor per column (`name()`, `quantity()`). To
  tighten types (say, an enum column), move the step method and the `XParam` class into the marker and edit
  them; the generator then uses yours.
- `LIST_OF_MAPS`: `List<Map<String, String>>`, header cells as keys.
- `CUCUMBER_DATA_TABLE`: a `DataTable`; needs a `getTableConverter()` in the hierarchy.

**JUnit-resolved parameters.** A step can also receive `TestInfo`, `TestReporter`, `@TempDir Path`, or a value
from a custom `ParameterResolver` (marked `@JUnitResolved`). The generator only sees the marker's hierarchy, so
declare such a step there with its full signature: as an abstract method in abstract mode, implemented by
`<Name>Test`; implemented directly on the marker in concrete mode.

**Never hand-guess a signature.** Build, open the generated class, and copy each step method's exact signature:
in abstract mode into `<Name>Test`, with `@Override`; in concrete mode onto the marker, without `@Override`,
since nothing above it declares the step. On the next build, the generator finds the marker's method and drops
the stub.

## Where step implementations live

In concrete mode every step goes on the marker, or on a base class or interface above it, so all the specs under
one marker share one implementation per step text. Name the marker's fields after what they hold, since they
serve every spec under it. The rest of this section is about abstract mode.

A step method placed on the marker, a base class it extends, or an interface it implements (as a `default`
method) is detected by the generator, which then leaves that step out of the generated class. Every `*Test`
under that marker inherits the one implementation. So:

- A step that only this spec uses goes on `<Name>Test`.
- A step several specs share, worded identically, goes once in the shared place the project already uses: the
  marker, a base step class, or a step interface (interfaces hold no state, so they declare an accessor such as
  `Cart cart()` that the test implements). Never implement the same step twice.
- Share `Given` and `When` steps. Keep `Then` steps on the concrete test unless they are truly generic:
  shared assertions couple unrelated specs and hide what each one checks.
- When you find two `*Test` classes with the same step and the same body, hoist it: move the method (and any
  private helper it uses) to the shared place, delete every copy, rebuild, and re-run each affected test.
  A copy you missed shows up as "method does not override or implement a method from a supertype".

## Finding the project's existing wiring pattern

Before creating anything, look at how the project already does it, and copy that:

1. Search the test sources for `@Gherkin2JUnit` and `@Gherkin2JUnitOptions`: which markers exist, which specs
   their value or glob covers, and which options apply.
2. Find classes that extend a `*Scenarios` class: where abstract mode's hand-written tests live and how they are
   named. In concrete mode there are none: the steps are on the markers.
3. Note what markers extend and implement: base test classes (fixtures, Spring context, browser setup) and
   step interfaces.
4. Search those for steps whose wording matches this spec's steps: they are already implemented and need no
   override.
5. Look at the nearest existing step implementations for the house style (in `*Test` classes in abstract
   mode, on markers in concrete mode): how they reach the system, the helpers and fixtures they use, and their
   assertion style.

**Prefer what is nearest.** When the project does things in more than one way, copy the pattern of the specs
closest to this one: first those in the spec's own package, then those in sibling packages (under the same
parent), then further out. Nearby specs usually exercise the same part of the system with the same fixtures,
while distant ones may follow another area's or an older convention. This applies to every choice above: the
marker, the base classes and step interfaces, where shared steps go, and the style of the step bodies.

If a marker already covers the spec's location, use it and add only `<Name>Test` in abstract mode, or only the
missing steps on the marker in concrete mode. If the project has a pattern for a new area (a marker per package
extending a common base, say), follow it. If the project has no SpecBinder
specs wired yet, use the simple layout: a marker `<Name>Feature` with a bare `@Gherkin2JUnit`, in the spec's
package, plus `<Name>Test extends <Name>Scenarios`, and tell the user that is the layout you chose.

If the build has no SpecBinder dependency at all (`dev.specbinder:annotations` and
`dev.specbinder:annotation-processor`, the latter on the test annotation-processor path), tell the user what is
missing and ask before changing build files.

## Writing step bodies

- **Exercise the code the way the project's specs and tests already do**: at the same level (a class, a
  service, an HTTP endpoint, the UI) and with the same fixtures and helpers for setting it up and reaching it.
- Keep scenario state in fields of `<Name>Test` (or behind the accessor a step interface asks for).
- Fake only what the project's tests already fake, and the same way. Don't stub the very code a step is
  meant to exercise.
- A step whose production code doesn't exist yet (no method, endpoint or UI control to call) must **fail, not
  pass**: throw
  `new UnsupportedOperationException("Not yet drivable: <what is missing, e.g. no endpoint to cancel an order>")`.
  The project still compiles, the Example fails with that message, and the missing piece goes in your report.
  Never leave a step empty or make it pass without checking anything.
- Don't change production code to make a step drivable, and don't edit the spec to fit the code. If a step's
  wording makes it impossible to automate faithfully, stop and tell the user.

### Assertions

Write `Then` steps so a failure shows the expected and actual values, and so the SpecBinder IntelliJ plugin
can offer **Accept actual** (it rewrites the expected value in the spec with the actual one):

- Use one equality assertion per step that fails with `org.opentest4j.AssertionFailedError` carrying both
  values: SpecBinder reads expected and actual from that error, whichever library threw it. Use the library the
  project's tests already use. JUnit's `assertEquals(expected, actual, message)` (expected first) and AssertJ's
  `assertThat(actual).isEqualTo(expected)` both qualify; JUnit 5 always puts opentest4j on the test classpath.
  Assertions that only report true or false, such as `assertTrue`, Hamcrest's `assertThat`, or a thrown
  exception, carry no values.
- Pass the spec's value as `expected` unchanged. The only allowed normalisation is trimming the doc string's
  trailing newline. Don't parse it, re-order it, change case or collapse whitespace.
- Render `actual` into exactly the format the spec uses for that value, deterministically (stable order, fixed
  formatting), so that when it is right it equals the expected text character for character.
- Compare a whole block in one call, not field by field. A value that can contain line breaks needs a doc
  string in the spec.

## Compiling and running one test class

Build and run tests with the project's build tool from the command line, even when an IDE is connected. If the
user explicitly asks you to use the IDE, use its build and run tools instead. On the command line, use the
project's wrapper (`./mvnw`, `./gradlew`) when there is one:
- Generate and compile: `mvn -q test-compile` or `gradle compileTestJava`.
- Run one class: `mvn -Dtest=CartTest test` (add `-pl <module> -am -Dsurefire.failIfNoSpecifiedTests=false`
  in a multi-module build), or `gradle test --tests 'com.acme.cart.CartTest'` (`:<module>:test` for a
  module).
- Check the run included the Examples under Rules (they run as nested classes such as `CartTest$Rule_1`). If
  they are missing, widen the filter to `CartTest*`.

If the project's own instructions (its agent instructions file, such as `AGENTS.md` or `CLAUDE.md`) say how to
build or run tests, follow them instead.

### Reading compile errors

| Error | Meaning |
|---|---|
| `<Name>Test is not abstract and does not override abstract method xyz(…) in <Name>Scenarios` | a step still to implement: copy its signature from the generated class |
| `method does not override or implement a method from a supertype` | the step's text changed (so its name did), or it is now implemented higher up; delete or rename the override |
| `incompatible types` at a call in the generated class | the spec's values infer a different type than your method takes, or a refined `XParam` / enum doesn't accept a value |
| `cannot find symbol: class <Name>Scenarios` | the processor didn't run for this spec: no marker covers it, the glob misses it, or the build has no annotation processor |
| `duplicate class: …<Name>Test` (concrete mode) | you wrote a `<Name>Test`, but the generator writes one: move its steps to the marker and delete it |
| errors on a `Step is not yet implemented` line in the generated class (concrete mode, `COMPILATION_ERROR`) | a step still to implement on the marker |
| `method does not override or implement a method from a supertype` on the marker (concrete mode) | remove `@Override`: a step on the marker overrides nothing |

After any change to a spec's step text, rebuild before trusting the generated class.

### Reading test results

Treat every failure as one of three things, and say which in your report:
- **A wiring bug** (wrong call, wrong setup, leaked state, a faulty assertion): fix it and run again.
- **Missing production code** (an `UnsupportedOperationException("Not yet drivable: …")`): expected; list it.
- **The system behaves differently from the spec**: the assertion's expected and actual values show how.
  Report it; don't change the spec or the production code to make it pass unless the user asks.

In concrete mode, a failure or abort saying "Step is not yet implemented" is none of these: it is a step still
to write on the marker. Write it and run again; it is never a result to report.
