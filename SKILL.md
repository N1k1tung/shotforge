---
name: shotforge
description: Use when operating Shotforge through MCP to create, revise, or export App Store screenshots. Trigger for requests to generate screenshot decks, import screenshot assets, inspect Shotforge documents, or render/export finished marketing screenshots from the macOS app.
---

# Shotforge MCP

Use this skill when the task should be completed inside Shotforge through MCP rather than by building HTML, editing raw JSON manually, or describing designs abstractly.

## What Shotforge Does Best

Shotforge is a macOS screenshot composer for App Store marketing images.

Prefer Shotforge MCP when the user wants to:
- build a vivid screenshot deck from a brief
- import app screenshots, icons, or supporting image assets
- revise an existing `.shotforge` document
- add floating callouts that crop and magnify details from an existing device screenshot
- render/export final PNG screenshots

Do not default to hand-authoring a full `ShotProject` JSON payload unless the high-level design tool is insufficient.

## Transport Options

Shotforge currently supports both MCP transports:

### Live GUI HTTP MCP

Use this when the Shotforge app is open and the user wants to work against the currently active editor document.

- Default endpoint: `http://127.0.0.1:32100/mcp`
- Operates on the active Shotforge window/document
- Best for iterative collaboration while the user has the app open

### Headless stdio MCP

Use this when the user wants a subprocess-style workflow or file-based automation.

Launch pattern:

```bash
/path/to/Shotforge.app/Contents/MacOS/Shotforge --mcp-stdio --document /absolute/path/to/project.shotforge
```

Notes:
- `--document` is optional; without it, Shotforge starts with a blank in-memory document.
- In stdio mode, save changes with `shotforge_save_document`.

## File permissions

Since Shotforge operates from sandbox, it can only access Downloads folder with convenient read/write access. It is highly advised to move working folder there, or copy assets there.

## Recommended Workflow

Follow this sequence unless the user is explicitly asking for a lower-level operation:

1. Call `shotforge_get_design_questionnaire`.
2. Ask the user a compact set of design questions based on that questionnaire.
3. Call `shotforge_get_active_document` to inspect the current project and available assets.
4. Import any missing screenshots or app icons with `shotforge_store_asset`.
5. Decide which of the four supported flows applies:
   - whole-project reset across one or more device families: use `shotforge_generate_design_deck`
   - add or replace one device family from scratch: use `shotforge_fill_deck_from_questionnaire`
   - add a new device family by deriving from another family already in the document: use `shotforge_fill_deck_from_existing_deck`
   - add one automated floating callout to an existing device already on canvas: use `shotforge_get_asset_preview` followed by `shotforge_add_callout`
6. Inspect `slideDiagnostics` and `notes` from the generation, fill, localization, or callout result.
7. If any generated slide still looks dense or risky, shorten the copy or choose a different layout and regenerate before export.
8. Call `shotforge_localize_selected_localization` when only one localization needs to be added or refreshed across the existing device rows.
9. Call `shotforge_render_pages` to export PNGs.
10. In stdio mode, call `shotforge_save_document` if the document should persist on disk.

## Floating Callouts

Shotforge supports first-class floating callouts from the canvas.

- In the live GUI, the user can right-click a non-window device that currently resolves to a screenshot and choose `Add Callout`.
- The app opens a visual crop editor over the current effective screenshot, so manual callouts should be created in the GUI rather than by asking the user to copy MCP commands.
- Existing callouts are stored on the source `DevicePlacement` and render from each localization's effective screenshot using the same crop rect.
- A callout inherits the source device's 3D transform, floats above the screen, and uses `contentZoom` instead of independent position or rotation.

Use automated callouts when an agent has enough visual context to choose a meaningful crop rect from an existing screenshot or while generating a deck.

Follow this sequence:
1. Call `shotforge_get_active_document` and identify the source non-window device placement plus its screenshot asset.
2. Call `shotforge_get_asset_preview` for that screenshot asset.
3. Choose a `normalizedCropRect` in `0...1` coordinates using a top-left origin and the full exact screenshot image as the reference space.
4. Call `shotforge_add_callout` with `sourceDeviceID`, the chosen crop rect, optional `contentZoom`, optional `cornerRadius`, optional `borderWidth`, optional `borderColorHex`, and optional `description`.

Important constraints:
- The source device must not already be a `*Window` preset.
- `normalizedCropRect` must stay inside `0...1` bounds.
- `contentZoom` defaults to `1.2` and must stay within `1.0...3.0`.
- `cornerRadius` defaults to `0.08` and is a fraction of the callout's shorter side within `0.0...0.5`.
- `borderWidth` defaults to `2.0` pixels and `borderColorHex` defaults to white.
- The created callout does not create an asset or a separate placement; it stays attached to the source device.
- For generated decks, pass `slide.callouts` directly when calling `shotforge_generate_design_deck` or `shotforge_fill_deck_from_questionnaire`.

## Discovery Questions

Ask for the smallest useful brief, not an open-ended brainstorm. Cover these areas:

1. App promise: what the app is and the single biggest user outcome.
2. Feature priority: the top 3-6 benefits to turn into slides, in order.
3. Device targets: which families should be generated now: `iphone`, `ipad`, `ipadLandscape`, or a combination.
4. Assets: which screenshot assets and app icon should be used.
5. Style direction: prefer a concrete preset such as `clean/productivity`, `editorial/premium`, `playful/bright`, `wellness/soft`, `performance/kinetic`, `data/trust`, `dark/cinematic`, or `showcase/gradient`. A reference app is also fine.
6. Brand system: preferred font plus 1-2 brand colors if available.
7. Copy approval: one headline per slide, with optional support text.
8. Localized screenshots: whether the deck should include additional App Store localizations and which locale IDs to generate.

Keep the tone practical. If the user has not written copy yet, help them draft it before generating the deck.

## Style Systems

Treat `styleDirection` as the whole art-direction system, not as a background-color note.

- `clean/productivity`: Notion or Raycast energy. Crisp whitespace, restrained angles, split layouts, very readable UI.
- `editorial/premium`: Bear or Mela energy. Warm neutrals, centered hero moments, premium pacing, low ornament.
- `playful/bright`: Duolingo energy. Saturated gradients, bolder contrast, more asymmetry, punchier copy.
- `wellness/soft`: Calm or Headspace energy. Ambient blur, softer backgrounds, gentle copy, breathing room.
- `performance/kinetic`: Strava energy. Edge bleed, motion, stronger device angles, more athletic contrast.
- `data/trust`: Wise or Robinhood energy. Stable layouts, cool controlled gradients, credibility and clarity first.
- `dark/cinematic`: premium-dark entertainment or fintech feel. Dramatic blur, statement slides, stronger contrast.
- `showcase/gradient`: luminous hero gradients and focal-device compositions for polished marketing moments.

If the user references an app, map it to the closest preset and mention that assumption.

For the add-device-type flow, always ask one extra decision:
- derive from an existing device family already present in the document
- or build that new device family from scratch using the same brief structure as a full generation

## Copy Rules

Shotforge can compose slides, but the copy still drives the result.

Follow these rules:
- One idea per slide.
- Keep headlines short and thumbnail-readable.
- Aim for roughly 3-5 words per line.
- Prefer 2-4 headline lines. If the idea needs more, shorten it instead of trusting auto-fit alone.
- Avoid joining two claims with “and”.
- Use `<br />` where line breaks should be preserved.
- Put the strongest promise on the first slide.
- Include one contrast slide when the deck is long enough.

For localized decks:
- Do not literally translate headlines if the result becomes long or awkward.
- Rewrite copy for the target market while preserving the same selling idea.
- Re-check line breaks for long-word locales such as German, French, and Portuguese.
- For RTL locales, prefer mirrored asymmetric compositions instead of only swapping the text.
- Verify the localized copy still fits before export.

## Core MCP Tools

### `shotforge_get_design_questionnaire`

Use first when you need to understand what information to collect before generating screenshots.

### `shotforge_get_active_document`

Returns the current Shotforge snapshot, including:
- full `ShotProject`
- asset references
- missing asset IDs

Use this to understand the current state before importing assets or generating a deck.

### `shotforge_store_asset`

Imports an asset into the current Shotforge document.

Use one of:
- `filePath`: absolute path on disk
- `base64Data`: base64 or data URL payload

Optional:
- `suggestedFilename`

Use this for:
- app screenshots
- app icon
- generated illustrations or supporting art

### `shotforge_get_asset_preview`

Returns a downscaled PNG preview for one exact Shotforge asset so the agent can reason about the correct pixels before making a crop.

Arguments:
- `assetID`: exact asset UUID

Result includes:
- `assetID`
- `originalFilename`
- `previewBase64PNG`
- `previewPixelWidth`
- `previewPixelHeight`

Use this before selecting an automated callout crop when the request depends on exact screenshot pixels.

### `shotforge_add_callout`

Adds one floating callout crop to an existing Shotforge device.

Arguments:
- `sourceDeviceID`: exact non-window device placement UUID that should host the callout
- `normalizedCropRect`: crop rect in `0...1` coordinates relative to the full screenshot image, using a top-left origin
- `contentZoom`: optional zoom for the floating callout content; defaults to `1.2` and must be within `1.0...3.0`
- `cornerRadius`: optional rounded-corner radius as a fraction of the callout's shorter side; defaults to `0.08` and must be within `0.0...0.5`
- `borderWidth`: optional border width in pixels; defaults to `2.0`
- `borderColorHex`: optional border color as `#RRGGBB` or `#RRGGBBAA`; defaults to white
- `description`: optional short label for the created callout
- `includeSnapshot`: optional boolean

Behavior:
- validates that the source device still exists and is not already a window preset
- validates the crop rect and content zoom
- appends a `DeviceCallout` to the source placement without creating a new asset or placement
- renders the crop from the source device's effective screenshot, including localized screenshot overrides

Result includes:
- `sourceDeviceID`
- `createdCalloutID`
- `documentUpdate`
- `notes`
- optional `snapshot`

### `shotforge_generate_design_deck`

This is the whole-project reset command. Use it for flow 1: generate one or more device families and any requested localizations in one pass.

Arguments:
- `projectName`: optional output project name
- `appName`: required
- `deviceFamily`: optional single-family shortcut
- `deviceFamilies`: optional ordered list of device families such as `iphone`, `ipad`, and `ipadLandscape`
- `appIconAssetID`: optional
- `styleDirection`: optional but strongly recommended; prefer one of the named presets above or a reference app
- `accentColorHex`: optional `#RRGGBB` or `#RRGGBBAA`
- `secondaryColorHex`: optional `#RRGGBB` or `#RRGGBBAA`
- `fontName`: optional font family name
- `sourceLocalizationID`: optional source locale, such as `en-US`
- `localizations`: optional array of localized variants, each with `localizationID` plus localized `slides`
- `slides`: required ordered array

Each slide can include:
- `name`
- `label`
- `headline`
- `supportingText`
- `screenshotAssetIDs`
- `backgroundAssetID`
- `symbolName`
- `layout`
- `emphasis`

Important behavior:
- Shotforge automatically varies layouts across slides.
- Shotforge may apply contrast styling automatically on longer decks.
- Style direction now influences layout bias, background treatment, symbol usage, and device energy, not just palette selection.
- `<br />` in text is converted into actual line breaks.
- If a slide has multiple screenshot asset IDs, Shotforge may use a layered/two-device composition.
- Shotforge now measures headline/support text, reduces font size when needed, and returns `slideDiagnostics` so agents can detect dense or risky slides.
- When `localizations` are provided, Shotforge creates additional in-app localization variants that reuse the same backgrounds and device placements while localizing text and per-device screenshots.
- When `deviceFamilies` contains more than one family, Shotforge creates multiple device rows in the same project while preserving the same story arc and localization structure across them.

### `shotforge_fill_deck_from_questionnaire`

Use this for flow 2 when the user wants to add or replace one device family from scratch without resetting the whole document.

Arguments:
- same shape as `shotforge_generate_design_deck`
- but provide exactly one target through `deviceFamily` or a single-item `deviceFamilies`

Behavior:
- replaces only the requested device-family row
- preserves the other device rows already in the active project
- supports localized variants for that device family in the same call

### `shotforge_fill_deck_from_existing_deck`

Use this for flow 3 when the user wants to add a new device family by deriving from another family that is already present in the active document.

Arguments:
- `sourceDeviceFamily`: existing family to derive from
- `targetDeviceFamily`: destination family to create or replace
- `projectName`: optional override

Behavior:
- copies the source family's page structure, device placements, and localization overrides from the active document
- scales page geometry, text, and device transforms for the target device family
- preserves other device rows in the document

### `shotforge_localize_selected_localization`

Use this after flows 1 through 3 when the deck already exists and you only want to add or refresh one localization across the current device rows.

Arguments:
- `localizationID`: optional target locale ID; defaults to the currently selected localization
- `slides`: localized slide definitions in page order, or with explicit `pageID` values

Behavior:
- updates localized text content for the target locale
- applies per-device screenshot overrides for that locale only
- keeps shared page backgrounds and device transforms intact
- works across every page in the active project, including mixed device-family rows
- returns diagnostics so you can spot dense localized copy before export

### `shotforge_render_pages`

Exports one or more pages to PNG files.

Arguments:
- `outputDirectoryPath`: required absolute path
- `pageIDs`: optional subset

### `shotforge_save_document`

Use in stdio mode when the document should be written to disk.

## Slide Brief Shape

Use a structured brief like this:

```json
{
  "projectName": "Stride Screenshots",
  "appName": "Stride",
  "deviceFamilies": ["iphone", "ipadLandscape"],
  "appIconAssetID": "UUID",
  "styleDirection": "clean/minimal",
  "accentColorHex": "#3B82F6",
  "secondaryColorHex": "#122033",
  "fontName": "Avenir Next",
  "slides": [
    {
      "name": "Hero",
      "label": "Runner Focus",
      "headline": "Keep your<br />streak alive",
      "supportingText": "Flexible plans that fit your week.",
      "screenshotAssetIDs": ["UUID"],
      "symbolName": "figure.run",
      "emphasis": "hero"
    },
    {
      "name": "Coaching",
      "headline": "Run with calm coaching",
      "supportingText": "Pace prompts that stay out of your way.",
      "screenshotAssetIDs": ["UUID"],
      "symbolName": "waveform.path.ecg"
    }
  ]
}
```

## Layout Guidance

You can let Shotforge choose layouts automatically, or set `slide.layout` explicitly.

Supported layout names:
- `hero-centered`
- `copy-left-device-right`
- `copy-right-device-left`
- `device-bleed-right`
- `device-bleed-left`
- `layered-devices`
- `contrast-statement`

## Overlap Prevention

Unlike HTML mockups, Shotforge compositions are generated from a constrained native canvas model. Treat overlap prevention as an explicit workflow concern.

Do this every time:
- Keep the copy disciplined before generation.
- Generate the deck.
- Read `slideDiagnostics`.
- If any slide has `hasOverflowRisk: true`, revise the copy or force a roomier layout and regenerate.
- Render proof PNGs before calling the deck final.

When a slide feels cramped, try this order:
1. shorten the headline
2. shorten or drop supporting text
3. add intentional `<br />` line breaks
4. switch to `hero-centered` or `contrast-statement`
5. only then consider low-level manual edits

## Practical Operating Rules

- Import assets before generating the deck if they are not already in the active document.
- Prefer the app icon on the hero slide and sometimes the final slide.
- If the user changes the story arc significantly, regenerate the deck instead of patching low-level project JSON by hand.
- If the user cares about a very specific look, set `slide.layout` intentionally instead of assuming the preset alone will force that composition.
- If the user wants a fresh language variant after the base deck exists, prefer `shotforge_localize_selected_localization` over replacing the whole project.
- If the user wants manual crop selection, tell them to use the right-click `Add Callout` or `Edit Callout` flow in the canvas.
- For automated callouts, inspect an asset preview first, then use `slide.callouts` during generation or `shotforge_add_callout` for incremental edits.
- Use `shotforge_replace_project` only for advanced repairs or deliberately custom edits that the design generator cannot express.
- After generation, inspect the returned `layoutSequence`, `slideDiagnostics`, `notes`, and snapshot before exporting.

## Good Outcomes

A strong Shotforge MCP run should:
- ask only the necessary questions
- produce one clear promise per slide
- vary composition across adjacent slides
- use brand colors and font when provided
- detect dense slides before export
- export ready-to-review PNGs, not just a modified project state

## If Something Is Missing

If results look weak, check these first:
- the copy is too long or sells two ideas at once
- `slideDiagnostics` is reporting overflow risk and the copy was not revised
- no app icon or screenshots were imported
- the style direction was vague
- the wrong screenshot asset IDs were assigned to slides
- the task really needs a custom low-level edit instead of the high-level deck generator
