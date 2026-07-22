# MyInvoice — CLAUDE.md

## Project Overview

A self-contained single-file invoice generator for **C&C Royal Service Inc.** (cleaning services). Everything lives in `invoice_generator.html` — no build system, no npm, no backend.

## File Structure

```
invoice_generator.html   — the entire app (HTML + CSS + JS, ~1200 lines)
index.html               — redirect or alias (not the main file)
logo.jpg                 — original logo (replaced by PNG in the PDF)
docs/superpowers/        — plans and specs from brainstorming sessions
.gitignore               — ignores .DS_Store, *.pdf, *.xlsx
```

## Dependencies (CDN, no install)

- **jsPDF 2.5.1** — `https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js`  
  Used for PDF generation in `buildPDF()` / `exportPDF()`.
- **Google Fonts** — DM Serif Display + DM Mono  
  Used for left-panel UI. The invoice preview and PDF use Arial/Helvetica.
- **Brevo REST API** — `https://api.brevo.com/v3/smtp/email`  
  Used for email sending with PDF attachment. Requires user to configure credentials.

No other dependencies.

## Architecture

Two-panel layout:
- **Left panel** (dark, 400px) — form inputs: business info, customer, line items, email settings, action buttons
- **Right panel** (cream) — live invoice preview (HTML table, updates on every input)

PDF output mirrors the live preview layout with gold header/footer bars (`#FFC000`), logo, and a DATE | DESCRIPTION | TOTAL table.

## Key Functions

| Function | Purpose |
|---|---|
| `updatePreview()` | Re-renders the right-panel invoice preview from current form values |
| `buildPDF()` | Builds and returns a jsPDF doc object (does not save) |
| `exportPDF()` | Calls `buildPDF().save(filename)` |
| `exportCSV()` | Downloads line items as CSV |
| `getItems()` | Returns line item rows as `{date, desc, amount}[]`, sorted chronologically |
| `getServiceMonth()` | Derives month string(s) from work item dates for the email body |
| `addLineItem()` | Adds a new work item row to the DOM |
| `removeItem(id)` | Removes a line item row |
| `syncMultiDateBtn(id)` | Enables/disables the `+Dates` button based on whether amount > 0 |
| `saveCustomer()` | Saves current customer to `localStorage` |
| `loadCustomer()` | Loads selected customer into form fields |
| `deleteCustomer()` | Removes a customer from `localStorage` |
| `exportCustomers()` | Downloads all saved customers as `customers.json` |
| `importCustomers(input)` | Merges imported JSON file into saved customers (same name = overwrite) |
| `saveEmailSettings()` | Persists Brevo API key, sender email, BCC to `localStorage` |
| `initEmailSettings()` | On page load: restores email settings from `localStorage` |
| `toggleApiKeyVis(btn)` | Toggles API key input between `password` and `text` type |
| `sendEmail()` | Validates settings, generates PDF base64, opens compose modal |
| `confirmSendEmail()` | Reads compose modal and POSTs to Brevo API |
| `closeEmailModal()` | Hides the email compose modal |
| `openMultiDatePicker(id)` | Opens the multi-date calendar picker for a line item |
| `confirmMultiPicker()` | Expands selected dates into individual line item rows |
| `toggleSection(id, toggle)` | Toggles collapsible sections in the left panel |

## localStorage Keys

| Key | Contents |
|---|---|
| `invoice_customers` | `Customer[]` — `{name, contactName, address, email, phone, notes}` |
| `invoice_smtp` | `{apiKey, senderEmail, bcc}` — Brevo credentials, never in source |

## CSS Variables (`:root`)

```css
--ink: #1a1a18       /* dark background */
--paper: #f5f2eb     /* warm off-white */
--cream: #ede9df     /* right panel background */
--rust: #b85c38      /* primary action color */
--rust-light: #d4724e
--gold: #FFC000      /* invoice header/footer, matches Excel template */
--muted: #7a7468
--border: #ccc7bb
--white: #fdfcf9
```

## PDF Generation Conventions

- Page size: A4 (jsPDF default)
- Constants defined inside `buildPDF()`: `W`, `MARGIN`, `GOLD` (`[255, 192, 0]`), `PAGE_H`
- Gold bars: `doc.rect(0, 0, W, 12, 'F')` top and `doc.rect(0, PAGE_H - 12, W, 12, 'F')` bottom
- Logo: PNG base64 embedded in `LOGO_B64` constant (extracted from Excel template)
- Table columns: DATE (left) | DESCRIPTION (center) | TOTAL (right-aligned)
- Images added via `doc.addImage(LOGO_B64, 'PNG', ...)`

## Email Flow

1. User configures Brevo API key + sender email in **Email Settings** section → saved to `localStorage`
2. User clicks **Send Email** → `sendEmail()` validates settings, generates PDF base64, pre-fills compose modal
3. User reviews/edits the compose modal (To, BCC, Subject, Body) → clicks **Send**
4. `confirmSendEmail()` POSTs to `https://api.brevo.com/v3/smtp/email` with attachment

**Important:** Brevo API requires serving the file over HTTP (not `file://` protocol). Use a local server (`python3 -m http.server` or Live Server extension) for development.

## Conventions

- **No build step** — edit `invoice_generator.html` directly, open in browser or local server to test
- **No comments in JS** unless the reason is non-obvious
- **No external state** — everything derives from the DOM and `localStorage`
- **Chronological sort** — `getItems()` always returns rows sorted by date ascending; CSV and PDF follow the same order
- **`+Dates` button** is disabled until the line item has `amount > 0`
- **Contact Name** field is optional; falls back to Full Name for email greeting
- **Customer import merges** by name — same name overwrites, new name appends
