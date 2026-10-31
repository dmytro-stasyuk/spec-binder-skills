## `user-perspective` — written outside-in, from the user's side

Kind: issue

- Steps describe what the user does and observes, in the user's own words.
- Flag: steps told from the system's or developer's side ("the service stores…", "the handler returns…").
- Flag: `Then` steps that assert internal state the user can't see (database rows, fields, logs, method calls),
  instead of an observable outcome.
- Flag: a user story (`As a / I want / So that`) whose role is not a real user or stakeholder.
- Flag (suggestion): a user story whose benefit just repeats the capability.
