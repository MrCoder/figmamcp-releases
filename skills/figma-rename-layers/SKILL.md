---
name: figma-rename-layers
description: >-
  Use when the user wants the layers of a Figma frame renamed clearly and
  consistently (for example "Frame 12", "Rectangle 4", "Group 7" turned into
  "Card / Header", "Avatar", "Price"), through the FigmaMCP server. Reads the
  layer tree, proposes a naming map, renames one layer at a time with
  verbatim-name verification, and re-reads to confirm.
---

# Figma: rename a frame's layers

Gives every layer in a frame a clear, consistent name based on what it is and what it contains, so that designers, developers and code-generation agents can find things. Requires **FigmaMCP** (a Mac app with a local MCP server and a Figma plugin, <https://figmamcp.dev>); the tools below are FigmaMCP tools.

## When to use

- "Clean up the layer names in the Checkout frame."
- Before a handoff or before generating code from a frame, when the layer panel is full of "Frame 12" and "Rectangle 4".
- To apply a naming convention the user states (for example `Section / Element`, kebab-case, or names that match component props).

## Preconditions

1. The Figma desktop app has the file open and the **FigmaMCP plugin is running** in it.
2. The plugin's **AI can edit this file** switch is on. If it is off, every rename fails with `READ_ONLY_MODE`; tell the user and stop.
3. The user has named the frame, or pasted a Figma link to it. A link's node ID (`12-34`) can be passed as is.

## Steps

1. **Find the frame.** With a link, pass its node ID to `node_info({ nodeIds: [id], maxDepth: 0, properties: ["name", "type"] })`. With only a name, `page_info()` then `page_info({ pageIds: [pageId] })` lists each page's top-level frames; pick the match and confirm with the user if two are close.
2. **Read the layer tree.** `node_info({ nodeIds: [frameId], properties: ["name", "type", "characters", "locked", "visible"], maxDepth: 6 })`. `characters` gives the text content, which is the best clue for a text layer's name. Check `descendantCount` first on a very large frame and work section by section.
3. **Propose a naming map.** For each layer, the new name follows the user's convention, or by default:
   - Text: what it is, not what it says ("Title", "Price", "Error message"), unless the text is a fixed label like a button's "Sign in".
   - Frames and groups: their role ("Header", "Product card", "Actions").
   - Shapes: their purpose ("Divider", "Avatar background").
   - Keep names that are already clear. Do not rename component instances' inner layers, and skip locked layers.
   Show the map to the user as a table (current name → new name) and wait for a yes before writing.
4. **Rename one layer at a time.** `node_rename({ nodeId, nodeName: <current name, verbatim from step 2>, name: <new name> })`. There is no batch rename. Pass the current name exactly as read: same spaces, same case, same characters. A mismatch is refused, which is the safety check doing its job.
5. **Handle refusals.** A name mismatch means someone changed the file since your read: re-read that layer and re-check the map. A locked layer, or a layer the plugin refuses for any other reason, comes back as `Operation Denied: …`; report that message to the user as is and continue with the rest.
6. **Verify.** Run the same `node_info` read as step 2 and compare against the map. Report what was renamed, what was skipped and why.

If the run stops partway (connection lost, plugin closed), the frame has a mix of old and new names. The re-read in step 6 shows exactly where it stopped; continue from there.

## What it will not do

- It does not change anything but layer names: no moving, resizing, restyling or deleting.
- It does not rename pages, styles, variables or components' properties.
- It does not guess when a layer's role is unclear; it asks, or keeps the old name and lists it.
- It does not edit locked layers.

## Worked example

Prompt:

> Rename the layers in the "Pricing card" frame on Page 1 so a developer can read them. Use short role names in Title Case. Show me the list before you change anything.

Expected calls, in order:

1. `page_info()`, then `page_info({ pageIds: ["<Page 1 id>"] })` → finds "Pricing card".
2. `node_info({ nodeIds: ["<Pricing card id>"], properties: ["name", "type", "characters", "locked"], maxDepth: 6 })`.
3. The agent shows a table, for example:

   | Current | New |
   |---|---|
   | Frame 3 | Header |
   | Text | Plan Name |
   | Text | Price |
   | Rectangle 2 | Divider |
   | Frame 4 | Feature List |

4. After the user says yes: one `node_rename` per row, each with `nodeName` copied verbatim from step 2 (two layers both named `Text` are told apart by `nodeId`).
5. `node_info` again on the frame, then a summary: 5 renamed, 0 skipped.

## Related

- The `figmamcp` skill explains the plugin's rules (name checks, scope, error codes) in depth.
- `figma-handoff-notes` adds developer notes once names are clean.
