# Gestalt Invoice Template — Project Guidelines

## Project Overview
Build a production-ready HTML/CSS invoice template called "Gestalt" for the Quaderno invoicing platform. The template must be compliant, accessible, and follow Quaderno's Liquid templating conventions.

## Reference Materials

### Existing Templates (analyse for patterns)
- All HTML files in the repo root are existing Quaderno invoice templates.
- Pay special attention to: Liquid variable names, label conventions, conditional patterns, and CSS approaches.
- The De Stijl template (`destijl.html`) is the most feature-complete reference.

### Design Reference
- The PDF in `/design/neo-invoice-design.pdf` is the visual design specification for the Gestalt template.
- Match the layout, spacing, typography hierarchy, and section ordering shown in the design.

### Documentation (read and follow)
- Quaderno template customisation: https://developers.quaderno.io/templates/
- Liquid templating language: https://shopify.github.io/liquid/basics/introduction/

## Technical Requirements

### HTML & CSS
- HTML5 semantic markup.
- CSS3 only — no preprocessors, no JavaScript.
- Single-file template (all CSS in a `<style>` block within the HTML).
- A4 page size (210mm × 297mm).
- **Measurement units: use px for all CSS values (fonts, spacing, borders, widths). The only exception is mm, which is used exclusively in the pdfkit `<meta>` tags for page margins.**
- Print-optimised with `@media print` rules.
- Use `@page` meta tags consistent with Quaderno's PDF rendering (pdfkit).
- Quaderno uses wkhtmltopdf for PDF rendering — do NOT use flexbox or CSS grid. Use `<table>` for all layouts. Follow the same structural approach as the existing templates.

### Liquid Templating
- Use ONLY Quaderno's Liquid variables and labels as documented at https://developers.quaderno.io/templates/
- Use `labels.*` for all user-facing text (never hardcode text like "Invoice" or "Subtotal").
- Use Quaderno's Liquid filters: `precision`, `date`, `textilize`, `newline_to_br`, `localize_document_type`.
- All labels and variables must match those used in the existing reference templates.

### Complete Element List & Conditional Logic

Every conditional field must be wrapped in `{% if variable != blank %}` (or `> 0` for discount). The template must render cleanly with ALL conditional fields absent.

**1. Header**
- Verification code — CONDITIONAL — for tax authority codes (e.g., Verifactu, QR)
- Logo — CONDITIONAL
- Document title — ALWAYS
- Document subject — CONDITIONAL
- Document number — ALWAYS
- Payment status tag — ALWAYS — styled per status (Draft, Outstanding, Paid, Overdue)
- Payment method — CONDITIONAL — only shown when status is Paid

**2. Metadata**
- Issue date — ALWAYS
- Service date — ALWAYS — Mandatory in France. Currently mirrors issue_date.
- Due date — CONDITIONAL
- PO number — CONDITIONAL
- Valid until — CONDITIONAL
- Referenced document — CONDITIONAL — shows type label, with the document number displayed as a clickable link to the original Proforma or Receipt
- Language / locale — ALWAYS (used for date formatting and label localisation via `contact.language`)

**3. Parties**
- From (Seller):
  - Name — ALWAYS
  - Address — ALWAYS
  - Registration number — CONDITIONAL
  - Tax ID / VAT — CONDITIONAL
  - Email — CONDITIONAL
- Billed to (Buyer):
  - Name — ALWAYS
  - Contact person — CONDITIONAL
  - Department — CONDITIONAL
  - Address — ALWAYS
  - Tax ID / VAT — CONDITIONAL
  - Email — CONDITIONAL
- Shipping address — CONDITIONAL (entire block)

**4. Line Items Table**
- Description — ALWAYS
- Quantity — ALWAYS
- Unit price — ALWAYS
- Line discount — CONDITIONAL (hide column if no items have discounts)
- Line subtotal — ALWAYS
- Line tax rate — ALWAYS
- Line amount (after tax) — ALWAYS

**5. Tax Breakdown (side by side with Summary, left side)**
- Tax label / name — ALWAYS
- Taxable base — ALWAYS
- Tax rate — ALWAYS
- Tax amount — ALWAYS
- Total tax — ALWAYS

**6. Summary / Totals (side by side with Tax Breakdown, right side)**
- Subtotal — ALWAYS
- Discount — CONDITIONAL (only if > 0, labeled simply "Discount")
- Total tax — ALWAYS
- Total — ALWAYS
- Amount paid — CONDITIONAL (hidden when no payment has been made)
- Balance due — CONDITIONAL (hidden when no payment has been made)

**7. Currency Conversion — CONDITIONAL (entire block, only if exchange data exists)**
- Exchange rate — ALWAYS (within block)
- Tax in domestic currency — ALWAYS (within block)
- Total in domestic currency — ALWAYS (within block)
- Balance due in domestic currency — ALWAYS (within block)
- Deposit rate note (exchange rate + date) — ALWAYS (within block)

**8. Other Info**
- Payment details — CONDITIONAL
- Notes — CONDITIONAL
- Legal notices — CONDITIONAL

**9. Footer** *(gestalt-footer.html — served via wkhtmltopdf --footer-html)*
- Color scheme bar (using `account.color_scheme`) — ALWAYS
- Invoice number — ALWAYS
- Page number — ALWAYS

### Expected Behaviors

**Locale & Language**
- Apply the user's locale language using `contact.language`. This affects date formatting, label translations, and the `lang` attribute on the `<html>` tag.

**Header — Status Tags**
- Invoice status is shown as a tag immediately after the document title.
- Each status has a different color scheme (see Payment Status Tags below).
- The four statuses are: Draft, Outstanding, Paid, and Overdue.
- If the status is "Paid" and a payment method is specified, the method is displayed after the tag.

**Subject — Separate Section**
- Document subject (conditional) is a standalone section between the Header and Metadata — it is NOT part of the header.
- 8px top margin separating it from the header above.
- Occupies full content width.

**Metadata — Flowing Layout**
- Metadata uses a flowing layout, NOT a fixed grid. Items fill cells sequentially left to right across 3 columns, wrapping to a new row after every 3 items.
- "Always displayed" items render first, followed by conditional items in order. Only conditionals that have data are included in the flow.
- The rendering order is:
  1. Issue Date — ALWAYS
  2. Due Date — CONDITIONAL
  3. Service Date — ALWAYS
  4. PO Number — CONDITIONAL
  5. Valid Until — CONDITIONAL
  6. Referenced Document — CONDITIONAL
- Implementation approach: build a Liquid array of metadata items that exist, then loop through the array rendering 3 per table row. Open a new `<tr>` every 3 items. This means the table could have 1 row (only 2 always items + 0 conditionals) or up to 2 rows (all 6 items present).
- Each cell contains a label (top) and value (below), stacked vertically.
- When only 2 items exist (Issue Date + Service Date with no conditionals), they fill columns 1 and 2 of a single row — column 3 is left empty.
- Row borders: if there are 2 rows, row 1 cells get a bottom border. If there is only 1 row, no bottom border.

**Parties — Responsive Columns**
- When a shipping address exists: 3 columns (From, Billed To, Shipping Address).
- When no shipping address: switches to 2 columns (From, Billed To) — the two columns expand to fill the full width.

**Line Items — Conditional Discount Column**
- Column order: Description | Qty | Unit Price | Subtotal | Discount | Tax Rate | Total
- If no line items have a discount, the Discount column is completely hidden (6-column layout: Description | Qty | Unit Price | Subtotal | Tax Rate | Total).
- When the Discount column is hidden, the Quantity, Unit Price, Subtotal, Tax Rate, and Total columns expand to fill the recovered space. The Description column does NOT expand.
- The Description column occupies 40–45% of the table width and remains fixed at that width regardless of whether the table has 7 columns (with discount) or 6 columns (without discount). Only the numeric columns redistribute.

**Other Info — Layout**
- Other Info (Payment Details, Notes, Legal Notices) is displayed immediately after the Summary area or Currency Conversion block. It is NOT part of the footer.
- Layout: Payment Details (left) + Notes (right) side by side, then Legal Notices full width below.
- Each section is conditional — if only one of Payment Details or Notes exists, it takes the full width.

**Footer — Separate File**
- Footer is in gestalt-footer.html, served via wkhtmltopdf `--footer-html`.
- Spacing between content and footer is controlled by engineering via `--footer-spacing`.
- Footer repeats on every page automatically via wkhtmltopdf.

**Pagination**
- When the invoice spills to multiple pages, content simply continues — no header or column headers are repeated on subsequent pages.
- Page numbers use wkhtmltopdf JS variables: `window.page` and `window.topage`, injected into `<span id="page">` and `<span id="topage">` via a `<script>` block in gestalt-footer.html.

### Payment Status Tags
The invoice displays a status tag in the header, positioned next to the document title.

**Tag styling (shared):**
- Shape: slightly rounded rectangle (small border-radius, approximately 4px).
- Text: capitalize (e.g., "Paid", "Draft"), slightly smaller font size than the document title.
- Border: 1px with 10% opacity of the text color, solid — EXCEPT Draft which uses dashed.
- Padding: enough to give the text breathing room inside the tag.

**Per-status styles:**

| Status      | Background | Text      | Border                          | Border Style |
|-------------|------------|-----------|---------------------------------|--------------|
| Draft       | `#EBEBEB`  | `#525252` | `#525252` at 10% opacity        | dashed       |
| Outstanding | `#FFEDD4`  | `#9F2D00` | `#9F2D00` at 10% opacity        | solid        |
| Paid        | `#A4F4C0`  | `#016630` | `#016630` at 10% opacity        | solid        |
| Overdue     | `#FFE2E2`  | `#9F0712` | `#9F0712` at 10% opacity        | solid        |

**Additional notes:**
- All status tags use `font-weight: 600` (semibold).
- The tag is rendered conditionally based on the document state.
- Use the Quaderno Liquid variable for document state (check the reference templates and documentation for the correct variable).

### Structural Colors (Borders, Backgrounds, Dividers)

**Page:**
- Background: `#FFFFFF`

**Table pattern (applies to Items, Tax Breakdown, Summary, and Currency Conversion tables):**
- Column headers: no background (transparent), bottom border `1px solid #171717`
- Row borders: bottom border `1px solid #E5E5E5`
- Last row in each table: NO bottom border

**Metadata:**
- Row 1 cells: bottom border `1px solid #E5E5E5` only when row 2 exists. No border when single row.

**Status tags:**
- Draft: border `1px dashed` `#525252` at 10% opacity
- Outstanding: border `1px solid` `#9F2D00` at 10% opacity
- Paid: border `1px solid` `#016630` at 10% opacity
- Overdue: border `1px solid` `#9F0712` at 10% opacity

**Footer:**
- Color bar: `background-color: {{account.color_scheme}}`, height 6px, fully rounded ends (`border-radius: 3px`), full content width (edge to edge).

### Typography System

**Font family:** Helvetica Neue, Arial, sans-serif (fallback stack)
**Line-height:** 1.4 ratio on all elements except Document title (1.2)
**Weights:** 400 (normal) and 600 (semibold) only — no 500 or 700. wkhtmltopdf doesn't visually distinguish 400 from 500, so 600 is used to ensure visible contrast.
**Colors:** Three-tier: `#171717` (primary), `#525252` (secondary), `#737373` (tertiary)
**Sizes:** Five-step scale: 8px, 9px, 11px, 13px, 24px. All sizes bumped +1px from the original design (7→8, 8→9, 10→11) to compensate for wkhtmltopdf's slight size reduction. Title stays at 24px. Invoice number (header) is 13px.

| Element | Size | Weight | Color | Style |
|---|---|---|---|---|
| Document title | 24px | 600 | #171717 | — |
| Status tag | 11px | 600 | (per status) | — |
| Payment method | 8px | 400 | #737373 | italic |
| Invoice number (header) | 13px | 400 | #525252 | — |
| Subject label | 11px | 600 | #171717 | — |
| Subject value | 11px | 400 | #525252 | — |
| Section labels (From, Billed To) | 11px | 600 | #171717 | — |
| Field labels (Issue Date, PO Number) | 11px | 600 | #171717 | — |
| Field values | 11px | 400 | #525252 | — |
| Table column headers | 8px | 600 | #171717 | — |
| Table cell content | 9px | 400 | #525252 | — |
| Subtotal / Discount / Total tax / Amount paid (labels + values) | 9px | 400 | #525252 | — |
| Total / Balance due (labels + values) | 9px | 600 | #171717 | — |
| Currency conversion label | 8px | 600 | #171717 | — |
| Exchange rate / Balance due / Total VAT labels | 9px | 600 | #737373 | — |
| Exchange rate / Balance due / Total VAT values | 9px | 400 | #737373 | — |
| Deposit paid note | 8px | 400 | #737373 | italic |
| Payment details label | 9px | 600 | #171717 | — |
| Payment details value | 9px | 400 | #525252 | — |
| Notes label | 9px | 600 | #171717 | — |
| Notes value | 9px | 400 | #525252 | — |
| Legal notices | 7px | 400 | #737373 | — |
| Footer — Invoice label | 9px | 600 | #171717 | — |
| Footer — Invoice value | 9px | 400 | #525252 | — |
| Pagination | 9px | 400 | #525252 | — |

### Design Specifications
- Logo frame: 150 × 55px.
- Financial Zone: Tax Breakdown (65% width, left) and Summary (35% width, right) side by side — implemented using a table, not flexbox. Currency Conversion section also uses 65% width, matching Tax Breakdown.
- Tax Breakdown and Summary tables do NOT need section title labels — column headers are self-explanatory.
- Currency Conversion label: "Currency Conversion" (hardcoded — no Quaderno label exists, see MISSING-LABELS.md).
- Balance Due: largest, boldest number on the invoice — visually unmissable.
- Global discount in summary: sits between Subtotal and Total Tax, labeled simply "Discount".
- Legal notices: smallest legible font (7–8pt), lighter text color.
- Color scheme: `account.color_scheme` applied to footer accent bar.
- Other Info layout: Payment Details (left) + Notes (right) side by side, Legal Notices full width below. Payment Details takes priority position (left) as the most actionable information.

### Summary Section Order
1. Subtotal
2. Discount (conditional, only if > 0)
3. Total Tax
4. Total
5. Amount Paid
6. Balance Due

## Quality Checklist
Before considering the template complete, verify:
- [ ] All Liquid variables match Quaderno's documentation exactly.
- [ ] All user-facing text uses `labels.*` — no hardcoded strings.
- [ ] Every CONDITIONAL field from the element list has a `{% if %}` wrapper.
- [ ] Template renders cleanly with ALL conditional fields empty.
- [ ] Template renders cleanly with ALL conditional fields populated.
- [ ] Payment status tag displays correctly for all four statuses (Draft, Outstanding, Paid, Overdue).
- [ ] Payment method only appears when status is Paid.
- [ ] Referenced document number renders as a clickable link.
- [ ] Amount Paid and Balance Due are fully hidden when no payment exists.
- [ ] Line discount column is hidden when no items have discounts.
- [ ] Discount row in summary only appears when > 0.
- [ ] Currency conversion block only appears when exchange data exists.
- [ ] Tax breakdown and summary numbers are mathematically consistent.
- [ ] Print styles produce clean A4 output.
- [ ] No deprecated HTML tags or CSS properties.
- [ ] No flexbox or CSS grid — tables only (wkhtmltopdf compatibility).
- [ ] CSS and structural patterns are consistent with existing templates in the repo.