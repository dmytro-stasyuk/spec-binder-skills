# SpecBinder review demo

A small shop project for trying the `review-spec` skill. Its cart spec,
`src/test/java/com/example/shop/cart/ShoppingCart.specb`, is written with a deliberate flaw for each kind of
finding, so one review shows all eight: blocker, issue, split, gap, suggestion, polish, question and praise. The
checkout spec beside it is there for the question finding to point at. The screenshots in the plugin's README
were taken from this project.

## Try it

1. Open this `demo` folder as a project of its own in IntelliJ IDEA, with the SpecBinder plugin installed and your
   agent connected to the IDE's MCP server (see the plugin's README).
2. Open `ShoppingCart.specb` and ask your agent to review it, or click **Review This Spec** on the spec's toolbar.
3. Turn on the **AI Review** tab under **Settings › SpecBinder › Split view** to see each finding as a card.

Open the demo on its own, not inside a checkout of the whole plugin repository. When both are open in the IDE,
the spec belongs to two open projects and the review can't tell which one to publish to.

## Guidelines

`.specbinder/review-guidelines/` holds the plugin's starter guidelines, copied as a project would copy them, and
`strengths.md`, a one-check category of the demo's own that gives the review something to praise. Edit them to
see how the review follows your guidelines.
