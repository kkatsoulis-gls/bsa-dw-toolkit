
---
name: "gls-branded-xlsx"
description: "Build or edit Excel workbooks for Global Lending Services (GLS) with automatic brand compliance. Use this skill whenever a user mentions a GLS spreadsheet, workbook, Excel file, tracker, data export, report, or model — even if they don't say 'use our format.' GLS branding is the default, not opt-in. Trigger on any of: 'GLS spreadsheet', 'GLS workbook', 'GLS Excel', 'GLS tracker', 'GLS report', 'GLS data export', 'Global Lending Services Excel', 'build a tracker for GLS', 'make an Excel file for GLS', or any request that combines spreadsheet/workbook/Excel/tracker/report with GLS or Global Lending Services in any phrasing."
---
 
# GLS Branded Excel
 
Always read and follow the `xlsx` skill for all mechanics (openpyxl patterns, recalc.py, formula gotchas, etc.). This skill **overrides the xlsx skill's design defaults** — use GLS brand instead of generic formatting or Arial/Times typography.
 
> **Sample baseline:** Two confirmed reference workbooks (a single-column list report and a multi-sheet expense report template). Multi-column table styling is now confirmed. Charts and conditional formatting rules below remain reasonable defaults. Flag assumptions marked ⚠️ to the user; confirmed patterns have no flag.
 
---
 
## STEP 1 — REQUIRED: Determine Layout Before Writing Any Code
 
**This step is mandatory. Do not skip it. Do not start building until the layout is confirmed.**
 
GLS Excel workbooks come in three distinct layouts. Each uses different structure, borders, and row organization. Choosing the wrong one produces an incorrectly formatted file. Determine the layout from the request and data first; ask if it is not clear.
 
### Layout Decision Tree
 
```
Does the request describe a form someone fills out?
  Keywords: "form", "template", "fill out", "submit", "reimbursement",
            "request", "application", "approval", labeled fields with blanks
  → Use FORM layout
 
Does the request describe rows of records, exports, or a scrollable list?
  Keywords: "tracker", "export", "list", "log", "data", "pipeline",
            "report" (with columnar data), structured rows of the same type
  → Use DATA TABLE layout
 
Does the request describe calculations, projections, or financial models?
  Keywords: "model", "budget", "forecast", "P&L", "projection",
            "variance", "scenario", formulas across periods
  → Use FINANCIAL MODEL layout
 
None of the above clearly apply, OR the request could fit more than one?
  → ASK — see "When to Ask" below
```
 
### When to Ask
 
If the layout is not obvious from the prompt and any data provided, **stop and ask before building.** Use this exact question:
 
> "Before I build this, I want to make sure the formatting matches how it will be used. Should this be:
> - A **form** people fill out (labeled fields, input cells, like an expense report)?
> - A **data table / tracker** (rows of records with a cyan header, like a list or export)?
> - A **financial model** (formulas, projections, budget rows)?
>
> Or describe how it will be used and I'll pick the right layout."
 
**Do not guess and proceed.** An incorrectly formatted workbook requires a full rebuild.
 
### Layout Summary
 
| Layout | Trigger | Key structure |
|---|---|---|
| **Form** | Fill-out template, submission, request | Header info block + labeled fields + yellow input cells + section labels + totals row |
| **Data Table** | Tracker, export, list, log, data | Logo → title → cyan header row → alternating data rows → timestamp |
| **Financial Model** | Budget, forecast, P&L, model | Blue input cells + black formula cells + period columns + bold totals rows |
 
Once the layout is confirmed, proceed to the matching section below.
 
---
 
## CRITICAL: Output File Path Length
 
The outputs folder path is already ~189 characters. Windows cannot open files with paths over 259 characters.
 
**Rules — always follow:**
- Save the final `.xlsx` directly in the outputs folder root — **never in a subdirectory**
- Use a **short filename of 15 characters or fewer** (e.g., `gls.xlsx`, `tracker.xlsx`, `export.xlsx`)
- Abbreviate aggressively: `q3loans.xlsx` not `q3_loan_performance_export.xlsx`
---
 
## Color Palette (Excel-specific)
 
This palette is **distinct** from both the PPTX brand (`#0065AC`) and the Word brand (navy). Use only these values.
 
| Role | Hex | Source | Usage |
|---|---|---|---|
| **Header Row Fill** | `049DDD` | Confirmed (list report) | All table column header rows — bright cyan/blue |
| **Header Row Text** | `FFFFFF` | Confirmed (list report) | White, bold |
| **Body Text** | `000000` | Confirmed (both samples) | Standard black body cells |
| **Section Label Text** | `222222` | Confirmed (expense template) | Section/category label rows — near-black |
| **Input Cell Fill** | `FFFF00` | Confirmed (expense template, B6) | Yellow highlight for user-editable input cells |
| **Semantic Red** | `FF0000` | Confirmed (expense template styles) | Data callout — negative/error/bad only |
| **Alt Row Shading** | `F2F2F2` | ⚠️ Assumed | Light gray alternating rows — not confirmed, reasonable default |
| **Border Color** | `CCCCCC` | Confirmed (expense template) | Thin borders on data cells |
| **Semantic Green** | `00B050` | ⚠️ Assumed (PPTX carryover) | Conditional formatting — positive callouts only |
 
**Not brand colors:** Do not use `#0065AC` (PPTX blue) in Excel workbooks.
Red/Green are **semantic data-callout only** — never decorative fills.
 
**Theme colors confirmed from expense template** (Excel default Office 2007 theme — do not override):
`dk2: #1F497D` · `accent1: #4F81BD` · `accent5: #4BACC6` — these appear in the theme XML but are not used decoratively; keep them as Excel defaults.
 
---
 
## Typography
 
Confirmed from both samples:
 
- **Font:** `Calibri` — confirmed primary font across both workbooks
- **Fallback:** `Arial` — confirmed as fallback in expense template (Dealers sheet)
- **Title / report heading:** Bold, black (`000000`), 16pt (confirmed from expense template `D1`)
- **Section labels:** Bold, black, 11pt (confirmed: "Training Expenses (a):", "Travel and Other Expenses:")
- **Header row:** Bold, white (`FFFFFF`), 11pt, on cyan fill (`049DDD`)
- **Body cells:** Regular, black, 11pt
- **Footer / timestamp / notes:** Regular, 10–11pt
- **Do NOT use** Franklin Gothic (PPTX only)
---
 
## Logo Assets
 
Bundled in this skill's `assets/` folder — **never use a placeholder rectangle.**
 
| File | px | Source | Use |
|---|---|---|---|
| `assets/gls-logo-expense.jpg` | 669×240 | Confirmed (expense template) | **Preferred for Excel** — confirmed in-use asset |
| `assets/gls-logo-color-lockup.jpg` | 1516×472 | Confirmed (PPTX) | High-res fallback if expense logo unavailable |
| `assets/gls-logo-grayscale-lockup.png` | 749×241 | Confirmed (PPTX) | Optional: additional sheets only |
 
**Confirmed placement from expense template:**
- Anchored at cell A1 (with slight x-offset ~0.36")
- Rendered at ~2.5" wide × 0.9" tall
- Spans rows 1–3 (TwoCellAnchor, rows 0–3 in 0-indexed EMU)
- `editAs="oneCell"` — stays anchored to top-left, scales with column/row changes
```python
import os
from openpyxl.drawing.image import Image as XLImage
 
SKILL_DIR = os.path.dirname(os.path.abspath(__file__))
logo_path = os.path.join(SKILL_DIR, 'assets', 'gls-logo-expense.jpg')
 
logo = XLImage(logo_path)
# Confirmed dimensions from expense template (669×240 native, displayed at 2.5"×0.9")
logo.width  = 180   # pixels at screen resolution — tune to ~2.5" display width
logo.height = 65
logo.anchor = 'A1'
ws.add_image(logo)
```
 
- Never stretch, recolor, or rotate the logo.
- Place only on the **first (summary) sheet** by default.
- After placing, set row heights for rows 1–3. Do not freeze any rows.
---
 
## Sheet Structure (confirmed from two samples)
 
### First / Summary Sheet
 
```
Row 1–3:  Logo area — rows 1-3 reserved, logo anchored at A1 (~0.36" x offset)
          Row heights ~16.5–21pt (confirmed from expense template)
Row 4:    Report title — Bold, 16pt, black (e.g., "Expense Reimbursement Form - 2026")
Row 5–7:  Header info block — name, employee number, department, date fields
          Use thin underline borders on label+value rows (confirmed: bottom:thin)
          Input cells: yellow fill (FFFF00) on user-editable fields
Row 8:    Section divider / blank
Row 9:    Column headers — cyan fill (049DDD), white bold text
Row 10+:  Data rows — plain, thin-bordered, section labels in bold
Last row: Timestamp — date/time right-aligned, 10pt (e.g., "9/1/2026 8:00:49 AM")
```
 
**Header info block borders (confirmed from expense template):**
- Name/Signature row: `bottom: thin` only (thin underline, no box)
- Employee Number/Date row: `top: thin` + `bottom: thin` (thin lines above and below)
- Department/Approval row: `bottom: thin` only
**Date/expense entry row (confirmed):** `medium` border all sides on the date input cell and the "Total" column header cell.
 
**Logo area row height setup:**
```python
ws.row_dimensions[1].height = 21
ws.row_dimensions[2].height = 16.5
ws.row_dimensions[3].height = 16.5
# Do NOT set freeze_panes — rows are never frozen in GLS workbooks
```
 
### Additional Sheets (confirmed from Dealers sheet)
 
- No logo — confirmed: Dealers sheet has no image
- Simple bold section headers, no fill
- Plain thin-bordered data rows
- Tab names: short, descriptive (`Summary`, `Dealers`, `Raw Data`)
- ⚠️ Tab color `049DDD` is an assumed default (not confirmed in expense template); apply unless file uses a different convention
---
 
## STEP 2 — Build the Confirmed Layout
 
Use only the section matching the layout chosen in Step 1.
 
### FORM Layout — Build Guide
 
Structure (confirmed from expense template):
```
Rows 1–3:   Logo area (logo anchored A1, row heights 21/16.5/16.5)
Row 4:      Report title — Bold 16pt black
Rows 5–8:   Header info block (Name, Employee #, Department, Date)
              Label cells: bold, thin bottom border
              Input cells: yellow fill FFFF00, thin top+bottom border
Row 9:      Blank spacer
Row 10:     Date/period entry — medium box border on key cells
Rows 11+:   Section labels (bold, no fill) + line-item rows (plain, thin borders)
              Totals column: medium box border, right-aligned
Last rows:  Footer notes (plain text, wrapped)
            Timestamp (10pt, gray, right-aligned)
```
 
Key style rules for forms:
- User-editable input cells → yellow fill `FFFF00` + thin top/bottom borders
- Field label cells → bold, thin bottom border only (underline style)
- Section category labels → bold black, no fill, height 17
- Key entry / totals cells → medium box border
- See "Form / Input Fields" section below for full openpyxl code
### DATA TABLE Layout — Build Guide
 
Structure (confirmed from list-report sample):
```
Rows 1–3:   Logo area (logo anchored A1, row heights 21/16.5/16.5)
Row 4:      Report title — Bold 16pt black, merged across columns
Row 5:      Instructional note box — thin black border, no fill, wrapped text
Row 6:      Blank spacer
Row 7:      Column header row — cyan fill 049DDD, white bold Calibri 11pt, height 20
Rows 8+:    Data rows — white / F2F2F2 alternating, CCCCCC thin borders
            Freeze panes at A8
Last row:   Timestamp (10pt, gray, right-aligned, bottom-right cell)
```
 
Key style rules for data tables:
- All headers: `style_header_row()` (cyan fill, white bold text)
- All data rows: `style_data_row()` (alternating white/gray, thin borders)
- Column widths: auto-fit to content (min 10, max 40)
- See "Table Styling" section below for full openpyxl code
### FINANCIAL MODEL Layout — Build Guide
 
> ⚠️ Not confirmed from GLS samples — apply these as reasonable defaults and confirm with user.
 
Structure:
```
Rows 1–3:   Logo area (same as other layouts)
Row 4:      Model title — Bold 16pt black
Rows 5–8:   Assumptions block — blue input cells (0000FF text), yellow fill FFFF00
Row 9:      Blank spacer
Row 10:     Column headers — cyan fill 049DDD, white bold, period labels (e.g. "FY2026", "FY2027")
Rows 11+:   Model rows — category labels bold, formula cells black text, no special fill
            Subtotal rows: bold + thin top border
            Grand total rows: bold + medium top border
Last row:   Timestamp
```
 
Key style rules for financial models:
- Hardcoded input cells → blue text `0000FF`, yellow fill `FFFF00`
- Formula cells → black text `000000`, no fill
- Subtotal/total rows → bold; grand totals use medium top border
- Currency: `$#,##0` format; percentages: `0.0%` stored as fractions
- Never hardcode computed values — always use `=SUM(...)` etc.
---
 
## Form / Input Fields (confirmed from expense template)
 
When building forms or templates with user-editable cells:
 
```python
from openpyxl.styles import PatternFill, Border, Side, Font, Alignment
 
# Input cell: yellow fill, thin borders top/bottom
YELLOW_FILL = PatternFill('solid', fgColor='FFFF00')
THIN_BOTTOM = Border(bottom=Side(style='thin', color='000000'))
THIN_TOP_BOTTOM = Border(
    top=Side(style='thin', color='000000'),
    bottom=Side(style='thin', color='000000')
)
MEDIUM_BOX = Border(
    left=Side(style='medium', color='000000'),
    right=Side(style='medium', color='000000'),
    top=Side(style='medium', color='000000'),
    bottom=Side(style='medium', color='000000'),
)
 
# Field label + underline (confirmed pattern)
ws['B5'].value = 'Name:'
ws['B5'].font = Font(bold=True, name='Calibri', size=11)
ws['B5'].border = THIN_BOTTOM
 
# Input cell with yellow fill (confirmed for employee number)
ws['B6'].fill = YELLOW_FILL
ws['B6'].border = THIN_TOP_BOTTOM
 
# Date/totals entry cell — medium box (confirmed pattern)
ws['C10'].border = MEDIUM_BOX
```
 
---
 
## Table Styling (openpyxl)
 
Apply consistently to every data table:
 
```python
from openpyxl.styles import PatternFill, Font, Border, Side, Alignment
 
CYAN       = PatternFill('solid', fgColor='049DDD')
GRAY_ALT   = PatternFill('solid', fgColor='F2F2F2')   # ⚠️ assumed
WHITE_FILL = PatternFill('solid', fgColor='FFFFFF')
THIN_BORDER = Border(
    left=Side(style='thin',   color='CCCCCC'),
    right=Side(style='thin',  color='CCCCCC'),
    top=Side(style='thin',    color='CCCCCC'),
    bottom=Side(style='thin', color='CCCCCC'),
)
MEDIUM_BOX = Border(
    left=Side(style='medium',   color='000000'),
    right=Side(style='medium',  color='000000'),
    top=Side(style='medium',    color='000000'),
    bottom=Side(style='medium', color='000000'),
)
 
def style_header_row(ws, row_num, col_count):
    for col in range(1, col_count + 1):
        cell = ws.cell(row=row_num, column=col)
        cell.fill = CYAN
        cell.font = Font(bold=True, color='FFFFFF', name='Calibri', size=11)
        cell.border = THIN_BORDER
        cell.alignment = Alignment(horizontal='center', vertical='center', wrap_text=True)
    ws.row_dimensions[row_num].height = 20
 
def style_data_row(ws, row_num, col_count, is_alt=False):
    fill = GRAY_ALT if is_alt else WHITE_FILL
    for col in range(1, col_count + 1):
        cell = ws.cell(row=row_num, column=col)
        cell.fill = fill
        cell.font = Font(name='Calibri', size=11, color='000000')
        cell.border = THIN_BORDER
 
def style_section_label(ws, row_num, label):
    """Bold section label row (e.g. 'Training Expenses (a):')"""
    cell = ws.cell(row=row_num, column=2, value=label)
    cell.font = Font(bold=True, name='Calibri', size=11, color='000000')
    ws.row_dimensions[row_num].height = 17
 
def style_totals_cell(ws, row_num, col_num):
    """Medium-bordered totals/summary cell"""
    cell = ws.cell(row=row_num, column=col_num)
    cell.border = MEDIUM_BOX
    cell.font = Font(bold=True, name='Calibri', size=11)
```
 
---
 
## Timestamp (confirmed from expense template)
 
```python
from datetime import datetime
last_row = ws.max_row + 2
ts_cell = ws.cell(row=last_row, column=ws.max_column)
# Format matches confirmed sample: "9/1/2026 8:00:49 AM"
ts_cell.value = datetime.now().strftime('%-m/%-d/%Y %-I:%M:%S %p')
ts_cell.font = Font(name='Calibri', size=10, color='888888')
ts_cell.alignment = Alignment(horizontal='right')
```
 
---
 
## Chart Styling (⚠️ assumed defaults — not confirmed from either sample)
 
- **Lead series:** `049DDD` (cyan — matches header row)
- **Comparison series:** neutral grays (`888888`, `AAAAAA`, `CCCCCC`)
- **Semantic series:** `00B050` green (positive), `FF0000` red (negative/bad)
- **Chart title font:** Calibri, bold, 12pt
- **Axis label font:** Calibri, 10pt
- **No 3D charts.** Flat bar, line, and pie only.
- **Legend:** below chart or right side — never overlapping data
---
 
## Conditional Formatting (⚠️ assumed — confirm before applying)
 
```python
from openpyxl.formatting.rule import CellIsRule
from openpyxl.styles import PatternFill
 
red_fill   = PatternFill('solid', fgColor='FF0000')
green_fill = PatternFill('solid', fgColor='00B050')
 
ws.conditional_formatting.add(f'B10:B{last_row}',
    CellIsRule(operator='lessThan',    formula=['0'], fill=red_fill))
ws.conditional_formatting.add(f'B10:B{last_row}',
    CellIsRule(operator='greaterThan', formula=['0'], fill=green_fill))
```
 
Only add when explicitly requested or obviously called for by data type.
 
---
 
## Column Width Defaults
 
Confirmed widths from expense template as reference:
 
| Column role | Confirmed width |
|---|---|
| Narrow index / spacer | ~5.5 |
| Label / description (wide) | ~38.8 |
| Data / value column | ~15–16.5 |
| Total / summary column | ~13 |
| Notes / GL Account | ~22.5 |
 
```python
# Auto-fit to content (min 10, max 40) — override with above reference widths for form-style sheets
for col in ws.columns:
    max_len = max((len(str(cell.value)) for cell in col if cell.value), default=8)
    ws.column_dimensions[col[0].column_letter].width = min(max(max_len + 2, 10), 40)
```
 
---
 
## QA Checklist
 
- [ ] **Layout was confirmed** (from request signals or by asking) before any code was written
- [ ] Layout-specific structure matches the chosen layout (Form / Data Table / Financial Model)
- [ ] `.xlsx` saved with short name (≤15 chars) directly in outputs root — no subdirectories
- [ ] GLS logo (`gls-logo-expense.jpg`) anchored at A1, not stretched or recolored
- [ ] Logo spans rows 1–3; row heights set (~21/16.5/16.5); no freeze_panes set
- [ ] Report title: Bold 16pt black
- [ ] Input cells (user-editable): yellow fill `FFFF00`
- [ ] Field label rows: thin bottom border only; Employee Number-style rows: thin top+bottom
- [ ] Key input/total cells: medium box border
- [ ] All table headers: cyan fill `049DDD`, white bold Calibri 11pt
- [ ] Section label rows: bold, black, no fill
- [ ] Timestamp at bottom: 10pt, right-aligned
- [ ] Font is Calibri throughout (Arial fallback only)
- [ ] No `0065AC` (PPTX blue) in any cell fill
- [ ] `recalc.py` passes (if workbook contains formulas)
- [ ] Column widths set (not all default)
---
 
## Quick Reference
 
```
Logo:           gls-logo-expense.jpg, A1, width=180px height=65px, rows 1-3
Title:          Bold 16pt black, row 4
Input cells:    FFFF00 yellow fill + thin top/bottom borders
Field labels:   Bold 11pt, thin bottom border
Key cells:      Medium box border (date entry, totals)
Header row:     Fill=049DDD, Font=FFFFFF bold Calibri 11pt, height=20
Section labels: Bold 11pt black, no fill, height=17
Data rows:      White / F2F2F2⚠️ alternating, CCCCCC thin borders, Calibri 11pt
Timestamp:      Bottom-right, 10pt Calibri, right-aligned
Totals row:     Bold, medium box borders
Semantic:       FF0000 red (error/bad) ✓ confirmed · 00B050 green (good) ⚠️ assumed
```
 
