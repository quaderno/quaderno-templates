# Neo Template — Known Issues & Pending Items

## Platform Limitations (requires Quaderno engineering)

### 1. Sticky Footer & Pagination
The footer cannot be fixed to the bottom of every page,
and dynamic page numbers (Page X of Y) cannot be generated
from within the template. Both require wkhtmltopdf
--footer-html server-side support.

Current state: footer renders at the end of the document
with a static "Page 1 of 1" placeholder.

Five CSS/HTML approaches were tested and documented in
/neo/analysis/footer-audit.md and footer-debug.md.

### 2. Service Date
No document.service_date variable or labels.service_date
label exists in Quaderno's documentation or any existing
template. The metadata section includes a conditional for
it that will render if/when the variable becomes available.

Documented in /neo/analysis/metadata-debug.md.

## Pending Engineering Confirmation

### 3. Line Item Pricing Mode
item.total_amount is used for the Total column (after
discount + tax). Awaiting confirmation on whether this
variable's behavior changes depending on the "Include
taxes and discounts" pricing mode setting.

Documented in /neo/analysis/financial-columns-debug.md.

### 4. Tax Type Distinction at Item Level
The Neo template introduces a per-line Tax Rate column —
a feature not present in any of the 8 existing templates.
This has surfaced a compliance issue: Quaderno attaches up
to two taxes per item (item.tax_1_rate, item.tax_2_rate),
but these can be fundamentally different in nature:

- Item-level / consumption taxes (VAT, GST, IGIC, IVA)
  belong per line item
- Document-level / withholding taxes (IRPF, TDS, Retenciones)
  belong only in the Tax Breakdown table, not per line item

Currently both tax rates are displayed stacked in the Tax
Rate column. This is misleading for withholding taxes.

The template needs a Liquid variable or property to
distinguish consumption taxes from withholding taxes so
only item-level taxes appear in the per-line Tax Rate
column. Until then, both rates are shown.

This distinction matters globally (Spain, India, Latin
America, and other jurisdictions with dual tax systems).

### 5. Unit Price in Tax-Inclusive Pricing Mode
In tax-inclusive pricing mode ("Include taxes and discounts"),
item.unit_price returns the back-calculated pre-discount,
pre-tax base price — not the customer-facing entered price.

Example: a seller enters 50,000 EUR per unit (tax-inclusive).
The template displays 55,555.56 EUR (the back-calculated base
price: 50,000 ÷ 0.9 for a 10% discount). There is no Liquid
variable that returns the original entered price.

This means the Unit Price column on the invoice won't match
the price the seller or buyer recognizes from the original
quote or order when tax-inclusive pricing is used.

Documented in /neo/analysis/items-table-audit.md.

### 6. Missing Quaderno Labels
The following user-facing text is hardcoded because no
Quaderno label exists for localisation:

- "Currency conversion" — section title (no labels.currency_conversion)
- "Exchange rate" — row label (labels.exchange resolves to
  "Exchange" which is incorrect; no labels.exchange_rate exists)
- "Deposit paid at..." — full sentence with embedded variables
  (no label system can support this without a template engine change)

Additionally, these labels are used in the template but
are not confirmed in Quaderno's official documentation:
- labels.shipping_address
- labels.service_date
- labels.registration_number
- labels.taxable_base
- labels.tax_amount
- labels.amount_paid
- labels.balance_due

These may work in practice (some are used in existing
templates) but are not guaranteed. Each has a default
fallback value.

Documented in /neo/analysis/hardcoded-labels-audit.md.

## Status
This is a work-in-progress template. The items above
require Quaderno engineering input before the template
can be considered production-ready.
