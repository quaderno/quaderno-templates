# Footer Audit Report

**Date:** 2026-03-11
**Template:** gestalt.html (Revision 4 — tfoot, no height declarations)

---

## Step 1 — Current State: Exact Code

### CSS — every layout-related rule

```css
/* Global reset */
* { box-sizing: border-box; }

body {
  margin: 0;
  padding: 0;             /* no body padding */
  background: #fff;
  color: #525252;
  font-family: "Helvetica Neue", Arial, sans-serif;
  font-size: 9px;
  font-weight: 400;
  line-height: 1.4;
}

table { border-collapse: collapse; border-spacing: 0; }  /* GLOBAL — affects page-table too */

/* Master layout */
.page-table {
  width: 100%;
  border-collapse: collapse;  /* redundant with global rule above */
}
.content-cell {
  vertical-align: top;
  padding: 0;
  height: 1px;              /* intended to expand cell — see audit below */
}
.footer-cell {
  vertical-align: bottom;
  padding-top: 10px;
}

/* @media print */
@media print {
  body { -webkit-print-color-adjust: exact; }
  tfoot { display: table-footer-group; }   /* global — applies to ALL tfoot in document */
  .item-row { page-break-inside: avoid; }
}
```

**No height on `html`, `body`, or `.page-table`.** No `position` properties. No `overflow` properties.

---

### HTML — full nesting hierarchy

```
<body>
  <table class="page-table">           ← outer wrapper, width:100%, no height
    <tfoot>                             ← SOURCE ORDER: tfoot BEFORE tbody ✓
      <tr>
        <td class="footer-cell">        ← vertical-align:bottom, padding-top:10px
          <div class="footer-bar">      ← 6px color bar
          <table class="footer-inner">  ← NESTED TABLE inside tfoot td
            <tr>
              <td class="footer-invoice">  ← invoice label + number
              <td class="footer-pagination"> ← "Page 1 of 1"
    </tfoot>
    <tbody>
      <tr>
        <td class="content-cell">       ← vertical-align:top, padding:0, height:1px
          <!-- Section 1: header-table (nested table, width:100%) -->
          <!-- Section 1b: subject div -->
          <!-- Section 2: meta-table (nested table) -->
          <!-- Section 2b: divider + parties-table (nested table) -->
          <!-- Section 3: parties-table -->
          <!-- Section 4: items-table (nested table, has thead + tbody) -->
          <!-- Section 5: financial-zone (nested table with 2 inner tables) -->
          <!-- Section 6: currency-section (conditional, nested table) -->
          <!-- Section 7: other-info-table (conditional, nested table) -->
          <!-- legal-text div (conditional) -->
    </tbody>
  </table>
</body>
```

**Everything is inside the single content-cell.** No content outside `<table class="page-table">`.

---

## Step 2 — Expected Behavior

### What wkhtmltopdf should do

1. **Render the `<tfoot>` as a repeating footer group.** The `@media print { tfoot { display: table-footer-group; } }` rule instructs wkhtmltopdf to treat the tfoot as a footer that appears at the bottom of every printed page.

2. **Repeat the tfoot on each page.** When the table content overflows to a second page, wkhtmltopdf should render the tfoot at the bottom of page 1 and repeat it at the bottom of page 2.

3. **The content-cell fills available height.** The `height: 1px` on `.content-cell` is intended to make the cell expand vertically to fill the remaining page height after the footer is reserved.

### Why the footer should appear at the bottom

The mechanism is two-part:
- `display: table-footer-group` tells wkhtmltopdf: render this after all tbody rows, at the bottom of each page
- `vertical-align: bottom` on `.footer-cell` tells the browser: align content to the bottom of its cell

### What pushes the footer down

**Nothing does, for short invoices.** This is the critical gap:
- `display: table-footer-group` places the footer *below the last tbody row*, not at the physical page bottom
- Without a table height ≥ the page height, the table is only as tall as its content
- On a short invoice (e.g., 1 line item), the table might be 100mm tall; the footer appears at 100mm, leaving 171mm of blank space below it before the page bottom

### How footer repeats on page 2

`display: table-footer-group` should cause wkhtmltopdf to render the tfoot at the bottom of every page. This is the standard tfoot mechanism and is used by mono, modern, frame, and classic templates. Whether wkhtmltopdf's specific build correctly implements this is the open question.

---

## Step 3 — Bug Audit

### Bug 1 — tfoot source order
**Status: ✓ CORRECT**
`<tfoot>` is at line 518, `<tbody>` is at line 542. tfoot precedes tbody in source. This is required for wkhtmltopdf footer repetition.

---

### Bug 2 — CSS on html / body / parent elements

| Element | CSS | Problem? |
|---|---|---|
| `html` | no rules | ✓ |
| `body` | `margin:0; padding:0;` | ✓ — no height, no overflow |
| `* { box-sizing: border-box }` | global | ✓ — no layout impact |
| `table { border-collapse: collapse; border-spacing: 0; }` | **GLOBAL** | ⚠️ see below |

**Global `table` rule:** `table { border-collapse: collapse; border-spacing: 0; }` applies to `.page-table`. The `.page-table` also explicitly sets `border-collapse: collapse` — redundant but not harmful.

**No height, overflow, display, or position properties on any ancestor.** No conflicts found.

---

### Bug 3 — Nested tables inside content-cell

Gestalt.html has at least 8 nested tables inside `.content-cell`:

- `.header-table` (header)
- `.meta-table` (metadata)
- `.parties-table` (from/to/shipping)
- `.items-table` (line items — has `<thead>` and `<tbody>`)
- `.financial-zone` (outer table) → `.tax-table` + `.summary-table` (inner)
- `.currency-table` (conditional)
- `.other-info-table` (conditional)
- `.footer-inner` (inside tfoot)

**Critical: `tfoot { display: table-footer-group; }` is a GLOBAL selector.** It will match:
1. The outer `<tfoot>` on `.page-table` — intended ✓
2. ANY `<tfoot>` inside `.content-cell` — if any inner table has a `<tfoot>`, this rule would also apply to it

**Checking inner tables for tfoot:** The items-table, tax-table, summary-table, and currency-table all use `<tbody>` only — no inner tfoot. The footer-inner table inside the tfoot has no tfoot. **No inner tfoot conflict found.**

However: **the items-table has `<thead>`.** The `@media print` block intentionally does NOT set `thead { display: table-header-group; }` (which would repeat column headers). This is correct per spec.

---

### Bug 4 — Content outside page-table

`<table class="page-table">` is the FIRST and ONLY element inside `<body>`. All content is inside the single `<td class="content-cell">`. No stray elements outside the table. ✓

---

### Bug 5 — pdfkit meta tags

```html
<meta name="pdfkit-page_size" content="A4">
<meta name="pdfkit-margin_top" content="13mm">
<meta name="pdfkit-margin_right" content="13mm">
<meta name="pdfkit-margin_bottom" content="13mm">
<meta name="pdfkit-margin_left" content="13mm">
```

These are identical to the reference templates (modern, frame). No conflicts. `body { padding: 0; }` is correct — pdfkit handles margins, body does not need to duplicate them.

---

### Bug 6 — Liquid conditionals altering structure

Checked all conditional blocks:
- `{% if has_discounts %}` — adds/removes `<th>`, `<col>`, and `<td>` inside items-table only. Does not affect outer table structure.
- `{% if document.exchange %}` — wraps the currency section div. Does not add/remove table wrappers.
- `{% if show_payment_details or show_notes %}` — wraps other-info-table. Does not affect outer structure.
- `{% if document.legal != blank %}` — adds a `<div>` inside content-cell. ✓
- Meta section `{% for item in meta_items %}` — operates inside meta-table only. ✓

No Liquid block opens or closes a `<tr>`, `<td>`, `<tbody>`, or `</table>` tag across a conditional boundary in the outer structure. ✓

---

### Bug 7 — Structural comparison with destijl.html

| Aspect | destijl.html | gestalt.html |
|---|---|---|
| Outer structure | `<body><table>` + flat `<tr>` rows for all sections `</table>` | `<body><table class="page-table"><tfoot>...<tbody><tr><td>` all content `</td></tr></tbody></table></body>` |
| Footer type | Regular `<tr>` (not tfoot) — content flow only | `<tfoot>` — intended to repeat |
| tfoot usage | `#items` table has tfoot for totals, set to `display:table-row-group` (prevents repeating) | Outer page-table has tfoot for footer bar + invoice number |
| Items table tfoot | ✓ exists | ✗ uses tbody only |
| Page footer | None — destijl has no repeating page footer | tfoot with footer bar + invoice number |
| body padding | `padding: 20px` | `padding: 0` |
| Nested depth | Content is directly in `<tr><td>` rows of the main table | All content is inside a single `<td class="content-cell">` which contains nested tables |

**Critical structural difference:** destijl's content is spread across multiple `<tr>` rows of the single outer table. Gestalt.html wraps ALL content inside ONE `<td class="content-cell">`. This means:

- destijl: `<table> → <tr> → <td>` for each section (shallow nesting)
- gestalt.html: `<table> → <tfoot>+<tbody> → <tr> → <td> → <table> → <tr> → <td>` for each section (deep nesting, all inside one content cell)

The depth itself is not a bug — it's the intended structure for tfoot repetition. But it does mean the single `content-cell` td must contain all the nested tables, and its `height: 1px` trick must make it expand to fill the available space.

**Also critical:** destijl sets `#items tfoot { display: table-row-group; }` — converting items tfoot to a row group. Gestalt.html's global `tfoot { display: table-footer-group; }` would conflict IF any inner table had a tfoot — but they don't.

---

## Step 4 — Root Cause Identification

### Problem a) Footer overlaps header on page 1

This symptom occurred in **Revision 1** (height:100%). It has NOT been confirmed to occur in Revision 4 (no height).

Root cause in Revision 1: `height: 100%` on the outer table, combined with `html/body { height: 100% }`, caused wkhtmltopdf to compute the tfoot position relative to the viewport height at initial layout time, placing it at the top of the document before any content rendered. Absolute location: this was caused by lines that set `html { height: 100%; }` and `.neo-layout { height: 100%; }` — both now removed.

Revision 4 does not have this problem because there is no height on any element.

---

### Problem b) Footer not anchored to physical page bottom

**Root cause: no mechanism pushes the table tall enough to fill the page.**

The sequence:
1. `.page-table` has no height → table is exactly as tall as its content
2. On a short invoice, content might be 100–150mm tall
3. tfoot appears immediately after the last tbody row (at ~100–150mm)
4. Physical page bottom is at 271mm (297mm A4 minus 13mm top and 13mm bottom margins)
5. Gap between footer and page bottom: ~120–170mm of blank space

The `height: 1px` on `.content-cell` does not help here. For `height: 1px` to expand a cell, the containing table must have a declared height larger than the cell's natural height. Without a height on `.page-table`, the table shrinks to fit content, and the cell has nothing to expand into. The `height: 1px` is effectively a no-op.

**Specific line: `.content-cell { height: 1px; }` — this rule cannot function without a height on `.page-table`.**

---

### Problem c) Footer not repeating on page 2

**Root cause: uncertain — this is an untested assumption about wkhtmltopdf behaviour.**

`display: table-footer-group` is the CSS property that should cause tfoot repetition. Whether Quaderno's specific wkhtmltopdf build supports this for a top-level outer table (not just the items sub-table) has NOT been confirmed by any successful test.

The templates that use this (mono, modern, frame, classic) use it for notes/legal footers that sit after content — not for a color-bar footer that must be at the physical page bottom. Their "repeating footer" test is less strict than gestalt.html's requirement.

---

## Step 5 — Proposed Fix

### Diagnosis summary

There are two separate problems requiring different solutions:

| Problem | Cause | Solution |
|---|---|---|
| Footer not at physical bottom on short invoices | Table has no height; tfoot falls after content | Give the table a height equal to the usable page height |
| `height: 1px` on content-cell does nothing | Cell can't expand without a table height | Pairs with the table height fix — once table has height, cell expands to fill it |

### Proposed CSS

The only reliable CSS approach that addresses the physical-page-bottom requirement:

```css
.page-table {
  width: 100%;
  border-collapse: collapse;
  height: 100vh;  /* Option A: viewport height — wkhtmltopdf viewport = one page */
}
```

Or:

```css
.page-table {
  width: 100%;
  border-collapse: collapse;
  min-height: 271mm;  /* Option B: explicit A4 usable height */
}
```

**Why `100vh` instead of `271mm`:**
- `100vh` = 100% of the wkhtmltopdf viewport, which is exactly one page height minus margins. It does not require knowing the margin values.
- `271mm` requires accurately knowing that margins are 13mm each. If margins change, this breaks.
- However, `100vh` in wkhtmltopdf's webkit may or may not equal the usable page area (vs the full page including margins) — this is uncertain.

**Why NOT `height: 100%` (the Revision 1 bug):**
`height: 100%` on a table requires a parent with a defined height. `body { height: 100% }` requires `html { height: 100% }`. This chain caused Revision 1's tfoot-at-top bug.

`100vh` and `271mm` are **absolute values** that do not depend on a parent height chain. They should not trigger the same wkhtmltopdf tfoot misplacement bug.

### Specific change

Replace:
```css
.page-table {
  width: 100%;
  border-collapse: collapse;
}
.content-cell {
  vertical-align: top;
  padding: 0;
  height: 1px;
}
```

With:
```css
.page-table {
  width: 100%;
  border-collapse: collapse;
  height: 100vh;  /* or: 271mm — see note above */
}
.content-cell {
  vertical-align: top;
  padding: 0;
  height: 100%;  /* now has a containing height to fill */
}
```

The `height: 1px` trick is replaced by `height: 100%` on the content-cell, which is now meaningful because the outer table has a declared height.

**Nothing else changes** — tfoot source order, `display: table-footer-group`, `vertical-align: bottom` on footer-cell all remain correct.

### Why this is different from the Revision 1 bug

Revision 1 chain:
```
html { height: 100% }      ← cascading relative chain
body { height: 100% }      ← cascading relative chain
table { height: 100% }     ← caused tfoot misplacement
```

Proposed chain:
```
html: no height
body: no height
table { height: 100vh }    ← absolute value, no parent dependency
td.content-cell { height: 100% }  ← % of table's 100vh = one page height
```

The `height: 100%` on the content-cell is safe here because its parent (`.page-table`) has an absolute height. The bug in Revision 1 was the cascading chain through html and body — not `height: 100%` on a table cell.

### Two options to test

**Option A (preferred):** `height: 100vh` on page-table
- Pros: no hardcoded mm value; adapts if pdfkit meta margins change
- Cons: `100vh` in wkhtmltopdf may equal the full page (297mm) rather than the usable area (271mm); footer might be cut off by the 13mm bottom margin

**Option B:** `height: 271mm` on page-table
- Pros: explicit, predictable
- Cons: hardcoded to 13mm margins; breaks if margins change
- Note: this is what Revision 3 tried, BUT Revision 3 used `height: 1px` on content-cell (which doesn't work) rather than `height: 100%`

Revision 3 failed not because of `height: 271mm` on the table, but because `content-cell { height: 1px }` couldn't expand to push the tfoot down. The table had the right height; the cell didn't use it.

**Recommendation:** Test Option B (`271mm`) first because it's been partially validated — Revision 3 used it, and the only reported failure was the footer not anchoring, which this fix directly addresses by correcting the content-cell height.
