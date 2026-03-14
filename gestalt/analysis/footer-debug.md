# Footer Debug Report — Sticky Footer & Pagination

**Date:** 2026-03-11

---

## Issue 12 — Sticky Footer

### Revision 1 (superseded): tfoot + height:100%

Applied `<tfoot>` + `display:table-footer-group` with `height: 100%` on html/body/outer table. Produced two failures:

**Issue 1: Footer rendered at TOP of page 1**
- Root cause: cascading `height: 100%` on html → body → table causes wkhtmltopdf to misplace the tfoot. With `height: 100%` on the table, wkhtmltopdf interprets the table as filling the viewport height, and renders the tfoot at the TOP (before tbody content). This is a wkhtmltopdf webkit quirk with relative heights on table elements.
- Key evidence: None of the 8 production templates set `height: 100%` on html, body, or their outer table — they all use content-driven heights only.

**Issue 2: Footer not at physical page bottom on the last page**
- Root cause: `display:table-footer-group` repeats the tfoot at the bottom of table content on each page, but on the final page (when content is shorter than a full page), the footer appears immediately after the last content row, not at the physical page bottom.

---

### Revision 2 (superseded): position:fixed

**Why it failed:** Quaderno's wkhtmltopdf build does not reliably support `position: fixed` for footer pinning. The footer was not at the bottom of page 1, not at the bottom of page 2, and did not repeat across pages.

---

### Revision 3 (superseded): tfoot + height:271mm (no height on html/body)

**Root cause analysis of Revision 1's failure:**

The bug was NOT in `<tfoot>` or `display:table-footer-group` — these work correctly in wkhtmltopdf. The bug was specifically the cascading `height: 100%` chain:

```css
/* THIS CAUSED THE BUG — do not repeat */
html  { height: 100%; }
body  { height: 100%; }
table { height: 100%; }  /* relative % = broken tfoot position */
```

wkhtmltopdf handles relative heights on tables differently from absolute heights. `height: 100%` on a table in a paginated PDF context causes wkhtmltopdf to render the tfoot at the top because the "100% height" is computed relative to an undefined parent context during pagination.

**The correct approach:**

Use an **absolute mm height** on the outer table only — no height on html or body:

```css
/* CORRECT — absolute value, no html/body height */
.neo-layout {
  width: 100%;
  height: 271mm;  /* A4 297mm − 13mm top margin − 13mm bottom margin */
}
```

**Why `height: 271mm` works:**

1. CSS spec: `height` on a `<table>` element is treated as a **minimum height** — the table will be at least 271mm tall but grows beyond that if content requires it.
2. With no `height` on html/body, the percentage-height chain that caused Revision 1's bug does not exist.
3. On short invoices (content < 271mm): table fills exactly 271mm, tfoot sits at the physical page bottom.
4. On long invoices (content > 271mm): table grows beyond 271mm; `display:table-footer-group` repeats the tfoot at the bottom of every page.

**Reference template confirmation:**

All 4 tfoot-using templates (mono, modern, frame, classic) confirmed to use:
- `<tfoot>` before `<tbody>` in DOM order
- `tfoot { display: table-footer-group; }` in `@media print`
- **Zero** use of `height` or `min-height` on html, body, or outer table

These templates do NOT require the footer to pin to the physical page bottom — their tfoot is a notes/legal block that appears after content. gestalt.html's requirement is different (color bar + invoice number always at physical page bottom), which is why the `height: 271mm` is added.

**CSS:**
```css
.neo-layout {
  width: 100%;
  height: 271mm;
}
.content-cell {
  vertical-align: top;
  padding: 0;
}
.footer-cell {
  padding-top: 10px;
}

/* @media print */
tfoot { display: table-footer-group; }
```

**HTML structure:**
```html
<table class="neo-layout">
  <tfoot>
    <tr>
      <td class="footer-cell">
        <div class="footer-bar"></div>
        <table class="footer-inner">
          <tr>
            <td class="footer-invoice">...</td>
            <td class="footer-pagination">...</td>
          </tr>
        </table>
      </td>
    </tr>
  </tfoot>
  <tbody>
    <tr>
      <td class="content-cell">
        <!-- all content sections -->
      </td>
    </tr>
  </tbody>
</table>
```

**Scenario verification:**
- Short invoice (< 271mm): table = 271mm tall; tfoot at physical page bottom ✓
- Full-page invoice (≈ 271mm): table fills page; tfoot at physical page bottom ✓
- Multi-page invoice: table > 271mm; `table-footer-group` repeats tfoot at bottom of each page ✓
- Footer NOT at top: no `height: 100%` on html/body = no cascading relative height bug ✓

---

### Revision 4 (superseded): tfoot + no height declarations

Tested. Audit in `/analysis/footer-audit.md` identified root cause: `height: 1px` on `.content-cell` is a no-op without a table height. Footer appeared after content, not at physical page bottom.

---

### Revision 5 (superseded): tfoot + height:271mm + content-cell height:100%

Applied the audit's recommendation: `height: 271mm` on `.page-table` + `height: 100%` on `.content-cell` (no height on html/body). Also failed.

---

### Final status: UNRESOLVED via template CSS/HTML

**All five approaches tested and failed:**

| Revision | Approach | Result |
|---|---|---|
| 1 | tfoot + `height:100%` on html/body/table | Footer rendered at top of page 1 |
| 2 | `position: fixed; bottom: 0` | Not at bottom, did not repeat across pages |
| 3 | tfoot + `height:271mm` on table only + `height:1px` on cell | Footer after content, not at physical page bottom |
| 4 | tfoot + no height declarations | Footer after content, not at physical page bottom |
| 5 | tfoot + `height:271mm` on table + `height:100%` on cell | Failed |

**Conclusion:** A page-fixed repeating footer is not achievable via template CSS/HTML alone in Quaderno's wkhtmltopdf build.

**Required:** `--footer-html` server-side support from Quaderno engineering, or an equivalent platform-level mechanism. See `/analysis/footer-audit.md` for the full structural audit and root cause analysis.

**Current implementation:** Footer placed at natural end of document (after Legal Notices) as a simple `<div class="footer-wrap">`. Styling retained (6px color bar, invoice number, page placeholder). This is the correct interim state pending engineering resolution.

---

## Issue 13 — Pagination

### Research Summary

| Method | Status |
|---|---|
| `@page { @bottom-right { content: counter(page) } }` | ❌ Not rendering — Quaderno's wkhtmltopdf build does not support @page margin boxes |
| `<script>` JS substitution using `vars.page` / `vars.topage` | ❌ These wkhtmltopdf URL-parameter variables work ONLY in files passed via `--footer-html` CLI flag, not in the main body HTML |
| CSS `counter(page)` in element `content` property | ❌ Same webkit limitation as @page |
| `display: table-footer-group` with page counter | ❌ Not supported |
| Hardcoded in template HTML | N/A — would always show static number |

**Confirmed:** None of the 8 production Quaderno templates implement pagination. This is a platform constraint, not a template design choice.

### Solution Applied
Static placeholder `Page 1 of 1` added to the footer with a prominent comment flagging it for Quaderno engineering. The placeholder:
- Maintains correct visual layout (right-aligned, matching typography spec)
- Makes the limitation visible during QA
- Can be replaced with a working mechanism once Quaderno engineering confirms the supported approach

### Recommended next step (for Quaderno engineering)
Investigate whether Quaderno's template rendering pipeline supports any of:
1. A wkhtmltopdf `--footer-html` injection mechanism at the platform level
2. A custom Liquid tag or filter that injects page numbers server-side
3. An updated wkhtmltopdf build with @page margin box support
