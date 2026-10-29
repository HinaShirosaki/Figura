# Figura

A [Hikari](https://github.com/HinaShirosaki/Hikari) plugin for editable scientific
figures and data plots. Describe a figure, Codex draws it on a canvas, and every
piece — SVG artwork, raster images, Plotly plots and text labels — stays a
separate layer you and Codex can keep editing.

Figura runs inside Hikari; it is not a standalone app. This repository is the
whole plugin: no npm install, no build step, no imports from Hikari's source.
Controls, agent tools and storage are covered in full in
[USER_GUIDE.md](USER_GUIDE.md).

## Install

1. In Hikari, open **Settings → Plugins**, paste
   `https://github.com/HinaShirosaki/Figura` and choose **Install from GitHub**.
2. Review its host access, turn it on, click **Reload App**, and configure
   Hikari storage.
3. Open **More → Figura** and sign in to Codex.

To install from a local copy instead, clone it into a folder named
`scientific-illustration` (the installer checks that the folder name matches the
manifest id) and use **Add Plugin Folder**:

```bash
git clone https://github.com/HinaShirosaki/Figura.git scientific-illustration
```

Saved figures live in Hikari storage, not in the plugin folder, so removing or
re-adding the plugin keeps them. An install made before the `layout` permission
existed must be re-added to grant it.

## Start a figure

- **From a description:** type it into the composer under an empty canvas and
  click **Draw with Codex** (⌘Enter), or send it from the chat rail. Later
  changes go through the chat rail.
- **From a protocol:** **Protocol → … → Generate illustration**.
- **From a paper:** select PDF text and choose **Generate illustration** in the
  hover toolbar.

The protocol and paper actions open a new figure and save their source (the
protocol, or the passage, page and paper Markdown) with it, so Codex can use it in
later edits. Hikari passes only the item you clicked; Figura needs no general
protocol or paper read permission.

Each figure has its own chat and Codex session. Switching figures restores that
conversation and its unsent draft.

## Editing

- **Layers:** vector, raster, plot and text layers with previews, visibility and
  property controls. Double-click a label to edit its text on the canvas.
- **Select & move (V)** and **Freehand select (L):** drag empty space to select,
  drag a selection to move it, use the eight handles to resize. Shift adds to the
  selection; Alt picks one member of a group. Clicks pass through transparent
  pixels to the layer underneath.
- **Rotate** with the round handle; hold Shift to snap to 15°.
- **Group / Ungroup** (⌘G / ⌘⇧G) and **Align and distribute** from the right
  toolbar.
- **Copy and paste** (⌘C / ⌘V) between figures; copies keep formatting, rotation
  and grouping.
- **Right-click** (or Shift+F10) for Copy, Paste, Group, Ungroup and
  **Save as an asset…**.
- **Assets** keeps reusable components shared by all figures. Save a selection,
  or use **Import asset…** for SVG, PNG, JPEG or WebP files.
- **Crop selected image** crops a raster layer in place; the saved asset is left
  untouched.
- **Canvas size** presets or **Custom…** dimensions; zoom and **Fit** change only
  the view.
- **Undo / Redo** use Hikari's system buttons, ⌘Z, and ⌘⇧Z or Ctrl+Y, for the
  last 25 edits of the open figure.
- **Export** as SVG, PNG or PowerPoint. The `.pptx` has one slide with each layer
  as its own object and labels as editable text boxes.

On Windows use Ctrl in place of ⌘.

## How Codex draws

- **Detail** (Simple, Standard, Detailed) sets how much scientific detail the
  brief asks for. It is saved per figure and does not redraw existing artwork.
- **Renderer per component:** SVG for schematics, arrows, panels and exact
  geometry; a Plotly layer for data; Codex's built-in `image_gen` only for
  realistic or textured detail that SVG cannot show clearly. Labels are always
  separate text layers, and artwork never contains text.
- **Image generation settings** (beside Detail): off means SVG first; on sets a
  target share of 0–100% for eligible artwork. Boxes, arrows, charts, exact
  geometry and labels stay SVG or text even at 100%. **Image style** and
  **Style notes** steer new generated images only.
- **Image generation** uses the signed-in Codex; there is no separate provider or
  API key. Hikari imports results from Codex's own `generated_images` folder and
  stores the bytes in the figure. If generation is unavailable, Codex says so and
  falls back to SVG.
- **Plots** are rendered by Hikari's bundled Plotly.js through `plotly.render`.
  Each plot keeps its data, layout and processing notes so Codex can revise it.
- **Assets first:** before each drawing or edit, Codex checks the asset library
  and reuses suitable components. Generated images and finished multi-layer groups
  are saved there automatically.
- **Visual review:** a run completes only after Codex renders the latest canvas
  and records what it checked with `inspect`. Hikari prompts once if the step is
  skipped; artwork already saved is kept either way.

Codex reaches the canvas through Hikari's `plugin_canvas` MCP tool. The drawing
instructions live in [agent/workflow.mjs](agent/workflow.mjs) and the request
schema in [agent/canvas-contract.mjs](agent/canvas-contract.mjs); both are
returned on every canvas read. See **Agent tools** in the user guide for the
action reference.

## Storage and limits

Figures are saved under `Plugins/Figura/` in the Hikari storage root:
`library.json` indexes them, each figure alternates between two scene files, and
reusable assets sit beside them. Older single-figure `workspace.json` files and
scratch canvases migrate automatically.

| Limit | Value |
| --- | --- |
| Figures in the library | 100 |
| Layers per figure | 200 |
| Saved figure size | 12 million characters |
| Raster image | PNG, JPEG or WebP, 5 MiB, 64 megapixels |

## Permissions

| Permission | Used for |
| --- | --- |
| `storage`, `files` | The figure and asset library |
| `downloads` | Export through a save dialog |
| `agent:chat` | Sending your prompt to the figure's chat |
| `agent:canvas` | Letting Codex read, edit and render the canvas |
| `layout` | Remembering the left rail's width and folded state |

## Development

[index.html](index.html) loads [main.mjs](main.mjs). The document model is in
[model.mjs](model.mjs), saving in [workspace.mjs](workspace.mjs) and
[library.mjs](library.mjs), and PowerPoint export in
[powerpoint.mjs](powerpoint.mjs). `vendor/` bundles Cropper.js 1.6.2,
PptxGenJS 4.0.1 and the Inter font, so everything works offline.
[icon.svg](icon.svg) is the dock icon; Hikari tints it to the current theme.

The unit tests need Node.js 22 or newer and no dependencies. GitHub Actions runs
them on every push and pull request.

```bash
npm test
```

## License

Apache-2.0; see [LICENSE](LICENSE). Bundled libraries and fonts are listed in
[THIRD-PARTY-NOTICES.md](THIRD-PARTY-NOTICES.md).
