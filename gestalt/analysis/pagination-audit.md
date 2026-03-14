# Pagination Audit Report

**Date:** 2026-03-11
**Template:** gestalt.html

---

## Step 1 — Current State: Exact Code

### HTML (line 952)
```html
<td class="footer-pagination">Page 1 of 1</td>
```

### CSS (lines 156–168)
```css
.footer-pagination {
  text-align: right;
  font-size: 9px;
  font-weight: 400;
  color: #525252;
}

/* ─── Page numbers ────────────────────────────────────────────────
   @page margin boxes and JS URL-param substitution (vars.page)
   both require features unavailable in Quaderno's wkhtmltopdf
   build. Pagination is omitted from the footer for now.
   TODO: Quaderno engineering to confirm supported mechanism
         for "Page X of Y" in the main body HTML template.        */
```

### Liquid logic
None. The string `Page 1 of 1` is hardcoded static text.

### JavaScript
None. No `<script>` tags anywhere in gestalt.html.

### @page CSS rules
None. No `@page` declarations anywhere in gestalt.html.

---

## Step 2 — Expected Behavior

### What mechanism is being used?
**Static text placeholder.** The string `"Page 1 of 1"` is a hardcoded literal. It is not generated dynamically. It will always display "Page 1 of 1" regardless of actual page count.

### How is the current page number determined?
It isn't. The value is hardcoded.

### How is the total page count determined?
It isn't. The value is hardcoded.

### Where does the page number appear in the DOM?
Inside `<td class="footer-pagination">` inside `<table class="footer-inner">` inside `<div class="footer-wrap">` — at the natural end of the document, after Legal Notices. It is right-aligned within the footer row (invoice number left, page number right).

---

## Step 3 — Bug Audit

### Check 1 — Mechanism compatibility

**Current mechanism: static text placeholder.**
Category: (c) Static text placeholder.

This is not a "bug" — it is an intentional interim state. The static text renders correctly as "Page 1 of 1". On single-page invoices it is technically accurate. On multi-page invoices it is incorrect (always shows "Page 1 of 1").

The real question is whether a dynamic mechanism is available.

---

### Check 2 — CSS @page counters

**Not used.** Would look like:
```css
@page {
  @bottom-right {
    content: "Page " counter(page) " of " counter(pages);
  }
}
```

**wkhtmltopdf support status:** @page margin boxes (`@bottom-right`, `@bottom-left`, etc.) are part of the CSS Paged Media spec. wkhtmltopdf is built on an old QtWebKit engine that has **incomplete** CSS Paged Media support. The `@page` rule itself is partially supported (for page size/margins), but `@page` margin boxes with `content: counter(page)` are **not supported** in wkhtmltopdf's webkit build.

**Verdict:** Would not work. Even if added to gestalt.html, it would render nothing.

---

### Check 3 — wkhtmltopdf JavaScript variables

wkhtmltopdf provides special JavaScript variables `page` and `topage` for page numbering. They work as follows:

```html
<!-- In --footer-html or --header-html file only: -->
<script>
  document.getElementById('page').textContent = window.page;
  document.getElementById('topage').textContent = window.topage;
</script>
Page <span id="page"></span> of <span id="topage"></span>
```

**Critical limitation:** These variables (`window.page`, `window.topage`) are **only available when the HTML file is passed as a `--footer-html` or `--header-html` argument** to the wkhtmltopdf CLI. They are populated by wkhtmltopdf's internal substitution engine at rendering time. They are **not available inside the main body HTML** (the template itself).

Since Quaderno renders invoice templates as the main body HTML (not via `--footer-html`), this mechanism is unavailable from within the template.

**Verdict:** Would not work in the template body. Requires `--footer-html` server-side support.

---

### Check 4 — All 8 reference templates

**Finding: Zero of 8 templates implement any form of page numbering.**

| Template | Page numbering | @page rules | `<script>` tags |
|---|---|---|---|
| destijl | None | None | None |
| mono | None | None | None |
| professional | None | None | None |
| newspaper | None | None | None |
| modern | None | None | None |
| minimal | None | None | None |
| frame | None | None | None |
| classic | None | None | None |

All 8 templates use `@media print` only for: `-webkit-print-color-adjust: exact`, `page-break-inside: avoid`, and `display: table-footer-group` / `table-row-group`. None use `@page` margin boxes. None have JavaScript. None display page numbers.

**This is a platform-level limitation, not a template design choice.** The absence of page numbering across all 8 production templates confirms that Quaderno's wkhtmltopdf build does not support any in-template page numbering mechanism.

---

### Check 5 — Quaderno documentation

**Finding: No mention of page numbering in Quaderno's template documentation.**

The official docs at https://developers.quaderno.io/templates/ and https://developers.quaderno.io/templates/tags/ contain zero references to:
- Page numbers or pagination
- `counter(page)` or `counter(pages)`
- wkhtmltopdf page variables
- `--footer-html` or `--header-html`

This absence, combined with zero usage across 8 production templates, confirms that page numbering is not a supported feature of Quaderno's template rendering pipeline.

---

### Check 6 — wkhtmltopdf built-in mechanisms available from body HTML

The only wkhtmltopdf page numbering mechanisms documented are:

| Mechanism | Available in body HTML? | Notes |
|---|---|---|
| `--footer-html` file with `window.page` / `window.topage` | No — requires server-side CLI support | Works only in separate footer HTML file passed to wkhtmltopdf |
| `--footer-right "[page] of [pages]"` CLI option | No — requires server-side CLI support | wkhtmltopdf string substitution on command line |
| CSS `@page { @bottom-right { content: counter(page) } }` | No — webkit engine does not support @page margin boxes | Spec is unsupported in wkhtmltopdf's webkit build |
| CSS `counter(page)` in `content` property on regular elements | No — CSS counters don't track PDF pages | CSS counters count DOM occurrences, not PDF page breaks |
| Liquid/server-side injection | Unknown — not documented by Quaderno | Would require Quaderno to inject page number as a Liquid variable |

**No mechanism exists for dynamic page numbers from within the main body HTML template.**

---

## Step 4 — Root Cause

### Why pagination is not rendering dynamically

The current implementation uses a static placeholder `"Page 1 of 1"` because **no mechanism exists to generate dynamic page numbers from within a Quaderno invoice template**.

The root cause is a platform constraint:
1. Quaderno renders templates as the main body HTML passed to wkhtmltopdf
2. wkhtmltopdf's page number injection (`window.page`, `window.topage`) only works in files passed via `--footer-html` / `--header-html` CLI flags — not in the main body HTML
3. wkhtmltopdf's webkit engine does not support CSS Paged Media `@page` margin boxes with `counter(page)`
4. Quaderno's template documentation provides no page number variables or filters
5. Zero of 8 production templates implement page numbering — confirming this is a known, accepted limitation

The static "Page 1 of 1" text is not a rendering failure — it renders correctly as text. The issue is that it cannot be made dynamic from within the template.

**Specific code:** Line 952 — `<td class="footer-pagination">Page 1 of 1</td>` — is the intentional interim state, not a bug.

---

## Step 5 — Proposed Fixes

### Option A — Keep static placeholder (current, no change required)

**Feasibility:** Already implemented.

**Behaviour:** Shows "Page 1 of 1" on all invoices. Correct on single-page invoices. Incorrect on multi-page invoices (always shows "1 of 1" even on page 3 of 5).

**Recommendation:** Acceptable interim state. Most invoices are single-page. The visual layout is correct.

---

### Option B — Remove page number entirely

**Feasibility:** Template change only. Immediate.

**Behaviour:** The footer shows only the invoice number (left-aligned). No page text.

**Trade-off:** Cleaner than a wrong "Page 1 of 1" on multi-page invoices. Loses the visual symmetry of the two-column footer layout.

---

### Option C — wkhtmltopdf `--footer-html` via Quaderno engineering (requires platform work)

**Feasibility:** Requires Quaderno to implement `--footer-html` injection at the platform level.

**How it would work:**
1. Quaderno renders a separate footer HTML file (passed as `--footer-html` to wkhtmltopdf)
2. That file uses `window.page` / `window.topage` JavaScript variables
3. The footer HTML file could be a separate Liquid template, or a fixed server-rendered snippet

**Behaviour:** True dynamic page numbers — "Page 2 of 5" etc. — rendered correctly on every page, at the physical page bottom (solving both the footer positioning and pagination problems simultaneously).

**This is the correct long-term solution.** It resolves both the sticky footer issue (Issue 12) and pagination in one platform feature.

---

### Option D — Quaderno Liquid page variable (requires platform work)

**Feasibility:** Requires Quaderno to expose a `document.page_count` or similar Liquid variable.

**How it would work:**
```liquid
Page 1 of {{ document.page_count }}
```

**Limitation:** This could show total pages ("of 5") but cannot show the current page number ("Page 2") — the current page is only known at PDF rendering time, not at Liquid template evaluation time. Partial improvement only.

---

### Recommendation

| Priority | Action | Who |
|---|---|---|
| Now | Keep static "Page 1 of 1" placeholder | Done — no change |
| Short term | Escalate to Quaderno engineering: request `--footer-html` support | Product/engineering |
| Long term | Implement `--footer-html` rendering at platform level | Quaderno engineering |

The static placeholder is the correct current state. It communicates the intent (page numbers should appear here) and is accurate for single-page invoices. No template change is needed at this time.
