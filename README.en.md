[日本語](./README.md) | **English**

# Figma Chart Skills for Claude Code

A Claude Code plugin that populates Figma bar chart and table templates with spreadsheet data (CSV/Excel/Google Sheets).

You want your charts to look beautiful in Figma — but updating numbers and adjusting bar heights is painfully tedious. AI handles that for you. Financial reports, IR materials, team rosters — the use cases are wide open. This plugin bridges spreadsheets and Figma template components, automating everything from data population to design correction by reading the template's design intent. Figma templates can stay minimal, and data values are populated faithfully as-is.

## Skills

### `figma-bar-chart`

**CSV → Bar Chart**

Populates Figma bar chart templates with CSV data.

- **Simple** — Single-value bar chart
- **Stacked** — Stacked bar chart
- **Parallel** — Grouped bar chart (2 values)

Chart type is specified in the last row of the CSV or auto-detected from the number of value columns.

### `figma-table`

**CSV → Table**

Populates Figma table templates with CSV data. Since the template design may vary each time, the structure is dynamically analyzed.

- Row and column counts adapt to the CSV
- Header hierarchy (period > quarter, period only, etc.) is dynamically analyzed
- Automatically detects style regions in the template and maps them to CSV column groups
- Reads the template's design intent and auto-corrects when adding rows/columns

## Prerequisites

- [Claude Code](https://claude.com/claude-code) installed
- [Figma MCP Server](https://help.figma.com/hc/en-us/articles/32132100833559-Guide-to-the-Figma-MCP-Server) connected (`use_figma`, `get_screenshot`, etc. — not Figma MCP Desktop)
- Design templates (components) prepared in Figma

## Installation

### Quick start (single session)

```bash
git clone https://github.com/tikeda/figma-chart-skill.git
claude --plugin-dir ./figma-chart-skill
```

### Permanent install

Inside a Claude Code session:

```
/plugin marketplace add tikeda/figma-chart-skill
/plugin install figma-bar-chart@figma-chart-skill
```

## Usage

```bash
# Provide the CSV file path and Figma template URL
figma-table samples/table.csv https://www.figma.com/design/xxx/yyy?node-id=47-658
```

You can drag and drop the CSV file from Finder into the terminal. The template URL, guide settings, and other options are confirmed interactively.

## How It Works

### figma-bar-chart (11 steps)

| Step | Description | Confirm |
|------|------|:------:|
| 1 | Get CSV file | yes |
| 2 | Read and parse CSV | yes |
| 3 | Determine chart type | yes |
| 4 | Get Figma template URL | yes |
| 5 | Get guide settings (max value, interval) | yes |
| 6 | Instantiate template | |
| 7 | Detach all levels | |
| 8 | Add guides and set labels | |
| 9 | Populate data (calculate bar heights) | |
| 10 | Set labels (month/year display control) | |
| 11 | Verify with screenshot | yes |

### figma-table (10 steps)

| Step | Description | Confirm |
|------|------|:------:|
| 1 | Get CSV file | yes |
| 2 | Read and parse CSV | yes |
| 3 | Get Figma template URL | yes |
| 4 | Analyze template structure | yes |
| 5 | Map column groups to style regions | |
| 6 | Instantiate template | |
| 7 | Adjust row/column counts | |
| 8 | Populate data | |
| 9 | Auto-correct design | |
| 10 | Verify with screenshot | yes |

## Design Philosophy

### Designers Keep the Reins

This Skill aims to **have AI work within the templates you've built**, not replace them. Designers keep the reins — AI handles the tedious parts.

### Minimal Templates, Skill-Driven Correction

Figma templates define only the design skeleton. Adjustments for row/column differences, text overflow, and stroke consistency are automatically corrected by the Skill. It reads the property patterns of existing cells in the template as "design intent" and applies those patterns to added or modified cells.

### CSV as the Source of Truth

All data processing and calculation is done on the CSV side. The Skill populates CSV values as-is, respecting notations like `▲` and `-` exactly as they appear in the CSV.

## Sample Data

| File | Purpose |
|------|------|
| `samples/simple.csv` | Simple bar chart |
| `samples/stacked.csv` | Stacked bar chart |
| `samples/parallel.csv` | Grouped bar chart |
| `samples/table.csv` | Table (quarterly matrix) |
| `samples/table2.csv` | Table (business segment) |
| `samples/table3.csv` | Table (project member list) |
| `samples/table4.csv` | Table (pricing plan comparison) |
| `samples/Figma-Chart_Sample.fig` | Figma design template |
