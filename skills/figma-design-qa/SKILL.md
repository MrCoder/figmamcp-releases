---
name: figma-design-qa
description: >-
  Use when asked to polish, clean up, QA, or "make this frame look right" in a
  live Figma file through the figmamcp server, or when a design should keep
  being checked and fixed until it scores 9/10. Runs a loop: export the frame
  with node_export_visual, score it against a written rubric (alignment,
  spacing, type scale, contrast, naming, auto-layout), fix the worst
  categories with figmamcp write tools, re-export, re-score, and stop at the
  target score or the round limit.
---

# figma-design-qa

A look-critique-fix loop on the **live** Figma file. The agent edits a frame, exports it with `node_export_visual`, scores it with a fixed rubric, fixes the lowest categories, and repeats until every category is 9 or more.

This skill sits on top of the `figmamcp` skill. Load that skill's `references/constraints.md` and `references/error-playbook.md` before the first write: every rule there applies here (scope lock, verbatim names, batch pre-validation, no default parent).

## Inputs and output

| | |
|---|---|
| **Input** | A frame (ID, Figma link, or name), or "pick one" on a page. Optional: target score (default every category ≥ 9), round limit (default 5), a type scale or palette the file already uses. |
| **Output** | Per-round table of category scores with the fix applied in each round, the final export, and anything left unfixed with the reason. |
| **Writes** | Only inside the chosen frame. Never other frames, never styles or variables unless the user asks. |

## Procedure

1. **Discover.** `page_info()` → pick the page → `page_info({ pageIds: [pageId] })` → `node_info({ nodeIds: [frameId], maxDepth: 3, properties: [...] })` (property list in [references/rubric.md](references/rubric.md)). Keep the returned names; you pass them back verbatim on every write.
2. **Confirm the target.** Say which frame you will edit, by name and ID. If the user named no frame, choose a small one and say why. If `channel_join` lists several files, join the one the user named; never guess.
3. **Round 0: look and score.** `node_export_visual({ nodeId: frameId, format: "PNG", scale: 1 })`. Score all six categories from the `node_info` numbers ([references/rubric.md](references/rubric.md)). Write the violations down as a list with the numbers that prove each one.
4. **Pick this round's fixes.** Take the 1 or 2 categories with the largest penalty. Fixing one or two per round keeps each diff small enough to review, and shows where each point came from.
5. **Apply.** Use the tool table below. After any `node_rename`, the old name no longer verifies: read the nodes again before the next write, or do renames last in the round.
6. **Look again.** Export the frame again and read `node_info` again. Rescore every category, not only the ones you fixed: a layout change can move text and break contrast or alignment elsewhere.
7. **Stop or repeat.** Stop when every category is 9 or more, or at the round limit. If a round raised no score, stop and report: the fix is outside what the rubric or the tools can do.
8. **Report.** One table, one row per round: fix applied, six category scores, overall. Then the final export, then anything left and why.

## Which tool fixes which category

| Category | Typical fix | Tool |
|---|---|---|
| Alignment, spacing | Turn the container into a stack with even padding and gap; set children to fill width | `node_set_auto_layout` (`layoutMode`, `padding*`, `itemSpacing`, `counterAxisAlignItems`, `layoutSizingHorizontal: "FILL"` on children) |
| Alignment (no auto-layout wanted) | Move or resize one node | `node_transform` |
| Type scale | Snap `fontSize` to the scale | `text_set_style` (change `fontSize` and keep `fontName`; a new font family can fail if the font is not loaded) |
| Contrast | Darken text or lighten the background | `node_set_fill` for a literal colour; `node_bind_variable` or `node_apply_style` if the file has a colour token or style for it |
| Naming | Give layers a role name | `node_rename` (last in the round) |
| Auto-layout | Single-child boxes (inputs, buttons) get horizontal auto-layout with padding and centered children | `node_set_auto_layout` |

Read-only tools for each round: `node_info`, `node_export_visual`. Neither changes the file.

## Rules that keep the loop safe

- **Names are verified on every write.** Pass the `nodeName` exactly as the last `node_info` returned it. A `nodeName does not match` refusal means your copy is stale: read again, do not retry with the same input.
- **Errors are deterministic.** A refused write returns the same refusal on retry. Change the input or stop and report (see the figmamcp error playbook).
- **Partial results.** If a result has `details.partialMutation: true`, read what changed with `node_info` before the next write.
- **Layer order is layout order.** When you add auto-layout to a frame, Figma orders children by layer order, not by their old `y`. If the layers are out of visual order, the stack will reorder them. Check the export after the first layout change.
- **Auto-layout changes size.** With `layoutSizingVertical: "HUG"` the frame height changes. Exports from different rounds can have different heights; that is expected.
- **Score from numbers.** The export shows the result; `node_info` gives the numbers you score. If they disagree (for example, a node is clipped), report it.
- **No scope widening.** If the frame is outside the editable scope, or the plugin is read-only (`READ_ONLY_MODE`), stop and tell the user. Do not move the work to another frame or file.

## Example result

From a live run on a deliberately messy sign-up card (full numbers in [references/rubric.md](references/rubric.md)):

| Round | Fix | Overall |
|---|---|---|
| 0 | none | 1.7 |
| 1 | Card auto-layout, padding 32, gap 16 | 5.2 |
| 2 | Type on scale, grey text to 4.5:1 or more | 7.7 |
| 3 | Field and button auto-layout, 10 layers renamed | 10.0 |
