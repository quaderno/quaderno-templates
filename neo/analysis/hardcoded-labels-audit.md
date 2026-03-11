# Hardcoded Labels Audit — neo.html

**Date:** 2026-03-11

---

## Complete Text Inventory

| Line | Text / Variable | Type | Notes |
|------|----------------|------|-------|
| 508 | `{{ title }}` | label (dynamic) | Resolves to document type label automatically |
| 515–521 | `{{ document.state \| capitalize }}` | variable | State rendered as "Draft", "Paid" etc. — not user text |
| 528 | `{{ payment.payment_method_text }}` | variable | — |
| 534 | `{{ document.number }}` | variable | — |
| 553 | `{{ labels.subject }}:` | label + `:` | Colon is formatting — Category C |
| 586 | `{{ labels.issue_date }}` | label ✅ | — |
| 589 | `{{ labels.due_date }}` | label ✅ | — |
| 596 | `{{ labels.service_date }}` | label ⚠️ | Undocumented — see Category B |
| 599 | `{{ labels.po_number }}` | label ✅ | — |
| 602 | `{{ labels.valid_until }}` | label ✅ | — |
| 605 | `{{ labels.related_document }}` | label ✅ | — |
| 645 | `{{ labels.from }}` | label ✅ | — |
| 650 | `{{ labels.registration_number }}:` | label ⚠️ + `:` | Undocumented label; colon is formatting |
| 653 | `{{ labels.tax_id }}:` | label ✅ + `:` | Colon is formatting — Category C |
| 663 | `{{ labels.to }}` | label ✅ | — |
| 687 | `{{ labels.shipping_address \| default: 'Shipping address' }}` | label ⚠️ + fallback | Undocumented — see Step 4 |
| 736 | `{{ labels.description }}` | label ✅ | — |
| 737 | `{{ labels.quantity }}` | label ✅ | — |
| 738 | `{{ labels.unit_price }}` | label ✅ | — |
| 739 | `{{ labels.subtotal }}` | label ✅ | — |
| 740 | `{{ labels.discount }}` | label ✅ | — |
| 742 | `{{ labels.rate }}` | label ✅ | — |
| 744 | `{{ labels.total }}` | label ✅ | — |
| 761 | `—` | hardcoded | Em dash for empty discount cell — Category C |
| 763 | `—` | hardcoded | Em dash for empty tax rate cell — Category C |
| 791 | `{{ labels.tax \| default: 'Tax name' }}` | label ⚠️ + fallback | Undocumented label — see Step 4 |
| 792 | `{{ labels.taxable_base \| default: 'Taxable amount' }}` | label ⚠️ + fallback | Undocumented label — see Step 4 |
| 793 | `{{ labels.rate \| default: 'Tax rate' }}` | label ✅ + fallback | Label documented; default unnecessary — see Step 4 |
| 794 | `{{ labels.tax_amount \| default: 'Total tax per rate' }}` | label ⚠️ + fallback | Undocumented label — see Step 4 |
| 810 | `{{ labels.tax \| default: 'Total Tax' }}*` | label ⚠️ + fallback + `*` | Undocumented; `*` is formatting — Category C |
| 823 | `{{ labels.subtotal }}` | label ✅ | — |
| 828 | `{{ labels.discount }}` | label ✅ | — |
| 833 | `{{ labels.tax \| default: 'Total Tax' }}*` | label ⚠️ + fallback + `*` | Same as line 810 |
| 837 | `{{ labels.total }}` | label ✅ | — |
| 844 | `{{ labels.amount_paid \| default: 'Amount paid' }}` | label ⚠️ + fallback | Undocumented — see Step 4 |
| 850 | `{{ labels.balance_due \| default: 'Balance due' }}` | label ⚠️ + fallback | Undocumented — see Step 4 |
| 868 | `Currency conversion` | **hardcoded** | No label exists — Category B |
| 876 | `Exchange rate` | **hardcoded** | No label exists — Category B |
| 877 | `1 {{ document.currency }} = ... {{ account.currency }}` | variables + formatting | Structural formatting — Category C |
| 881 | `{{ labels.tax }}* ({{ account.currency }})` | label ⚠️ + `*` + variable | `*` and `()` are formatting |
| 885 | `{{ labels.total }}` | label ✅ | — |
| 893 | `{{ labels.balance_due \| default: 'Balance due' }}` | label ⚠️ + fallback | Undocumented |
| 901 | `Deposit paid at ... on ... (...)` | **hardcoded sentence** | No label exists — Category B |
| 922 | `{{ labels.payment_details }}` | label ✅ | — |
| 926 | `{{ labels.notes }}` | label ✅ | — |
| 960 | `{{ labels.invoice }}:` | label ✅ + `:` | Colon is formatting — Category C |
| 963 | `Page 1 of 1` | **hardcoded** | Platform limitation — Category C |

---

## Step 3 — Categorised Findings

### Category A — Label exists in Quaderno docs but not used
*These are fixable bugs.*

**None found.** Every piece of user-facing text either uses a `labels.*` variable or is hardcoded for a reason (Categories B or C). There are no cases where a documented label is available and neo.html uses a hardcoded string instead.

---

### Category B — Hardcoded, no label exists
*These need new Quaderno labels or accepted as hardcoded with localisation comments.*

| Line | Text | Notes |
|------|------|-------|
| 868 | `Currency conversion` | No `labels.currency_conversion` exists. Already has localisation comment. |
| 876 | `Exchange rate` | `labels.exchange` = "Exchange" (wrong); no `labels.exchange_rate` in docs. Already flagged. |
| 901 | `Deposit paid at 1 {cur} = {rate} {acct_cur} on {date} ({amount} {cur})` | Structural sentence with variables. No label system can cover this without a template engine change. Needs engineering. |

---

### Category C — Intentionally hardcoded
*Formatting characters and structural text that cannot or should not be labels.*

| Line | Text | Reason |
|------|------|--------|
| 553, 650, 653, 960 | `:` after labels | Punctuation, not translatable content |
| 761, 763 | `—` | Em dash for empty numeric cell — formatting convention |
| 810, 833, 881 | `*` in "Tax*" | Asterisk footnote marker referring to tax breakdown — formatting |
| 829, 845 | `-` before discount/amount_paid values | Accounting sign convention — not text |
| 763 | `%` after tax rates | Unit symbol — not translatable |
| 877 | `1 ... = ...` | Exchange rate formula structure — not translatable |
| 963 | `Page 1 of 1` | Static placeholder pending engineering support — documented in KNOWN-ISSUES.md |

---

## Step 4 — Default Fallback Audit

| Line | Variable | Label in docs? | Assessment |
|------|----------|---------------|------------|
| 687 | `labels.shipping_address \| default: 'Shipping address'` | ❌ Not documented | Default is the only thing that can render. Label variable is speculative. |
| 791 | `labels.tax \| default: 'Tax name'` | ❌ Not in official docs | Used in all 8 reference templates without a default — `labels.tax` works in practice. Default should be `'Tax'` not `'Tax name'` to match what `labels.tax` actually resolves to. |
| 792 | `labels.taxable_base \| default: 'Taxable amount'` | ❌ Not documented | Used in newer reference templates without a default — likely works. |
| **793** | **`labels.rate \| default: 'Tax rate'`** | **✅ In docs** | **Default is unnecessary — `labels.rate` is confirmed. Remove the default.** |
| 794 | `labels.tax_amount \| default: 'Total tax per rate'` | ❌ Not documented | Not used in any reference template; most likely does not exist. Default is the only thing rendering. |
| 810, 833 | `labels.tax \| default: 'Total Tax'` | ❌ Not documented | Default differs from `labels.tax`'s actual value ("Tax"). Should be `'Tax'` for consistency. |
| 844 | `labels.amount_paid \| default: 'Amount paid'` | ❌ Not documented | Default is the only thing that can render reliably. |
| 850, 893 | `labels.balance_due \| default: 'Balance due'` | ❌ Not documented | Default is the only thing that can render reliably. |

---

## Step 5 — Summary

| Count | Category |
|-------|----------|
| 0 | Category A — fixable bugs (label exists, not used) |
| 3 | Category B — needs new Quaderno labels (Currency conversion, Exchange rate, Deposit note sentence) |
| 8 | Category C — intentionally hardcoded (formatting characters, structural text) |
| 1 | Unnecessary default fallback (`labels.rate` — label confirmed in docs) |
| 2 | Incorrect default values (`labels.tax` defaults say 'Tax name' / 'Total Tax'; actual value is 'Tax') |

### Recommended fixes (minor cleanup, not bugs)

1. **Line 793:** Remove `| default: 'Tax rate'` — `labels.rate` is confirmed in docs.
2. **Lines 791, 810, 833:** Change `| default: 'Tax name'` → `| default: 'Tax'` and `| default: 'Total Tax'` → `| default: 'Tax'` to match what `labels.tax` actually resolves to in English.
3. **Lines B above:** Keep all Category B items as hardcoded with existing localisation comments — no Quaderno label fix is possible without engineering support.

### Overall assessment

Neo.html has **excellent labels coverage**. All documented Quaderno labels are used correctly. The only issues are minor: one unnecessary default fallback and two mismatched default values. The three Category B hardcoded strings are genuine gaps in Quaderno's label system, not template errors.
