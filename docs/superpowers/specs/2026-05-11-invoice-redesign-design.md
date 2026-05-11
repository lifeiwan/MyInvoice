# Invoice Redesign + Customer Save — Design Spec

**Date:** 2026-05-11  
**File:** `invoice_generator.html`

---

## Overview

Redesign the invoice generator to match the C&C Royal Service sample invoice format (clean minimal style), update line items to a date-based flat-fee structure, add Tips and Cleaning Supplies rows, implement a save/load/delete customer feature backed by localStorage, and reserve a placeholder Send Email button for future EmailJS integration.

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
Tips and Cleaning Supplies rows are completely hidden (not rendered) when their value is $0.

### Remarks
Payment instructions text shown below the table.

---

## Form Panel Fields

### Your Business *(collapsible, collapsed by default)*
Pre-filled with C&C Royal Service defaults:
- Business Name: `C&C ROYAL SERVICE INC.`
- Address: `75-08 Bell Blvd, Bayside, NY 11364`
- Contact Name: `Wendy Chiu`
- Phone: `917-518-1718`
- Email: `ccroyalservice@gmail.com`

Section is collapsed by default (click to expand) since this data rarely changes. A collapse toggle shows the business name as a summary when folded.

### Invoice Details
- Invoice # — default empty, user types (e.g. WE041326)
- Invoice Date — date picker, default today

### Customer
- **Saved Customers dropdown** — lists saved customer names; selecting auto-fills all four fields below
- **Save Customer button** — saves/overwrites current customer by name in localStorage
- **Delete button** — removes the currently selected saved customer (only active when a saved customer is selected)
- Full Name
- Address
- Phone
- Email

### Work Items
Each line item row: **Date picker | Description | Amount**  
- Date picker defaults to today's date
- Description defaults to `Cleaning Service`
- Amount: flat dollar input

### Tax & Extras
- Tax Rate (%) — default 5%
- Tips ($) — input, default 0; row hidden in preview/PDF when $0
- Cleaning Supplies ($) — input, default 0; row hidden in preview/PDF when $0

### Remarks
Payment instructions textarea.

### Export / Action Buttons
- **Export PDF** — generates and downloads the invoice as PDF
- **Export CSV** — downloads invoice data as CSV
- **Send Email** *(placeholder, disabled)* — reserved for future EmailJS integration; shown as a grayed-out button with tooltip "Coming soon"

---

## Customer Save Feature

- Storage: `localStorage` key `invoice_customers` → JSON array of `{name, address, phone, email}`
- **Save Customer**: stores/overwrites entry matching current customer name
- **Dropdown**: placeholder "Select saved customer…"; selecting auto-fills the four customer fields and activates the Delete button
- **Delete**: removes the selected customer from localStorage and clears the dropdown selection

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

Columns: Date, Description, Amount  
Footer rows: Tax, Tips (if >0), Cleaning Supplies (if >0), Grand Total

---

## Future: Email Feature

A "Send Email" button is reserved in the UI. Future implementation will use **EmailJS** (browser-side email API, free tier 200/month) to send the generated PDF as a base64 attachment. Requires user to provide an EmailJS API key and template ID — to be configured when implemented.

---

## Out of Scope

- Customer edit UI (save overwrites, delete removes)
- Multiple saved business profiles
- Cloud sync
