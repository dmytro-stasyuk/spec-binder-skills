## `structure` — Given / When / Then roles

Kind: issue

- Flag: actions hidden in `Given` or `Then`, or assertions in `When`.
- Flag (blocker): an example with no observable outcome (no `Then`).
- Flag: an example missing the context it depends on.
- Flag: a `Given` that the example's outcome doesn't depend on, which hides the context that matters. A
  `Given` that sets the baseline other Examples of the Rule change is not extra.
- Flag: a `Background` holding a `When` or a `Then`. It holds only the `Given` setup its Examples share.
