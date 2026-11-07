## `title` — titles name the situation

Kind: suggestion

- Flag: an Example or Scenario title that restates the outcome the Rule already states, or names the action
  instead of the context or input that makes the case distinct ("with an expired card", "when the cart is
  empty").
- To test an Example title, read it as "the one where …" ("the one where the card has expired"): it
  should name the situation, not the action taken or the outcome. The title needn't start with those words.
- Flag: sibling Example titles under one Rule that don't tell the Examples apart when read side by side, such as
  near-identical titles or ones that differ only by "valid" and "invalid".
- Flag: a Rule title that is a label or a topic ("pricing", "discount handling") rather than a statement, with a
  subject and a verb, of what holds ("the cart totals the price of what it holds").
- Flag (polish): sibling Example titles under one Rule phrased in different shapes ("with an expired card", "card
  blocked", "when the CVV is wrong"), so they don't read as a set.
- Flag (blocker): a title that no longer matches what the steps do.
- Flag: a Feature or Ability title that names a component or screen rather than a capability.
- Flag: a spec file whose name doesn't say what the spec is about, naming something else (`Checkout.specb` for a
  spec about refunding an order) or nothing in particular (`NewSpec.specb`). The fix renames it after the
  spec's title, in the form the project's other spec file names take (`RefundOrder.specb`).
- A file name that is an issue tracker's key, such as `BR-123.specb`, names the ticket the spec came from, not
  what it is about: leave it as the team named it. When a name pairs a key with words, as `BR-123-checkout.specb`
  does, flag only words that miss what the spec is about, and keep the key in the new name
  (`BR-123-refund-order.specb`).
