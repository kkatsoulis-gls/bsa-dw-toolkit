---
name: "gls-branded-pptx"
description: "Build or edit PowerPoint decks for Global Lending Services (GLS) with automatic brand compliance. Use this skill whenever a user mentions a GLS presentation, deck, slides, pitch deck, PowerPoint, or any visual deliverable for GLS — even if they don't say \"use our brand.\" GLS branding is the default, not opt-in. Trigger on any of: \"GLS deck\", \"GLS slides\", \"GLS presentation\", \"Global Lending Services PowerPoint\", \"make slides for GLS\", \"reformat to GLS brand\", \"GLS pitch deck\", \"GLS all-hands\", \"GLS quarterly\", or any request that combines deck/slides/presentation with GLS or Global Lending Services in any phrasing."
---
 
# GLS Branded PowerPoint
 
Always read and follow the `pptx` skill for all mechanics (pptxgenjs gotchas, QA steps, validate.py, etc.). This skill **overrides the pptx skill's design defaults** — use GLS brand instead of generic palettes or the "Avoid" color bar guidance (GLS deliberately uses header bars as structural elements).
 
---
 
## CRITICAL: Output File Path Length
 
The outputs folder path is already ~189 characters. Windows cannot open files with paths over 259 characters.
 
**Rules — always follow:**
- Save the final `.pptx` directly in the outputs folder root — **never in a subdirectory**
- Use a **short filename of 15 characters or fewer** (e.g., `gls.pptx`, `q3.pptx`, `deck.pptx`)
- Save all intermediate files (`.js`, `.pdf`, slide images) with short names in the same root folder
- Abbreviate aggressively: `q3perf.pptx` not `q3_loan_performance_review.pptx`
---
 
## Color Palette (Ground Truth)
 
Verified against two official GLS decks. No `#` prefix in pptxgenjs — 6-digit hex only.
 
| Role | Hex | Usage |
|---|---|---|
| **Brand Blue** | `0065AC` | Sidebar, chart lead series, links |
| **Structural Charcoal** | `696464` | Content slide header bar; footer rule; border box outline |
| **White** | `FFFFFF` | Header bar text; slide body background |
| Logo Gradient Blue (bright) | `3FAEE1` | **Logo artwork only** |
| Logo Gradient Blue (deep) | `3996D0` | **Logo artwork only** |
 
**Not brand colors:** `D34817` (theme XML only, never decorative). Green `00B050`/`92D050` and Red `FF0000` are **semantic data-callout only** — good/bad data highlights, never decorative.
 
**Charts:** Default Office palette unless user requests brand-consistent; then lead series → `0065AC`, comparison → neutral grays, red/green → semantic callouts only.
 
---
 
## Slide Dimensions — REQUIRED
 
```js
pres.layout = 'LAYOUT_4x3';   // 10" × 7.5" — NEVER LAYOUT_16x9 or LAYOUT_WIDE
```
 
Verify after generating: `unzip -p out.pptx ppt/presentation.xml | grep -i "sldSz"` → must show `cx="9144000" cy="6858000"`.
 
---
 
## Typography
 
- **Font:** `Franklin Gothic Medium (Body)` — exact name, all text, headings and body both
- **Fallback:** `Arial Bold` — only if Franklin Gothic unavailable; apply uniformly
- **Header bar titles:** White, ALL CAPS, `charSpacing` (never `letterSpacing` — silently ignored)
- Embed fonts so text doesn't reflow
> LibreOffice will substitute Franklin Gothic in QA renders — trust layout math, not visual QA, for text fit.
 
---
 
## Logo Assets
 
Bundled in this skill's `assets/` folder — **never use placeholder rectangles.**
 
| File | px | Use |
|---|---|---|
| `assets/gls-logo-grayscale-lockup.png` | 749×241 | Footer of every content slide |
| `assets/gls-logo-color-lockup.jpg` | 1516×472 | Title slide (large) |
 
```js
const fs = require('fs'), path = require('path');
// SKILL_DIR = directory containing this SKILL.md
const grayLogo  = 'image/png;base64,'  + fs.readFileSync(path.join(SKILL_DIR, 'assets/gls-logo-grayscale-lockup.png')).toString('base64');
const colorLogo = 'image/jpeg;base64,' + fs.readFileSync(path.join(SKILL_DIR, 'assets/gls-logo-color-lockup.jpg')).toString('base64');
```
 
Never stretch, recolor, or rotate. Always `sizing: { type: 'contain' }`. Grayscale → footer only; color → title only.
 
---
 
## Slide Anatomy
 
### Title Slide
 
**Exact measurements** — all from Top Left Corner, 10" × 7.5" slide.
**0.16" is the one consistent padding value** used everywhere on this slide: box outer margin from slide edges, box outer margin from sidebar, and box inner padding from content.
 
| Element | x | y | w | h | Notes |
|---|---|---|---|---|---|
| Blue sidebar | 7.83" | 0" | 2.17" | 7.5" | Solid `0065AC`; flush right edge of slide |
| Grey border box | 0.16" | 0.16" | 7.51" | 7.18" | `696464` outline ~0.5pt, NO fill; right edge = 7.67" (0.16" gap from sidebar); 0.16" from all slide edges |
| Color logo lockup | 0.5" | 1.4" | 6.89" | 2.15" | Inside box; `gls-logo-color-lockup.jpg`; 100% native size |
| Main title | 0.5" | 3.74" | 6.83" | ~0.6" | Inside box; **22pt**, black |
| Subtitle | 0.5" | 4.5" | 6.83" | ~0.4" | Inside box; ~14pt, `696464` |
| Sidebar text | 7.99" | 1.67" | 1.85" | auto | Inset 0.16" from bar edges (x=7.83+0.16, w=2.17−0.32); white, 14pt, centered |
 
```js
// Draw order: sidebar first (behind), then box, then content on top
slide.addShape(pptx.ShapeType.rect, { x: 7.83, y: 0,    w: 2.17, h: 7.5,
    fill: { color: '0065AC' } });
slide.addShape(pptx.ShapeType.rect, { x: 0.16, y: 0.16, w: 7.51, h: 7.18,
    fill: { type: 'none' }, line: { color: '696464', width: 0.5 } });
slide.addImage({ data: colorLogo, x: 0.5, y: 1.4, w: 6.89, h: 2.15,
    sizing: { type: 'contain', w: 6.89, h: 2.15 } });
slide.addText(titleText,    { x: 0.5,  y: 3.74, w: 6.83, h: 0.6,
    color: '000000', fontFace: 'Franklin Gothic Medium (Body)', fontSize: 22, bold: true });
slide.addText(subtitleText, { x: 0.5,  y: 4.5,  w: 6.83, h: 0.5,
    color: '696464', fontFace: 'Franklin Gothic Medium (Body)', fontSize: 14 });
slide.addText(sidebarText,  { x: 7.99, y: 1.67, w: 1.85, h: 2.0,
    color: 'FFFFFF', fontFace: 'Franklin Gothic Medium (Body)', fontSize: 14,
    align: 'center', valign: 'top', wrap: true });
```
 
### Content Slide
 
```
Header bar:   x:0,   y:0,    w:10,  h:0.6   fill: 696464
Header text:  x:0.2, y:0.05, w:9.6, h:0.5   FFFFFF, bold, ALL CAPS, charSpacing:2
Body area:    x:0.4, y:0.75, w:9.2, h:6.0   black, square bullets (■ / U+25A0)
```
 
**Footer — exact template coordinates, required on every content slide:**
 
| Element | x | y | w | h |
|---|---|---|---|---|
| Grayscale logo | 0.15" | 6.89" | 1.96" | 0.63" |
| Horizontal rule | 2.15" | 7.2" | 7.05" | 0.03" |
| Page number | 9.0" | 7.1" | 0.8" | 0.25" |
 
```js
slide.addImage({ data: grayLogo, x: 0.15, y: 6.89, w: 1.96, h: 0.63,
    sizing: { type: 'contain', w: 1.96, h: 0.63 } });
slide.addShape(pptx.ShapeType.rect, { x: 2.15, y: 7.2, w: 7.05, h: 0.03,
    fill: { color: '696464' }, line: { color: '696464' } });
slide.addText(String(pageNum), { x: 9.0, y: 7.1, w: 0.8, h: 0.25,
    color: '696464', fontSize: 10, align: 'right',
    fontFace: 'Franklin Gothic Medium (Body)' });
```
 
### Table/Data Slide
 
Same `696464` header bar. Table: dark header row (white text) → optional light-tint second header → alternating `F2F2F2`/white body rows → `92D050` green / `FF0000` red highlight cells for semantic good/bad callouts only.
 
### Section Divider
 
Same `696464` header bar, white body, otherwise blank.
 
### Emphasis/Callout
 
Thin outlined rectangle, `696464` or `0065AC` stroke, no fill — used sparingly, ad hoc only.
 
---
 
## Quick-Reference Coordinates
 
```
── Content slide ─────────────────────────────────────────
Header bar:      x:0,    y:0,    w:10,   h:0.6
Grayscale logo:  x:0.15, y:6.89, w:1.96, h:0.63
Footer rule:     x:2.15, y:7.2,  w:7.05, h:0.03  fill:696464
Page number:     x:9.0,  y:7.1,  w:0.8,  h:0.25  color:696464 fontSize:10
 
── Title slide ───────────────────────────────────────────
Blue sidebar:    x:7.83, y:0,    w:2.17, h:7.5   fill:0065AC
Grey border box: x:0.16, y:0.16, w:7.51, h:7.18  outline:696464 no-fill
Color logo:      x:0.5,  y:1.4,  w:6.89, h:2.15
Main title:      x:0.5,  y:3.74, w:6.83, h:0.6   22pt black
Subtitle:        x:0.5,  y:4.5,  w:6.83, h:0.5   14pt 696464
Sidebar text:    x:7.99, y:1.67, w:1.85, h:auto  white 14pt centered
```
 
---
 
## QA Checklist
 
- [ ] `.pptx` saved with short name (≤15 chars) directly in outputs root — no subdirectories
- [ ] `sldSz` = cx="9144000" cy="6858000" (4:3 confirmed)
- [ ] Every content slide: `696464` header bar + white ALL-CAPS title
- [ ] Footer trio at exact coordinates on every content slide
- [ ] Title slide: sidebar x:7.83 w:2.17; border box x:0.16 y:0.16 w:7.51 h:7.18; logo at x:0.5 y:1.4; title 22pt
- [ ] No `3FAEE1`/`3996D0` outside logo; no `D34817`; red/green in data callouts only
- [ ] Font: `Franklin Gothic Medium (Body)` or `Arial Bold` fallback
- [ ] validate.py passes
 
