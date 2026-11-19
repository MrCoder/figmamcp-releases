---
name: figma-handoff-notes
description: >-
  Use when the user wants developer handoff notes for a Figma frame: read the
  frame's layout, spacing, typography, colors, styles, tokens and components,
  then write native Figma annotations on the right layers and a written handoff
  summary, through the FigmaMCP server. Checks existing annotations first so it
  never duplicates them.
---

# Figma: write handoff notes for a frame

Reads a frame the way a developer needs it (structure, auto layout, sizes, type, colors, the styles and tokens behind them, components and their props) and writes the result two ways: native Figma annotations on the layers that need them, and a handoff summary in the chat. Requires **FigmaMCP** (a Mac app with a local MCP server and a Figma plugin, <https://figmamcp.dev>); the tools below are FigmaMCP tools.

## When to use

- "Write handoff notes for the Settings screen."
- "Annotate this frame with spacing and tokens for the dev team."
- Before a developer or a code-generation agent builds a screen from the design.

For a read-only summary with no annotations, skip step 6.

## Preconditions

1. The Figma desktop app has the file open and the **FigmaMCP plugin is running** in it.
2. To write annotations, the plugin's **AI can edit this file** switch is on. If it is off, `annotation_set` fails with `READ_ONLY_MODE`; give the summary in the chat only and say why.
3. The user has named the frame, or pasted a Figma link to it.

## Steps

1. **Find the frame.** With a link, `node_info({ nodeIds: [id], maxDepth: 0, properties: ["name", "type", "width", "height"] })`. With only a name, `page_info()` then `page_info({ pageIds: [pageId] })`.
2. **Read structure and layout.** `node_info({ nodeIds: [frameId], maxDepth: 6, properties: ["name", "type", "width", "height", "layoutMode", "itemSpacing", "paddingTop", "paddingRight", "paddingBottom", "paddingLeft", "layoutSizingHorizontal", "layoutSizingVertical", "cornerRadius", "fills", "strokes", "characters", "fontName", "fontSize", "lineHeight"] })`. Ask only for what you need; large frames return large results.
3. **Read what is shared, not literal.** `node_info` with `properties: ["fillStyleId", "textStyleId", "effectStyleId", "boundVariables"]` on the same frame shows which values come from styles and tokens; the response resolves each to `{ id, name }`. `style_list()` and `variable_list()` list the file's local styles and variables. For instances, read `mainComponent` and `componentProperties` (slower; only on instance layers), or `instance_get_overrides({ nodeId })` for one instance.
4. **Look at it.** `node_export_visual({ nodeId: frameId, format: "PNG", scale: 0.5 })` so the notes match what the design shows.
5. **Check existing annotations.** `annotation_list({ nodeId: frameId })`. It returns the notes already on the frame's layers and the file's annotation categories. Do not write a note a layer already has. Use a `categoryId` from this result if the file has a fitting category (for example "Development").
6. **Write the annotations.** One `annotation_set({ annotations: [ { nodeId, nodeName, labelMarkdown, categoryId? }, … ] })` call with every note. `nodeName` is the layer's current name, verbatim from step 2. Keep each note short and specific: "Auto layout vertical, gap 16 (`spacing/md`), padding 24". Only add the optional `properties` field (for example `[{ type: "itemSpacing" }]`) when the layer clearly has that property; a text property on a rectangle is refused.
7. **Handle the result.** A `success` status is done. For `partial_success` or a failed row: annotations are appended, not replaced, so a failed row may already be on the layer. Run `annotation_list` again and compare labels before retrying anything.
8. **Give the summary** in the chat: screen size, layout tree with gaps and padding, text styles, colors and tokens, components with their variant props, and anything that is not on a token (a literal color or size) as a list of follow-ups.

## What it will not do

- It does not change the design. Its only write is appending annotations.
- It does not delete or edit existing annotations.
- It does not invent token or style names. A value with no style or variable behind it is reported as a literal.
- It does not write code. Its output is notes for whoever writes the code.

## Worked example

Prompt:

> Write developer handoff notes for the "Login" frame on Page 1. Annotate the auto-layout frames with gap and padding, and the text layers with size and weight. Then give me a short summary.

Expected calls, in order:

1. `page_info()`, `page_info({ pageIds: ["<Page 1 id>"] })` → finds "Login".
2. `node_info` on "Login" with the layout and text properties from step 2.
3. `node_info` with `fillStyleId`, `textStyleId`, `boundVariables` → shows whether colors and text use styles or tokens.
4. `node_export_visual({ nodeId: "<Login id>", format: "PNG", scale: 0.5 })`.
5. `annotation_list({ nodeId: "<Login id>" })` → no existing notes.
6. `annotation_set` with one row per layer, for example:
   - "Login": "Vertical auto layout, gap 24, padding 24, width 375, height hugs content"
   - "Form": "Vertical, gap 12; fields fill the width"
   - "Title": "24 / Bold, Inter; literal color #12171F (no style)"
   - "Sign in button": "Height 48, radius 8, fill #2663EB (no token)"
7. Summary in the chat, ending with "Follow-ups: 3 literal colors have no style or token."

## Related

- The `figmamcp` skill explains the plugin's rules (name checks, batch results, error codes) in depth.
- `figma-rename-layers` gives layers clear names first, which makes the notes easier to read.
