# WLED Preset Editor

A standalone browser-based editor for WLED `presets.json` files. No installation, no server — just open the HTML file and drag in your file.

## What it does

Load a `presets.json` exported from your WLED device, edit it visually, and export it back. You can then upload the modified file to your device via the WLED web interface (Config > Security > Backup/Restore).

The editor lets you:

- Browse all presets in a sidebar and open them one at a time
- Edit every field defined in the WLED JSON API: name, brightness, transition, on/off state, and all segment parameters (effect, speed, intensity, palette, colors, LED range, grouping, CCT, custom sliders, flags)
- Select multiple presets using the checkboxes and apply bulk changes to all of them at once — useful for normalizing brightness, swapping effects across a whole set of presets, or adjusting LED ranges after rewiring
- See a diff of what changed before confirming any save or bulk operation
- View and copy the raw JSON of any preset

## Usage

1. Open `index.html` in a browser
2. Click "Open Editor"
3. Load your `presets.json` via the button or by dragging it onto the page
4. Edit presets individually or select multiple with the checkboxes for bulk editing
5. Export the result with "Export JSON" and upload it back to your WLED device

The file never leaves your browser — everything runs locally.

## Multi-select

Clicking a preset row focuses it in the editor. The small checkbox on the left of each row adds it to the bulk selection independently. Shift-click on a row selects a range of checkboxes. The bulk edit panel appears automatically when at least one checkbox is ticked.

## Links

- WLED project: https://github.com/Aircoookie/WLED
- WLED knowledge base: https://kno.wled.ge
- Effects reference: https://kno.wled.ge/features/effects/
- Presets documentation: https://kno.wled.ge/features/presets/
- JSON API reference: https://kno.wled.ge/interfaces/json-api/
- WLED-Utils (effect previews and tooling): https://github.com/scottrbailey/WLED-Utils
