# ECULegPC

**v0.87** — Professional ECU binary editor for desktop.  
PC port of the ECULeg Android app, built with Python + PyQt6.


<p align="center">
  <img src="https://github.com/Switchleg1/ECULegPC/blob/main/eculeg.png?raw=true" width="400">
</p><br>
---

## Features

### Definition & Binary loading

- Supports **TunerPro XDF**, **ECU XML**, and native **XLEG v0.2** definition files
- Drag-and-drop a definition or BIN directly onto the window
- Recent-file menus for definitions, BINs, PID CSVs, and log files
- **Definition Editor** — create and modify table definitions (name, description, axis addresses, data types, math equations, storage order) without leaving the app
- Export the loaded definition back to `.xdf`, `.xml`, or `.xleg`
- **Overlap detection** — an amber warning bar appears on any table whose BIN memory region intersects another table, with clickable links to the conflicting tables

---

### Compare BIN

A second BIN file can be loaded alongside the working BIN as a **Compare BIN**.  It is used for side-by-side inspection in every open table and as the reference for the Compare Bins diff tool.

- **Load** — toolbar **CMP BIN** button, or **File → Load Compare BIN…**
- **Recent** — **File → Recent BINs (as Compare)** lists the same recent BINs menu so previously used files are one click away
- **Auto-restore** — the last-used compare BIN is remembered and reloaded automatically on the next app launch
- **Status indicator** — a small cyan label on the toolbar shows the filename when a compare BIN is active; grey when none is loaded
- **Close** — **File → Close Compare BIN** unloads the reference and resets all open table editors to BIN mode

---

### Table grid view

- **Colour-coded heat map** — cells shade blue → green → red relative to the z-axis min/max
- **Frozen axis headers** — the X-axis breakpoint row and Y-axis breakpoint column stay pinned while scrolling large tables
- **Swap ⇄** — the toolbar swap button transposes rows and columns for easier editing of wide tables; state is saved per-table
- Multi-cell selection: click, drag, Shift+click (range), Ctrl+click (individual toggle)
- **Inline cell editing**
  - Double-click, press **F2**, or just start typing on a selected cell to open the inline editor
  - Works on both data cells and axis header breakpoint cells
  - **Tab / Shift+Tab** to commit and move to the next/previous cell
  - **Escape** cancels without saving
- **Toolbar arithmetic** — enter a value in the *Set* box and press Enter to set all selected cells; or use **+  −  ×  ÷** to apply arithmetic to every selected cell at once
- **Alt + scroll** — increments or decrements the selected cells by the step amount in the *Set* box (or all data cells if nothing is selected); scroll up to add, scroll down to subtract
- **Copy / Paste** — **Ctrl+C** copies the current selection to the app clipboard and the system clipboard (TSV format); **Ctrl+V** pastes into the current selection (single-cell anchor or bounding-box alignment); also available from the right-click context menu
- **Undo / Redo** — 100-step history covering cell edits, arithmetic, axis range changes, smoothing, and paste operations

#### BIN / CMP / DIFF view modes

When a Compare BIN is loaded, three exclusive view-mode buttons appear in the table toolbar:

| Button | Colour scheme | Cell values shown |
|--------|--------------|-------------------|
| **BIN** | Standard blue → green → red heat map | Working BIN values (default) |
| **CMP** | Brighter teal → cyan → magenta | Compare BIN values — the vivid palette immediately signals that the working BIN is not being shown |
| **DIFF** | Diverging blue → grey → amber | Delta (compare − BIN): blue = compare is lower, grey = identical, amber = compare is higher |

- **BIN** and **CMP** modes use the same range normalisation rules as the rest of the editor (relative or definition-defined — see Settings)
- **DIFF** mode scales symmetrically around zero so equal-magnitude positive and negative deltas appear as equal-intensity colours
- The orange "modified" cell border is only shown in **BIN** mode; it is suppressed in CMP and DIFF to avoid confusion between working edits and compare data
- Header breakpoint cells always display working BIN values regardless of mode
- CMP and DIFF buttons are disabled when no Compare BIN is loaded

---

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

---

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

---

### Unit Conversion

The **Convert Units** toolbar button applies display-only unit conversions to all views simultaneously.  The raw values stored in the BIN are **never modified** — conversions happen at paint time only, so the file stays clean.

#### How it works

1. Define conversion rules in **Edit → Settings → Units** (see Settings section below)
2. Open any table whose axis or data units match one of your enabled rules
3. Click **Convert Units** in the toolbar — the button highlights blue when active
4. All three views update instantly:
   - **Grid** — every cell displays the converted value; axis header breakpoints also convert if the axis has a matching rule
   - **2D waterfall** — Y-axis tick labels and X-axis tick labels update to converted values; axis names update in cyan
   - **3D surface** — all three axis rails (X, Y, Z) update tick labels and names

A **conversion badge** appears in the top-right corner of the 2D and 3D views showing which axes are converted (e.g. `Units: X: RPM  |  Data: bar`).

**Inline editing while converted** — if a conversion rule has an inverse equation defined, you can edit cells while the conversion is active and type values in the converted unit; the app automatically back-converts to raw before storing.

Click **Convert Units** again to revert all views to the original units.  The tab title shows `*` while conversions are active (just like unsaved edits) to make the state obvious.

#### Matching rules

Each conversion rule specifies a **Unit name** (e.g. `kPa`).  When you press Convert Units, the app scans the current table's X, Y, and Z axis unit strings and applies the first enabled rule whose unit name matches (case-insensitive by default; the rule can opt in to case-sensitive matching).

---

### Alternate View panel (2D / 3D)

The **Alternate View** panel lives in the bottom tab bar and stays linked to whichever table tab is currently selected.

| Table type | Panel shows |
|------------|-------------|
| **Scalar (1D)** | Placeholder — no chart available |
| **Single-axis (row or column only)** | Waterfall (2D) — one line per row |
| **Two-axis** | Toggle between **2D** waterfall and **3D** surface |

Both views:
- Share the same underlying data as the grid — any edit made in the grid, 2D, or 3D view is reflected in all three instantly
- Respect the **Convert Units** state — toggling the unit button updates axes and data in all views at once
- Support vertex drag-to-edit: drag a point in the 2D view vertically, or drag a vertex on the 3D surface, to change its value; changes are undoable

#### Waterfall (2D line chart)

- One colour-coded line per Y-axis row (green → red gradient)
- X axis = table X-axis breakpoints; Y axis = cell value scale
- Row labels displayed along the right edge at each line's endpoint
- Drag data points vertically to change their value
- Converted unit labels shown in cyan when Convert Units is active

#### 3D surface

- Orthographic polygon surface with painter's-algorithm depth sort
- Rotate by dragging, zoom with the mouse wheel
- Click and drag vertices to change their value directly on the surface
- Colour scheme matches the grid heat map (blue → green → red)
- Back-wall grid lines and axis rails with tick marks for all three axes
- Selecting a vertex syncs the selection highlight back to the grid

---

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

### Compare Bins (`Advanced → Compare Bins…`)

Byte-level diff between the currently loaded BIN and a second BIN of the same structure.  Differences are mapped back to definition tables and presented in a collapsible results tree; you can cherry-pick individual changes and copy them into the working BIN.

> **Prerequisite:** a definition and a working BIN must be loaded.  If a Compare BIN is already loaded in the main window, the dialog opens with its path pre-filled and the Compare button immediately enabled.

#### Setup

1. **Compare Binary** — the path field is pre-populated if a Compare BIN is already loaded; otherwise click **Browse…** to select a file
2. **Search Window** — a dual-handle range slider restricts the scan to a specific byte range (the entire file by default).  Hex address fields allow precise entry; **Full Range** resets the slider

#### Results tree

After clicking **Compare**, diffs appear in a tree grouped by table name (or **Unregistered** for regions between known tables):

- Each group header shows the table name and the number of diffs it contains
- Expand any group to see individual diff rows — each row shows the byte-range address and size
- **Expand a diff row** to reveal an inline detail pane with one row per affected cell:

| Column | Content |
|--------|---------|
| **Address** | Absolute byte offset of the cell |
| **Row** | Table row number (or — for 1-D tables) |
| **Col** | Table column number (or — for 1-D tables) |
| **BIN** | Decoded working BIN value with the table's units and precision |
| **Compare BIN** | Decoded compare BIN value with the same formatting |

  Bytes not covered by a registered table z-axis are shown as raw hex.

- **Double-click a table group** to open that table in the main editor
- **Checkboxes** on every row and group control which diffs will be copied; checking or unchecking a group propagates to all its children

#### Actions

| Button | Effect |
|--------|--------|
| **Copy Selected to Current BIN** | Writes the byte ranges of all checked diffs from the compare BIN into the working BIN; all open table editors reload automatically |
| **Remove** | Deletes selected diff rows from the results (does not affect either BIN file) |
| **Clear All** | Removes all results |

---

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
| **Ctrl+D** | Open Definition file |
| **Ctrl+O** | Open BIN file |
| **Ctrl+S** | Save BIN |
| **Ctrl+Shift+S** | Save BIN As… |
| **Ctrl+T** | Save Table (commit to working BIN) |
| **Ctrl+Z** | Undo |
| **Ctrl+Y** | Redo |
| **Ctrl+A** | Select all data cells |
| **Ctrl+C** | Copy selected data cells (app clipboard + system TSV) |
| **Ctrl+V** | Paste into current selection |
| **Ctrl+W** | Close current tab |
| **Ctrl+L** | Open Log CSV |
| **Ctrl+E** | Open Definition Editor |
| **Ctrl+\\** | Toggle tree panel |
| **Ctrl+Q** | Quit |
| **F2** | Edit selected cell |
| **Enter** | Apply toolbar value to selection |
| **Tab / Shift+Tab** | Commit edit and move right / left |
| **Escape** | Cancel edit |
| **Arrow keys** | Move selection (crosses axis/data boundary) |
| **Alt + scroll up** | Add step value to selected cells |
| **Alt + scroll down** | Subtract step value from selected cells |
| **Any printable key** | Start inline edit with that character |

---

## Settings

**Edit → Settings…** persists preferences to `%APPDATA%\ECULegPC\config.json`.

### General

| Setting | Description |
|---------|-------------|
| **Exit behaviour** | Prompt / auto-save / discard on close with unsaved changes |
| **Drag & drop — Definitions** | Ask for confirmation or replace silently when dropping a definition while one is already open |
| **Drag & drop — BINs** | Ask for confirmation or replace silently when dropping a BIN while one is already open |
| **Definition tree** | Keep all categories collapsed (default) or expand all on load |

### Tables

| Setting | Description |
|---------|-------------|
| **Cell width / height** | Pixel dimensions of every table cell.  Updated live across all open tabs immediately — no restart needed |
| **Default precision** | Decimal places used for tables whose definition does not specify a precision value |
| **Value range mode** | **Relative** (default) — heat-map and chart axes scale to the actual data min/max; **Definition defined** — use the XDF z-axis min/max bounds, expanding outward if cell values fall outside them |

### Units

Define reusable unit conversion rules applied globally across all tables.  Rules are matched against a table's axis unit strings when you click **Convert Units**.

Each rule has:

| Field | Description |
|-------|-------------|
| **Unit name** | The raw unit string to match (e.g. `kPa`).  Matched case-insensitively unless **Case-sensitive** is checked. |
| **Convert name** | The display name shown in the status bar and conversion badge when the rule is active (e.g. `bar`) |
| **Description** | Optional free-text note |
| **Equation** | Python math expression converting the raw value.  Use `X` for the raw value.  Standard math functions (`sin`, `cos`, `sqrt`, `log`, …) are available.  Example: `X / 100` to convert kPa → bar |
| **Inverse equation** | Optional reverse expression (`X` = converted value → raw).  Required to allow inline editing while conversions are active.  Click **Build ⟳** to auto-generate for simple linear equations |
| **Override precision** | When checked, overrides the axis decimal-places setting for this conversion (useful when the converted unit needs more or fewer digits) |

Rules can be individually enabled/disabled from the rules table without deleting them.  The **Enable unit conversion by default** checkbox causes the Convert Units button to activate automatically whenever a table is opened.

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
