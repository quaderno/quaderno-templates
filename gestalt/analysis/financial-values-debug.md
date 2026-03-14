# Financial Values Debug Report

**Date:** 2026-03-11
**Symptom:** Line item Amount values and Summary totals in gestalt.html do not match the correct values shown in the Quaderno app.

---

## Step 1 — Root Cause Analysis

### Line Items Table

gestalt.html has **two separate monetary columns** in the line items table:

| Column | Header label | Variable used | Line |
|--------|-------------|---------------|------|
| 5th (Subtotal) | `labels.subtotal` | `item.subtotal` | 804 |
| 7th (Amount)   | `labels.amount`   | `item.total_amount` | 808 |

**This is the primary bug.** `item.total_amount` is not used by any of the 8 production templates. Every reference template uses `item.subtotal` as its single line value column.

### Document-Level Summary

gestalt.html uses:
- `document.subtotal` → Subtotal row
- `document.discount` → Discount row
- `document.total` → Total row

**These are identical to all 8 reference templates** — confirmed correct Quaderno variable names.

### Tax Breakdown

- Per-tax rows: `document_tax.taxable_base`, `document_tax.rate`, `document_tax.amount` — confirmed correct.
- Total tax row taxable base: `document.subtotal` — best available aggregate variable.

---

## Step 2 — Cross-Reference With All 8 Templates

Line item value column across every reference template:

| Template | Column 5 (last value) | Variable |
|---|---|---|
| destijl | Amount | `item.subtotal` |
| mono | Amount | `item.subtotal` |
| professional | Amount | `item.subtotal` |
| newspaper | Amount | `item.subtotal` |
| modern | Amount | `item.subtotal` |
| minimal | Amount | `item.subtotal` |
| frame | Amount | `item.subtotal` |
| classic | Amount | `item.subtotal` |
| **gestalt** | **Subtotal** | **`item.subtotal`** |
| **gestalt** | **Amount** | **`item.total_amount` ← BUG** |

`item.total_amount` is used by **zero** production templates.

Document-level summary variables:

| Row | Variable | Used by |
|---|---|---|
| Subtotal | `document.subtotal` | All 8 templates + gestalt ✓ |
| Discount | `document.discount` | All 8 templates + gestalt ✓ |
| Total tax | `document.tax_amount` | Modern, classic, frame, mono, minimal + gestalt ✓ |
| Total | `document.total` | All 8 templates + gestalt ✓ |

No calculation discrepancy in the summary — variables are identical to production templates.

---

## Step 3 — Quaderno Documentation

From official docs and confirmed usage:

| Variable | Definition | Status |
|---|---|---|
| `item.subtotal` | quantity × unit_price | ✅ Documented |
| `item.gross_amount` | subtotal − per-line discount | 📄 Documented but unused in templates |
| `item.total_amount` | subtotal − discount + tax | 📄 Documented but unused in templates |
| `document.subtotal` | Net amount before global discount and tax | ✅ Documented + used in all templates |
| `document.total` | Final invoice total | ✅ Documented + used in all templates |

**`item.total_amount` is documented but is not the right variable for the last "Amount" column** — it includes per-line discount and tax, producing values that diverge from what the app displays.

---

## Step 4 — Root Cause: Two Competing Amount Columns

gestalt.html has an extra "Subtotal" column before the tax rate that no reference template has:

```
Description | Qty | Unit Price | [Discount] | Subtotal | Tax Rate | Amount
                                              ^^^^^^^^^ extra column not in reference templates
```

Reference template structure:
```
Description | Qty | Unit Price | [Discount] | Subtotal/Amount
```

gestalt.html was built with a two-column approach (showing both a pre-tax subtotal and a post-tax amount per line), but the Amount column was assigned `item.total_amount` which:
1. Is not used in any production template
2. Gives values that don't match what the app displays

**Proof from test data:**
- Sum of App Amount values: 10,000.00 + 722,222.222 + 30,612.245 + 1,789.286 + 2,400.00 + 1,302.088 + 36,296.795 = **804,622.636 ≈ App Subtotal (804,622.65)**
- This confirms App Amount column = `item.subtotal` (qty × unit_price, before discount/tax)
- `item.total_amount` produces different (smaller) values — it appears to apply per-line discounts, making it ≠ `item.subtotal`

---

## Fix Applied

### Change 1: Remove the redundant "Subtotal" column

The "Subtotal" column (`item.subtotal`) was redundant with the "Amount" column once Amount is corrected to also use `item.subtotal`. Removing it aligns gestalt.html with all 8 reference templates:

```
Before: Description | Qty | Unit Price | [Discount] | Subtotal | Tax Rate | Amount
After:  Description | Qty | Unit Price | [Discount] | Tax Rate | Amount
```

Column counts: 7-col (with discount) → 6-col. 6-col (no discount) → 5-col.

### Change 2: Amount column variable — `item.total_amount` → `item.subtotal`

| Location | Before | After |
|---|---|---|
| Line items `<td>` last column | `{{ item.total_amount }}` | `{{ item.subtotal }}` |

### Change 3: Colgroup widths redistributed

**With discount (6 cols after fix):**

| Col | Before | After |
|---|---|---|
| Description | 40% | 40% |
| Qty | 8% | 9% |
| Unit price | 11% | 13% |
| Discount | 8% | 9% |
| ~~Subtotal~~ | ~~11%~~ | removed |
| Tax rate | 8% | 10% |
| Amount | 14% | 19% |

**Without discount (5 cols after fix):**

| Col | Before | After |
|---|---|---|
| Description | 40% | 40% |
| Qty | 10% | 12% |
| Unit price | 13% | 15% |
| ~~Subtotal~~ | ~~13%~~ | removed |
| Tax rate | 10% | 13% |
| Amount | 14% | 20% |

---

## Summary of Discrepancy for Document-Level Summary Values

| Row | App value | Template value | Variable | Verdict |
|---|---|---|---|---|
| Subtotal | 804,622.65 | 722,303.50 | `document.subtotal` | Variable is correct (same as all 8 templates). Discrepancy is because Quaderno's `document.subtotal` represents the net after per-line discounts, whereas the app UI shows the gross before per-line discounts. No alternative variable available. Requires Quaderno engineering clarification. |
| Discount | 77,486.29 | 69,093.07 | `document.discount` | Variable is correct. The app may include per-line discounts in this figure while `document.discount` = global discount only. Requires Quaderno engineering clarification. |
| Total | 722,303.50 | 649,594.63 | `document.total` | Variable is correct. `document.total` = `document.subtotal` − global discount + tax. Different result because `document.subtotal` baseline differs. Same root cause. |

**Conclusion:** The document-level summary variable names are correct and identical to all 8 production templates. The value differences are a Quaderno data model question, not a template bug. The template cannot fix this without performing calculations (prohibited). Requires Quaderno engineering to clarify the correct variable for displaying gross-before-per-line-discounts subtotal.
