---
name: figma-bar-chart
description: Populate a Figma bar chart template with spreadsheet data (CSV/Excel/Google Sheets). Supports simple, stacked, and parallel chart types.
---

# Figma Chart - Populate Figma Bar Charts with Spreadsheet Data

A skill that generates bar charts by populating the Figma Graph_Sample template with spreadsheet data (CSV/Excel/Google Sheets).

## Workflow

### Step 1: Obtain the Data File
Ask the user for the path to the data file. CSV, Excel (.xlsx/.xls), and Google Sheets URLs are accepted. (Dragging and dropping from Finder into the terminal is fine.)
- **CSV**: Use as-is
- **Excel**: Read the file and convert to CSV format for processing
- **Google Sheets**: Export as CSV and use for processing

### Step 2: Read and Parse the CSV
Read the CSV and parse the following structure:
- `year` row: Year data (empty cells inherit the previous value)
- `month` row: Month or quarter labels (10, 11, Q1, Q2, etc.)
- `value1` row: Required. The first value series
- `value2` row: Optional. The second value series
- `value3` row: Optional. The third value series
- Final row: Chart type specifier (`simple`, `stacked`, `parallel`)

### Step 3: Determine the Chart Type
If a type is specified in the CSV, use it. Otherwise, ask the user:
- value1 only → `simple` (simple bar chart)
- value1 + value2 + `parallel` → `parallel` (parallel bar chart)
- value1 + value2 + value3 + `stacked` → `stacked` (stacked bar chart)

### Step 4: Get the Figma URL
Ask the user for the URL of the Figma file containing the Graph_Sample component set.

### Step 5: Get Guide Settings
Ask the user for the following:
- Maximum guide value (e.g., 1400)
- Guide interval (e.g., 200)

### Step 6: Instantiate the Template
1. Create an instance of the matching variant from the Graph_Sample COMPONENT_SET
   - simple → `Property 1=Simple`
   - stacked → `Property 1=stacked`
   - parallel → `Property 1=parallel`
2. Place it to the right of the component set

### Step 6.5: Rename the Instance
Rename the top-level frame to the CSV filename (without extension).
Example: `sales_2025Q1.csv` → frame name becomes `sales_2025Q1`

### Step 7: Detach All Levels
1. Detach the top-level instance
2. Detach all Graph instances inside Graph_Container
3. For stacked/parallel types, also detach children inside Bar_Body_Container
4. Set `layoutSizingVertical` to `FILL` on all `Bar` nodes

### Step 8: Add Guides and Set Labels
1. Duplicate guides as needed (max value / interval + 1)
2. Set the text label on each guide (from max value at the top down to 0)

### Step 9: Populate the Data

#### Simple
- Control each Graph's bar height via the Graph node's own height
- `newGraphHeight = (value / guideMax) * maxBarArea + labelArea`
- labelArea = 88px (month 38 + gap 6 + year 38 + gap 6)
- maxBarArea = 332px

#### Stacked
- Set the height of Bar_Body-Value2 and Value3 individually: `(value / guideMax) * maxBarArea`
- Bar_Body-Value1 uses FILL (fills the remaining space)
- Set the overall Graph height based on the total: `(total / guideMax) * maxBarArea + labelArea`

#### Parallel
- Set the height of each Bar_Body-Value individually: `(value / guideMax) * maxBarArea`
- Do not change the overall Graph height (each bar is independent in a parallel layout)

### Step 10: Set Labels
Default rules (can be changed if the user requests):
- **Month/quarter labels**: Show every third label; replace the rest with a single space
- **Year labels**: Show only the first occurrence of each year; replace the rest with a single space

### Step 11: Verify with a Screenshot
Use `get_screenshot` to check the result and show it to the user.

## Fonts
Load the following fonts when modifying text:
- `{ family: "Noto Sans", style: "Regular" }` — for guide labels
- `{ family: "Noto Sans", style: "Bold" }` — for month/year labels

## Notes
- This skill requires Figma MCP Remote (`figma-remote`), NOT Figma MCP Desktop (`figma-desktop`). Never use `figma-desktop` tools
- When the CSV year row contains empty cells, fill them by carrying forward the previous value
- Add guides by cloning existing guides inside Guide_Container
- Do not forget to set Bar's layoutSizingVertical to FILL after detaching
- Always specify `skillNames: "figma-use"` when calling `use_figma`

$ARGUMENTS
