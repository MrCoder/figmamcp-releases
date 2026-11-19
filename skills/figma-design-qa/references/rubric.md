# Design QA rubric

Six categories. Each starts at 10 and loses points per violation; the floor is 0. The overall score is the mean of the six, to one decimal.

Score from **numbers you read with `node_info`**, not from the picture. Use the `node_export_visual` image to confirm what the numbers say and to catch problems the rubric does not cover (report those separately; do not add points for them).

Read this once per round, for the target frame:

```
node_info({ nodeIds: [targetId], maxDepth: 3,
  properties: ["name", "type", "x", "y", "width", "height", "layoutMode",
               "itemSpacing", "paddingTop", "paddingRight", "paddingBottom", "paddingLeft",
               "fontSize", "fontName", "fills"] })
```

`x`/`y` of a child are relative to its parent. A "stack" is a container whose children are laid out in one direction (most cards, forms, lists).

## 1. Alignment

| Violation | Penalty |
|---|---|
| Each distinct left edge (`x`) among the direct children of a stack, beyond the first | −2 |
| Each member of a same-role group (for example input fields) whose inner content offset differs from the first member's | −1 |
| Each label meant to be centered (button label, badge) whose center is more than 1 px from its container's center | −1 |

## 2. Spacing

| Violation | Penalty |
|---|---|
| Each gap between consecutive children of a stack that is not a multiple of 4 | −1 |
| Each distinct gap value in one stack beyond 2 (one item gap and one section gap are allowed) | −2 |
| Container padding spread: largest minus smallest of top/right/bottom/left padding is more than 8 px (count empty space below the last child as bottom padding) | −2 |

Gap = next child's `y` − (this child's `y` + `height`), children sorted by `y`.

## 3. Type scale

Declared scale: **12, 14, 16, 20, 24, 32**. If the file has text styles or a documented scale, use that instead and say so in the report.

| Violation | Penalty |
|---|---|
| Each text node whose `fontSize` is not on the scale | −1 |
| Each same-role group (all placeholders, all body text) that mixes sizes | −1 |

## 4. Contrast

WCAG 2.x contrast ratio between the text fill and the fill directly behind it (walk up the parents to the first solid fill).

| Violation | Penalty |
|---|---|
| Each text node below 4.5:1 (normal text) or below 3:1 (large text: 24 px or more, or 18.66 px or more at weight 700) | −2 |

Formula, with channels in 0–1 as `fills` returns them:

```
lin(c) = c <= 0.03928 ? c / 12.92 : ((c + 0.055) / 1.055) ^ 2.4
L      = 0.2126 lin(r) + 0.7152 lin(g) + 0.0722 lin(b)
ratio  = (L_light + 0.05) / (L_dark + 0.05)
```

Reference points on white: `#BBBBBB` 1.9, `#AAAAAA` 2.3, `#64748B` 4.8, `#475569` 7.6.

## 5. Naming

| Violation | Penalty |
|---|---|
| Each layer with a default name (`Frame`, `Frame 12`, `Text`, `Rectangle 3`, `Group 5`, `Ellipse 2`, `Vector`) or a name that only repeats its own text content | −1 |

## 6. Auto-layout

| Violation | Penalty |
|---|---|
| Each container with 2 or more children stacked in one direction and `layoutMode: "NONE"` | −3 |
| Each single-child container (input, button, chip) that pads or centers its child by hand (`layoutMode: "NONE"`) | −1 |

## Pass

Stop when **every category is 9 or more**. Otherwise stop after the round limit (default 5) and report what is left.

## Worked example (live run, 2026-11-20)

A sign-up card built messy on purpose: title, subtitle, two input frames, a button frame, a log-in line.

| Round | Fix applied | Align | Spacing | Type | Contrast | Names | Auto-layout | Overall |
|---|---|---|---|---|---|---|---|---|
| 0 | (none) | 0 | 0 | 4 | 2 | 0 | 4 | 1.7 |
| 1 | Card: vertical auto-layout, padding 32, gap 16, hug height; fields and button fill width | 8 | 10 | 4 | 2 | 0 | 7 | 5.2 |
| 2 | Sizes 27/15/13/14/17/11 → 24/16/14/14/16/12; three greys to `#64748B`/`#6B7380` | 9 | 10 | 10 | 10 | 0 | 7 | 7.7 |
| 3 | Fields and button: horizontal auto-layout, padding 16, centered; button height 46 → 48; 10 layers renamed | 10 | 10 | 10 | 10 | 10 | 10 | 10.0 |

Round 0 detail, to show how a number is reached: left edges 19, 24, 27, 31, 40 (5 edges, −8); placeholder offsets 12 vs 17 (−1); button label center 6 px off (−1) → Alignment 0. Gaps 9, 25, 14, 28, 33: three off the 4 px grid (−3), five distinct values (−6), padding 19 to 217 px (−2) → Spacing 0.
