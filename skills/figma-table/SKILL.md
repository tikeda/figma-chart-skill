---
name: figma-table
description: Populate a Figma table template with spreadsheet data (CSV/Excel/Google Sheets). Supports variable row/column counts and multi-level headers.
---

# Figma Table - Populate a Figma Table with Spreadsheet Data

This skill generates a table by populating a Figma table template with spreadsheet data (CSV/Excel/Google Sheets).
Because the template design may vary each time, the structure is analyzed dynamically.

## Workflow

### Step 1: Obtain the Data File
Ask the user for the path to the data file. CSV, Excel (.xlsx/.xls), and Google Sheets URLs are accepted. (Dragging from Finder into the terminal is fine.)
- **CSV**: Use as-is
- **Excel**: Read the file and convert to CSV format for processing
- **Google Sheets**: Export as CSV and use for processing

### Step 2: Read and Parse the CSV
Read the CSV and analyze the following structure:
- **Header rows**: `year`, `month`, etc. (multi-row headers indicate hierarchical headers)
  - `year` row: Period data (empty cells inherit the previous value)
  - `month` row: Quarter/month labels (Q1, Q2, 10, 11, etc.). May not be present
- **Data rows**: Rows consisting of a row label + values (e.g., Revenue, Operating Profit, EBITDA)
  - The number of rows is variable. Process as many rows as the CSV contains
  - `▲` is a negative notation and should be displayed as-is

#### >> User Confirmation 1: Verify CSV Parse Results
Present the parse results in table format and confirm the following:
- Whether the header hierarchy (year / month) was recognized correctly
- Whether the data row count and labels are correct
- Whether the column grouping is correct

### Step 3: Obtain the Figma Template URL
Ask the user for the URL of the Figma file containing the table template.

### Step 4: Analyze the Template Structure
Use `use_figma` to traverse the template's node tree and identify:
- Whether the template is a COMPONENT or COMPONENT_SET
- The header hierarchy (e.g., two levels with period header then quarter header, or one level with period header only)
- The number of cells in data rows
- Cell component variants (Design=A/B/C/D, etc.)

### Step 5: Map Column Groups to Style Regions
Auto-detect the style regions in the template (column groups distinguished by differences in background color, text color, and strokes), then map them to the CSV column groups.

#### Detecting Style Regions
Traverse the template's column frames and identify groups with visually distinct styles (background color, text color, Cell variant).
Examples:
- Columns with gray background + gray text (subdued style)
- Columns with green background + white/green text (emphasis style)
- Label column with white background + black text

#### Splitting CSV Column Groups
Group CSV columns by shared values in the header row (year, etc.).
Examples:
- Columns belonging to `2024/9期`
- Columns belonging to `2025/10期`
- Calculated column groups such as `前年比`, `前四半期比`

#### Mapping Rules
- Compare the number of style regions in the template with the number of CSV column groups
- If the counts match, map them in order
- If the counts differ, ask the user for confirmation

#### >> User Confirmation 2: Verify Template Analysis and Mapping
Present the template structure analysis results and mapping, and confirm the following:
- Whether the style regions were recognized correctly
- Whether the mapping between CSV column groups and style regions is correct
- Any row/column count differences between the CSV and template (rows/columns that need to be added or removed)

### Step 6: Instantiate the Template
1. Create an instance from the template
2. Place it to the right of the template
3. Detach the top-level instance
4. Rename the top-level frame to the CSV filename (without extension)

### Step 7: Adjust Row and Column Counts
If the template and CSV have different row/column counts:
- **If columns are insufficient**: Clone cells (INSTANCEs) within existing columns to add more
- **If there are excess columns**: Delete the extra columns
- **If rows are insufficient**: Clone cells in the Label column and each data column to add more
- **If there are excess rows**: Delete the extra cells

### Step 8: Populate the Data
1. **Label column**: Set the CSV data row labels (e.g., Revenue, Operating Profit)
2. **Period headers**: Set the CSV year values (for Previous/Current respectively)
3. **Quarter/month headers**: Set the CSV month values (only if a month row exists)
4. **Value cells**: Set each data row's values directly from the CSV
5. **Percentage cells (Design=E, etc.)**: The value and percent sign may be in separate text nodes
   - If the value contains `%`, remove the percent sign and set it in the value node; leave the `%` node as-is
   - If the value is `-`, set `-` in the value node and replace the `%` node with a half-width space

### Step 9: Automatic Design Corrections
After populating the data, apply the following corrections automatically to preserve the template's design intent.
Core principle: **Read the property patterns of existing cells in the template as the "design intent," then apply those patterns to added or modified cells.**

#### 9.1 Record the Template's Design Patterns
**Before** populating data, record the following properties from each column and cell in the template:
- **Per-cell strokes**: `strokeTopWeight`, `strokeBottomWeight`, `strokeLeftWeight`, `strokeRightWeight`
- **Per-cell backgrounds**: `fills` (color, opacity)
- **Cell dimensions**: `width`, `height`, `layoutSizingVertical`, `layoutSizingHorizontal`
- **Text styles**: `fontSize`, `fontName`, `textAlignHorizontal`, `textAutoResize`, `fills` (text color)
- **Pattern variations within a column**: If properties differ between cells in the same column, this is intentional design (e.g., the last row has different strokes = hierarchical representation)

#### 9.2 Infer and Apply Cell Property Intent
When rows or columns are added, determine the properties to apply to added cells using the following rules:

**Strokes (borders)**
- Analyze the stroke pattern of each cell in the column and identify the **majority pattern** and **exception patterns**
- Exceptions are intentional design. Typical examples:
  - Only the last row (total row) has `strokeLeft=0` → hierarchical representation (the summary row is at the same level as the category)
  - Only the first row has a thicker `strokeTop` → section divider
- Determine which pattern applies to the added row based on the CSV row label or position, and apply it accordingly
  - Total / Subtotal / Sum → exception pattern (summary row)
  - Everything else → majority pattern (normal row)

**Background colors**
- Record the background color pattern for each Design variant (A/B/C/D/E, etc.) of cells and apply the same variant's background to added cells

**Text styles**
- Apply the font settings (`fontSize`, `fontName`, text color, alignment) from existing cells in the same column and same variant to the added cells' text

#### 9.3 Detect and Fix Text Overflow
- Traverse all TEXT nodes and detect whether text is wrapping
  - Detection: For nodes with `textAutoResize=HEIGHT`, check if the actual height exceeds the expected single-line height (`fontSize * 1.5`)
- If wrapping is detected, reduce the font size by 2px at a time until the text fits on a single line (minimum 10px)
- After resizing, maintain other styles such as font weight and text color

#### 9.4 Fix Layout Consistency
**Unify cell heights**
- Traverse the data cells (excluding headers) in each column, and if heights are inconsistent, unify them
- Use the template's reference height (the original cell height before text overflow correction)
- If `layoutSizingVertical: HUG` causes a cell's height to break, set it to `FIXED` and align to the reference height

**Sync span cell heights**
- If there are cells spanning multiple rows (e.g., category labels), recalculate their height based on changes in the number of spanned rows
- Target cell height = total height of the spanned rows (cell height x number of rows)
- Resize both the frame and its inner cell (`layoutSizingVertical: FILL`)

**Column width consistency**
- Verify that added columns' widths match the existing template columns, and set `layoutSizingHorizontal: FILL` if needed

#### 9.5 Handle Compound Text Nodes
- If a single cell contains multiple text nodes (e.g., value + unit like `%` or `円`):
  - Identify the value node and unit node (the unit node is a short, fixed string)
  - Separate the unit portion from the CSV value and set each in the appropriate node
  - If the value is `-` or empty, replace the unit node with a half-width space to hide it

### Step 10: Verify with a Screenshot
Use `get_screenshot` to check the result and show it to the user.

#### >> User Confirmation 3: Final Review
Present the screenshot and confirm the following:
- Whether the data has been reflected correctly
- Whether there are any display issues (text clipping, misalignment, etc.)
- Whether any corrections are needed

## CSV Format

```
year,2024/9期,,,,2025/10期,,,
month,Q1,Q2,Q3,Q4,Q1,Q2,Q3,Q4
売上,100,80,85,82,10,91,95,95
営業利益,▲52,▲50,58,53,42,42,40,40
EBITDA,124,219,6,55,9,10,14,14
```

- The first column is the row type/label
- Empty cells in the `year` row inherit the previous value
- The `month` row is optional (omitted for period-only tables)
- Any number of data rows is supported

## Fonts
When modifying text, load and use the node's existing font.
Respect the fonts used in the template as-is.

## Notes
- This skill requires Figma MCP Remote (`figma-remote`), NOT Figma MCP Desktop (`figma-desktop`). Never use `figma-desktop` tools
- The template design may vary each time. Analyze the structure dynamically to handle it
- If the CSV year row has empty cells, fill them by inheriting the previous value
- Detach only as needed (only when detaching is required for text replacement)
- When calling `use_figma`, always specify `skillNames: "figma-use"`
- If a Cell is an instance, detaching may be required before modifying text

$ARGUMENTS
