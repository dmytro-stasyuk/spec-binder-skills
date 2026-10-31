## `language` — clear, consistent domain language

Kind: polish

- Flag (suggestion): the same domain concept named differently across the spec ("customer", "client", "user" for one
  role).
- Flag (suggestion): jargon or abbreviations a business reader would not know, where a plain domain word exists.
- Flag (issue): vague outcomes that can't be checked ("works correctly", "is handled", "succeeds").
- Flag: an official or legal voice where plain words would do ("shall", "it is required that", "the system is
  expected to", "in the event that").
- Flag: filler that adds nothing a reader needs ("successfully", "correctly", "basically", "Given that …",
  "in order to"): "Then the order is placed", not "Then the order is successfully placed".
- Flag: a double negative or a negated opposite where the positive says the same ("should not be unpaid",
  "is not disabled"). A plain "not" that states the outcome ("Then I should not see the warning") is fine.
- Flag (suggestion): a passive `When` that leaves out who acts, where it matters ("When the order is cancelled", where
  the shopper, the shop or a timeout could cancel it).
