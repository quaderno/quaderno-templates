# Financial Columns Debug Report

**Date:** 2026-03-11
**Source:** Quaderno Tags documentation — https://developers.quaderno.io/templates/tags/

---

## Corrected Item Variable Definitions

From the official Quaderno Tags documentation:

| Variable | Definition |
|---|---|
| `item.subtotal` | Quantity × Unit Price (before discount, before tax) |
| `item.gross_amount` | Subtotal − Discount (after discount, before tax) |
| `item.total_amount` | Subtotal − Discount + Tax Amount (after discount AND tax) |
| `item.discount_rate` | Discount percentage for this line |
| `item.tax_1_rate` | Tax rate shortcut for the item's primary tax |
| `item.tax_2_rate` | Additional rate for a second tax (e.g. local tax), if applicable |

---

## Why item.total_amount Was Lower Than item.subtotal

In the previous test, `item.total_amount` produced smaller values than `item.subtotal` for several lines. This was misidentified as a bug.

**Actual cause:** The test invoice uses negative taxes (IRPF at −7% and −15%), which is a Spanish withholding tax applied as a deduction. In that context:

```
item.total_amount = item.subtotal − discount + (negative tax amount)
                  = item.subtotal − discount − withholding
```

The result is correctly lower than `item.subtotal`. This is not a bug — `item.total_amount` behaves as documented.

---

## Correct 7-Column Line Items Layout

| Column | Variable | Notes |
|---|---|---|
| Description | `item.description` | Fixed 40% width |
| Qty | `item.quantity` | |
| Unit Price | `item.unit_price` | |
| Discount | `item.discount_rate` | Conditional column (has_discounts pre-scan) |
| Subtotal | `item.subtotal` | Qty × Unit Price, taxable base |
| Tax Rate | `item.tax_1_rate` [+ `item.tax_2_rate`] | See multi-rate note below |
| Amount | `item.total_amount` | After discount + tax (can be < subtotal with negative taxes) |

**With Discount column: 7 columns. Without: 6 columns.**

---

## Handling Two Tax Rates (item.tax_1_rate + item.tax_2_rate)

No existing template shows per-line tax rates — there is no reference pattern. Decision for neo.html:

Show `item.tax_1_rate` always; conditionally append `+ item.tax_2_rate` when it is present and non-zero:

```liquid
{{ item.tax_1_rate | precision }}%{% if item.tax_2_rate != 0 and item.tax_2_rate != blank %} + {{ item.tax_2_rate | precision }}%{% endif %}
```

This produces "21%" for single-tax items and "21% + 5.2%" for dual-tax items, which is readable and complete.

---

## Column Widths

**7-column (with discount):** Description 40% + Qty 8% + Unit Price 11% + Discount 8% + Subtotal 11% + Tax Rate 8% + Amount 14% = 100%

**6-column (no discount):** Description 40% + Qty 10% + Unit Price 13% + Subtotal 13% + Tax Rate 10% + Amount 14% = 100%

---

## service_date — Confirmed Absent from Docs

- `document.service_date` is **not listed** in the Quaderno Tags documentation.
- `labels.service_date` is **not listed** in the labels documentation.
- No existing template uses either variable.
- The conditional guard `{% if document.service_date != blank %}` in neo.html's metadata section prevents any visible error if the variable is absent.
- Status: kept in template with a `<!-- service_date: not in Quaderno docs, pending engineering confirmation -->` comment.

---

## Previous Analysis Correction

The `/analysis/financial-values-debug.md` report concluded that `item.total_amount` "produces incorrect values" and recommended removing the Subtotal column. Both conclusions are now superseded:

1. `item.total_amount` is correct — values lower than `item.subtotal` reflect negative IRPF taxes, not a variable error.
2. The 7-column layout (with both Subtotal and Amount columns) is correct per spec and documentation.
3. The Subtotal column (removed in the previous fix) is restored.
