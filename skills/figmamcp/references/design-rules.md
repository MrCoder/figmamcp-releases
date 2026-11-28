# Design rules (set before you draw)

A short starter rule set for clean screens. Read it **before** you create frames. Write the rules into the file as variables and text styles first, then bind every new node to them, so the screen follows the rules as it is drawn and not only after a QA pass.

The rules use the same six categories as the design-QA rubric (`skills/figma-design-qa/references/rubric.md`) and the "Visual self-check loop (design QA)" recipe in [workflows.md](workflows.md): **Alignment**, **Spacing**, **Type scale**, **Contrast**, **Naming**, **Auto-layout**. The rules are written to avoid every penalty those categories define, so a screen built to them should need few fixes in the QA loop. Radius is an extra rule; the rubric does not score it.

**When to use this file.** Use it when the user asks you to build new screens or frames, or asks you to set up rules for a file. When you only QA or polish an existing frame, follow the self-check loop instead: it does not create variables or styles unless the user asks.

## The starter rules

### Alignment

- The direct children of a stack share one left edge. Auto-layout does this for you; do not position children by hand.
- Labels inside buttons, badges and chips are centered: `primaryAxisAlignItems: "CENTER"` and `counterAxisAlignItems: "CENTER"` on their container.
- Members of a same-role group (all input fields, all list rows) use the same padding, so their content starts at the same offset.

### Spacing

One scale, based on 4. Every gap and padding is a value from it.

| Variable | Value |
|---|---|
| `space/4` | 4 |
| `space/8` | 8 |
| `space/12` | 12 |
| `space/16` | 16 |
| `space/24` | 24 |
| `space/32` | 32 |
| `space/48` | 48 |

- One stack uses at most two gap values: one item gap and one section gap (for example 12 inside a form, 24 between sections).
- Container padding is the same on all four sides, or differs by 8 or less (for example 16 vertical, 24 horizontal).
- Screen padding: 24 on a 375-wide phone frame, 32 on a 1440-wide desktop frame.

### Type scale

The rubric's declared scale: **12, 14, 16, 20, 24, 32**. One text style per step. Line heights are on the 4 grid.

| Text style | `fontSize` | `lineHeight` (PIXELS) | `fontName` |
|---|---|---|---|
| `Type/Caption` | 12 | 16 | Inter Regular |
| `Type/Body small` | 14 | 20 | Inter Regular |
| `Type/Body` | 16 | 24 | Inter Regular |
| `Type/Title small` | 20 | 28 | Inter Semi Bold |
| `Type/Title` | 24 | 32 | Inter Bold |
| `Type/Display` | 32 | 40 | Inter Bold |

- All text of one role (all body text, all placeholders) uses one style.
- The font must be available to Figma. If a style write fails on the font, report it and ask which font the file uses; do not substitute one silently.

### Contrast

A small colour set with roles. Every text colour is listed with the surfaces it may sit on. Ratios are computed with the rubric's WCAG 2.x formula.

| Variable | Hex | `value` (0–1) | Role |
|---|---|---|---|
| `color/text/primary` | `#0F172A` | `{ r: 0.059, g: 0.09, b: 0.165 }` | Titles and body text |
| `color/text/secondary` | `#475569` | `{ r: 0.278, g: 0.333, b: 0.412 }` | Hints, captions, placeholders |
| `color/text/on-accent` | `#FFFFFF` | `{ r: 1, g: 1, b: 1 }` | Text on an accent fill |
| `color/surface/default` | `#FFFFFF` | `{ r: 1, g: 1, b: 1 }` | Screen and card background |
| `color/surface/subtle` | `#F8FAFC` | `{ r: 0.973, g: 0.98, b: 0.988 }` | Input and section background |
| `color/border/default` | `#CBD5E1` | `{ r: 0.796, g: 0.835, b: 0.882 }` | Strokes only, never text |
| `color/accent/default` | `#2563EB` | `{ r: 0.145, g: 0.388, b: 0.922 }` | Primary button fill, links |
| `color/danger/default` | `#B91C1C` | `{ r: 0.725, g: 0.11, b: 0.11 }` | Error text |

| Text on surface | Ratio |
|---|---|
| text/primary on surface/default | 17.85 |
| text/primary on surface/subtle | 17.06 |
| text/secondary on surface/default | 7.58 |
| text/secondary on surface/subtle | 7.24 |
| text/on-accent on accent/default | 5.17 |
| accent/default (links) on surface/default | 5.17 |
| accent/default (links) on surface/subtle | 4.94 |
| danger/default on surface/default | 6.47 |

- Use only these pairs for text. Each is 4.5:1 or more, so it passes for normal and large text.
- If you add a colour, compute its ratio against every surface it will sit on before you use it for text.

### Naming

- Layers are named for their role, in the spec's words: `Email field`, `Sign in button`, `Header`. Never leave a default name (`Frame 12`, `Text`, `Rectangle 3`).
- Variables use the names in the tables above (`space/16`, `color/text/primary`); the slash groups them in Figma's variables panel. Text styles use the `Type/…` names.

### Auto-layout

- Every container with 2 or more children uses auto-layout (`layoutMode: "VERTICAL"` or `"HORIZONTAL"`).
- Single-child containers (button, input, chip) also use auto-layout, with padding from the spacing scale and the child centered or start-aligned. They are not padded by hand.
- Pass all four paddings on every auto-layout frame, 0 where none is wanted: a new auto-layout frame starts with 10 px padding, which is off the scale.
- Sections fill the width of their parent (`layoutSizingHorizontal: "FILL"`) and hug their content (`layoutSizingVertical: "HUG"`).

### Radius (not scored by the rubric)

| Variable | Value | Use |
|---|---|---|
| `radius/4` | 4 | Chips, small tags |
| `radius/8` | 8 | Inputs, buttons |
| `radius/16` | 16 | Cards, sheets |

- One radius per component type. Nested corners use a smaller radius than their container.

## Procedure: rules first, then frames

Follow the discover-before-acting rule and the hard constraints in [constraints.md](constraints.md) on every step: pass names back verbatim, give every creator an explicit parent, and never retry a refused write with the same input.

```
1. Discover.
   page_info()                                  → the target page and its name
   variable_list()                              → collections and variables the file already has
   style_list()                                 → text and paint styles the file already has
   If the file already has a spacing scale, colours or text styles, use them and create only
   what is missing. Say which rules came from the file and which from this rule set.

2. Create the variables (once per file).
   variable_manage({ action: "CREATE_COLLECTION", name: "Design rules" })
                                                → returns the collection id and name; variable_list
                                                  returns the same name
   variable_manage({ action: "CREATE_VARIABLE", collectionId, collectionName: <name verbatim>,
                     name: "space/16", type: "FLOAT", value: 16, scopes: ["GAP"] })
   variable_manage({ action: "CREATE_VARIABLE", collectionId, collectionName,
                     name: "radius/8", type: "FLOAT", value: 8, scopes: ["CORNER_RADIUS"] })
   variable_manage({ action: "CREATE_VARIABLE", collectionId, collectionName,
                     name: "color/text/primary", type: "COLOR",
                     value: { r: 0.059, g: 0.09, b: 0.165 }, scopes: ["TEXT_FILL"] })
   … one call per row of the tables above. `scopes` is required on every create:
     text/* and danger/default   → ["TEXT_FILL"]
     surface/*                   → ["FRAME_FILL", "SHAPE_FILL"]
     accent/default              → ["ALL_FILLS"]   (button fills and link text)
     border/default              → ["STROKE_COLOR"]
     space/*                     → ["GAP"]
     radius/*                    → ["CORNER_RADIUS"]
   The value set on create goes to the collection's default mode.

3. Create the text styles.
   style_manage({ type: "TEXT", name: "Type/Body",
                  properties: { fontName: { family: "Inter", style: "Regular" },
                                fontSize: 16, lineHeight: { value: 24, unit: "PIXELS" } } })
   … one call per row of the type table.

4. Read them back.
   variable_list() and style_list()             → the IDs and exact names to use from now on

5. Create the frames, values from the scale.
   create_frame({ parentId, parentNodeName, name: "Login", x, y, width: 375, height: 640,
                  layoutMode: "VERTICAL", itemSpacing: 24,
                  paddingTop: 24, paddingRight: 24, paddingBottom: 24, paddingLeft: 24,
                  layoutSizingVertical: "HUG" })
   create_text({ parentId, parentNodeName, name: "Title", text: "Welcome back", x: 0, y: 0,
                 fontSize: 24 })

6. Bind every node to the rules, names verbatim from the create result or node_info.
   node_bind_variable({ nodeId, nodeName, bindVariables: {
       itemSpacing: <space/24 id>, paddingTop: <space/24 id>, paddingRight: <space/24 id>,
       paddingBottom: <space/24 id>, paddingLeft: <space/24 id>,
       fills: <color/surface/default id>,
       topLeftRadius: <radius/16 id>, topRightRadius: <radius/16 id>,
       bottomLeftRadius: <radius/16 id>, bottomRightRadius: <radius/16 id> } })
   node_apply_style({ nodeId: textId, nodeName, styleId: <Type/Title id>, styleType: "TEXT" })
   node_bind_variable({ nodeId: textId, nodeName, bindVariables: { fills: <color/text/primary id> } })

7. Check.
   node_info({ nodeIds: [frameId], maxDepth: 3, properties: ["name", "boundVariables", "textStyleId"] })
                                                → every gap, padding, fill and radius is bound
   Optional: run the "Visual self-check loop (design QA)" in workflows.md on the frame.
```

Order rules that the tools enforce or that Figma needs:

- Set auto-layout before you bind padding or spacing (step 5 sets `layoutMode` on create; use `node_set_auto_layout` for an existing frame).
- A node needs zero fills or a solid fill before you bind a colour variable to `fills`. `create_frame` without `fillColor` and `create_text` (which sets a solid text colour) both qualify.
- There is no single radius field to bind: bind all four corner fields.
- A paint style is created with `style_manage` `type: "PAINT"` but applied with `node_apply_style` `styleType: "FILL"`.
- The plugin does not check `scopes` when it binds; scopes decide where Figma's own panels offer the variable. Whether Figma accepts a bind to a field outside the variable's scopes (for example a `["GAP"]` variable on padding) was not checked on a live file. If a bind is refused, report the error; do not change scopes without telling the user.

Stop and tell the user when a write is refused with `VARIABLE_EDITS_DISABLED` or `STYLE_EDITS_DISABLED`: the plugin connection does not allow variable or style edits, and only the user can reconnect with them allowed. Do not fall back to literal values without saying so.

## Bring your own rules

A team can keep its own rules instead of, or on top of, this starter set.

1. **Where the files live.** The user keeps rule files in a `design-rules/` folder at the root of their project, one Markdown file per rule set (for example `design-rules/brand.md`, `design-rules/mobile.md`). Each file uses the same six category headings as this file, plus Radius if it sets one. Do not keep them inside the installed `figmamcp` skill folder: setup replaces that folder when it updates the skill.
2. **Pick or merge.** If the user names a file, use it. If the folder has one file, use it. If it has several and the user did not say, list them and ask. To merge several, apply them in the order the user gives; for each category, a later file replaces an earlier file's value. If two files the user did not order set different values for the same thing, ask; do not choose silently. This starter set fills any category none of the files sets, and you say so.
3. **One sample frame first.** Write the merged rules as variables and styles (steps 1–4 above), then build **one** frame (steps 5–7), show it with `view_navigate` and a `node_export_visual` PNG, and list the rules you applied.
4. **Apply to the rest only after the user accepts.** Wait for the user to accept the preview. If they ask for changes, update the variables and styles (`variable_manage` `UPDATE_VARIABLE` with `currentVariableName`, `style_manage` with `styleId` and `currentStyleName`; bound nodes follow), rebuild the sample, and ask again. Then build or rebind the other frames, one at a time.
