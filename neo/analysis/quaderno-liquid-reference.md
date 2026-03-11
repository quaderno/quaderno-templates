# Quaderno Liquid Reference

Sources:
- Quaderno template docs: https://developers.quaderno.io/templates/
- Liquid language docs: https://shopify.github.io/liquid/basics/introduction/
- Cross-referenced with all 8 templates in this repo (see `template-patterns.md`)

Legend for cross-reference column:
- ✅ In docs AND used in templates
- 📄 In docs, NOT used in any template
- ⚠️ Used in templates, NOT listed in official docs
- 🔶 In docs only for common-use-cases examples, not in main variable table

---

## 1. Account Variables

| Variable | Description | Status |
|---|---|---|
| `{{ account.full_name }}` | Legal full name of the account | ✅ |
| `{{ account.trade_name }}` | Trade / brand name | ✅ |
| `{{ account.tax_id }}` | Seller's VAT / tax ID | ✅ |
| `{{ account.formatted_address }}` | Full address as a single formatted string | ✅ |
| `{{ account.logo_url }}` | URL of the account logo | ✅ |
| `{{ account.color_scheme }}` | Hex colour, used directly in CSS `<style>` blocks | ✅ |
| `{{ account.phone_1 }}` | Phone number | ✅ (classic only) |
| `{{ account.street_line_1 }}` | First line of street address | 📄 |
| `{{ account.street_line_2 }}` | Second line of street address | 📄 |
| `{{ account.postal_code }}` | Postal code | 📄 |
| `{{ account.city }}` | City | 📄 |
| `{{ account.region }}` | Region / state | 📄 |
| `{{ account.country }}` | Country ISO code | 📄 |
| `{{ account.email }}` | E-mail address | 📄 |
| `{{ account.web }}` | Website URL | 📄 |
| `{{ account.registration_number }}` | Company registration number | ⚠️ Used in all 8 templates, absent from docs |
| `{{ account.currency }}` | Account's default currency code | ⚠️ Used in exchange-rate rows, absent from docs |

---

## 2. Document Variables

### Core fields

| Variable | Description | Status |
|---|---|---|
| `{{ document.number }}` | Unique sequential document identifier | ✅ |
| `{{ document.contact }}` | Customer full name at time of creation | ✅ |
| `{{ document.issue_date }}` | Date of issue | ✅ |
| `{{ document.subject }}` | Optional summary description | ✅ |
| `{{ document.po_number }}` | Purchase order number | ✅ |
| `{{ document.billing_address }}` | Customer billing address | ✅ |
| `{{ document.tax_id }}` | Customer tax ID at time of creation | ✅ |
| `{{ document.type }}` | Invoice / Receipt / Credit / Expense / Estimate / Recurring | ✅ (common-use-cases) |
| `{{ document.state }}` | Status: "outstanding", "paid", "refunded", etc. | ✅ |
| `{{ document.currency }}` | Three-letter ISO currency code (uppercase) | ✅ |
| `{{ document.due_date }}` | Payment due date | ✅ |
| `{{ document.valid_until }}` | Estimate expiry date | ✅ |
| `{{ document.notes }}` | Any note added to the document | ✅ |
| `{{ document.payment_details }}` | Payment instructions | ✅ |
| `{{ document.legal }}` | Account's legal notes (from Preferences) | ✅ |

### Financial fields

| Variable | Description | Status |
|---|---|---|
| `{{ document.subtotal }}` | Amount before discount and tax | ✅ |
| `{{ document.discount }}` | Discount amount (numeric, check `> 0`) | ✅ |
| `{{ document.gross_amount }}` | Subtotal − Discount | 📄 (shown in common-use-cases, not in templates) |
| `{{ document.total }}` | Final total | ✅ |
| `{{ document.amount_paid }}` | Amount already paid | 📄 |
| `{{ document.amount_due }}` | Total − Amount Paid | 📄 |
| `{{ document.exchange }}` | Truthy when transaction currency ≠ account currency | ✅ |
| `{{ document.exchange_rate }}` | Exchange rate used for conversion | 📄 |
| `{{ document.tax_amount }}` | Total tax amount | ⚠️ Used in modern/classic/frame/mono/minimal; absent from docs variable table |
| `{{ document.tax_amount_exchange }}` | Tax amount in account's currency | ⚠️ Used in all templates for exchange rows; absent from docs |
| `{{ document.total_exchange }}` | Total in account's currency | ⚠️ Used in all templates for exchange rows; absent from docs |

### Related document fields

| Variable | Description | Status |
|---|---|---|
| `{{ document.related_document_number }}` | Number of related doc (invoice ↔ credit note) | ✅ |
| `{{ document.related_document_type }}` | "Invoice" or "Credit" | ✅ |
| `{{ document.related_document_date }}` | Related document issue date | ✅ |

### Boolean / metadata fields

| Variable | Description | Status |
|---|---|---|
| `{{ document.reverse_charged? }}` | True when reverse charge applies | 🔶 |
| `{{ document.auto? }}` | True when imported via integration | 📄 |
| `{{ document.secure_id }}` | Non-revealing document identifier | 📄 |
| `{{ document.processor }}` | Integration name used to import | 📄 |
| `{{ document.processor_id }}` | External ID from integration | 📄 |
| `{{ document.days_past_due }}` | Days past the due date | 📄 |
| `{{ document.shipping_address }}` | Shipping address (physical products only) | 📄 |
| `{{ document.country }}` | Customer billing country | 📄 |
| `{{ document.tag_list }}` | Array of classification tags | 📄 |
| `{{ document.payments }}` | Array of payment objects (use `| last` for most recent) | ⚠️ Used in all templates; not listed explicitly in docs |

### Custom metadata

Any field sent via the API `custom_metadata` hash is accessible as `{{ document.your_field_name }}`.

---

## 3. Document Item Variables

Iterated with `{% for item in document.items %}`.

> **Naming discrepancy:** The docs define these as `document_item.*` but all 8 templates use `item.*` as the loop variable name. Both work — the variable name is the one declared in the `for` tag.

| Variable | Description | Status |
|---|---|---|
| `{{ item.description }}` | Line item description | ✅ |
| `{{ item.quantity }}` | Quantity | ✅ |
| `{{ item.unit_price }}` | Unit price | ✅ |
| `{{ item.subtotal }}` | Quantity × Unit Price | ✅ |
| `{{ item.discount_rate }}` | Discount percentage (check `> 0` before display) | ✅ |
| `{{ item.gross_amount }}` | Subtotal − Discount | 📄 (shown in common-use-cases) |
| `{{ item.total_amount }}` | Subtotal − Discount + Tax | 📄 |
| `{{ item.tax_1_rate }}` | Rate of first tax applied | 📄 |
| `{{ item.tax_2_rate }}` | Rate of second (local) tax applied | 📄 |

---

## 4. Document Tax Variables

Iterated with `{% for document_tax in document.taxes %}` (newer templates) or `{% for tax in document.taxes %}` (older templates).

> **Pattern discrepancy:** The docs use `document_tax.*` as the example variable name. Older templates (destijl, newspaper, professional) use `tax.*` and render taxes inline as `tax.label × tax.taxable_base | precision` without a breakdown table. Newer templates use `document_tax.*` with a separate breakdown table.

| Variable | Description | Status |
|---|---|---|
| `{{ document_tax.label }}` | Human-readable tax name, e.g. "VAT 21%" | ✅ |
| `{{ document_tax.taxable_base }}` | The base amount tax is calculated on | ✅ |
| `{{ document_tax.rate }}` | Tax rate percentage | ✅ |
| `{{ document_tax.amount }}` | Tax amount charged | ✅ |
| `{{ document_tax.country }}` | Tax jurisdiction country | 📄 |
| `{{ document_tax.region }}` | Tax jurisdiction region | 📄 |
| `{{ document_tax.currency }}` | Tax currency | 📄 |
| `{{ document_tax.local_taxable_base }}` | Taxable base in customer's currency | 📄 |
| `{{ document_tax.local_amount }}` | Tax amount in customer's currency | 📄 |
| `document.taxes.size` | Number of taxes (used as `{% if document.taxes.size > 0 %}`) | ✅ |

---

## 5. Payment Variables

Accessed after `{% assign payment = document.payments | last %}`.

> **Naming discrepancy:** The docs define these as `document_payment.*` but templates always use a locally-assigned name (conventionally `payment.*`).

| Variable | Description | Status |
|---|---|---|
| `{{ payment.payment_method_text }}` | Payment method description | ✅ |
| `{{ payment.date }}` | Payment date | 📄 |
| `{{ payment.amount }}` | Payment amount | 📄 |

---

## 6. Contact Variables

| Variable | Description | Status |
|---|---|---|
| `{{ contact.full_name }}` | First + last name | ✅ (newer templates; fallback: `| default: document.contact`) |
| `{{ contact.country }}` | Country ISO code — passed to `date` filter | ✅ |
| `{{ contact.contact_person }}` | Contact person at a company | ✅ |
| `{{ contact.department }}` | Department at a company | ✅ |
| `{{ contact.eu_member? }}` | True when contact is EU-based | 🔶 (common-use-cases) |
| `{{ contact.language }}` | Contact's language code | ⚠️ Used on `<html lang="">` in all templates; absent from docs |
| `{{ contact.first_name }}` | First name | 📄 |
| `{{ contact.last_name }}` | Last name | 📄 |
| `{{ contact.tax_id }}` | Contact's current tax ID | 📄 |
| `{{ contact.formatted_address }}` | Full formatted address | 📄 |
| `{{ contact.street_line_1 }}` | First street line | 📄 |
| `{{ contact.street_line_2 }}` | Second street line | 📄 |
| `{{ contact.postal_code }}` | Postal code | 📄 |
| `{{ contact.city }}` | City | 📄 |
| `{{ contact.region }}` | Region / state | 📄 |
| `{{ contact.phone_1 }}` | Phone 1 | 📄 |
| `{{ contact.phone_2 }}` | Phone 2 | 📄 |
| `{{ contact.email }}` | E-mail | 📄 |
| `{{ contact.web }}` | Website | 📄 |

---

## 7. Top-level Variables

| Variable | Description | Status |
|---|---|---|
| `{{ title }}` | Resolves to `labels.invoice`, `labels.estimate`, etc. based on document type | ✅ |

---

## 8. Label Keys

Labels auto-translate based on the contact's language. **Labels work in document templates only — not in email templates.**

### Documented labels

| Key | Default English output |
|---|---|
| `{{ labels.invoice }}` | Invoice |
| `{{ labels.estimate }}` | Estimate |
| `{{ labels.receipt }}` | Receipt |
| `{{ labels.number }}` | Number |
| `{{ labels.issue_date }}` | Issue date |
| `{{ labels.subject }}` | Subject |
| `{{ labels.po_number }}` | PO Number |
| `{{ labels.description }}` | Description |
| `{{ labels.quantity }}` | Quantity |
| `{{ labels.unit_price }}` | Unit price |
| `{{ labels.amount }}` | Amount |
| `{{ labels.subtotal }}` | Subtotal |
| `{{ labels.discount }}` | Discount |
| `{{ labels.total }}` | Total |
| `{{ labels.exchange }}` | Exchange |
| `{{ labels.due_date }}` | Due date |
| `{{ labels.valid_until }}` | Valid until |
| `{{ labels.payment_details }}` | Payment details |
| `{{ labels.notes }}` | Notes |
| `{{ labels.tax_id }}` | Tax ID |
| `{{ labels.phone }}` | Phone |
| `{{ labels.email }}` | E-mail |
| `{{ labels.from }}` | From |
| `{{ labels.to }}` | To |
| `{{ labels.paid }}` | Paid |
| `{{ labels.related_document }}` | Related document |

### Labels used in templates but absent from docs

| Key | Used in | Notes |
|---|---|---|
| `{{ labels.tax }}` | All templates | Tax total label in totals section |
| `{{ labels.registration_number }}` | All templates | Seller registration number label |
| `{{ labels.tax_breakdown }}` | modern, classic, frame, mono, minimal, professional | Header for the tax breakdown table |
| `{{ labels.taxable_base }}` | modern, classic, frame, mono, minimal, professional | Column header in tax breakdown |
| `{{ labels.rate }}` | modern, classic, frame, mono, minimal, professional | Column header in tax breakdown |

### Labels in docs but not used in any template

| Key | Notes |
|---|---|
| `{{ labels.invoice }}` | `{{ title }}` is used instead (resolves automatically) |
| `{{ labels.estimate }}` | Same — use `{{ title }}` |
| `{{ labels.receipt }}` | Same — use `{{ title }}` |
| `{{ labels.exchange }}` | Not rendered as a label; exchange values shown directly |
| `{{ labels.email }}` | Not used in any sample template |

> `labels` is also passed as an argument to the `localize_document_type` filter:
> `{{ document.related_document_type | localize_document_type: labels }}`

---

## 9. Quaderno-Specific Filters

### Documented filters

| Filter | Signature | Output | Notes |
|---|---|---|---|
| `money` | `{{ document.total \| money }}` | `57,33€` | Formats number as currency string using document currency |
| `money: true` | `{{ document.total \| money: true }}` | `57.33 EUR` | Shows ISO currency code instead of symbol |
| `precision` | `{{ item.quantity \| precision }}` | `17.33` | Formats number with 2 decimal places |
| `precision: N` | `{{ item.quantity \| precision: 3 }}` | `17.333` | Formats with N decimal places |
| `country_name` | `{{ contact.country \| country_name }}` | `United Kingdom` | Country ISO code → localised name |

### Filters used in templates but NOT documented

| Filter | Example usage | Observed behaviour | Status |
|---|---|---|---|
| `textilize` | `{{ document.notes \| textilize }}` | Renders Textile markup to HTML | ⚠️ Undocumented Quaderno filter |
| `localize_document_type: labels` | `{{ document.related_document_type \| localize_document_type: labels }}` | Translates "Invoice"/"Credit" via `labels` object | ⚠️ Undocumented Quaderno filter |
| `date: 'default', contact.country` | `{{ document.issue_date \| date: 'default', contact.country }}` | Locale-aware date format | ⚠️ Custom Quaderno `date` format keyword |
| `date: 'long'` | `{{ document.issue_date \| date: 'long' }}` | Verbose date format (e.g. "1 January 2024") | ⚠️ Custom Quaderno `date` format keyword (used in older templates: classic, newspaper) |
| `newline_to_br` | `{{ account.formatted_address \| newline_to_br }}` | Converts `\n` to `<br />` | Standard Liquid filter ✓ |

> **Note on `date`:** Standard Liquid `date` uses `strftime` format strings (e.g. `%Y-%m-%d`). The Quaderno templates use custom keywords `'default'` and `'long'` with an optional second argument for locale, which is a Quaderno extension of the standard filter.

---

## 10. Standard Liquid Filters Used in Templates

These are standard Liquid filters (documented at shopify.github.io/liquid) that appear in the templates:

| Filter | Used for |
|---|---|
| `default: 'en'` | `contact.language \| default: 'en'` on `<html lang="">` |
| `default: 'Tax'` | `labels.tax \| default: 'Tax'` fallback in some templates |
| `last` | `document.payments \| last` to get most recent payment |
| `upcase` | `labels.subtotal \| upcase` in mono.html |
| `strip_newlines` | Address formatting in classic.html footer |
| `replace: "<br />", " – "` | Inline address formatting in classic.html footer |
| `size` | `document.taxes.size > 0` to conditionally show breakdown table |

---

## 11. Liquid Language Constructs Used

### Control flow
```liquid
{% if variable != blank %}...{% endif %}
{% if variable == "value" %}...{% elsif ... %}...{% else %}...{% endif %}
{% unless contact.eu_member? %}...{% endunless %}
```

### Iteration
```liquid
{% for item in document.items %}...{% endfor %}
{% for document_tax in document.taxes %}...{% endfor %}
{% for tax in document.taxes %}...{% endfor %}   ← older template style
```

### Variable assignment
```liquid
{% assign payment = document.payments | last %}
```

### Whitespace control
Not used in existing templates (no `{%-` / `-%}` syntax) but available.

### Useful operators available
- `==`, `!=`, `>`, `<`, `>=`, `<=`
- `and`, `or`
- `contains` (substring / array membership check)
- `blank` (Liquid falsy for nil, false, empty string, empty array)

---

## 12. PDF / Rendering Constraints

### pdfkit meta tags (required in `<head>`)
```html
<meta name="pdfkit-page_size" content="A4">
<meta name="pdfkit-margin_top" content="13mm">
<meta name="pdfkit-margin_right" content="13mm">
<meta name="pdfkit-margin_bottom" content="13mm">
<meta name="pdfkit-margin_left" content="13mm">
```
These control PDF output. Standard value is `13mm` for all margins. Use `0mm` if you want to handle margins yourself (e.g. the classic template's bleed-to-edge hero).

### Print CSS (required for correct pagination)
```css
@media print {
  body { -webkit-print-color-adjust: exact; }   /* preserve background colours */
  thead { display: table-header-group; }         /* repeat header across pages */
  tfoot { display: table-footer-group; }         /* anchor footer to page bottom */
  tr { page-break-inside: avoid; }               /* keep rows together */
}
```
To allow long item tables to break across pages (instead of pushing whole table to next page):
```css
#items tr { page-break-inside: auto !important }
```

### Template upload constraints
- **One custom template per account** — only a single customised template can be uploaded at a time.
- Upload via: Quaderno account → Appearance page.
- A preview is available before confirming.
- Templates are HTML + Liquid. No external CSS files or JS — everything must be inline.

### CSS colour injection
`{{ account.color_scheme }}` is rendered **inside `<style>` blocks** as a literal CSS value:
```css
h1 { color: {{ account.color_scheme }}; }
```
This is valid because Liquid processes the template before the browser/PDF renderer sees it.

### Labels constraint
Labels (`labels.*`) work **only in document templates**. They are not available in email templates.

---

## 13. Cross-Reference: Gaps & Discrepancies

### Variables used in templates that are NOT in the official docs

| Variable | Where used | Severity |
|---|---|---|
| `account.registration_number` | All 8 templates | **High** — core seller identity field |
| `account.currency` | All 8 templates (exchange rows) | **High** — required for multi-currency |
| `contact.language` | All 8 templates (`<html lang="">`) | **High** — drives i18n |
| `document.tax_amount` | modern, classic, frame, mono, minimal | **High** — needed for totals summary |
| `document.tax_amount_exchange` | All 8 templates | **High** — needed for exchange display |
| `document.total_exchange` | All 8 templates | **High** — needed for exchange display |
| `document.payments` (array) | All 8 templates | **Medium** — implied by payment vars but not listed |
| `labels.tax` | All 8 templates | **High** — core totals label |
| `labels.registration_number` | All 8 templates | **Medium** — standard seller detail |
| `labels.tax_breakdown` | 6 templates | **Medium** — needed for breakdown table |
| `labels.taxable_base` | 6 templates | **Medium** — needed for breakdown table |
| `labels.rate` | 6 templates | **Medium** — needed for breakdown table |
| `textilize` filter | All 8 templates | **High** — used for notes, payment_details, legal |
| `localize_document_type` filter | All 8 templates | **High** — used for related document type |
| `date: 'default'` format keyword | All 8 templates | **High** — primary date format |
| `date: 'long'` format keyword | classic, newspaper | **Medium** — verbose date format |

### Variables in the docs NOT used in any sample template

| Variable | Notes |
|---|---|
| `document.gross_amount` | Useful for discount breakdowns — see common-use-cases |
| `document.amount_paid` / `document.amount_due` | Partial payment scenarios |
| `document.exchange_rate` | Could show rate alongside exchange totals |
| `document.shipping_address` | Physical goods only |
| `document.tag_list` | Classification / grouping |
| `document.secure_id` | Useful for links/QR codes |
| `document.days_past_due` | Dunning / overdue notices |
| `document.auto?` | Integration-originated docs |
| `contact.tax_id` | Note: templates use `document.tax_id` (snapshot at creation time) instead |
| `contact.formatted_address` | Templates use `document.billing_address` instead |
| `contact.first_name` / `contact.last_name` | Templates use `contact.full_name` |
| `account.street_line_1/2`, `.postal_code`, `.city`, `.region`, `.country` | Templates use `account.formatted_address` |
| `labels.exchange` / `labels.email` | Not rendered in any sample template |
| `document_tax.country/region/currency` | Jurisdiction details, not shown |
| `document_tax.local_taxable_base/local_amount` | Cross-border local currency display |

### Loop variable naming conventions (docs vs templates)

| Object | Docs example name | Templates use |
|---|---|---|
| Items | `document_item` | `item` (all templates) |
| Taxes (newer) | `document_tax` | `document_tax` (modern, classic, frame, mono, minimal, professional) |
| Taxes (older) | — | `tax` (destijl, newspaper) |
| Payments | `document_payment` | `payment` (via `{% assign payment = ... %}`) |

The loop variable name is arbitrary Liquid — any name declared in the `for` tag works. The docs' naming convention (`document_*`) makes the object type explicit but is not required.
