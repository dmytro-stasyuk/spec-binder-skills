# Releasing and distributing updates

This repository is both the `specbinder` plugin and its own marketplace (`.claude-plugin/marketplace.json`, with
the plugin entry at `"source": "./"`). Users install it from the public GitHub repository:

```
/plugin marketplace add dmytro-stasyuk/spec-binder-skills
/plugin install specbinder@specbinder
```

## How Claude Code detects a new version

Claude Code works out a plugin's version from, in order:

1. `version` in `.claude-plugin/plugin.json`;
2. `version` in the plugin's marketplace entry;
3. the git commit SHA, when neither is set.

It compares that version with the one it has installed and only updates when they differ. **Pushing commits
without changing the version does not reach existing users**: they keep their cached copy. New installs are
different: they clone the latest commit of `main`, so they get unreleased changes straight away.

This plugin sets `version` in `plugin.json` only. Leave it out of the marketplace entry, so there is one source of
truth. Removing `version` altogether would make every push to `main` an update, which is simpler but ships work
in progress the moment it lands.

## Changelog

`CHANGELOG.md` follows [Keep a Changelog](https://keepachangelog.com). Every user-visible change adds an entry
under `## Unreleased`, in `Added`, `Changed`, `Fixed` or `Removed`. The `/changelog` command writes it. Leave the
empty category headings in place: the release fills them in and recreates them. Released sections are a record
of what shipped and are never edited. A correction goes in as a new entry.

An entry starts with `**⚠️ BREAKING:** ` when an existing user loses something: the release needs a newer
SpecBinder IntelliJ plugin, or a project's guidelines (`.specbinder/review-guidelines.md` or
`.specbinder/review-guidelines/`) are read differently.

## Version numbers

Versions follow the SpecBinder IntelliJ plugin's scheme, `YEAR.N.P`:

- the major part is the calendar year of the release;
- the minor part counts releases and carries on across years rather than starting again, so 2026.7.0 is followed
  by 2027.8.0;
- the patch part is for a fix-only release on top of a release from the same year: 2026.4.0 → 2026.4.1.

The plugin's first release, 0.1.0, predates the scheme; the release after it is 2026.2.0.

## Making a release

The version in `.claude-plugin/plugin.json` is always the last released one, and only the release workflow
(`.github/workflows/release.yml`) changes it. It writes the same version to `.codex-plugin/plugin.json`, the
manifest Codex reads, so the two never differ.

1. Check that `## Unreleased` says everything the release contains, and validate the plugin:
   ```bash
   claude plugin validate --strict .
   ```
2. Push the `rc` tag on `main`:
   ```bash
   git tag rc && git push origin rc
   ```
   This makes a minor release. For a patch release, run the **Release** workflow from the repository's Actions
   tab instead and pick `patch` there.
3. The workflow then:
   - writes the new version to `plugin.json`;
   - turns `## Unreleased` into `## [X.Y.Z] - YYYY-MM-DD` (dropping empty categories) and puts a fresh, empty
     `## Unreleased` above it;
   - commits `release X.Y.Z`, tags it `X.Y.Z`, pushes to `main`, and renames the `rc` tag to `rc_X.Y.Z`;
   - creates a GitHub Release with that section as its notes.

   It stops without changing anything if `## Unreleased` has no entries.
4. Pull the release commit (`git pull`), then check the update from a Claude Code session:
   `/plugin marketplace update specbinder`, then `/reload-plugins`.

If a run fails before pushing, delete the `rc` tag (`git push origin :refs/tags/rc && git tag -d rc`), fix the
cause and push it again.

### Compatibility with the IntelliJ plugin

The `review-spec` skill calls the `specbinder_get_review_scope`, `specbinder_publish_spec_review` and
`specbinder_get_spec_review` tools that the SpecBinder IntelliJ plugin adds to the IDE's MCP server. When a
release depends on a change to those tools, release the IntelliJ plugin first and note the minimum IntelliJ
plugin version in the README. Otherwise users can end up with a skill that calls tools their IDE does not have
yet.

## How users receive updates

### Auto-update

Auto-update is **off by default** for third-party marketplaces such as this one. Each user turns it on once:
`/plugin` › **Marketplaces** › `specbinder` › **Enable auto-update**.

With it on, Claude Code refreshes the marketplace in the background, up to 10 minutes after the first message of
an interactive session. The running session keeps the old version and shows
`Plugin updated: specbinder · Run /reload-plugins to apply`; new sessions load the new version.

A project can turn this on for everyone who works in it by listing the marketplace in its
`.claude/settings.json`, under `extraKnownMarketplaces`, with `autoUpdate: true`.

### Manual update

- In a session: `/plugin marketplace update specbinder`, then `/reload-plugins`.
- From the shell: `claude plugin update specbinder@specbinder`.

### Environment variables that affect updates

- `DISABLE_AUTOUPDATER=1` turns off background updates; `DISABLE_UPDATES=1` also blocks manual ones.
- `CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC=1` turns off all background network activity, marketplace refresh
  included.
- `FORCE_AUTOUPDATE_PLUGINS=1` keeps plugin auto-update on despite the three above.

## Release channels (optional)

Not used yet, but available if a "stable" and a "latest" track are ever wanted:

- Users can pin a branch or tag when adding the marketplace:
  `/plugin marketplace add dmytro-stasyuk/spec-binder-skills#stable`.
- Alternatively, publish two marketplaces with different `name` values, whose plugin entries point at different
  refs (a plugin `source` object of type `github` accepts `ref` and `sha`). Users add the one they want.
