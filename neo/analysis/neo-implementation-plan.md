# Neo Template — Implementation Plan

Sources:
- `/design/neo-invoice-design.pdf` — visual specification
- `/analysis/template-patterns.md` — patterns from existing templates
- `/analysis/quaderno-liquid-reference.md` — variable/filter reference
- `CLAUDE.md` — technical requirements and element list

---

## PDF Design Observations

### Overall layout
- Clean white background, no bleed areas or hero panels
- Standard 13mm pdfkit margins on all sides
- Single-column document flow (no sidebar)
- Content width ≈ 696px (A4 794px − 2 × 49px margins)
- Font: Helvetica Neue / Arial / sans-serif
- Base body text: 8px (per CLAUDE.md typography table — body/value text)

### Typography system (authoritative — from CLAUDE.md)

**Font family:** `"Helvetica Neue", Arial, sans-serif`
**Line-height:** `1.4` on all elements except Document title (`1.2`)
**Weights:** `400` (normal) and `500` (medium) ONLY — **no bold/700 anywhere in the template**
**Sizes:** `7px`, `8px`, `10px`, `24px` — all in px (no pt except inside `@page` margin boxes)

> **No `<strong>` tags in Neo:** Because `<strong>` maps to font-weight 700 which is prohibited, use CSS classes with `font-weight: 500` instead. Reserve `<strong>` for semantic emphasis only — if used, override with `font-weight: 500` in CSS.

| Element | Size | Weight | Color | Style |
|---|---|---|---|---|
| Document title | 24px | 500 | #171717 | — |
| Status tag | 10px | 500 | (per status) | — |
| Payment method | 7px | 400 | #737373 | italic |
| Invoice number (header) | 10px | 400 | #525252 | — |
| Subject label | 8px | 500 | #171717 | — |
| Section labels (From, Billed To) | 8px | 500 | #171717 | — |
| Field labels (Issue Date, PO Number) | 8px | 500 | #171717 | — |
| Field values | 8px | 400 | #525252 | — |
| Table column headers | 7px | 500 | #171717 | — |
| Table cell content | 8px | 400 | #525252 | — |
| Subtotal / Discount / Total tax / Amount paid (labels + values) | 8px | 400 | #525252 | — |
| Total / Balance due (labels + values) | 8px | 500 | #171717 | — |
| Currency conversion label | 7px | 500 | #171717 | — |
| Exchange rate / Balance due / Total VAT labels | 8px | 500 | #737373 | — |
| Exchange rate / Balance due / Total VAT values | 8px | 400 | #737373 | — |
| Deposit paid note | 7px | 400 | #737373 | italic |
| Payment details label | 8px | 500 | #171717 | — |
| Payment details value | 8px | 400 | #525252 | — |
| Notes label | 8px | 500 | #171717 | — |
| Notes value | 8px | 400 | #525252 | — |
| Legal notices | 7px | 400 | #737373 | — |
| Footer — Invoice label | 8px | 500 | #171717 | — |
| Footer — Invoice value | 8px | 400 | #525252 | — |
| Pagination | 8px | 400 | #525252 | — |

> **Balance Due:** The typography table specifies 8px/500/#171717 — same scale as Total. Its "visually unmissable" quality comes from colour (#171717 vs #525252 for subtotals), weight (500 vs 400), and contextual position — not a larger font size.

### Colour palette (authoritative — from CLAUDE.md)
| Usage | Value |
|---|---|
| Primary text | `#171717` |
| Secondary text / values | `#525252` |
| Tertiary text / labels in currency block | `#737373` |
| Divider lines | `#E5E5E5` |
| Footer bar | `{{account.color_scheme}}` |
| Draft badge bg/text/border | `#EBEBEB` / `#525252` / `rgba(82,82,82,0.1)` |
| Outstanding badge bg/text/border | `#FFEDD4` / `#9F2D00` / `rgba(159,45,0,0.1)` |
| Paid badge bg/text/border | `#A4F4C0` / `#016630` / `rgba(1,102,48,0.1)` |
| Overdue badge bg/text/border | `#FFE2E2` / `#9F0712` / `rgba(159,7,18,0.1)` |

---

## Section-by-Section Breakdown

---

### Section 1 — Header

#### Visual description
Two-column table row (title area left ~70%, logo right ~30%). Title row contains: large document title, inline status badge, inline italic payment method text. Document number sits directly below the title on the next line. Subject is a **separate section** immediately below the header (not part of it) — see Section 1b.

```
┌─────────────────────────────────────────┬──────────────┐
│ Invoice  [Paid]  Payment method here.   │  [LOGO BOX]  │
│ INT-99-TK421                            │              │
└─────────────────────────────────────────┴──────────────┘
```

#### Liquid variable mapping

| Visual element | Variable / value | Conditional? |
|---|---|---|
| Verification code | `{{ document.secure_id }}` (or injected by Quaderno — see note) | CONDITIONAL, top of header |
| Document title | `{{ title }}` | ALWAYS |
| Status badge text | Derived from `{{ document.state }}` | ALWAYS |
| Payment method text | `{{ payment.payment_method_text }}` (after assign from `document.payments \| last`) | CONDITIONAL: only when `document.state == "paid"` |
| Document number | `{{ document.number }}` | ALWAYS |
| Logo | `{{ account.logo_url }}` | CONDITIONAL: `{% if account.logo_url != blank %}` |

> **Verification code note:** The `.verification-code` CSS class is reserved across all existing templates. Quaderno likely injects this element — include the class in CSS but no Liquid output tag needed in markup.

#### Status badge logic

```liquid
{% assign state = document.state %}
{% if state == "draft" %}
  <span class="badge badge-draft">Draft</span>
{% elsif state == "outstanding" %}
  <span class="badge badge-outstanding">Outstanding</span>
{% elsif state == "paid" %}
  <span class="badge badge-paid">Paid</span>
{% elsif state == "overdue" %}
  <span class="badge badge-overdue">Overdue</span>
{% endif %}
```
Badge border style: `dashed` for Draft, `solid` for all others (see CLAUDE.md spec).

#### Payment method logic
```liquid
{% if document.state == "paid" %}
  {% assign payment = document.payments | last %}
  {% if payment.payment_method_text != blank %}
    <span class="payment-method-inline">{{ payment.payment_method_text }}</span>
  {% endif %}
{% endif %}
```

#### HTML structure
```html
<table class="header-table">
  <tr>
    <td class="header-title-cell">
      <div class="title-row">
        <h1 class="doc-title">{{ title }}</h1>
        <!-- status badge -->
        <!-- payment method (conditional) -->
      </div>
      <div class="doc-number">{{ document.number }}</div>
    </td>
    <td class="header-logo-cell">
      {% if account.logo_url != blank %}
        <img src="{{ account.logo_url }}" class="logo-img" alt="{{ account.full_name }}">
      {% endif %}
    </td>
  </tr>
</table>
```

#### wkhtmltopdf notes
- Logo box: max-width 150px, max-height 55px (per CLAUDE.md spec).
- Title and badge must be inline — use `display: inline-block` on badge (table-cell layout, not flex).
- `vertical-align: middle` on badge td/span to align with title baseline.

---

### Section 1b — Subject (Separate Section)

#### Visual description
Standalone section sitting between the Header and Metadata. Full content width. Rendered only when `document.subject` is present. 8px top margin separating it from the header above.

```
                                              ↑ 8px margin
Subject: Here's a summary of the invoice subject in a brief sentence.
```

#### Liquid variable mapping

| Visual element | Variable | Conditional? |
|---|---|---|
| "Subject:" label | `{{ labels.subject }}` | CONDITIONAL: entire block |
| Subject text | `{{ document.subject }}` | CONDITIONAL: `{% if document.subject != blank %}` |

#### HTML structure
```html
{% if document.subject != blank %}
<div class="subject-section">
  <strong>{{ labels.subject }}:</strong> {{ document.subject }}
</div>
{% endif %}
```

#### CSS
```css
.subject-section {
  margin-top: 8px;
  font-size: 8px;
  font-weight: 500;
  color: #171717;
  line-height: 1.4;
  width: 100%;
}
```

> **Not a table row:** Subject is a standalone `<div>` between the header table and the metadata table, not a row inside either table. This ensures the 8px top margin is a true gap between the two sections.

---

### Section 2 — Metadata

#### Visual description
Two-row × three-column grid. Labels in bold, values below. Thin horizontal separator between the two rows. No visible vertical dividers between columns.

```
┌──────────────────┬──────────────────┬────────────────────────┐
│ Issue date:      │ Service date:    │ Due date:              │
│ 10 December 2025 │ 10 December 2025 │ 10 December 2025       │
├──────────────────┼──────────────────┼────────────────────────┤
│ PO number:       │ Valid until:     │ Referenced document:   │
│ ORD-66-EXECUTE   │ 10 December 2025 │ Proforma [INV-…], date │
└──────────────────┴──────────────────┴────────────────────────┘
```

#### Liquid variable mapping

| Visual element | Variable | Conditional? |
|---|---|---|
| Issue date label | `{{ labels.issue_date }}` | ALWAYS |
| Issue date value | `{{ document.issue_date \| date: 'default', contact.country }}` | ALWAYS |
| Service date label | `{{ labels.service_date }}` ⚠️ | ALWAYS |
| Service date value | `{{ document.service_date \| date: 'default', contact.country }}` ⚠️ | ALWAYS |
| Due date label | `{{ labels.due_date }}` | CONDITIONAL: `{% if document.due_date != blank %}` |
| Due date value | `{{ document.due_date \| date: 'default', contact.country }}` | CONDITIONAL |
| PO number label | `{{ labels.po_number }}` | CONDITIONAL: `{% if document.po_number != blank %}` |
| PO number value | `{{ document.po_number }}` | CONDITIONAL |
| Valid until label | `{{ labels.valid_until }}` | CONDITIONAL: `{% if document.valid_until != blank %}` |
| Valid until value | `{{ document.valid_until \| date: 'default', contact.country }}` | CONDITIONAL |
| Referenced doc label | `{{ labels.related_document }}` | CONDITIONAL: `{% if document.related_document_type != blank %}` |
| Referenced doc type | `{{ document.related_document_type \| localize_document_type: labels }}` | CONDITIONAL |
| Referenced doc number | `{{ document.related_document_number }}` (as `<a>` link — see note) | CONDITIONAL |
| Referenced doc date | `{{ document.related_document_date \| date: 'default', contact.country }}` | CONDITIONAL |

> **⚠️ `service_date` gap:** `document.service_date` and `labels.service_date` are not listed in the official Quaderno docs, but are specified in CLAUDE.md. Implement as `{{ document.service_date | date: 'default', contact.country }}` — they are likely undocumented variables following the same pattern as `account.registration_number`. If `document.service_date` is blank on documents that don't have a service date, wrap with `{% if document.service_date != blank %}`.

> **Referenced document link note:** The design shows the related document number as a clickable underlined link. No URL variable is documented for the related document. Implement with an `<a>` tag and a `#` href as a placeholder, or use underline-only styling. Raise with Quaderno to confirm if a URL variable exists.

#### Conditional logic

**Row 2 visibility — single compound check (per CLAUDE.md):**
Rather than per-cell conditionals that leave an empty second row, a single `{% if %}` wraps the entire second `<tr>`. If all four row-2 fields are blank, the row is omitted entirely and the metadata table becomes 1-row × 3-col.

```liquid
{% assign show_meta_row2 = false %}
{% if document.due_date != blank or document.po_number != blank or document.valid_until != blank or document.related_document_type != blank %}
  {% assign show_meta_row2 = true %}
{% endif %}
```

Row 2 is then wrapped:
```liquid
{% if show_meta_row2 %}
<tr>
  <!-- Row 2 cells, each individually conditional inside -->
</tr>
{% endif %}
```

Within that row, each cell still guards its own content — because cells must be rendered (even if empty) to maintain the 3-column grid, but they render blank when their specific field is absent:

```liquid
<!-- Row 2, Col 1: PO Number -->
<td class="meta-cell">
  {% if document.po_number != blank %}
    <span class="meta-label">{{ labels.po_number }}:</span>
    <span class="meta-value">{{ document.po_number }}</span>
  {% endif %}
</td>

<!-- Row 2, Col 2: Valid until -->
<td class="meta-cell">
  {% if document.valid_until != blank %}
    <span class="meta-label">{{ labels.valid_until }}:</span>
    <span class="meta-value">{{ document.valid_until | date: 'default', contact.country }}</span>
  {% endif %}
</td>

<!-- Row 2, Col 3: Referenced document -->
<td class="meta-cell">
  {% if document.related_document_type != blank %}
    <span class="meta-label">{{ labels.related_document }}:</span>
    <span class="meta-value">
      {{ document.related_document_type | localize_document_type: labels }}
      <a href="#" class="related-doc-link">{{ document.related_document_number }}</a>,
      {{ document.related_document_date | date: 'default', contact.country }}
    </span>
  {% endif %}
</td>
```

**Full metadata table structure:**
```html
{% assign show_meta_row2 = false %}
{% if document.due_date != blank or document.po_number != blank or document.valid_until != blank or document.related_document_type != blank %}
  {% assign show_meta_row2 = true %}
{% endif %}

<table class="meta-table">
  <tr>
    <td class="meta-cell" style="width:33%">
      <span class="meta-label">{{ labels.issue_date }}:</span>
      <span class="meta-value">{{ document.issue_date | date: 'default', contact.country }}</span>
    </td>
    <td class="meta-cell" style="width:33%">
      <span class="meta-label">{{ labels.service_date }}:</span>
      <span class="meta-value">{{ document.service_date | date: 'default', contact.country }}</span>
    </td>
    <td class="meta-cell" style="width:34%">
      {% if document.due_date != blank %}
        <span class="meta-label">{{ labels.due_date }}:</span>
        <span class="meta-value">{{ document.due_date | date: 'default', contact.country }}</span>
      {% endif %}
    </td>
  </tr>
  {% if show_meta_row2 %}
  <tr>
    <td class="meta-cell">
      {% if document.po_number != blank %}
        <span class="meta-label">{{ labels.po_number }}:</span>
        <span class="meta-value">{{ document.po_number }}</span>
      {% endif %}
    </td>
    <td class="meta-cell">
      {% if document.valid_until != blank %}
        <span class="meta-label">{{ labels.valid_until }}:</span>
        <span class="meta-value">{{ document.valid_until | date: 'default', contact.country }}</span>
      {% endif %}
    </td>
    <td class="meta-cell">
      {% if document.related_document_type != blank %}
        <span class="meta-label">{{ labels.related_document }}:</span>
        <span class="meta-value">
          {{ document.related_document_type | localize_document_type: labels }}
          <a href="#" class="related-doc-link">{{ document.related_document_number }}</a>,
          {{ document.related_document_date | date: 'default', contact.country }}
        </span>
      {% endif %}
    </td>
  </tr>
  {% endif %}
</table>
```

> **Note on Due date placement:** In row 1 col 3, due date is shown when present. It is part of the row-2 compound check (`show_meta_row2`) because its absence alone could still require row 2 for other fields — but it renders in row 1 col 3 since that slot is otherwise empty in the first row.

#### wkhtmltopdf notes
- Use `border-bottom` on row 1 cells for the horizontal separator (only rendered when row 2 exists — apply via `{% if show_meta_row2 %}class="meta-cell meta-cell-bordered"{% endif %}`).
- Three equal-width columns: each td at `width: 33%` (last col `34%` to avoid rounding gaps).
- No separator line needed when table is 1-row.

---

### Section 3 — Parties

#### Visual description
Three columns (From, Billed to, Shipping address) when shipping address exists; two equal columns otherwise. Labels are bold, uppercase or semibold.

```
┌──────────────────────┬──────────────────────┬──────────────────────┐
│ From:                │ Billed to:           │ Shipping address:    │
│ Name                 │ Name                 │ Name                 │
│ Address              │ Contact person       │ Address              │
│ Email                │ Department           │                      │
│ Tax ID               │ Address              │                      │
│                      │ Email                │                      │
│                      │ Tax ID               │                      │
└──────────────────────┴──────────────────────┴──────────────────────┘
```

#### Liquid variable mapping

**From (seller):**
| Visual element | Variable | Conditional? |
|---|---|---|
| Section label | `{{ labels.from }}` | ALWAYS |
| Name | `{{ account.full_name }}` | ALWAYS |
| Address | `{{ account.formatted_address \| newline_to_br }}` | ALWAYS |
| Registration number | `{{ labels.registration_number }}: {{ account.registration_number }}` | CONDITIONAL: `{% if account.registration_number != blank %}` |
| Tax ID | `{{ labels.tax_id }}: {{ account.tax_id }}` | CONDITIONAL: `{% if account.tax_id != blank %}` |
| Email | `{{ account.email }}` | CONDITIONAL: `{% if account.email != blank %}` |

**Billed to (buyer):**
| Visual element | Variable | Conditional? |
|---|---|---|
| Section label | `{{ labels.to }}` | ALWAYS |
| Name | `{{ contact.full_name \| default: document.contact }}` | ALWAYS |
| Contact person | `{{ contact.contact_person }}` | CONDITIONAL: `{% if contact.contact_person != blank %}` |
| Department | `{{ contact.department }}` | CONDITIONAL: `{% if contact.department != blank %}` |
| Address | `{{ document.billing_address \| newline_to_br }}` | ALWAYS |
| Tax ID | `{{ labels.tax_id }}: {{ document.tax_id }}` | CONDITIONAL: `{% if document.tax_id != blank %}` |
| Email | `{{ contact.email }}` | CONDITIONAL: `{% if contact.email != blank %}` |

**Shipping address:**
| Visual element | Variable | Conditional? |
|---|---|---|
| Section label | (hardcoded label needed — see note) | CONDITIONAL: entire column |
| Name | `{{ account.full_name }}` ⚠️ | CONDITIONAL |
| Address | `{{ document.shipping_address \| newline_to_br }}` | CONDITIONAL |

> **⚠️ Shipping address label:** `labels.shipping_address` is not in the official labels list. Either add a hardcoded "Shipping address:" fallback or use `{{ labels.shipping_address | default: 'Shipping address' }}`. Raise with Quaderno.

> **⚠️ Shipping address name:** The PDF shows the company name under "Shipping address". It's not clear which variable provides this — may be `account.full_name`, `contact.full_name`, or a dedicated `document.shipping_name` field. The safest assumption is `account.full_name` or `contact.full_name`. Flag for clarification.

#### Conditional layout logic (2 vs 3 columns)
```liquid
{% if document.shipping_address != blank %}
  <!-- 3-column layout: each col width: 33% -->
{% else %}
  <!-- 2-column layout: each col width: 50% -->
{% endif %}
```

#### wkhtmltopdf notes
- Use a single `<table>` with conditional `<td>` count and width.
- Cannot use flex `flex: 1` — use explicit percentage widths via `width` attribute or inline style.
- Column structure: for 3-col use `width: 33%` per td; for 2-col use `width: 50%`.

---

### Section 4 — Line Items Table

#### Visual description
Full-width table. Header row with grey column labels. Body rows with light bottom borders. 7 columns when discount present (Description, Qty, Unit price, Discount, Subtotal, Tax rate, Amount), 6 columns when no discounts.

```
┌─────────────────────────┬──────┬────────────┬──────────┬──────────┬──────────┬────────────┐
│ Description             │ Qty  │ Unit price │ Discount │ Subtotal │ Tax rate │ Amount     │
├─────────────────────────┼──────┼────────────┼──────────┼──────────┼──────────┼────────────┤
│ TIE/LN Starfighter…     │ 2.00 │ 2,500.00   │ 21.00%   │ 5,000.00 │ 21.00%   │ 50,000.00  │
└─────────────────────────┴──────┴────────────┴──────────┴──────────┴──────────┴────────────┘
```

#### Liquid variable mapping

| Visual element | Variable | Notes |
|---|---|---|
| Description header | `{{ labels.description }}` | |
| Qty header | `{{ labels.quantity }}` | |
| Unit price header | `{{ labels.unit_price }}` | |
| Discount header | `{{ labels.discount }}` | Hidden when no discounts |
| Subtotal header | `{{ labels.subtotal }}` | Column (not totals row) |
| Tax rate header | `{{ labels.rate }}` | ⚠️ undocumented label |
| Amount header | `{{ labels.amount }}` | |
| Item description | `{{ item.description }}` | |
| Item quantity | `{{ item.quantity \| precision }}` | |
| Item unit price | `{{ item.unit_price \| precision }}` | |
| Item discount | `{{ item.discount_rate \| precision }}%` | Conditional cell |
| Item subtotal | `{{ item.subtotal }}` | Qty × Unit price (pre-discount) |
| Item tax rate | `{{ item.tax_1_rate \| precision }}%` | ⚠️ using tax_1_rate |
| Item amount | `{{ item.total_amount }}` | Subtotal − Discount + Tax |

> **⚠️ Tax rate per line:** The design shows a "Tax rate" column per item. Docs provide `item.tax_1_rate` and `item.tax_2_rate`. Use `item.tax_1_rate` as primary. For items with two tax rates, concatenation may be needed — flag for clarification.

> **⚠️ `labels.rate`:** Used as the tax rate column header. This label is in existing templates but not in official docs. Use `{{ labels.rate }}` consistent with existing templates.

#### Conditional discount column: pre-scan
```liquid
{% assign has_discounts = false %}
{% for item in document.items %}
  {% if item.discount_rate > 0 %}
    {% assign has_discounts = true %}
  {% endif %}
{% endfor %}
```
Use `has_discounts` throughout the table to conditionally render the discount `<th>` and each row's discount `<td>`.

#### Column widths
With discount (7 columns):
```
Description: auto (fills remaining)
Qty:         7%
Unit price:  11%
Discount:    8%
Subtotal:    11%
Tax rate:    8%
Amount:      11%
```

Without discount (6 columns):
```
Description: auto
Qty:         9%
Unit price:  13%
Subtotal:    13%
Tax rate:    9%
Amount:      13%
```

Use `<colgroup>` with conditional column definitions inside `{% if has_discounts %}`.

#### HTML structure
```html
{% assign has_discounts = false %}
{% for item in document.items %}
  {% if item.discount_rate > 0 %}{% assign has_discounts = true %}{% endif %}
{% endfor %}

<table class="items-table">
  <colgroup>
    <col>
    <col style="width:9%">
    <col style="width:13%">
    {% if has_discounts %}<col style="width:8%">{% endif %}
    <col style="width:13%">
    <col style="width:9%">
    <col style="width:13%">
  </colgroup>
  <thead>
    <tr class="items-header">
      <th class="left">{{ labels.description }}</th>
      <th class="right">{{ labels.quantity }}</th>
      <th class="right">{{ labels.unit_price }}</th>
      {% if has_discounts %}<th class="right">{{ labels.discount }}</th>{% endif %}
      <th class="right">{{ labels.subtotal }}</th>
      <th class="right">{{ labels.rate }}</th>
      <th class="right">{{ labels.amount }}</th>
    </tr>
  </thead>
  <tbody>
    {% for item in document.items %}
    <tr class="item-row">
      <td>{{ item.description }}</td>
      <td class="right">{{ item.quantity | precision }}</td>
      <td class="right">{{ item.unit_price | precision }}</td>
      {% if has_discounts %}<td class="right">{{ item.discount_rate | precision }}%</td>{% endif %}
      <td class="right">{{ item.subtotal }}</td>
      <td class="right">{{ item.tax_1_rate | precision }}%</td>
      <td class="right">{{ item.total_amount }}</td>
    </tr>
    {% endfor %}
  </tbody>
</table>
```

#### wkhtmltopdf notes
- Use `page-break-inside: avoid` on `tr` in `@media print`.
- No `thead` repeat needed for items (CLAUDE.md: "no header repeat on subsequent pages").
- But `<thead>` must still exist for correct tfoot/tbody rendering order.

---

### Section 5 — Financial Zone (Tax Breakdown + Summary)

#### Visual description
Two panels side by side: Tax Breakdown (55% left) and Summary (45% right). No section title labels — column headers are self-explanatory.

```
┌──────────────────────────────────────────────┬─────────────────────────────────┐
│ Tax name    Taxable amt  Tax rate  Total tax  │        Subtotal   8,450.00 USD  │
│ VAT (Std)   6,200.00    21.00%    1,302.00   │        Discount   -397.50 USD   │
│ VAT (Red)   2,250.00    9.00%     202.50     │      Total tax*   1,774.50 USD  │
│ Zero-rated  0.00        0.00%     0.00       │           Total  10,224.50 USD  │
│ Total tax*  8,450.00    —         1,504.50   │    Amount paid   -3,000.00 USD  │
│                                              │     Balance due   7,224.50 USD  │
└──────────────────────────────────────────────┴─────────────────────────────────┘
```

#### Tax Breakdown — Liquid mapping

| Visual element | Variable | Notes |
|---|---|---|
| Tax name header | `{{ labels.tax }}` or `{{ labels.tax_breakdown }}` | ⚠️ `labels.tax_breakdown` is undocumented |
| Taxable amount header | `{{ labels.taxable_base }}` | ⚠️ undocumented label |
| Tax rate header | `{{ labels.rate }}` | ⚠️ undocumented label |
| Total tax per rate header | `{{ labels.amount }}` | |
| Loop | `{% for document_tax in document.taxes %}` | |
| Tax label | `{{ document_tax.label }}` | |
| Taxable base | `{{ document_tax.taxable_base }}` | |
| Rate | `{{ document_tax.rate }}%` | |
| Tax amount | `{{ document_tax.amount }}` | |
| "Total tax" row label | Static "Total tax" or `{{ labels.tax }}` | |
| Total taxable base | Sum row — likely `document.subtotal` or requires calculation | ⚠️ |
| Total tax amount | `{{ document.tax_amount }}` | ⚠️ undocumented |

> **⚠️ Total row in tax breakdown:** The "Total tax*" row shows total taxable base (8,450.00) and total tax (1,504.50). `document.tax_amount` provides the total tax. The total taxable base requires either a Quaderno variable or summing the loop — no documented aggregate. Use `document.subtotal` with discount adjustment, or flag for clarification and leave as `document.tax_amount` for the total column only.

> **⚠️ `labels.tax_breakdown`:** Used as tax breakdown table section label in newer templates. Apply consistently.

#### Summary — Liquid mapping

| Visual element | Variable | Conditional? |
|---|---|---|
| "Subtotal" label | `{{ labels.subtotal }}` | ALWAYS |
| Subtotal value | `{{ document.subtotal }}` | ALWAYS |
| "Discount" label | `{{ labels.discount }}` | CONDITIONAL: `{% if document.discount > 0 %}` |
| Discount value | `-{{ document.discount }}` (with negative sign) | CONDITIONAL |
| "Total tax" label | `{{ labels.tax }}` | ALWAYS |
| Total tax value | `{{ document.tax_amount }}` ⚠️ | ALWAYS |
| "Total" label | `{{ labels.total }} ({{ document.currency }})` | ALWAYS |
| Total value | `{{ document.total }}` | ALWAYS |
| "Amount paid" label | `{{ labels.paid }}` or similar | CONDITIONAL: `{% if document.amount_paid > 0 %}` |
| Amount paid value | `-{{ document.amount_paid }}` (with negative sign) | CONDITIONAL |
| "Balance due" label | (no documented label — see note) | CONDITIONAL: same as amount_paid |
| Balance due value | `{{ document.amount_due }}` | CONDITIONAL |

> **⚠️ "Amount paid" / "Balance due" labels:** Neither `labels.amount_paid` nor `labels.balance_due` appear in the documented labels list or in existing templates. These may need to be hardcoded with a `| default:` fallback, or Quaderno may provide them undocumented. Use `{{ labels.amount_paid | default: 'Amount paid' }}` and `{{ labels.balance_due | default: 'Balance due' }}` as a safe fallback.

> **⚠️ `document.amount_paid` and `document.amount_due`:** Both are in the official docs but not used in any existing template. These are the correct variables per the spec.

#### Conditional for amount paid / balance due block
```liquid
{% if document.amount_paid > 0 %}
  <tr>
    <td class="summary-label">{{ labels.paid }}</td>
    <td class="summary-value">-{{ document.amount_paid }}</td>
  </tr>
  <tr class="balance-due-row">
    <td class="summary-label balance-label">{{ labels.balance_due | default: 'Balance due' }}</td>
    <td class="summary-value balance-value">{{ document.amount_due }}</td>
  </tr>
{% endif %}
```

#### HTML structure
```html
<table style="width:100%">
  <tr>
    <!-- Tax breakdown panel: 55% -->
    <td style="width:55%; vertical-align:top; padding-right:20px;">
      <table class="tax-breakdown-table">
        <thead>
          <tr class="tax-header-row">
            <th class="left">{{ labels.tax_breakdown }}</th>
            <th class="right">{{ labels.taxable_base }}</th>
            <th class="right">{{ labels.rate }}</th>
            <th class="right">{{ labels.amount }}</th>
          </tr>
        </thead>
        <tbody>
          {% for document_tax in document.taxes %}
          <tr class="tax-body-row">
            <td>{{ document_tax.label }}</td>
            <td class="right">{{ document_tax.taxable_base }}</td>
            <td class="right">{{ document_tax.rate }}%</td>
            <td class="right">{{ document_tax.amount }}</td>
          </tr>
          {% endfor %}
        </tbody>
        <tfoot>
          <tr class="tax-total-row">
            <td><strong>{{ labels.tax }}</strong></td>
            <td class="right"><strong><!-- sum of taxable bases --></strong></td>
            <td class="right">—</td>
            <td class="right"><strong>{{ document.tax_amount }}</strong></td>
          </tr>
        </tfoot>
      </table>
    </td>
    <!-- Summary panel: 45% -->
    <td style="width:45%; vertical-align:top;">
      <table class="summary-table">
        <tr>
          <td class="summary-label">{{ labels.subtotal }}</td>
          <td class="summary-value">{{ document.subtotal }}</td>
        </tr>
        {% if document.discount > 0 %}
        <tr>
          <td class="summary-label">{{ labels.discount }}</td>
          <td class="summary-value">-{{ document.discount }}</td>
        </tr>
        {% endif %}
        <tr>
          <td class="summary-label">{{ labels.tax }}</td>
          <td class="summary-value">{{ document.tax_amount }}</td>
        </tr>
        <tr class="summary-total-row">
          <td class="summary-label"><strong>{{ labels.total }} ({{ document.currency }})</strong></td>
          <td class="summary-value"><strong>{{ document.total }}</strong></td>
        </tr>
        {% if document.amount_paid > 0 %}
        <tr>
          <td class="summary-label">{{ labels.amount_paid | default: 'Amount paid' }}</td>
          <td class="summary-value">-{{ document.amount_paid }}</td>
        </tr>
        <tr class="balance-due-row">
          <td class="summary-label balance-label">{{ labels.balance_due | default: 'Balance due' }}</td>
          <td class="summary-value balance-value">{{ document.amount_due }}</td>
        </tr>
        {% endif %}
      </table>
    </td>
  </tr>
</table>
```

---

### Section 6 — Currency Conversion (Conditional)

#### Visual description
Full-width block, only shown when exchange data exists. Section label "Currency conversion". Three data rows (exchange rate, balance due in domestic, total tax in domestic). Italic note for deposit payment details.

```
Currency conversion
──────────────────────────────────────────────────────────────────
Exchange Rate (ECB, 22 Jan 2026)               1.00 USD = 0.92 EUR
Balance Due (EUR)                                      6,646.54 EUR
Total VAT (EUR)                                        1,632.54 EUR
Deposit paid at 1.00 USD = 0.91 EUR on 15 Jan 2026 (€2,730.00)
```

#### Conditional wrapper
```liquid
{% if document.exchange %}
  <!-- currency conversion block -->
{% endif %}
```

#### Liquid variable mapping

| Visual element | Variable | Notes |
|---|---|---|
| Section label | `{{ labels.exchange | default: 'Currency Conversion' }}` | ⚠️ CLAUDE.md contradiction: line 224 says "EUR Equivalent", line 226 says "Currency Conversion" — use `labels.exchange` with fallback; resolve with designer before building |
| Exchange rate row label | Constructed string | |
| Exchange rate value | `1.00 {{ document.currency }} = {{ document.exchange_rate }} {{ account.currency }}` | `document.exchange_rate` is documented |
| Balance due (domestic) | `{{ document.amount_due }}` converted | ⚠️ No `document.amount_due_exchange` variable exists |
| Total tax (domestic) | `{{ document.tax_amount_exchange }}` | ⚠️ undocumented but in all templates |
| Total (domestic) | `{{ document.total_exchange }}` | ⚠️ undocumented but in all templates |
| Deposit note | `{{ document.payment_details \| newline_to_br }}` (if it contains this text) | ⚠️ |

> **⚠️ Balance due in domestic currency:** No documented variable exists for this. `document.amount_due_exchange` is not listed anywhere. Options: (a) calculate from `document.amount_due` × `document.exchange_rate`, (b) use `document.total_exchange` − amount paid equivalent, (c) flag as requiring a Quaderno undocumented variable. Implement as a placeholder and confirm with Quaderno.

> **⚠️ Deposit paid note:** The italic "Deposit paid at…" text in the design likely comes from `document.payment_details` or `document.notes` — it describes the payment's exchange rate at time of payment. The payment object (`payment.date`, `payment.amount`) could be used to construct this dynamically. Implement using `document.payment_details | newline_to_br` and note that the exchange rate at payment time is not documented.

> **⚠️ `labels.exchange`:** This label IS documented (`labels.exchange` → "Exchange") but never used in any template. Use it for the section heading: `{{ labels.exchange | default: 'Currency conversion' }}`.

---

### Section 7 — Other Info

#### Visual description
Payment Details (left half) and Notes (right half) side by side. If only one exists, it expands to full width. Legal notices render full width below, in small text.

```
┌──────────────────────────────────┬──────────────────────────────────┐
│ Payment details:                 │ Notes:                           │
│ Bank name: ING Bank N.V.         │ Late payments are subject to…    │
│ IBAN: NL91 ABNA…                 │                                  │
└──────────────────────────────────┴──────────────────────────────────┘
1. Registry: Sienar Fleet Systems is a registered supplier… [small, full width]
```

#### Liquid variable mapping

| Visual element | Variable | Conditional? |
|---|---|---|
| Payment details label | `{{ labels.payment_details }}` | CONDITIONAL: `{% if document.payment_details != blank %}` |
| Payment details value | `{{ document.payment_details \| newline_to_br }}` | CONDITIONAL |
| Notes label | `{{ labels.notes }}` | CONDITIONAL: `{% if document.notes != blank %}` |
| Notes value | `{{ document.notes \| newline_to_br }}` | CONDITIONAL |
| Legal notices | `{{ document.legal \| textilize }}` | CONDITIONAL: `{% if document.legal != blank %}` |

> **Note on `textilize` vs `newline_to_br`:** Existing templates use `| textilize` for payment_details, notes, and legal. The design uses `| newline_to_br` for payment_details and notes (bank details are plain text) and `| textilize` for legal notices (numbered list). Use `| newline_to_br` for payment_details and notes, `| textilize` for legal — consistent with modern/classic templates.

#### Conditional layout logic
```liquid
{% assign show_payment = false %}
{% assign show_notes = false %}
{% if document.payment_details != blank %}{% assign show_payment = true %}{% endif %}
{% if document.notes != blank %}{% assign show_notes = true %}{% endif %}

{% if show_payment or show_notes %}
<table class="other-info-table">
  <tr>
    {% if show_payment and show_notes %}
      <td class="other-info-cell" style="width:50%; padding-right:15px;">
        <strong>{{ labels.payment_details }}:</strong>
        <div>{{ document.payment_details | newline_to_br }}</div>
      </td>
      <td class="other-info-cell" style="width:50%; padding-left:15px;">
        <strong>{{ labels.notes }}:</strong>
        <div>{{ document.notes | newline_to_br }}</div>
      </td>
    {% elsif show_payment %}
      <td class="other-info-cell" colspan="2">
        <strong>{{ labels.payment_details }}:</strong>
        <div>{{ document.payment_details | newline_to_br }}</div>
      </td>
    {% elsif show_notes %}
      <td class="other-info-cell" colspan="2">
        <strong>{{ labels.notes }}:</strong>
        <div>{{ document.notes | newline_to_br }}</div>
      </td>
    {% endif %}
  </tr>
</table>
{% endif %}

{% if document.legal != blank %}
<div class="legal-text">{{ document.legal | textilize }}</div>
{% endif %}
```

---

### Section 8 — Footer

#### Visual description
Fixed to bottom of every page. Color accent bar (3px, `account.color_scheme`) across the full width. Below: left-aligned invoice label + number, right-aligned page number.

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━ [account.color_scheme] bar ━━━━━━━━━━
Invoice: INT-99-TK421                                    Page 1 of 1
```

#### Liquid variable mapping

| Visual element | Variable / mechanism | Notes |
|---|---|---|
| Color bar | `{{account.color_scheme}}` as CSS `background-color` | ALWAYS |
| Invoice label | `{{ labels.invoice }}` | ALWAYS |
| Invoice number | `{{ document.number }}` | ALWAYS |
| Page number | CSS `@page` margin box — see mechanism below | ALWAYS |

#### Page numbering mechanism

**Investigation findings:** Zero of the 8 existing Quaderno templates implement page numbers. There are no `[page]`, `counter(page)`, or any page-numbering patterns in any existing template. The only page-related CSS in all templates is `page-break-inside: avoid` and `display: table-footer-group`.

**wkhtmltopdf options assessed:**

| Approach | Feasible? | Reason |
|---|---|---|
| `--footer-html` with `[page]` / `[topage]` substitution | ✗ | Requires a separate HTML file passed via CLI — not available in single-file templates |
| JavaScript `window.onload` / `window.status` | ✗ | Disallowed by CLAUDE.md |
| CSS `content: counter(page)` on body elements | ✗ | `counter(page)` is a paged-media CSS counter — WebKit only exposes it in `@page` context, not in body element `content:` properties |
| **CSS `@page` margin boxes** | ✅ | Supported in wkhtmltopdf (QtWebKit). Places content in the physical page margin area. |

**Correct approach — CSS `@page` margin boxes:**

wkhtmltopdf's QtWebKit engine supports CSS Paged Media Level 3 margin boxes. The page number is rendered via:

```css
@page {
  @bottom-right {
    content: "Page " counter(page) " of " counter(pages);
    font-size: 8pt;
    color: #666666;
    font-family: "Helvetica Neue", Arial, sans-serif;
  }
}
```

This renders the page number in the bottom-right margin of every page, outside the page body. It is separate from the `<tfoot>` visual footer (color bar + invoice number), which lives in the page body.

**Two-component footer layout:**
- **`<tfoot>` body component** — color accent bar + invoice number (rendered as part of the page body, repeats via `display: table-footer-group`)
- **`@page` margin component** — "Page X of Y" number (rendered in the bottom page margin via CSS)

The design shows these as a single unified footer row. In practice with wkhtmltopdf, the page number will appear in the bottom-right margin area, while the color bar and invoice number appear at the bottom of the body content. These will be visually adjacent at the bottom of the page.

> **Testing note:** The exact vertical alignment between the `<tfoot>` bar and the `@page` margin content depends on the pdfkit margin setting (`13mm`) and font sizes. Adjust `padding-top` on `.footer-cell` to align them after testing with Quaderno's actual renderer.

#### HTML structure (using tfoot pattern from newer templates)

The tfoot cell contains the color bar and invoice number. The right-hand cell holds the `footer-right` slot — left empty in the primary approach (page number comes from `@page` margin box) but used by the fallback (see CSS below).

```html
<table class="master-table">
  <tfoot>
    <tr>
      <td colspan="2" class="footer-cell">
        <div class="footer-bar"></div>
        <table style="width:100%">
          <tr>
            <td class="footer-invoice"><span class="footer-invoice-label">{{ labels.invoice }}:</span> <span class="footer-invoice-value">{{ document.number }}</span></td>
            <!-- PAGE NUMBER SLOT
                 Primary approach: rendered by @page CSS margin box (see CSS below).
                 This td is intentionally empty — page number appears in the @page
                 bottom-right margin, visually adjacent to the footer bar.
                 If @page fails: see the FALLBACK comment in the CSS block. -->
            <td class="footer-right"></td>
          </tr>
        </table>
      </td>
    </tr>
  </tfoot>
  <tbody>
    <!-- all content sections -->
  </tbody>
</table>
```

#### CSS

```css
/* ─── Footer: color bar + invoice number ─────────────────────────── */
.footer-bar {
  height: 3px;
  background-color: {{ account.color_scheme }};
  margin-bottom: 6px;
}
.footer-cell {
  padding-top: 10px;
  font-size: 8px;
  vertical-align: top;
}
/* Footer — Invoice: label 500/#171717, value 400/#525252 (see typography table) */
.footer-invoice { text-align: left; }
.footer-right   { text-align: right; }
.footer-invoice-label { font-weight: 500; color: #171717; }
.footer-invoice-value { font-weight: 400; color: #525252; }

/* ─── Page numbers: PRIMARY approach ─────────────────────────────────
   CSS Paged Media Level 3 @page margin boxes.
   Rendered by wkhtmltopdf's QtWebKit in the physical page margin area
   (bottom-right), outside the normal page body flow.
   The @page rule MUST be a top-level CSS rule — do NOT nest it inside
   @media print.
   Note: 8pt used here (not px) — @page margin boxes use a different
   rendering context; px may not be reliable in paged-media contexts. ─ */
@page {
  @bottom-right {
    content: "Page " counter(page) " of " counter(pages);
    font-size: 8pt;
    color: #525252;
    font-family: "Helvetica Neue", Arial, sans-serif;
  }
}

/* ─── Page numbers: FALLBACK approach ────────────────────────────────
   Use this if @page margin boxes do not render in Quaderno's
   wkhtmltopdf build. Steps to swap:
     1. Remove or comment out the @page block above.
     2. Uncomment the .footer-right::after block below.
   This injects the counter directly into the tfoot right cell as
   CSS-generated content. Some wkhtmltopdf builds support
   counter(page) in element content even when @page boxes fail.
   ─────────────────────────────────────────────────────────────────── */
/*
.footer-right::after {
  content: "Page " counter(page) " of " counter(pages);
}
*/
```

> **If both approaches fail:** Neither `@page` margin boxes nor element-level `counter(page)` are supported by the wkhtmltopdf build in use. In that case the page number slot will simply be blank — the footer will still render correctly with the color bar and invoice number. Dynamic "Page X of Y" would then require either (a) Quaderno to inject it server-side, or (b) enabling JavaScript and using wkhtmltopdf's `--footer-html` URL-parameter mechanism, which is outside the scope of a single-file template.

---

## wkhtmltopdf Adaptation Notes

### Flexbox → Table conversions required

| Design element | Flexbox equivalent | Table implementation |
|---|---|---|
| Header (title left, logo right) | `display:flex; justify-content:space-between` | 2-cell `<tr>`: left ~70%, right ~30% |
| Metadata 3-column grid | `display:flex; flex-wrap:wrap` | `<table>` with 2 `<tr>`, 3 `<td>` each |
| Parties 2/3 columns | `display:flex` | `<table>` with conditional column count |
| Financial zone side-by-side | `display:flex` | 2-cell `<tr>`: 55% + 45% |
| Other info 2-col | `display:flex` | `<table>` 2-cell row or colspan |
| Status badge inline with title | `display:inline-flex; align-items:center` | `display:inline-block` + `vertical-align:middle` |

### CSS properties to avoid
- `flexbox` — any `display:flex`, `flex-direction`, `justify-content`, `align-items`
- `grid` — any `display:grid`, `grid-template-*`
- `calc()` — unreliable in older wkhtmltopdf versions; use fixed percentages
- `border-radius` on table cells — use wrapping `<div>` if needed
- CSS variables (`var(--color)`) — not reliably supported

### CSS that works safely in wkhtmltopdf
- `display: table`, `table-row`, `table-cell`
- `float: left/right` with clearfix
- `display: inline-block`
- `position: relative/absolute`
- `-webkit-print-color-adjust: exact` (required for background colours)
- `page-break-inside: avoid` on `tr`
- `display: table-header-group` / `table-footer-group` on thead/tfoot

### Measurement units (per CLAUDE.md)
- All CSS values: `px` only
- Exception: pdfkit margin `<meta>` tags use `mm`

---

## Proposed Implementation Order

### Phase 1 — Scaffold
1. `<!DOCTYPE html>`, `<html lang="{{ contact.language | default: 'en' }}">`, `<head>` with pdfkit meta tags (A4, 13mm margins)
2. Empty `<style>` block with CSS reset (`* { box-sizing: border-box }`)
3. Master outer `<table>` with `<tfoot>` (footer) and `<tbody>` (content)
4. `@media print` block skeleton

### Phase 2 — Header
5. Header table (title cell + logo cell)
6. Status badge CSS (all 4 states with correct colours)
7. Badge Liquid conditionals + payment method conditional
8. Logo conditional (`!= blank`)
9. Document number display

### Phase 2b — Subject Section
10. Subject `<div>` between header table and metadata table, `margin-top: 8px`, conditional on `document.subject != blank`

### Phase 3 — Metadata
11. `{% assign show_meta_row2 %}` compound check (single `{% if %}` across all four row-2 fields)
12. Metadata table: row 1 always (issue date, service date, due date conditional)
13. Row 2 conditional (`{% if show_meta_row2 %}`): PO, valid until, related document cells
14. Related document number link markup
15. Row-separator border: applied to row 1 cells only when `show_meta_row2` is true

### Phase 4 — Parties
16. Shipping address pre-check: `{% if document.shipping_address != blank %}`
17. 3-column party table (with shipping)
18. 2-column party table (without shipping) using `{% else %}`
19. All conditional fields per party (contact_person, dept, tax_id, email, reg_number)

### Phase 5 — Line Items
20. Discount pre-scan loop (`{% assign has_discounts %}`)
21. `<colgroup>` with conditional discount column width
22. Table header row (conditional discount `<th>`)
23. Item loop rows (conditional discount `<td>`)
24. Section separator above items table

### Phase 6 — Financial Zone
25. Outer 55/45 split table
26. Tax breakdown table (loop + total row)
27. Summary table (subtotal, conditional discount, tax, total)
28. Conditional amount paid + balance due rows
29. Balance due CSS (larger, bolder)

### Phase 7 — Currency Conversion
30. `{% if document.exchange %}` wrapper
31. Section label and divider
32. Exchange rate row (constructed string)
33. Balance due (EUR) row
34. Total VAT (EUR) row — `document.tax_amount_exchange`
35. Deposit note row (italic)

### Phase 8 — Other Info
36. Payment details + notes conditional logic
37. Full-width / split layout conditional
38. Legal text conditional + `textilize`

### Phase 9 — Footer
39. `<tfoot>` cell: `.footer-bar` div (3px, `background-color: {{account.color_scheme}}`)
40. Footer inner table: invoice label + number (left), empty `.footer-right` td (right)
41. `@page { @bottom-right { ... } }` PRIMARY CSS rule (top-level, outside `@media print`)
42. Commented-out FALLBACK: `.footer-right::after { content: "Page " counter(page) " of " counter(pages); }` — with swap instructions in comment
43. Adjust `.footer-cell` padding to visually align tfoot bar with `@page` number after testing

### Phase 10 — Polish
43. Typography: apply sizes (7/8/10/24px), weights (400/500 only, no 700), line-heights (1.4; title 1.2), colors (#171717/#525252/#737373) per element table in "Typography System" section above
44. Spacing: section gaps, cell padding
45. Divider lines between sections
46. Print CSS: `page-break-inside: avoid`, `thead/tfoot display` rules
47. Verification code CSS class (reserved, empty)
48. Test: all conditional fields empty
49. Test: all conditional fields populated
50. Test: 4 status badge states
51. Test: discount column present/absent
52. Test: exchange rate block

---

## Open Questions / Flags for Quaderno

| # | Issue | Impact | Suggested resolution |
|---|---|---|---|
| 1 | `document.service_date` not in docs | Section 2 — always-present field | Confirm variable exists; test against a real document |
| 2 | `labels.service_date` not in docs | Section 2 label | Same as above |
| 3 | Related document URL (link href) | Section 2 referenced doc link | Confirm if a URL variable exists; use `#` or underline-only as fallback |
| 4 | `labels.tax`, `labels.registration_number`, `labels.tax_breakdown`, `labels.taxable_base`, `labels.rate` not in docs | Sections 3, 5 | These work in all existing templates — use them; they are de-facto standard |
| 5 | `labels.amount_paid` / `labels.balance_due` not in docs or templates | Section 5 summary | Use `| default: 'Amount paid'` / `| default: 'Balance due'` as English fallback |
| 6 | `labels.shipping_address` not in docs | Section 3 | Use `| default: 'Shipping address'` as English fallback |
| 7 | `document.amount_due_exchange` doesn't exist | Section 6 currency conversion | Confirm or calculate from `document.amount_due` × `document.exchange_rate` |
| 8 | Deposit paid note — payment exchange rate variable | Section 6 | Confirm if `payment.exchange_rate` exists; use `document.payment_details` as fallback |
| 9 | Page numbers: `@page { @bottom-right { content: counter(page) " of " counter(pages) } }` — no existing template does this; must be tested with Quaderno's actual wkhtmltopdf version | Section 8 footer | ✅ Mechanism identified. No further Quaderno input needed — test during build. |
| 10 | Shipping address recipient name variable | Section 3 | Confirm correct variable (`document.shipping_name`? `contact.full_name`?) |
| 11 | `item.total_amount` vs `item.gross_amount` for "Amount" column | Section 4 | Confirm: "Amount" = after tax (`total_amount`) or after discount before tax (`gross_amount`) |
| 12 | "Proforma" as related document type | Section 2 | `localize_document_type` translates "Invoice"/"Credit" per docs; confirm if "Proforma"/"Estimate" is also supported |
| 13 | CLAUDE.md contradiction: Currency Conversion label | Section 6 | Line 224 says `"EUR Equivalent"`, line 226 says `"Currency Conversion"`. Use `labels.exchange` with `\| default:` fallback. Resolve with designer. |
