# New Invoice Button + Folder Picker on Export — Design Spec

**Date:** 2026-06-25

## Overview

Two independent UX improvements to `invoice_generator.html`:

1. **New Invoice button** — resets the form for a fresh invoice without reloading the page.
2. **Folder picker on export** — lets the user choose where to save PDF and CSV files via the browser's native Save As dialog.

---

## Feature 1: New Invoice Button

### Placement

A `+ New Invoice` button in the brand header row at the top of the left panel, right-aligned next to the title. Styled as a small secondary button (`.btn-small` class already exists).

### Behavior

Clicking calls `newInvoice()`:

1. `confirm("Start a new invoice? Current data will be cleared.")` — returns early if user cancels.
2. Generate a new 5-letter random prefix and assign it to `invPrefix` (a `let` module-level variable replacing the current `const INV_PREFIX`).
3. Set `#inv-date` to today's ISO date string.
4. Call `autoInvoiceNumber()` to rebuild `#inv-number` from the new prefix + today's date.
5. Clear customer fields: `#cust-name`, `#cust-address`, `#cust-phone`, `#cust-email`, `#cust-contact`, `#cust-notes` (any that exist).
6. Reset `#cust-saved` dropdown to its first option ("Select saved customer…").
7. Remove all `.line-item-row` elements from `#line-items-container`.
8. Call `addLineItem()` to insert one blank row.
9. Call `updatePreview()`.

### What is NOT reset

- Business info fields (`#biz-name`, `#biz-address`, `#biz-contact`, `#biz-phone`, `#biz-email`)
- Email settings (Brevo API key, sender email, BCC)
- Tax rate (`#tax-rate`), tips (`#tips`), supplies (`#supplies`)
- Remarks textarea (`#inv-note`)

### Code change: `INV_PREFIX` → `invPrefix`

The current `const INV_PREFIX` is a module-level constant set once at page load. Change it to:

```js
let invPrefix = randomPrefix(); // function extracted for reuse

function randomPrefix() {
  return Array.from({length: 5}, () =>
    String.fromCharCode(65 + Math.floor(Math.random() * 26))
  ).join('');
}
```

`autoInvoiceNumber()` uses `invPrefix` instead of `INV_PREFIX`.

---

## Feature 2: Folder Picker on Export

### Mechanism

Use the [File System Access API](https://developer.mozilla.org/en-US/docs/Web/API/File_System_API) — specifically `window.showSaveFilePicker` — to open a native Save As dialog. Available in Chrome and Edge. Falls back to the current auto-download behavior in unsupported browsers.

### `exportPDF()` changes

```js
async function exportPDF() {
  const invNumber = document.getElementById('inv-number').value;
  const blob = buildPDF().output('blob');
  if (window.showSaveFilePicker) {
    try {
      const handle = await window.showSaveFilePicker({
        suggestedName: (invNumber || 'invoice') + '.pdf',
        types: [{ description: 'PDF', accept: { 'application/pdf': ['.pdf'] } }],
      });
      const writable = await handle.createWritable();
      await writable.write(blob);
      await writable.close();
    } catch {}
  } else {
    const a = document.createElement('a');
    a.href = URL.createObjectURL(blob);
    a.download = (invNumber || 'invoice') + '.pdf';
    a.click();
  }
}
```

### `exportCSV()` changes

At the end of the existing function, replace the `a.click()` block:

```js
  const blob = new Blob([csv], { type: 'text/csv' });
  if (window.showSaveFilePicker) {
    try {
      const handle = await window.showSaveFilePicker({
        suggestedName: (invNumber || 'invoice') + '.csv',
        types: [{ description: 'CSV', accept: { 'text/csv': ['.csv'] } }],
      });
      const writable = await handle.createWritable();
      await writable.write(blob);
      await writable.close();
    } catch {}
  } else {
    const a = document.createElement('a');
    a.href = URL.createObjectURL(blob);
    a.download = (invNumber || 'invoice') + '.csv';
    a.click();
  }
```

`exportCSV` must also become `async`.

### Error handling

The `try/catch` silently swallows the `AbortError` thrown when the user cancels the picker dialog. No user-facing error is shown for cancels. Actual write errors (disk full, permissions) would also be silent — acceptable for a local single-user tool.

---

## Affected Code in `invoice_generator.html`

| Area | Change |
|---|---|
| Brand header HTML | Add `+ New Invoice` button |
| `const INV_PREFIX` | Replace with `let invPrefix` + `randomPrefix()` function |
| `autoInvoiceNumber()` | Reference `invPrefix` instead of `INV_PREFIX` |
| New `newInvoice()` function | Confirmation + field reset + new prefix + re-init |
| `exportPDF()` | Async, use `output('blob')` + `showSaveFilePicker` with fallback |
| `exportCSV()` | Async, use `showSaveFilePicker` with fallback |

No new dependencies. No new CSS classes needed beyond what exists.
