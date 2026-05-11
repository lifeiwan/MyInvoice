# Invoice Redesign + Customer Save — Design Spec

**Date:** 2026-05-11  
**File:** `invoice_generator.html`

---

## Overview

Redesign the invoice generator to match the C&C Royal Service sample invoice format (clean minimal style), update line items to a date-based flat-fee structure, add Tips and Cleaning Supplies rows, and implement a save/load customer feature backed by localStorage.

---

## Invoice Format

### Overall Style
- Invoice document: white background, black text, simple borders — no dark headers, no rust color strip
- Left form panel: keep existing dark theme

### Header Block (two-column)
```
[Company Name]                    SERVICE INVOICE
[Address]
Contact: [Contact Name]           Invoice No. [#]
TEL: [Phone]                      [Invoice Date]
Email: [Email]
```

### Bill To Block
```
BILL TO
[Customer Name]
[Customer Address]
TEL: [Customer Phone]
```
(Customer email shown if provided)

### Line Items Table
Columns: **DATE | DESCRIPTION | AMOUNT**  
Each row: a date (MM/DD/YYYY), a description, and a flat dollar amount.

### Footer Rows (below table body)
| Label | Value |
|-------|-------|
| Tax (N%) | $X.XX |
| Tips *(hidden if $0)* | $X.XX |
| Cleaning Supplies *(hidden if $0)* | $X.XX |
| **Grand Total** | **$X.XX** |

Grand Total = sum of line items + tax + tips + cleaning supplies.

### Remarks
Payment instructions text shown below the table.

---

## Form Panel Fields

### Your Business
- Business Name
- Address
- Contact Name *(new field)*
- Phone
- Email

### Invoice Details
- Invoice #
- Invoice Date (single date, not issue+due)

### Customer
- **Saved Customers dropdown** — lists saved names; selecting auto-fills fields below
- **Save Customer button** — saves/overwrites current customer by name in localStorage
- Full Name
- Address
- Phone
- Email

### Work Items
Each line item row: **Date picker | Description | Amount**  
(replaces Hours + Rate/hr)

### Tax & Extras
- Tax Rate (%)
- Tips ($) — input field; row hidden in preview/PDF when $0
- Cleaning Supplies ($) — input field; row hidden in preview/PDF when $0

### Remarks
Payment instructions textarea.

### Export Buttons
- Export PDF
- Export CSV

---

## Customer Save Feature

- Storage: `localStorage` key `invoice_customers` → JSON array of `{name, address, phone, email}`
- **Save Customer**: stores current customer fields; if name already exists, overwrites it
- **Dropdown**: populated from saved list; placeholder "Select saved customer…"; selecting an entry auto-fills the four customer fields
- No delete UI in this version (YAGNI)

---

## PDF Export

jsPDF-rendered PDF matches the invoice preview layout exactly:
- Clean white page, black text
- Header two-column block
- Bill To block
- Table with DATE | DESCRIPTION | AMOUNT columns
- Footer rows: Tax, Tips (if >0), Cleaning Supplies (if >0), Grand Total
- Remarks at bottom

---

## CSV Export

Columns updated to: Date, Description, Amount  
Footer rows: Tax, Tips (if >0), Cleaning Supplies (if >0), Grand Total

---

## Out of Scope

- Customer delete/edit UI
- Multiple saved business profiles
- Cloud sync
