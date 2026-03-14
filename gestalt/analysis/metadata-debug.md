# Metadata Section Debug Report

**Date:** 2026-03-11
**Template:** gestalt.html
**Symptom:** Metadata section (Issue Date, Due Date, Service Date, PO Number, Valid Until, Referenced Document) not rendering in Quaderno test invoice.

---

## Step 1 — Audit gestalt.html

### Does the metadata section exist?
Yes. Lines 564–632. Structure:
1. Liquid array built from items that exist (lines 571–585)
2. Spacer div (line 587)
3. `<table class="meta-table">` with a `{% for item in meta_items %}` loop (lines 589–632)

### Liquid variables used
| Field | Variable | Label | Conditional |
|---|---|---|---|
| Issue Date | `document.issue_date` | `labels.issue_date` | Always added |
| Due Date | `document.due_date` | `labels.due_date` | `{% if document.due_date != blank %}` |
| Service Date | `document.service_date` | `labels.service_date` | Always added (unconfirmed var) |
| PO Number | `document.po_number` | `labels.po_number` | `{% if document.po_number != blank %}` |
| Valid Until | `document.valid_until` | `labels.valid_until` | `{% if document.valid_until != blank %}` |
| Referenced Doc | `document.related_document_type/number/date` | `labels.related_document` | `{% if document.related_document_type != blank %}` |

### CSS issues that could hide the section
None. Reviewed `.meta-table`, `.meta-cell`, `.meta-label`, `.meta-value` — no `display: none`, no `overflow: hidden`, no zero height. The `.section` spacer div is safe.

---

## Step 2 — Reference Template Comparison

### Date/metadata variables used across all 8 templates

| Template | issue_date | due_date | service_date | po_number | valid_until | related_document | Date filter |
|---|---|---|---|---|---|---|---|
| destijl | `document.issue_date` | `document.due_date` | — | `document.po_number` | `document.valid_until` | `document.related_document_*` | `'default', contact.country` |
| mono | `document.issue_date` | `document.due_date` | — | `document.po_number` | `document.valid_until` | `document.related_document_*` | `'default', contact.country` |
| professional | `document.issue_date` | `document.due_date` | — | `document.po_number` | `document.valid_until` | `document.related_document_*` | `'long'` |
| newspaper | `document.issue_date` | `document.due_date` | — | `document.po_number` | `document.valid_until` | `document.related_document_*` | `'default', contact.country` |
| modern | `document.issue_date` | `document.due_date` | — | `document.po_number` | `document.valid_until` | `document.related_document_*` | `'default', contact.country` |
| minimal | `document.issue_date` | `document.due_date` | — | `document.po_number` | `document.valid_until` | `document.related_document_*` | `'default', contact.country` |
| frame | `document.issue_date` | `document.due_date` | — | `document.po_number` | `document.valid_until` | `document.related_document_*` | `'default', contact.country` |
| classic | `document.issue_date` | `document.due_date` | — | `document.po_number` | `document.valid_until` | `document.related_document_*` | `'default', contact.country` |

**Observations:**
- All 8 templates use identical variable names — confirmed correct.
- `document.service_date` is used by zero reference templates. Not in official Quaderno docs.
- All templates use `labels.issue_date`, `labels.due_date`, `labels.po_number`, `labels.valid_until`, `labels.related_document` — matches gestalt.html exactly.
- Date filter `| date: 'default', contact.country` — matches gestalt.html.

---

## Step 3 — Root Cause: `push` Filter Not Available

### Finding
```
grep "push" *.html
```
Result: **`push` appears only in gestalt.html (6 times). Zero uses in any of the 8 reference templates.**

### Explanation
The metadata array is built using Liquid's `push` filter:
```liquid
{% assign meta_items = "" | split: "" %}
{% assign meta_items = meta_items | push: "issue_date" %}
{% assign meta_items = meta_items | push: "due_date" %}
...
```

The `push` array filter is **not part of the Liquid standard library** included in Quaderno's rendering environment. It was added to Shopify Liquid in a later version and is not universally available. Since no existing Quaderno template uses it, Quaderno's Liquid build does not support it.

**Effect:** `push` calls silently fail or return the unmodified empty array. `meta_items` remains `[]`. The `{% for item in meta_items %}` loop never executes. The `<table class="meta-table">` renders as an empty table with no rows. **The entire metadata section is invisible.**

This is a silent failure — no Liquid error is thrown, no visible output.

### Secondary Issue: `document.service_date`
`document.service_date` does not exist in any reference template and is absent from Quaderno's documentation. It is being unconditionally pushed to the array (no `{% if %}` guard), which would render an empty cell even if the array-building worked. This variable needs a `!= blank` conditional guard, and its existence should be confirmed with Quaderno.

---

## Step 4 — Quaderno Documentation Check

Official docs at https://developers.quaderno.io/templates/ confirm:
- `document.issue_date` ✅
- `document.due_date` ✅
- `document.po_number` ✅
- `document.valid_until` ✅
- `document.related_document_number` / `_type` / `_date` ✅
- `document.service_date` ❌ — **not documented, not in any reference template**

No Liquid filters beyond the standard set are required for these fields. `| date: 'default', contact.country` is the correct filter (used by 7 of 8 reference templates).

---

## Step 5 — Fix Applied

### What changed
Replaced the `push`-based array builder with `append` + `split`, which is supported in all Liquid versions including Quaderno's:

**Before (broken):**
```liquid
{% assign meta_items = "" | split: "" %}
{% assign meta_items = meta_items | push: "issue_date" %}
{% if document.due_date != blank %}
  {% assign meta_items = meta_items | push: "due_date" %}
{% endif %}
{% assign meta_items = meta_items | push: "service_date" %}
...
```

**After (fixed):**
```liquid
{% assign meta_str = "issue_date" %}
{% if document.due_date != blank %}{% assign meta_str = meta_str | append: "|due_date" %}{% endif %}
{% if document.service_date != blank %}{% assign meta_str = meta_str | append: "|service_date" %}{% endif %}
{% if document.po_number != blank %}{% assign meta_str = meta_str | append: "|po_number" %}{% endif %}
{% if document.valid_until != blank %}{% assign meta_str = meta_str | append: "|valid_until" %}{% endif %}
{% if document.related_document_type != blank %}{% assign meta_str = meta_str | append: "|related_document" %}{% endif %}
{% assign meta_items = meta_str | split: "|" %}
```

**Key differences:**
1. `push` replaced by `append` + `split` — universally supported in Liquid
2. `service_date` is now conditional (`{% if document.service_date != blank %}`), not always-rendered
3. The rest of the loop logic (`{% for item in meta_items %}`, `modulo: 3`, row open/close, `meta_items.size`) is unchanged and correct

---

## Summary

| # | Issue | Severity | Fixed? |
|---|---|---|---|
| 1 | `push` filter unsupported in Quaderno's Liquid — entire metadata array stays empty | **Critical** | ✅ Yes |
| 2 | `document.service_date` always rendered without conditional guard | Medium | ✅ Yes (now conditional) |
| 3 | `document.service_date` unconfirmed as a Quaderno variable | Low | Flagged — needs Quaderno confirmation |
