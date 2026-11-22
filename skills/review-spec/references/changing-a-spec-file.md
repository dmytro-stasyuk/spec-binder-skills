# Changing a spec file

How a skill changes a spec file that already exists. Shared by `review-spec`, `automate-spec`,
`move-under-new-rule`, `move-under-new-spec`, `separate-out-specs` and `extract-composite-step`.

The author may have the spec open in IntelliJ IDEA, with Rules and Examples folded. A spec written over on disk is
reloaded by the IDE, and the reload unfolds every fold the change spans: a moved Example or a new block at the end
can open them all. So change the spec in the IDE's editor when you can, with the first of these that is available.

A new spec file is written with your own file tools: no editor has it open yet.

## 1. `specbinder_edit_spec`

A tool the SpecBinder IntelliJ plugin, 2026.23.0 or later, adds to the IDE's MCP server; its name ends with
`specbinder_edit_spec`. It makes the change in the spec's editor and saves the spec. The author's folds stay as
they were, but for a folded Rule or Example the change adds lines to, which opens so the author sees them; one Undo
in the IDE takes the whole change back.

1. **Read the spec** through the IDE's file tool (`read_file`, or `get_file_text_by_path` on an IDE that lacks it),
   not from disk: the edits name lines as the editor has them, unsaved changes included.
2. **Give the whole change in one call**, as `specPath` and `edits`: one edit per place the change touches, such
   as where an Example is moved from and where it goes. Each edit is
   `{"fromLine": n, "toLine": m, "replaces": "...", "text": "..."}`. It replaces lines `n` to `m`, both
   inclusive; `replaces` is those lines copied word for word, with `\n` between them, and `text` is the new
   lines, without a trailing line break.
   - An empty `text` removes the lines, line breaks and all.
   - To add lines, rewrite the line they go after, kept at the start of `text`.
   - The edits' lines must not overlap.
3. **If the answer starts with `Error:`**, nothing was changed: the author may have edited the spec since you
   read it. Read it again, and give the edits for it as it now reads. Otherwise the answer names each edit's lines
   as they are now.

If you reach the IDE's tools only through its `execute_tool`, quote `edits` as `review-spec`'s
[SKILL.md](../SKILL.md) says to quote `findings`.

## 2. The IDE's `replace_text_in_file`

When the IDE's MCP server is connected but has no `specbinder_edit_spec`, as with an older SpecBinder plugin. It
replaces text in the spec's editor and saves the spec, so the folds away from the change stay too; but it takes one
call per place, and the author one Undo per call.

- Read the spec through the IDE's file tool first, as above.
- Pass `replaceAll: false`, and quote enough of the old text that it occurs once in the spec: the tool replaces
  the first place it occurs.
- To add lines, rewrite the line after them, kept at the end of `newText`; at the end of the spec, rewrite the
  last line with the line break after it, kept at the start of `newText`. Rewriting only the line before them
  puts the added lines in its fold when the author has its Example or Rule folded, out of sight.

## 3. Your own file tools

Without the IDE's MCP server, write the file as usual.
