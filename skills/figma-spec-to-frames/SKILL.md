---
name: figma-spec-to-frames
description: >-
  Use when the user gives a written spec for a screen (a list of sections,
  fields, buttons and copy) and wants it built as real Figma frames in the open
  file, with auto layout, through the FigmaMCP server. Builds the screen frame,
  nested auto-layout sections and text layers with clear names, then exports a
  PNG to check the result.
---

# Figma: build a screen from a written spec

Turns a plain-text screen spec into a structured Figma screen in the file the user has open: one screen frame with vertical auto layout, one auto-layout frame per section, and named text layers. Requires **FigmaMCP** (a Mac app with a local MCP server and a Figma plugin, <https://figmamcp.dev>); the tools below are FigmaMCP tools.

## When to use

- "Build the login screen from this spec in Figma."
- "Turn this PRD section into frames I can refine."
- A wireframe-level first draft that a designer will then style.

Not for pixel-perfect visual design from a screenshot, and not for copying an existing design system component by component (see "What it will not do").

## Preconditions

1. The Figma desktop app has the file open and the **FigmaMCP plugin is running** in it.
2. The plugin's **AI can edit this file** switch is on. If it is off, every write fails with `READ_ONLY_MODE`; tell the user and stop.
3. The user has said where the screen goes: a page, or an existing frame or section. If they did not, ask.
4. The spec states the screen width (for example 375 for a phone, 1440 for desktop). If it does not, ask or use 375 and say so.

## Steps

1. **Find the destination.** `page_info()` lists pages. `page_info({ pageIds: [pageId] })` lists the page's top-level frames. If the user named a frame or section, read it with `node_info({ nodeIds: [id], maxDepth: 0, properties: ["name", "type", "width", "height"] })`. Keep its exact `name` for the next step. Do not build inside a component instance; the plugin refuses structural edits there.
2. **Turn the spec into a tree before writing.** One screen frame, one child frame per section (header, form, actions, footer), and the text and field frames inside each. Pick layer names from the spec's own words ("Email field", "Sign in button"), not "Frame 12". Show the tree to the user if the spec is long or vague.
3. **Create the screen frame.** `create_frame({ parentId, parentNodeName, x, y, width, height, name, layoutMode: "VERTICAL", itemSpacing, paddingTop, paddingRight, paddingBottom, paddingLeft, layoutSizingVertical: "HUG", fillColor })`. Pick `x` to the right of existing frames so nothing overlaps (read their `x` and `width` in step 1). Keep the returned `id` and `name`.
4. **Create each section** as a child of the screen: `create_frame({ parentId: <screen id>, parentNodeName: <screen name>, x: 0, y: 0, width, height, name, layoutMode, itemSpacing, layoutSizingHorizontal: "FILL", layoutSizingVertical: "HUG" })`. `FILL` works only for children of an auto-layout frame; `HUG` works only for frames and text. Auto layout ignores `x` and `y`, but they are required, so pass 0. Always pass all four paddings (0 when the spec gives none): a new auto-layout frame created without them gets a 10 px padding.
5. **Add the text.** `create_text({ parentId: <section id>, parentNodeName: <section name>, x: 0, y: 0, text, name, fontSize, fontWeight, fontColor })`. For a field or button, create a small auto-layout frame (`layoutMode: "HORIZONTAL"`, padding, `strokeColor` or `fillColor`) and put its label text inside it. Use `node_set_corner_radius` for rounded fields and buttons, and `text_set_style` for alignment or line height.
6. **Use the file's styles when they exist.** `style_list()` and `variable_list()` show local styles and tokens. Link a paint style with `node_apply_style` or a color token with `node_bind_variable` instead of a literal color when a matching one exists.
7. **Check the result.** `node_export_visual({ nodeId: <screen id>, format: "PNG", scale: 0.5 })` and look at it. Fix spacing or order with `node_set_auto_layout` (on a frame) or `node_transform` (size). Then `view_navigate({ ids: [<screen id>] })` so the user sees it.
8. **Report** the screen's layer tree and anything in the spec you could not build.

Every child call passes the parent's `id` and exact `name` from the previous call's result. If a call fails with `details.partialMutation: true`, read the survivor evidence it returns before you retry, so you do not create a duplicate.

## What it will not do

- It does not delete or move existing layers. It only adds the new screen.
- It does not invent copy, images or icons the spec does not state. A missing image gets a gray placeholder frame named "Image placeholder".
- It does not create components or variants. Ask for that as a separate step (`create_component`, `create_component_set`).
- It does not edit inside component instances or locked layers. The plugin refuses those writes.

## Worked example

Prompt:

> In Figma, on Page 1, build a 375-wide "Login" screen from this spec: title "Welcome back"; subtitle "Sign in to continue"; an Email field and a Password field, each a 48px-high outlined box with the label as placeholder text; a full-width blue "Sign in" button; a "Forgot password?" link under it. Use auto layout everywhere.

Expected calls, in order:

1. `page_info()`, then `page_info({ pageIds: ["<Page 1 id>"] })` to see existing frames and choose a free `x`.
2. `create_frame` "Login" on Page 1: `layoutMode: "VERTICAL"`, `itemSpacing: 24`, padding 24, `layoutSizingVertical: "HUG"`, white fill.
3. `create_frame` "Header" (VERTICAL, spacing 8, padding 0, FILL × HUG) in "Login"; `create_text` "Title" ("Welcome back", 24, 700) and "Subtitle" ("Sign in to continue", 14, 400) in "Header".
4. `create_frame` "Form" (VERTICAL, spacing 12, padding 0, FILL × HUG) in "Login"; in it two `create_frame` fields "Email field" and "Password field" (HORIZONTAL, padding 12, height 48, gray stroke, FILL × FIXED), each with a `create_text` placeholder; `node_set_corner_radius` 8 on each field.
5. `create_frame` "Actions" (VERTICAL, spacing 12, padding 0, centered, FILL × HUG); in it `create_frame` "Sign in button" (HORIZONTAL, centered, blue fill, FILL × FIXED 48) with `create_text` "Sign in" (white, 600), and `create_text` "Forgot password?" (blue, 14).
6. `node_export_visual({ nodeId: "<Login id>", format: "PNG", scale: 0.5 })`, fix anything off, then `view_navigate`.

The agent reports the tree:

```
Login (VERTICAL, 375 wide)
├─ Header: Title, Subtitle
├─ Form: Email field, Password field
└─ Actions: Sign in button, Forgot password?
```

## Related

- The `figmamcp` skill explains the plugin's rules (name checks, scope, error codes) in depth.
- `figma-rename-layers` cleans up the names of an existing screen.
- `figma-handoff-notes` adds developer notes to the finished screen.
