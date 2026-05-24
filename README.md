# ECULegPC

**v0.81** — Professional ECU binary editor for desktop.  
PC port of the ECULeg Android app, built with Python + PyQt6.

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
- **Label axes** — axes defined with human-readable string labels (e.g. gear-shift names from a TunerPro XDF) are displayed as-is in the axis header row/column and are read-only; selecting a label cell shows the label text in the status bar

### Building out tables (Set Axis Range)

The **Set Axis Range** dialog lets you fully rebuild an axis from scratch without manually editing every breakpoint:

1. Right-click any axis header cell and choose **Set axis range…**
2. Enter the new **Start** and **End** values for the axis
3. Click **Apply** — all breakpoints are redistributed evenly across the new range, and every data cell is resampled at its new position using the original data as a reference

Key details:
- **Even spacing** — breakpoints are spaced uniformly between start and end; no manual entry required
- **Linear resampling** — each data cell's value is interpolated (or extrapolated) from the original data, so the shape of the table is preserved even after a range change
- **Extrapolation supported** — the new range can be wider than the original; values beyond the original endpoints are linearly extended
- **Full undo** — the entire range change (axis + all resampled cells) is recorded as a single undo step
- Requires at least 2 breakpoints on the axis

### Smooth / Interpolate

Toolbar button **Smooth…** opens a dialog to blend selected cells with their neighbours.  Select at least two data cells first, then click **Smooth…**:

- **Compass widget** — drag the lit indicator into the right/down quadrant to set the blend direction visually
  - Drag straight right → pure horizontal (X) smoothing
  - Drag straight down → pure vertical (Y) smoothing
  - Drag diagonally → equal X/Y mix
  - Near centre → near-zero smoothing (almost no change)
- **X weight / Y weight spinboxes** — type weights directly (0.00 – 1.00); compass and spinboxes stay in sync
- **Passes** (1 – 20) — apply the smoothing kernel multiple times for a stronger effect
- **Normalisation** — weights are automatically normalised if their sum exceeds 1.0; the center weight is never driven negative; if both weights are zero the dialog warns and does nothing
- Each pass snapshots the selected cell values first so intra-pass neighbours do not feed back into the same pass
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
- **PID CSV import** — load a PID definition file to add channel descriptions, units and scaling.  If a PID CSV is not loaded items will be scaled by data swing within the log (NOT DESIRABLE) 

---

## Advanced Tools

### Pattern Matching (`Advanced → Pattern Matching…`)

Pattern Matching lets you take a **known definition** (definition + BIN pair already open in the editor) and locate where those same tables live in a **different, unknown BIN** — for example, a different ECU variant or software revision.  The result is a new definition file remapped to the target binary's addresses.

> **Prerequisite:** both a definition and a BIN must be loaded before opening the wizard.

The wizard has three pages:

#### Page 1 — Load & Configure

1. **Target BIN** — browse or drop the unknown binary you want to match against
2. **Search window** — a dual-handle range slider lets you restrict the scan to a specific byte range within the target file (useful when the calibration region is known).  The **Full Range** button resets to the entire file.  Hex address fields allow precise entry.
3. **Options:**

| Slider | Range | Default | Effect |
|--------|-------|---------|--------|
| **Sensitivity** | 0 – 100 | 50 | Sets the minimum match percentage required (100 = 90 % threshold, 0 = 40 % threshold).  Lower values return more candidates but with higher false-positive rates. |
| **Thoroughness** | 0 – 100 | 50 | Controls the scan stride.  At 0 the scanner steps 16 bytes at a time (fast, may miss offset tables); at 100 it checks every byte (exhaustive). |
| **Max Candidates** | 1 – 10 | 3 | Number of best-scoring match locations to keep per table.  Higher values let you choose between multiple plausible hits on the Review page. |
| **Min Table Size** | 1 – 10 | 3 | Tables with fewer cells than this are skipped.  Single-cell tables have too little data for a reliable match; the default of 3 eliminates most false positives on tiny constants. |

4. Click **Run Match** — progress is shown live with the current table name.  All available CPU cores are used in parallel for maximum speed.  Click **Abort** to stop early.

#### Page 2 — Review Matches

After matching completes, every matched table is listed in a tree grouped by tier (Exact / Good / Acceptable).  For each table you can:

- **Filter** — type in the filter box above the tree to narrow by table name (case-insensitive)
- **Preview** — click any match to see a side-by-side comparison:
  - *Left panel* — the original reference table (values from the loaded definition + BIN)
  - *Right panel* — the best-matched location in the target BIN, rendered with the same heat-map colouring
- **Candidate pane** — below the preview, a row for each candidate location shows its address, match %, and X/Y axis scores.  The highest-scoring candidate is pre-selected.  **Double-click** any row to switch which candidate will be exported.
- **Delete** — click **✕ Remove Match** to exclude a single table, or Ctrl/Shift-click multiple rows and click **✕ Remove N Matches** to mass-delete unwanted matches before export
- Click **Next** when satisfied

#### Page 3 — Export

Choose an output format and file path, then click **Export**:

| Format | Notes |
|--------|-------|
| **XLEG (.xleg)** | Native format — recommended; preserves all metadata |
| **TunerPro XDF (.xdf)** | For use with TunerPro RT and compatible tools |
| **ECU XML (.xml)** | For use with the ECULeg Android toolchain |

The exported file contains only the matched (non-deleted) tables, with every address remapped to the selected candidate's location in the target binary.  Load the exported definition alongside the target BIN to begin tuning.

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

## Definition formats

| Format | Read | Write | Notes |
|--------|:----:|:-----:|-------|
| `.xdf` | ✓ | ✓ | TunerPro XDF — widely supported tuning format |
| `.xml` | ✓ | ✓ | ECU XML — used by the ECULeg Android toolchain |
| `.xleg` | ✓ | ✓ | Native XLEG v0.2 — unified internal format |

### XDF label axes

TunerPro XDF files can define axes whose breakpoints are human-readable string labels rather than numeric values stored in the binary (e.g. gear-change descriptions such as `1->2 (1)`, `2->3 (2)`).  ECULegPC reads, displays, and re-exports these correctly:

- **Import** — `<LABEL index="N" value="…" />` elements inside an `<XDFAXIS>` are parsed and stored; the axis count is derived from the number of labels and no binary address is assigned
- **Display** — label strings appear directly in the axis header cells; the header background is dimmed to indicate the axis is read-only; selecting a label cell shows the text in the status bar
- **Export** — label axes are written back as `<LABEL>` elements in XDF and XLEG output; the XDF `<EMBEDDEDDATA>` uses the correct no-binary sentinel (`mmedmajorstridebits="-32"`) with the proper `<indexcount>`; ECU XML export omits label axes as that format has no equivalent representation

---

## Credits

Built by **switchleg** / switch-developments.  
Python + PyQt6. Icon and UI designed to match the ECULeg Android aesthetic.
