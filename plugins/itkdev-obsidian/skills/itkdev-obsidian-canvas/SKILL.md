---
name: itkdev-obsidian-canvas
description: Create and edit JSON Canvas (.canvas) files — the open format Obsidian Canvas uses for infinite-canvas diagrams with nodes (text, files, links, groups) and edges (connections). Use when the user asks to build or edit an Obsidian canvas, a visual map/diagram, an architecture or flow diagram as a `.canvas` file, or mentions JSON Canvas.
---

# JSON Canvas

JSON Canvas (jsoncanvas.org) is the open file format behind Obsidian Canvas. A
`.canvas` file is a single JSON object with two top-level arrays: `nodes` and
`edges`. Adapted from kepano/obsidian-skills.

To place a canvas in the vault, `Write` it as a `.canvas` file under the vault
root (see `itkdev-obsidian-vault`); follow `itkdev-obsidian-markdown` for any
text-node content.

> [!important] Linking TO a canvas from a note
> A bare `[[Name]]` wikilink resolves to `Name.md`, never `Name.canvas`. When a
> Markdown note links to a canvas you MUST include the extension:
> `[[Architecture.canvas]]` or `[[Architecture.canvas|Architecture]]`. Omitting
> it makes Obsidian create an empty `.md` stub on click (a duplicate). Prefer
> keeping one canvas per topic and always reference it with the `.canvas`
> extension.

## Top-level structure

```json
{
  "nodes": [],
  "edges": []
}
```

Coordinates: `x`/`y` are in pixels, origin top-left, y increases downward.
`width`/`height` are pixels. All ids are unique strings within the file.

## Nodes

Every node has: `id` (string), `type`, `x`, `y`, `width`, `height`, and an
optional `color`.

**Color** is either a preset digit string `"1"`–`"6"` (1 red, 2 orange, 3
yellow, 4 green, 5 cyan, 6 purple) or a hex string like `"#8a5cf5"`.

### `text` node — Markdown content
```json
{ "id": "n1", "type": "text", "x": 0, "y": 0, "width": 260, "height": 120,
  "text": "## Web app\nEntry point for users" }
```

### `file` node — embeds a vault file (note, image, PDF)
```json
{ "id": "n2", "type": "file", "x": 320, "y": 0, "width": 300, "height": 200,
  "file": "Services/Web App.md", "subpath": "#Architecture" }
```
`subpath` (optional) links to a `#heading` or `#^block`.

### `link` node — external URL
```json
{ "id": "n3", "type": "link", "x": 0, "y": 200, "width": 300, "height": 160,
  "url": "https://example.com" }
```

### `group` node — a labeled container
```json
{ "id": "g1", "type": "group", "x": -40, "y": -40, "width": 720, "height": 320,
  "label": "Docker stack", "background": "assets/bg.png", "backgroundStyle": "cover" }
```
`background`/`backgroundStyle` (`cover` | `ratio` | `repeat`) are optional. Nodes
are grouped by geometric containment, not by a reference.

## Edges

Edges connect nodes. Required: `id`, `fromNode`, `toNode`. Optional:
`fromSide`/`toSide` (`top`|`right`|`bottom`|`left`), `fromEnd`/`toEnd`
(`none`|`arrow`, default `none`/`arrow`), `color`, `label`.

```json
{ "id": "e1", "fromNode": "n1", "fromSide": "right",
  "toNode": "n2", "toSide": "left", "toEnd": "arrow", "label": "serves" }
```

## Full example

```json
{
  "nodes": [
    { "id": "a", "type": "text", "x": 0,   "y": 0, "width": 240, "height": 100, "color": "5", "text": "## Client" },
    { "id": "b", "type": "text", "x": 340, "y": 0, "width": 240, "height": 100, "color": "6", "text": "## Web app" },
    { "id": "c", "type": "text", "x": 680, "y": 0, "width": 240, "height": 100, "text": "## Database" }
  ],
  "edges": [
    { "id": "e1", "fromNode": "a", "fromSide": "right", "toNode": "b", "toSide": "left", "toEnd": "arrow" },
    { "id": "e2", "fromNode": "b", "fromSide": "right", "toNode": "c", "toSide": "left", "toEnd": "arrow", "label": "query" }
  ]
}
```

## Authoring rules

- Output must be valid JSON (no comments, no trailing commas).
- Every `id` is unique; every edge's `fromNode`/`toNode` references an existing
  node id.
- Lay out nodes so they don't overlap: leave gaps between `x`/`width` spans;
  increment `y` for new rows.
- Use `color` sparingly and consistently (e.g. one color per layer/category).
- When editing an existing canvas, `Read` it first and preserve untouched node
  ids/positions.
