## `declarative-steps` — intent over procedure

Kind: suggestion

- Flag: long imperative step sequences that walk through a procedure, where one declarative step would state the
  intent ("Given I am signed in as an admin", not five steps filling a login form).
- Flag (polish): incidental detail that doesn't affect the outcome under test (exact timestamps, IDs or unrelated
  field values), which hides what matters.
