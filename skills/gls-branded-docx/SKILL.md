---
name: "gls-branded-docx"
description: "Build or edit Word documents for Global Lending Services (GLS) with automatic brand compliance. Use this skill whenever a user mentions a GLS Word document, report, memo, requirements doc, write-up, documentation, or notes — even if they don't say 'use our format.' GLS branding is the default, not opt-in. Trigger on any of: 'GLS document', 'GLS report', 'GLS memo', 'GLS requirements doc', 'GLS write-up', 'GLS documentation', 'GLS Word doc', 'GLS .docx', 'Global Lending Services document', 'write up for GLS', 'document for GLS', 'create a doc for GLS', or any request that combines document/report/memo/write-up/notes with GLS or Global Lending Services in any phrasing. Important: this is the Word document brand — it uses a distinct style derived from real GLS published documents, NOT the PowerPoint brand (#0065AC / Franklin Gothic Medium from gls-branded-pptx)."
---
 
# GLS Branded Word Document Skill
 
Always read and follow the `docx` skill for all creation/editing mechanics (docx-js gotchas, rendering, validate.py, etc.). This skill **overrides the docx skill's design defaults** with the real GLS document standard, derived from analyzing official GLS published documents (Clean Desk Policy, Data Protection Policy, AI Governance Policy) and confirmed informal examples.
 
---
 
## Required Clarifying Question
 
**Before generating any document**, always ask:
 
> "Is this a **formal document** (Article-style sections, Times New Roman, centered logo, policy metadata) or an **informal document** (Calibri, blue headings, running header logo, lighter structure)?"
 
The answer determines everything — font, layout, heading style, footer, logo placement.
 
---
 
## ⚠️ Logo
 
**Always use the formal GLS logo:** `assets/gls-logo-formal.jpg`
 
This is the correct current logo — the blue/gray GLS globe lockup with "GLOBAL LENDING SERVICES". Do not use the grayscale or color-lockup PNG alternatives unless the user explicitly requests them.
 
---
 
---
# FORMAL DOCUMENT STANDARD
---
 
*Derived from: Clean Desk Policy, Data Protection Policy, AI Governance Policy*
 
## Formal — Visual Summary
 
- Logo large, centered, top of page 1 only
- Title centered, bold, ALL CAPS, black Times New Roman
- Headings: "Article I. Purpose", "Article II. Scope" — Times New Roman, large, black, not bold
- Body: Times New Roman 12pt, fully justified
- No running header on content pages
- Footer: 3-line GLS proprietary footer, page number right-aligned
- Clean white pages — no color blocks, no callout boxes, no shading
## Formal — Color Palette
 
| Role | Hex | Notes |
|---|---|---|
| All text | `#000000` | Black throughout — no colored headings |
| Table header fill | `#D6E4F0` | Light blue — only color element permitted |
| Table alt row | `#EBF3FB` | Light blue tint |
 
## Formal — Typography
 
| Element | Font | Size | Style |
|---|---|---|---|
| Title | Times New Roman | 12pt | Bold, ALL CAPS, centered, black |
| Heading 1 | Times New Roman | 16pt | Not bold, black — "Article I.   Purpose" |
| Heading 2 | Times New Roman | 12pt | Underlined, black — "Section IV.1   Label" |
| Body | Times New Roman | 12pt | Black, fully justified (`AlignmentType.BOTH`) |
| Footer | Times New Roman | 10pt | Bold lines 1–2, normal line 3 |
 
## Formal — Logo Placement
 
```js
// Large, centered, top of page 1 — no running header
new Paragraph({
  alignment: AlignmentType.CENTER,
  spacing: { before: 0, after: 200 },
  children: [new ImageRun({ data: formalLogo, type: 'jpg', transformation: { width: 210, height: 68 } })],
})
```
 
## Formal — Page Setup
 
```js
page: {
  size: { width: 12240, height: 15840 },   // US Letter
  margin: { top: 1440, right: 1350, bottom: 1440, left: 1440, header: 720, footer: 342 },
}
```
 
## Formal — Header & Footer
 
**Header:** Empty — real GLS formal docs have NO running header.
 
```js
function emptyHeader() {
  return new Header({ children: [new Paragraph({ children: [] })] });
}
```
 
**Footer:** 3 lines, separated by a thin top border. Bold lines 1–2, normal line 3 with right-tab page number.
 
```js
new Footer({
  children: [
    new Paragraph({
      border: { top: { style: BorderStyle.SINGLE, size: 6, color: '000000' } },
      spacing: { before: 80 },
      children: [new TextRun({ text: 'Global Lending Services LLC', bold: true, size: 20, font: 'Times New Roman' })],
    }),
    new Paragraph({
      children: [new TextRun({ text: 'This document is proprietary.  Do not reproduce without permission.', bold: true, size: 20, font: 'Times New Roman' })],
    }),
    new Paragraph({
      tabStops: [{ type: TabStopType.RIGHT, position: TabStopPosition.MAX }],
      children: [
        new TextRun({ text: 'Page ', size: 20, font: 'Times New Roman' }),
        new TextRun({ children: [PageNumber.CURRENT], size: 20, font: 'Times New Roman' }),
        // Optionally: " of " + PageNumber.TOTAL_PAGES
      ],
    }),
  ],
})
```
 
## Formal — Document Structure
 
```
[LOGO]         Large color logo, centered ~210px wide
[TITLE]        "DOCUMENT TITLE IN ALL CAPS"  — bold, centered, Times New Roman 12pt
[blank space]
Article I.   Purpose
[body text — justified]
 
Article II.   Scope
[body text]
 
Article III.  Overview
[body text]
 
Article IV.   Statement of Policy / Proposal
  Section IV.1   Subsection Name      ← underlined
  [body text]
  Section IV.2   Next Subsection
  [body text]
 
Article V.    Recommendation
Article VI.   Document Review and Approval
[review table]
```
 
## Formal — Tables
 
```js
// Header row: fill #D6E4F0, bold black text, centered
// Body rows: alternate white / #EBF3FB
// All borders: thin black lines (BorderStyle.SINGLE, size 6, color '000000')
// Font: Times New Roman, 11pt
```
 
## Formal — What NOT to Do
 
- ❌ No colored headings (blue, navy, gray)
- ❌ No running header with logo on every page
- ❌ No callout boxes (blue info box, amber warning box)
- ❌ No cover page with navy bars
- ❌ No gray title text
- ❌ No Calibri font
- ❌ No `#1F3864`, `#2E74B5`, or any colored text
---
---
# INFORMAL DOCUMENT STANDARD
---
 
*Derived from: lsr_informal.docx (confirmed user example)*
 
## Informal — Visual Summary
 
- Small logo top-left in running header on every page
- Large bold navy title, left-aligned, Calibri
- Headings: plain section names, navy `#1F3864`, bold, Calibri
- Body: Calibri 11pt, left-aligned
- Optional callout boxes (info blue, amber warning)
- Footer: simple "Doc Name | Page X" right-aligned, gray, Calibri
## Informal — Color Palette
 
| Role | Hex | Notes |
|---|---|---|
| Title | `#1F3864` | Dark navy, bold |
| Heading 1 | `#1F3864` | Dark navy, bold — confirmed from lsr_informal |
| Heading 2 | `#1F3864` | Dark navy, bold |
| Subtitle / gray text | `#595959` | Italic gray subtitle and footer text |
| Body text | `#000000` | Black |
| Info callout bg | `#D9E2F3` | Light blue |
| Info callout text | `#1F3864` | Navy |
| Warning callout bg | `#FFF2CC` | Amber |
| Warning callout text | `#7F6000` | Dark gold |
| Table header fill | `#1F3864` | Dark navy, white text |
| Table alt row | `#F2F2F2` | Light gray |
 
> **Note:** Heading color is `#1F3864` (dark navy) — NOT `#2E74B5` (medium blue). Confirmed from the user's updated informal example.
 
## Informal — Typography
 
| Element | Font | Size | Style |
|---|---|---|---|
| Title | Calibri | 26pt | Bold, `#1F3864` |
| Subtitle | Calibri | 11pt | Italic, `#595959` |
| Metadata line | Calibri | 10–11pt | Bold labels + plain values |
| Heading 1 | Calibri | 16pt | Bold, `#1F3864` |
| Heading 2 | Calibri | 13pt | Bold, `#1F3864` |
| Body | Calibri | 11pt | Black, left-aligned |
| Header text | Calibri | 9pt | Gray `#595959`, italic |
| Footer text | Calibri | 9pt | Gray `#595959` |
 
## Informal — Logo Placement
 
Small logo in the **running header**, top of every page. Left-aligned. No large logo on page 1.
 
```js
// Logo original size: 3.17" x 0.78" — aspect ratio 4.06:1
// Confirmed user-preferred size in Word: 1.19" x 0.29" = 114x28px at 96dpi
// Always calculate height as: width / 4.06
// e.g. width=114 → height=28 ✅ (confirmed), width=90 → height=22, width=130 → height=32
function makeHeader(docTitle) {
  return new Header({
    children: [new Paragraph({
      children: [
        new ImageRun({ data: formalLogo, type: 'jpg', transformation: { width: 114, height: 28 } }),
        // Optionally add right-aligned doc title text via tab stop
      ],
    })],
  });
}
```
 
## Informal — Page Setup
 
```js
page: {
  size: { width: 12240, height: 15840 },
  margin: { top: 1080, right: 1080, bottom: 1080, left: 1080, header: 705, footer: 705 },
}
// Tighter margins (0.75") vs formal (1.0")
```
 
## Informal — Header & Footer
 
**Header:** Small logo left + optional italic gray doc title right. Thin bottom border.
 
**Footer:** Simple one-line, gray, Calibri 9pt. Format: `"[Team] — [Doc Name]  |  Page X"` right-aligned.
 
```js
new Footer({
  children: [new Paragraph({
    border: { top: { style: BorderStyle.SINGLE, size: 4, color: 'BFBFBF' } },
    spacing: { before: 60 },
    tabStops: [{ type: TabStopType.RIGHT, position: TabStopPosition.MAX }],
    children: [
      new TextRun({ text: 'GLS Auto — [Doc Name]', size: 18, color: '595959', font: 'Calibri' }),
      new TextRun({ text: '\t', size: 18, font: 'Calibri' }),
      new TextRun({ text: 'Page ', size: 18, color: '595959', font: 'Calibri' }),
      new TextRun({ children: [PageNumber.CURRENT], size: 18, color: '595959', font: 'Calibri' }),
    ],
  })],
})
```
 
## Informal — Document Structure
 
```
[HEADER: small logo every page]
 
[Large bold navy title]           ← 26pt, Calibri, #1F3864
[thin blue rule line]
[Italic gray subtitle]
Date: ___   Owner: ___
 
[Optional: "In short" info callout box]
 
Current Process / [Section Name]       ← H1: bold, #1F3864
[body text]
 
[Optional: "Risk" or "Note" warning callout]
 
What's Changing / [Next Section]       ← H1
  Subsection Name                      ← H2: bold, #1F3864, smaller
  [body text]
 
[Tables as needed]
 
[Footer: gray, right-aligned, every page]
```
 
## Informal — Callout Boxes
 
Use a single-cell borderless table with background fill. Permitted in informal docs (not in formal).
 
```js
// Info / summary callout (light blue)
// Background: #D9E2F3, label bold navy #1F3864, body navy #1F3864
 
// Warning / open-item callout (amber)
// Background: #FFF2CC, label bold #7F6000, body #7F6000
```
 
## Informal — Tables
 
```js
// Header row: fill #1F3864 (dark navy), white bold text
// Alternating body rows: #F2F2F2 / white
// Borders: thin gray (BFBFBF) bottom border on rows, no outer borders
// Font: Calibri 10pt
```
 
---
 
## docx-js Color Note
 
No `#` prefix — 6 hex digits only:
- ✅ `color: '1F3864'`
- ❌ `color: '#1F3864'`
---
 
## Output File Path
 
- Save to `/mnt/user-data/outputs/` root — no subdirectories
- Filename ≤ 20 characters
---
 
## QA Checklist — Formal
 
- [ ] Asked formal vs. informal before starting
- [ ] Logo: `gls-logo-formal.jpg`, centered, ~210px wide, page 1 only
- [ ] Empty header (no running header)
- [ ] 3-line GLS footer with thin top border, page number right-tab
- [ ] Title: Times New Roman, 12pt, bold, ALL CAPS, centered, black
- [ ] H1: Times New Roman, 16pt, NOT bold, black — "Article I.   Purpose"
- [ ] H2: Times New Roman, 12pt, underlined, black — "Section IV.1   Label"
- [ ] Body: Times New Roman 12pt, black, fully justified
- [ ] No colored headings, no callout boxes, no cover page
- [ ] Tables: light blue header `#D6E4F0`, alternating `#EBF3FB`/white
- [ ] US Letter, 1" margins
- [ ] Filename ≤ 20 chars, saved to outputs root
- [ ] Rendered with soffice → pdftoppm → visually verified
## QA Checklist — Informal
 
- [ ] Asked formal vs. informal before starting
- [ ] Logo: `gls-logo-formal.jpg`, small (~90px wide) in running header, every page
- [ ] Title: Calibri 26pt, bold, `#1F3864`, left-aligned
- [ ] Thin blue rule after title
- [ ] Italic gray subtitle + metadata line (Date / Owner)
- [ ] H1: Calibri 16pt, bold, `#1F3864` — plain section names (no "Article" prefix)
- [ ] H2: Calibri 13pt, bold, `#1F3864`
- [ ] Body: Calibri 11pt, black, left-aligned
- [ ] Footer: gray Calibri 9pt, "Team — Doc Name  |  Page X", right-aligned
- [ ] Callout boxes optional — info (blue `#D9E2F3`) or warning (amber `#FFF2CC`)
- [ ] Tables: navy `#1F3864` header, alternating `#F2F2F2`/white rows
- [ ] Margins 0.75" (1080 DXA) all sides
- [ ] No Article-style headings, no Times New Roman, no justified body text
- [ ] Filename ≤ 20 chars, saved to outputs root
- [ ] Rendered with soffice → pdftoppm → visually verified
 
