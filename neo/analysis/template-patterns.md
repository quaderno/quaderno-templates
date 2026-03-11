# Quaderno Template Patterns

Analysis of all 8 HTML invoice templates in the repository root.
Templates: `classic.html`, `destijl.html`, `frame.html`, `minimal.html`, `modern.html`, `mono.html`, `newspaper.html`, `professional.html`

---

## 1. Liquid Variables

### `document.*`
| Variable | Notes |
|---|---|
| `document.number` | Used in `<title>` and display |
| `document.subject` | Optional — always guarded with `!= blank` |
| `document.contact` | Display name of the contact |
| `document.billing_address` | Multi-line address |
| `document.tax_id` | Contact's tax ID on the document |
| `document.issue_date` | Formatted with `date` filter |
| `document.due_date` | Optional |
| `document.valid_until` | Optional (quotes/estimates) |
| `document.po_number` | Optional purchase order number |
| `document.related_document_type` | Optional, used with `localize_document_type` filter |
| `document.related_document_number` | Optional |
| `document.related_document_date` | Optional |
| `document.items` | Array — iterated with `{% for item in document.items %}` |
| `document.subtotal` | Pre-formatted currency string |
| `document.discount` | Numeric; checked `> 0` before display |
| `document.taxes` | Array of tax objects; also has `.size` property |
| `document.tax_amount` | Total tax (modern/classic only) |
| `document.total` | Pre-formatted currency string |
| `document.currency` | Currency code, e.g. `EUR` |
| `document.exchange` | Boolean — triggers exchange-rate rows |
| `document.tax_amount_exchange` | Tax in account currency |
| `document.total_exchange` | Total in account currency |
| `document.notes` | Optional |
| `document.payment_details` | Optional payment instructions |
| `document.state` | Checked for `"paid"` to show payment method |
| `document.payments` | Array; `| last` used to get most recent |
| `document.legal` | Optional legal text |

### `document.items[]` (item loop variables)
| Variable | Notes |
|---|---|
| `item.description` | Line item description |
| `item.quantity` | Filtered with `| precision` |
| `item.unit_price` | Filtered with `| precision` |
| `item.subtotal` | Pre-formatted currency string |
| `item.discount_rate` | Optional; checked `> 0` (modern, frame, mono, professional) |

### `document.taxes[]` (tax loop variables)
Two iteration patterns are used:

**Pattern A — `{% for tax in document.taxes %}`** (destijl, newspaper, professional):
- `tax.label` — e.g. "VAT 21%"
- `tax.taxable_base` — filtered with `| precision`
- `tax.amount` — formatted currency

**Pattern B — `{% for document_tax in document.taxes %}`** (modern, classic, frame, mono, minimal):
- `document_tax.label`
- `document_tax.taxable_base`
- `document_tax.rate` — displayed as `{{ document_tax.rate }}%`
- `document_tax.amount`

> **Note:** Pattern B exposes `.rate` as a separate field and is used in the dedicated tax breakdown table. Pattern A is inline-only (no separate breakdown table).

### `document.payments[]`
| Variable | Notes |
|---|---|
| `payment.payment_method_text` | Used after `{% assign payment = document.payments | last %}` |

### `account.*`
| Variable | Notes |
|---|---|
| `account.color_scheme` | Used extensively in CSS inline values |
| `account.logo_url` | Optional; guarded with `!= blank` |
| `account.full_name` | Account/seller legal name |
| `account.trade_name` | Fallback when no logo (modern, frame, mono, professional) |
| `account.formatted_address` | Multi-line; filtered with `| newline_to_br` |
| `account.registration_number` | Optional |
| `account.tax_id` | Seller VAT/tax number |
| `account.currency` | Used in exchange-rate rows |
| `account.phone_1` | Phone number — classic only |

### `contact.*`
| Variable | Notes |
|---|---|
| `contact.language` | Used in `<html lang="">` with `| default: 'en'` |
| `contact.country` | Passed to `date` filter for locale-aware formatting |
| `contact.full_name` | Used with `| default: document.contact` fallback (modern, frame, mono, professional) |
| `contact.contact_person` | Optional |
| `contact.department` | Optional |

### Top-level variables
| Variable | Notes |
|---|---|
| `title` | Document type title (e.g. "Invoice", "Credit Note") — rendered prominently |

---

## 2. Label References (`labels.*`)

All templates use a `labels` object for i18n strings.

### Always present
| Key | Purpose |
|---|---|
| `labels.number` | "Invoice #" / "Number" |
| `labels.issue_date` | Issue date label |
| `labels.from` | Seller section header |
| `labels.to` | Buyer section header |
| `labels.tax_id` | VAT / tax ID label |
| `labels.registration_number` | Company reg. number label |
| `labels.description` | Items table column header |
| `labels.quantity` | Items table column header |
| `labels.unit_price` | Items table column header |
| `labels.amount` | Items table column header |
| `labels.subtotal` | Totals section |
| `labels.discount` | Totals section (shown when discount > 0) |
| `labels.tax` | Totals section |
| `labels.total` | Totals section |
| `labels.notes` | Footer section |
| `labels.payment_details` | Footer section |
| `labels.paid` | Shown when `document.state == "paid"` |
| `labels.legal` | Legal text section |
| `labels.related_document` | Credit note / related doc label |

### Conditionally present
| Key | Used in | Purpose |
|---|---|---|
| `labels.due_date` | All | Due date label |
| `labels.valid_until` | All | Quote/estimate expiry |
| `labels.po_number` | All | Purchase order number |
| `labels.subject` | classic, newspaper, professional | Subject line label |
| `labels.phone` | classic | Phone label |
| `labels.tax_breakdown` | modern, classic, frame, mono, minimal, professional | Tax breakdown table header |
| `labels.taxable_base` | modern, classic, frame, mono, minimal, professional | Tax breakdown column |
| `labels.rate` | modern, classic, frame, mono, minimal, professional | Tax rate column |

> `labels` is also passed as an argument to `localize_document_type`: `{{ document.related_document_type | localize_document_type: labels }}`

---

## 3. Conditional Patterns

### Logo / trade name fallback
```liquid
{% if account.logo_url != blank %}
  <img src="{{ account.logo_url }}" ...>
{% else %}
  <h2>{{ account.trade_name }}</h2>   ← modern, frame, mono, professional only
{% endif %}
```
Older templates (destijl, newspaper, classic, minimal) show nothing when no logo.

### Document subject
```liquid
{% if document.subject != blank %}
  <p class="subject">{{ document.subject }}</p>
{% endif %}
```

### Optional metadata fields (common pattern)
```liquid
{% if document.po_number != blank %}...{% endif %}
{% if document.due_date != blank %}...{% endif %}
{% if document.valid_until != blank %}...{% endif %}
```

### PO / related document slot (mutual exclusion — modern, frame, mono, professional)
```liquid
{% if document.po_number != blank %}
  ...po_number...
{% elsif document.related_document_type != blank %}
  ...related_document...
{% endif %}
```
Older templates (destijl, newspaper, classic, minimal) show both independently.

### Due date / valid_until slot (mutual exclusion — modern, frame, mono, professional)
```liquid
{% if document.due_date != blank %}
  ...
{% elsif document.valid_until != blank %}
  ...
{% endif %}
```

### Contact optional fields
```liquid
{% if contact.contact_person != blank %}<p>{{contact.contact_person}}</p>{% endif %}
{% if contact.department != blank %}<p>{{contact.department}}</p>{% endif %}
```

### Seller optional fields
```liquid
{% if account.registration_number != blank %}...{% endif %}
{% if account.tax_id != blank %}...{% endif %}
```

### Discount row
```liquid
{% if document.discount > 0 %}
  <tr>..discount..</tr>
{% endif %}
```

### Tax rows (two patterns)

**Older pattern** (destijl, newspaper, professional) — inline in tfoot, no breakdown table:
```liquid
{% for tax in document.taxes %}
  <tr>
    <td>{{tax.label}} × {{tax.taxable_base | precision}}</td>
    <td>{{tax.amount}}</td>
  </tr>
{% endfor %}
```

**Newer pattern** (modern, classic, frame, mono, minimal) — summary line + breakdown table:
```liquid
{% if document.taxes %}
  <tr><td>{{ labels.tax }}</td><td>{{ document.tax_amount }}</td></tr>
{% endif %}
...
{% if document.taxes.size > 0 %}
  <table>{% for document_tax in document.taxes %}...{% endfor %}</table>
{% endif %}
```

### Exchange rate rows
```liquid
{% if document.exchange %}
  <tr>{{labels.tax}} ({{account.currency}}): {{document.tax_amount_exchange}}</tr>
  <tr>{{labels.total}} ({{account.currency}}): {{document.total_exchange}}</tr>
{% endif %}
```

### Item discount rate (modern, frame, mono, professional)
```liquid
{% if item.discount_rate > 0 %}
  <small>({{ labels.discount }} {{ item.discount_rate | precision: 0 }}%)</small>
{% endif %}
```

### Payment details / paid state
```liquid
{% if document.payment_details != blank %}
  ...payment_details | textilize...
{% elsif document.state == "paid" %}
  {% assign payment = document.payments | last %}
  <strong>{{labels.paid}}</strong> — {{ payment.payment_method_text }}
{% endif %}
```

### Notes
```liquid
{% if document.notes != blank %}
  ...document.notes | textilize...
{% endif %}
```

### Legal text
```liquid
{% if document.legal != blank %}
  ...document.legal | textilize...
{% endif %}
```

---

## 4. Liquid Filters

| Filter | Usage | Templates |
|---|---|---|
| `default: 'en'` | `contact.language \| default: 'en'` | All |
| `date: 'default', contact.country` | Locale-aware date formatting | All |
| `date: 'long'` | Long date format | classic, newspaper |
| `precision` | `item.quantity \| precision`, `item.unit_price \| precision`, `tax.taxable_base \| precision` | All |
| `precision: 0` | `item.discount_rate \| precision: 0` | modern, frame, mono, professional |
| `newline_to_br` | Addresses, notes, payment_details | All |
| `textilize` | notes, payment_details, legal | All |
| `localize_document_type: labels` | `document.related_document_type \| localize_document_type: labels` | All |
| `last` | `document.payments \| last` | All (via assign) |
| `strip_newlines` | `account.formatted_address \| newline_to_br \| strip_newlines` | classic footer only |
| `replace: "<br />", " – "` | Inline address formatting | classic footer only |
| `upcase` | `labels.subtotal \| upcase` etc. | mono only |
| `default: 'Tax'` | `labels.tax \| default: 'Tax'` | modern, frame, classic |
| `size` | `document.taxes.size > 0` | modern, classic, frame, mono, minimal, professional |

---

## 5. CSS Patterns, Table Structures & Print Optimisations

### PDF Meta Tags (pdfkit)
All templates set A4 page size and margins via `<meta>` tags:
```html
<meta name="pdfkit-page_size" content="A4">
<meta name="pdfkit-margin_top" content="13mm">
<meta name="pdfkit-margin_right" content="13mm">
<meta name="pdfkit-margin_bottom" content="13mm">
<meta name="pdfkit-margin_left" content="13mm">
```
- **classic** uses `0mm` margins and handles spacing in CSS body padding + the "ghost spacer" pattern
- **mono** uses `15mm` margins (slightly wider)

### Print Optimisations
All templates include an `@media print` block. Common rules:

```css
@media print {
  body { -webkit-print-color-adjust: exact; }  /* preserve background colors */
  thead { display: table-header-group; }        /* repeat header on each page */
  tfoot { display: table-footer-group; }        /* push footer to page bottom */
  tr { page-break-inside: avoid; }              /* keep rows together */
}
```
Variations:
- `destijl` and older templates reference `#items tr` / `#items tfoot` rather than generic selectors
- `classic` uses `thead { display: table-header-group; }` for ghost-header repeat

### `account.color_scheme` Usage in CSS
All templates embed `{{account.color_scheme}}` directly in `<style>` blocks as a CSS value:
```css
h1 { color: {{account.color_scheme}}; }
border-bottom: 2px solid {{account.color_scheme}};
background-color: {{account.color_scheme}};
```
This is the primary theming mechanism across all templates.

### Verification Code
All templates include this CSS class (not always used in markup, reserved for injected content):
```css
.verification-code { margin: 10px 0; text-align: right; font-size: small; }
```

### Table Structure Patterns

#### Simple float-free layout (destijl — oldest)
Uses a single outer `<table>` with `colspan` for layout. Items table nested inside. No flex/grid.

#### Full-width master table with tfoot-as-footer (modern, frame, mono, professional)
```html
<table class="master-table">
  <tfoot><!-- footer always at page bottom --></tfoot>
  <tbody>
    <!-- header row -->
    <!-- metadata row -->
    <!-- address row -->
    <!-- items header row -->
    <!-- items loop rows -->
    <!-- totals row -->
    <!-- tax breakdown row -->
  </tbody>
</table>
```
Placing `<tfoot>` before `<tbody>` in source is intentional — browsers render it at the bottom of the table.

#### Frame pattern (frame.html)
Adds a `<thead>` with a coloured 3px top bar (`frame-top-bar`) so the page border continues across print pages. The outer `<table>` has `border-left`, `border-right`, `border-bottom` set to `{{account.color_scheme}}` but `border-top: none` (the thead handles the top).

#### Ghost header / margin spacer (classic.html)
Uses `0mm` pdfkit margins. Provides `padding` on `.hero-container` with negative margins to bleed the grey background to the page edge. A ghost `<thead>` row (`.margin-spacer`) with `height: 13mm` recreates the top margin on every print page so the first real content row doesn't sit against the paper edge.

### Items Table Column Widths

Standard 4-column layout across all templates:

| Column | Typical width | Alignment |
|---|---|---|
| Description | remaining / ~40–50% | left |
| Quantity | 10–18% | left or right |
| Unit price | 15–20% | left or right |
| Amount | 15–22% | right |

### Totals Layout Patterns

**Inline paragraph style** (minimal, professional — older):
```html
<tfoot><tr><td colspan="4">
  <p>{{labels.subtotal}}: {{document.subtotal}}</p>
  <p><strong>{{labels.total}}: {{document.total}}</strong></p>
</td></tr></tfoot>
```

**Two-column 60/40 split** (modern, frame, mono):
```html
<tr>
  <td style="width:60%"></td>   <!-- spacer -->
  <td style="width:40%">        <!-- totals table -->
    <table class="totals-table">...</table>
  </td>
</tr>
```

**Row-per-total in master tbody** (classic.html):
Each total line is its own `<tr class="total-row">` in the master table, using `colspan` for alignment.

### Typography / Font Stacks by Template

| Template | Primary font | Notable style |
|---|---|---|
| destijl | Helvetica, Arial | Bold bordered labels |
| classic | Calibri / Segoe UI / Roboto | 9pt, grey hero area |
| minimal | Arial, Helvetica | 10pt, coloured thead |
| modern | Segoe UI / Tahoma / Verdana | Rounded-corner pill rows |
| frame | Helvetica Neue | Bordered frame, uppercase title |
| mono | Lucida Console / Courier | Monospace, all uppercase |
| newspaper | Georgia / Times | Serif, Impact for totals |
| professional | Tahoma / Geneva | Compact, coloured borders |

---

## 6. destijl.html — Detailed Reference

`destijl.html` is the most complete and long-standing reference template.

### Structure
```
<html>
  <head> — pdfkit meta + CSS </head>
  <body>
    <table>                          ← single outer layout table
      <tr> header (logo + title) </tr>
      <tr> from/to/number columns </tr>
      <tr>
        <td> optional meta (PO, dates, related doc) </td>
        <td colspan="2">
          <table id="items">         ← items table
            <thead> column headers </thead>
            <tbody> item rows </tbody>
            <tfoot> totals </tfoot>
          </table>
        </td>
      </tr>
      <tr> notes + payment details </tr>
    </table>
    <div class="legal"> legal text </div>
  </body>
</html>
```

### Unique Features
- **Three-column header row**: [Number + dates] | [From/seller] | [To/buyer]
- **Left metadata column**: PO number, due date, valid_until, related document all stack vertically in a narrow left column alongside the items table
- **Border system**: Uses `border-first` (4px color), `border2` (2px color), `border1` (1px black) CSS classes for visual hierarchy
- **Tax loop uses `tax.label × tax.taxable_base`** format — no separate breakdown table
- **Payment fallback**: `{% elsif document.state == "paid" %}` with `{% assign payment = document.payments | last %}`
- **Notes conditional width**: Notes `<td>` gets `class="space12 w47per"` only when payment_details is also present

### CSS Classes (destijl-specific)
| Class | Purpose |
|---|---|
| `.label` | Uppercase small label text (11px) |
| `.smalltxt` | 14px text for notes/payment |
| `.txt` | 16px bold text |
| `.border-first` | 4px color top border |
| `.border2` | 2px color top border |
| `.border1` | 1px black top border + margin-top |
| `.space12` | `padding-right: 12px` |
| `.w300`, `.w18per`, `.w20per`, etc. | Fixed width utilities |
| `.valign-bottom`, `.valign-top` | Vertical alignment |
| `.legal` | Centred footer legal text |
| `.verification-code` | Right-aligned small verification text |

---

## 7. Cross-Template Summary

### Variables present in ALL templates
`contact.language`, `document.number`, `document.issue_date`, `document.contact`, `document.billing_address`, `document.items`, `item.description`, `item.quantity`, `item.unit_price`, `item.subtotal`, `document.subtotal`, `document.total`, `document.currency`, `document.taxes`, `document.exchange`, `document.tax_amount_exchange`, `document.total_exchange`, `document.notes`, `document.payment_details`, `document.legal`, `account.logo_url`, `account.full_name`, `account.formatted_address`, `account.registration_number`, `account.tax_id`, `account.color_scheme`, `account.currency`, `contact.country`, `contact.contact_person`, `contact.department`, `document.tax_id`, `document.state`, `document.payments`, `title`

### Variables only in newer templates (modern, classic, frame, mono, professional)
`account.trade_name`, `contact.full_name`, `item.discount_rate`, `document.tax_amount`, `document_tax.rate`, `document_tax.taxable_base` (as separate field in breakdown table)

### Variables only in specific templates
- `account.phone_1` — classic only
- `document.subject` displayed as inline text in header — all, but shown differently
- `labels.subject` — classic, newspaper, professional only (label for subject field)
