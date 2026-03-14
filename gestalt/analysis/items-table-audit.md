# Line Items Table Audit

**Date:** 2026-03-11
**Template:** gestalt.html

---

## Step 1 — Current Implementation: Exact Code

### Pre-scan (lines 704–709)
```liquid
{% assign has_discounts = false %}
{% for item in document.items %}
  {% if item.discount_rate > 0 %}
    {% assign has_discounts = true %}
  {% endif %}
{% endfor %}
```

### Colgroup (lines 712–732)
```html
<colgroup>
  <col style="width:40%"><!-- Description — fixed -->
  {% if has_discounts %}
    <col style="width:8%"><!-- Qty -->
    <col style="width:11%"><!-- Unit price -->
    <col style="width:8%"><!-- Discount -->
    <col style="width:11%"><!-- Subtotal -->
    <col style="width:8%"><!-- Tax rate -->
    <col style="width:14%"><!-- Amount -->
  {% else %}
    <col style="width:10%"><!-- Qty -->
    <col style="width:13%"><!-- Unit price -->
    <col style="width:13%"><!-- Subtotal -->
    <col style="width:10%"><!-- Tax rate -->
    <col style="width:14%"><!-- Amount -->
  {% endif %}
</colgroup>
```

### Column headers (lines 734–743)
```html
<tr class="items-header">
  <th class="col-left">{{ labels.description }}</th>
  <th class="col-right">{{ labels.quantity }}</th>
  <th class="col-right">{{ labels.unit_price }}</th>
  {% if has_discounts %}<th class="col-right">{{ labels.discount }}</th>{% endif %}
  <th class="col-right">{{ labels.subtotal }}</th>
  <th class="col-right">{{ labels.rate }}</th>
  <th class="col-right">{{ labels.amount }}</th>
</tr>
```

### Item row — column by column (lines 747–761)

**Column 1 — Description:**
```html
<td>{{ item.description }}</td>
```
No conditional, no filter.

**Column 2 — Quantity:**
```html
<td class="col-right">{{ item.quantity | precision }}</td>
```

**Column 3 — Unit Price:**
```html
<td class="col-right">{{ item.unit_price | precision }}</td>
```

**Column 4 — Discount (conditional):**
```html
{% if has_discounts %}
  <td class="col-right">
    {% if item.discount_rate > 0 %}{{ item.discount_rate | precision }}%{% else %}—{% endif %}
  </td>
{% endif %}
```
Only present when any item has a discount. Shows `—` for items with no discount.

**Column 5 — Subtotal:**
```html
<!-- item.subtotal = Qty × Unit Price (before discount, before tax) -->
<td class="col-right">{{ item.subtotal }}</td>
```
No filter.

**Column 6 — Tax Rate:**
```html
{% assign has_tax1 = false %}
{% assign has_tax2 = false %}
{% if item.tax_1_rate != blank and item.tax_1_rate != 0 %}{% assign has_tax1 = true %}{% endif %}
{% if item.tax_2_rate != blank and item.tax_2_rate != 0 %}{% assign has_tax2 = true %}{% endif %}
<td class="col-right">
  {% if has_tax1 == false and has_tax2 == false %}—
  {% elsif has_tax1 and has_tax2 %}{{ item.tax_1_rate | precision }}%<br>{{ item.tax_2_rate | precision }}%
  {% elsif has_tax1 %}{{ item.tax_1_rate | precision }}%
  {% elsif has_tax2 %}{{ item.tax_2_rate | precision }}%
  {% endif %}
</td>
```
4-scenario boolean logic handles: no tax (—), tax1 only, tax2 only, both stacked with `<br>`.

**Column 7 — Amount:**
```html
<!-- item.total_amount = Subtotal − Discount + Tax Amount.
     Can be lower than item.subtotal when negative taxes apply (e.g. IRPF).
     Documented in Quaderno Tags docs; confirmed correct. -->
<td class="col-right">{{ item.total_amount }}</td>
```
No filter.

---

## Step 2 — Variable Behavior Analysis

### Output formats (from Quaderno Tags documentation)

| Variable | Format | Before/after discount | Before/after tax |
|---|---|---|---|
| `item.unit_price` | `[amount] [ISO code]` e.g. `"55555.56 EUR"` | Before discount | Before tax |
| `item.quantity` | Number | N/A | N/A |
| `item.discount_rate` | Bare number, no `%` e.g. `10` | N/A | N/A |
| `item.tax_1_rate` | Bare number, no `%` e.g. `7` | N/A | N/A |
| `item.tax_2_rate` | Bare number, no `%` e.g. `-7` | N/A | N/A |
| `item.subtotal` | `[amount] [ISO code]` | Before discount | Before tax |
| `item.gross_amount` | `[amount] [ISO code]` | After discount | Before tax |
| `item.total_amount` | `[amount] [ISO code]` | After discount | After tax |

### Effect of `| precision` filter on currency strings

`item.unit_price` raw output: `"55555.556 EUR"`
`item.unit_price | precision` output: `"55555.56"` (strips ISO code, rounds to 2dp)

`item.subtotal` raw output: `"722222.222 EUR"` (no filter in gestalt.html → ISO code present)
`item.total_amount` raw output: `"650000.00 EUR"` (no filter in gestalt.html → ISO code present)

**Inconsistency:** Unit Price strips the ISO code via `| precision`; Subtotal and Amount do not. This is consistent with all 8 reference templates — they apply `| precision` to unit_price but not to subtotal. Likely intentional: the ISO code on monetary totals provides currency context.

---

## Step 3 — Carbonite Freezing Chamber Math Verification

### Inputs
- Qty: 13
- App price (tax-inclusive, discount-inclusive): 50,000 EUR per unit
- Discount: 10%
- IGIC: +7%
- IRPF: −7%
- Net tax: 0% (IGIC and IRPF cancel exactly)

### Working backwards from tax-inclusive pricing

When the Quaderno app is set to "Include taxes and discounts" mode, the entered price (50,000) is the **final customer price after tax and after discount**. Quaderno back-calculates the base unit price:

```
Final price (per unit) = base_unit_price × (1 − discount_rate) × (1 + net_tax_rate)
50,000 = base_unit_price × (1 − 0.10) × (1 + 0)
50,000 = base_unit_price × 0.90
base_unit_price = 50,000 / 0.90 = 55,555.556 EUR
```

Net tax = 0% (IGIC 7% + IRPF −7%), so tax does not affect the back-calculation.

### Expected variable values

| Variable | Calculation | Expected value |
|---|---|---|
| `item.unit_price` | Back-calculated base price | `55,555.556 EUR` → filtered: `55555.56` |
| `item.quantity` | Given | `13` |
| `item.discount_rate` | Given | `10` (bare number, no %) |
| `item.subtotal` | 13 × 55,555.556 | `722,222.222 EUR` |
| `item.gross_amount` | 722,222.222 × (1 − 0.10) | `650,000.000 EUR` |
| IGIC 7% tax amount | 650,000 × 0.07 | `+45,500.000 EUR` |
| IRPF −7% tax amount | 650,000 × (−0.07) | `−45,500.000 EUR` |
| Net tax | 45,500 − 45,500 | `0.000 EUR` |
| `item.total_amount` | gross_amount + net_tax = 650,000 + 0 | `650,000.000 EUR` |
| `item.tax_1_rate` | IGIC rate | `7` |
| `item.tax_2_rate` | IRPF rate | `-7` |

### What gestalt.html renders for each column

| Column | Code | Expected output |
|---|---|---|
| Description | `{{ item.description }}` | "Carbonite Freezing Chamber…" |
| Qty | `{{ item.quantity \| precision }}` | `13` |
| Unit Price | `{{ item.unit_price \| precision }}` | `55,555.56` (no ISO code) |
| Discount | `{{ item.discount_rate \| precision }}%` | `10%` or `10.00%` (see Bug 2 below) |
| Subtotal | `{{ item.subtotal }}` | `722,222.222 EUR` (ISO code present) |
| Tax Rate | has_tax1+has_tax2 logic | `7%` *(line 1)* / `-7%` *(line 2)* |
| Amount | `{{ item.total_amount }}` | `650,000.00 EUR` (ISO code present) |

---

## Step 4 — Cross-Reference with App Values

### App values (from test invoice, "Include taxes and discounts" mode)
- Qty: 13
- Price: 50,000.00 (tax-inclusive per unit as entered)
- Discount: 10.00%
- Taxes: IGIC 7.00%, IRPF −7.00%
- Amount: 722,222.222 EUR

### Discrepancy analysis

| Column | App shows | Gestalt renders | Match? | Notes |
|---|---|---|---|---|
| Qty | 13 | 13 | ✓ | |
| Unit Price | 50,000.00 | 55,555.56 | ✗ | **See Bug 1** |
| Discount | 10.00% | 10% or 10.00% | ✓ | Minor precision formatting difference |
| Subtotal | — (app has no subtotal column) | 722,222.222 EUR | N/A | iso code included |
| Tax Rate | IGIC 7%, IRPF −7% | 7% / −7% | ✓ | Negative rate correct |
| Amount | 722,222.222 EUR | 650,000.00 EUR | ✗ | **See Bug 3** |

---

## Step 5 — Pricing Mode Investigation

### Tax-exclusive mode vs tax-inclusive mode

In **tax-exclusive** mode (standard):
- The entered price is the base price before tax and before discount
- `item.unit_price` = the entered price (e.g., 50,000)
- `item.subtotal` = 13 × 50,000 = 650,000
- `item.gross_amount` = 650,000 × 0.9 = 585,000
- `item.total_amount` = 585,000 + 0 = 585,000 (net tax 0%)

In **tax-inclusive** mode (current test):
- The entered price is the final customer price (after all taxes and after discount)
- Quaderno back-calculates: `item.unit_price` = 50,000 / 0.9 = 55,555.556
- `item.subtotal` = 13 × 55,555.556 = 722,222.222
- `item.gross_amount` = 722,222.222 × 0.9 = 650,000
- `item.total_amount` = 650,000 + 0 = 650,000 (net tax 0%)

### What the app's "Amount" column represents

The app's single "Amount" column (722,222.222) matches `item.subtotal` in tax-inclusive mode. This is the back-calculated Qty × base_unit_price value — NOT the tax-inclusive customer-facing price.

The app's "Price" column (50,000.00) is the customer-facing entered price — which is **not** the value returned by `item.unit_price` in tax-inclusive mode.

### Does tax-inclusive mode break the column definitions?

**Unit Price column:** In tax-inclusive mode, `item.unit_price` = 55,555.56 (back-calculated base price), not the entered 50,000. The customer-entered price is not accessible as a Liquid variable. This is a platform behavior — the template cannot show "50,000.00" because no variable holds that value.

**Subtotal column:** `item.subtotal` = 722,222.222 = Qty × back-calculated unit_price. The column label "Subtotal" is still semantically correct (it IS the subtotal before discount and tax). The value differs from what a customer might expect if they entered 50,000 as the price.

**Amount column:** `item.total_amount` = 650,000. In tax-inclusive mode with net-zero taxes, this equals `item.gross_amount` (post-discount, pre-tax net amount). The column label "Amount" with this definition is semantically correct for a row's bottom-line contribution.

---

## Step 6 — All Bugs and Inconsistencies

### Bug 1 — Unit Price column: shows back-calculated base price, not entered price

**Severity:** Medium — cosmetic/UX, not a calculation error.

**Code:** `{{ item.unit_price | precision }}` (line 750)

**Issue:** In tax-inclusive pricing mode, `item.unit_price` returns the back-calculated pre-discount, pre-tax base price (55,555.556), not the customer-facing entered price (50,000.00). The app's "Price" column shows 50,000.00. Gestalt's Unit Price column shows 55,555.56.

**Root cause:** Quaderno stores `item.unit_price` as the pre-discount, pre-tax base price. In tax-inclusive mode, the back-calculation changes this value. There is no separate `item.entered_price` or `item.customer_price` Liquid variable. This is a Quaderno data model limitation, not a template bug.

**Impact:** The Unit Price column will not match the price the seller or buyer recognizes from the original quote/order when tax-inclusive pricing is used.

**Fix:** Not fixable from the template — requires a Quaderno platform variable. Flag for engineering.

---

### Bug 2 — Discount: `| precision` vs `| precision: 0`

**Severity:** Low — cosmetic only.

**Code:** `{{ item.discount_rate | precision }}%` (line 752)

**Issue:** All 5 reference templates that show discount use `| precision: 0` (zero decimal places), displaying "10%". Gestalt uses `| precision` (no argument), which may display "10.00%" (2 decimal places) or "10%" (no trailing zeros) depending on Quaderno's filter implementation.

**Reference template pattern:** `{{ item.discount_rate | precision: 0 }}%`

**Impact:** Possible minor inconsistency: "10.00%" vs "10%". The app shows "10.00%" which would mean gestalt actually matches the app better, but diverges from all reference templates.

**Recommended fix:** Align with reference templates: change to `| precision: 0` unless 2dp is preferred to match the app. Note: if a discount is 10.5%, `| precision: 0` would show "11%" (rounded) while `| precision` would show "10.5%". For non-integer discounts, `| precision` is more accurate.

---

### Bug 3 — Amount column: `item.total_amount` vs `item.subtotal` (app discrepancy)

**Severity:** Medium — values differ from app display. Intentional per spec.

**Code:** `{{ item.total_amount }}` (line 760)

**Issue:** The app's "Amount" column shows 722,222.222 (= `item.subtotal`). Gestalt's Amount column shows 650,000.00 (= `item.total_amount`, post-discount post-tax). The values differ.

**Is this a bug?** No — it is intentional. Gestalt has a 7-column layout with both a Subtotal column AND an Amount column. The financial-columns-debug.md documents this explicitly:
- Subtotal column = `item.subtotal` = 722,222.222 ← matches app's Amount value
- Amount column = `item.total_amount` = 650,000 ← post-discount, post-tax

The app has only one amount column (their "Amount" = gestalt's "Subtotal"). Gestalt adds an extra column showing the post-discount, post-tax figure, which is unique to this template's design.

**Impact:** A user comparing the gestalt invoice to the Quaderno app will see the Subtotal column match the app's Amount column (722,222.222), and gestalt's Amount column shows additional information (650,000). This is correct and expected per spec.

**Recommended action:** No change. Add a comment clarifying that the Amount column differs from the app by design.

---

### Bug 4 — Subtotal and Amount columns missing `| precision` filter

**Severity:** Low — potential ISO code display in cells.

**Code:** `{{ item.subtotal }}` and `{{ item.total_amount }}` (lines 755, 760)

**Issue:** These variables output `"[amount] [ISO code]"` format. Without `| precision`, the ISO code (e.g. "EUR") appears in the cell, producing "722,222.222 EUR" and "650,000.00 EUR".

**Is this a bug?** No — all 8 reference templates also apply no filter to `item.subtotal`. The ISO code in the cell is consistent with production templates. The currency label provides useful context on each row.

**Potential concern:** If the ISO code looks visually cluttered in the narrower Amount column (14% width), consider `| precision` to strip it. But this is a design choice, not a bug.

**Recommended action:** No change. Consistent with all reference templates.

---

### Bug 5 — Tax Rate column: negative IRPF rate displays as "−7%"

**Severity:** Informational — correct but potentially confusing to end customers.

**Code:** Tax rate cell with `{{ item.tax_2_rate | precision }}%` (line 756)

**Issue:** IRPF is a withholding tax at −7%. The Tax Rate cell for items with both IGIC and IRPF displays:
```
7%
-7%
```
The negative value is mathematically correct and reflects the actual tax rates. However, it may confuse customers who are unfamiliar with withholding taxes.

**Is this a bug?** No — it is accurate. The rates exactly match what Quaderno stores and the signed convention is standard for IRPF.

**Recommended action:** No change. The negative sign is correct and informative.

---

### Bug 6 — `labels.rate` used for Tax Rate column header

**Severity:** Low — label may not be the most precise but is used by all reference templates.

**Code:** `{{ labels.rate }}` (line 741)

**Issue:** `labels.rate` is used for the Tax Rate column header. This is consistent with the reference templates (modern, classic, frame, mono, minimal all use `labels.rate` for their tax rate column header). However, `labels.rate` might translate to just "Rate" rather than "Tax Rate" in some locales.

**Recommended action:** No change. Consistent with all reference templates.

---

## Step 7 — Proposed Fixes

### Fix 1 — Discount filter: `| precision` → `| precision: 0` (optional)

**Current:** `{{ item.discount_rate | precision }}%`
**Proposed:** `{{ item.discount_rate | precision: 0 }}%`

**Trade-off:** `precision: 0` rounds to integer (e.g. 10.5% → "11%") and matches all reference templates. `precision` (no args) preserves decimals (e.g. "10.5%") and likely matches the app's display.

**Recommendation:** Keep `| precision` (no args) for accuracy on non-integer discounts. This matches the app display and is more informative than rounding to integers. Document the divergence from reference templates as intentional.

---

### Fix 2 — Flag Unit Price discrepancy for engineering

Add a comment in the template:
```html
<!-- item.unit_price returns the back-calculated pre-discount, pre-tax base price.
     In tax-inclusive pricing mode, this differs from the customer-facing entered price.
     No Liquid variable exists for the entered price. Engineering confirmation needed. -->
<td class="col-right">{{ item.unit_price | precision }}</td>
```

No code change — documentation only.

---

### Fix 3 — Clarify Amount column intent

Add a comment distinguishing gestalt's Amount column from the app's Amount column:
```html
<!-- item.total_amount = Subtotal − Discount + Tax (post-discount, post-tax per line).
     This differs from the Quaderno app's "Amount" column (which = item.subtotal).
     Gestalt has both a Subtotal column (item.subtotal) and an Amount column (item.total_amount)
     — a 7-column design unique to this template. See /analysis/financial-columns-debug.md. -->
<td class="col-right">{{ item.total_amount }}</td>
```

No code change — documentation only.

---

## Summary

| # | Issue | Severity | Code change? |
|---|---|---|---|
| 1 | Unit Price shows back-calculated base price, not entered price in tax-inclusive mode | Medium | No — platform limitation |
| 2 | `\| precision` vs `\| precision: 0` on discount_rate | Low | Optional |
| 3 | Amount (total_amount) differs from app Amount (subtotal) | Medium | No — intentional by spec |
| 4 | Subtotal/Amount missing `\| precision` (ISO code in cells) | Low | No — consistent with all templates |
| 5 | Negative IRPF rate displays as "−7%" | Informational | No — correct |
| 6 | `labels.rate` for Tax Rate header | Low | No — consistent with all templates |

**No critical bugs found.** The implementation is consistent with Quaderno documentation and reference templates. Two medium issues exist (unit price discrepancy in tax-inclusive mode, Amount vs app Amount) but both are platform behaviors or intentional design choices, not fixable via template changes.
