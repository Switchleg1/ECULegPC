# ECULegPC

**v0.78** — Professional ECU binary editor for desktop.  
PC port of the ECULeg Android app, built with Python + PyQt6.

---

## Requirements

```
pip install PyQt6
```

> `PyQt6-Qt6` and `PyQt6-sip` are pulled in automatically.  
> For SVG toolbar icons (recommended): `pip install PyQt6-Qt6 PyQt6.QtSvg` — included with a standard PyQt6 install.

## Run

```
python main.py
```

---

## Features

### Definition & Binary loading

- Supports **TunerPro XDF**, **ECU XML**, and native **XLEG v0.2** definition files
- Drag-and-drop a definition or BIN directly onto the window
- Recent-file menus for definitions, BINs, PID CSVs, and log files
- **Definition Editor** — create and modify table definitions (name, description, axis addresses, data types, math equations, storage order) without leaving the app
- Export the loaded definition back to `.xdf`, `.xml`, or `.xleg`
- **Overlap detection** — an amber warning bar appears on any table whose BIN memory region intersects another table, with clickable links to the conflicting tables

### Table grid view

- **Colour-coded heat map** — cells shade blue → green → red relative to the z-axis min/max
- **Frozen axis headers** — the X-axis breakpoint row and Y-axis breakpoint column stay pinned while scrolling large tables
- Multi-cell selection: click, drag, Shift+click (range), Ctrl+click (individual toggle)
- **Inline cell editing**
  - Double-click, press **F2**, or just start typing on a selected cell to open the inline editor
  - Works on both data cells and axis header breakpoint cells
  - **Tab / Shift+Tab** to commit and move to the next/previous cell
  - **Escape** cancels without saving
- **Toolbar arithmetic** — enter a value in the *Set* box and press Enter to set all selected cells; or use **+  −  ×  ÷** to apply arithmetic to every selected cell at once
- **Undo / Redo** — 100-step history covering cell edits, arithmetic, axis range changes, and smoothing

### Axis breakpoint editing

- Click any axis header cell and edit its value directly (inline editor, same as data cells)  
  *(only available when the axis has a binary address in the definition)*
- **Set axis range** — right-click any axis header cell to redistribute all breakpoints evenly across a new start/end range; every data cell is resampled via linear interpolation (extrapolation beyond the original range is supported)

### Smooth / Interpolate

Toolbar button **Smooth…** opens a dialog to blend selected cells with their neighbours:

- **Compass widget** — drag the indicator into the right/down quadrant to set the X/Y blend direction visually
- **X weight / Y weight spinboxes** — type weights directly; compass and spinboxes stay in sync
- **Passes** (1–20) — apply the smoothing kernel multiple times for a stronger effect
- Each pass snapshots cell values first so intra-pass neighbours don't feed back into each other
- Weights are automatically normalised if their sum exceeds 1.0 (center weight is never negative)
- Full undo in a single step regardless of pass count

### Waterfall (line chart) view

Switch to **Lines ↔** (row mode) or **Lines ↕** (column mode):

- One colour-coded line per row or column
- Drag data points vertically/horizontally to change their value
- Zoom and pan with mouse wheel + drag
- Changes are reflected live in the grid and 3D views

### 3D surface view

Switch to **3D**:

- Orthographic polygon surface with painter's-algorithm depth sort
- Rotate by dragging, zoom with the mouse wheel
- Click vertices to select and edit values directly on the surface
- Colour scheme matches the grid heat map

### Log file integration

- Load a **CSV log file** (File → Open Log or drag-and-drop)
- **Log Viewer** — multi-channel scrollable waveform pane with zoom, pan, and drag-to-reorder channels
- Per-table **X axis / Y axis channel dropdowns** — link any log channel to either table axis
- Channels are auto-matched to axis names/units on first load; previous selections are restored when the log is reloaded
- **Live cursor overlay** — as the log playhead moves, the grid highlights up to four cells bracketing the current value using bilinear interpolation, with weighted opacity showing proximity
- Cyan cursor triangles on the frozen axis header strips mark the exact interpolated position
- **PID CSV import** — load a PID definition file to add channel descriptions and units

---

## Keyboard shortcuts

| Key | Action |
|-----|--------|
| **Ctrl+O** | Open Definition |
| **Ctrl+B** | Open BIN |
| **Ctrl+S** | Save BIN |
| **Ctrl+Z** | Undo |
| **Ctrl+Y** | Redo |
| **F2** | Edit selected cell |
| **Enter** | Apply toolbar value to selection |
| **Tab / Shift+Tab** | Commit edit and move right / left |
| **Escape** | Cancel edit |
| **Arrow keys** | Move selection (crosses axis/data boundary) |
| **Ctrl+A** | Select all data cells |
| **Any printable key** | Start inline edit with that character |

---

## Settings

**Edit → Settings…** persists preferences to `%APPDATA%\ECULegPC\config.json`.

| Setting | Description |
|---------|-------------|
| **Cell width / height** | Pixel dimensions of every table cell. Updated live across all open tabs immediately — no restart needed. |
| **Default precision** | Decimal places used for tables whose definition does not specify a precision value. |
| **Drag & drop behaviour** | Ask for confirmation or replace silently when dropping a file while one is already open. |
| **Exit behaviour** | Prompt / auto-save / discard on close with unsaved changes. |

---

## File structure

```
main.py                  Entry point; applies config cell-size settings at startup
core/
  definition.py          Internal XLEG data model (Definition, Table, Axis, StorageType)
  bin_file.py            Binary file read/write with endian and float support
  table_data.py          Table load/save, undo/redo, axis range redistribution,
                         linear interpolation helpers
  math_eval.py           Expression evaluator + automatic inverse equation builder
  config.py              JSON config (recent files, window state, display prefs)
  log_data.py            Log/PID data model
  version.py             Application version string
parsers/
  xdf_parser.py          TunerPro XDF parser
  xml_parser.py          ECU XML parser
  xleg_parser.py         XLEG native parser + XDF / XML / XLEG writers
  pid_csv.py             PID definition CSV importer
  log_csv.py             Log data CSV importer
ui/
  main_window.py         Main window: tree panel, tabbed workspace, toolbar, menus
  table_editor.py        Grid view (frozen headers, inline editing, smoothing,
                         axis range dialog, log overlay), waterfall/3D view switcher
  waterfall_view.py      Line-chart view with draggable data points
  surface_view.py        3D orthographic polygon surface
  log_viewer.py          Multi-channel waveform viewer with zoom and pan
  definition_editor.py   In-app definition creator/editor
  settings_dialog.py     Preferences dialog
  about_dialog.py        About screen
  icons.py               SVG toolbar icons + application icon (rendered at runtime)
```

---

## Definition formats

| Format | Read | Write | Notes |
|--------|:----:|:-----:|-------|
| `.xdf` | ✓ | ✓ | TunerPro XDF — widely supported tuning format |
| `.xml` | ✓ | ✓ | ECU XML — used by the ECULeg Android toolchain |
| `.xleg` | ✓ | ✓ | Native XLEG v0.2 — unified internal format |

---

## Credits

Built by **switchleg** / switch-developments.  
Python + PyQt6. Icon and UI designed to match the ECULeg Android aesthetic.
