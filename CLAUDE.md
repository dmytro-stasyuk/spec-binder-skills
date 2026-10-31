# CLAUDE.md

This repository is the SpecBinder skills plugin (plugin name `specbinder`) for coding agents: it installs in
Claude Code and in Codex. It has no build: it is plugin manifests, a marketplace file and skills.

- `.claude-plugin/plugin.json` (read by Claude Code) and `.codex-plugin/plugin.json` (read by Codex) are the same
  plugin's manifests: keep their name, description and keywords in step. `.claude-plugin/marketplace.json` lets
  the repository itself be added as a marketplace, by Claude Code and by Codex.
- Skills are agent-neutral. Write "the agent", not Claude; don't name one agent's tools or commands in a skill,
  except where a step only exists for that agent; and name other skills by plain name in backticks
  (`automate-spec`), never with a plugin prefix.
- The writing skills (`write-spec`, `improve-story`, `add-rules`, `add-examples`, `add-steps`, `example-map`) and
  `answer-from-specs`, `spec-from-code`, `spec-from-tests`, `spec-changes`, `spec-map` and `spec-to-ticket` live in
  the companion GLAD plugin, `../glad-workflow-skills/` (plugin `glad`). No skill here depends on it. Its
  `write-spec/starter-guidelines/` overlap with `review-spec/starter-guidelines/`: keep the two in step where they
  cover the same ground.
- `skills/review-spec/`: the `review-spec` skill (`SKILL.md`) and its starter review guidelines
  (`starter-guidelines/`, one file per category). A review reads only the project's own guidelines; the starter
  files are copied into a project when its author asks for them.

The skill depends on MCP tools that the SpecBinder IntelliJ plugin (`../spec-binder-intellij-plugin/`) adds to
the IDE's MCP server: `specbinder_get_review_scope`, `specbinder_publish_spec_review` and
`specbinder_get_spec_review`.
Their names, parameters and result text are defined in that plugin (`specreview/mcp/SpecReviewToolset.kt`) and
specified by its `tests/specreview/*.specb` and `tests/split_view/ai_review/*.specb` specs. When either side
changes, update the other.

- Stage changes with `git add` when done. The user commits.
- Keep user-facing text free of Cucumber references. The plugin is presented as SpecBinder's.

## Changelog and releases

- Whenever a change alters what a user of the plugin sees — how the review skill behaves, what the starter
  guidelines flag, the setup it needs — add an entry to `CHANGELOG.md` under `## Unreleased`, in `Added`,
  `Changed`, `Fixed` or `Removed` (in Claude Code, the `/changelog` command does this). Entries are for end users:
  describe the effect on a review, not the skill's wording or the tool calls behind it. Changes to docs, the
  workflow or this file get no entry.
- Mark an entry `**⚠️ BREAKING:** ` when an existing user loses something: the release needs a newer SpecBinder
  IntelliJ plugin, or a project's guidelines (`.specbinder/review-guidelines.md` or
  `.specbinder/review-guidelines/`) are read differently. Say which IntelliJ plugin version is needed when that is
  the reason.
- Never edit `version` in `.claude-plugin/plugin.json` or `.codex-plugin/plugin.json`, or the released sections of
  `CHANGELOG.md`, by hand.
  Pushing the `rc` tag runs `.github/workflows/release.yml`, which bumps the version, stamps `## Unreleased` into
  a `## [X.Y.Z] - date` section and creates the GitHub Release. See `docs/releasing.md`.
