# New Invoice Button + Folder Picker on Export — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add a "New Invoice" reset button at the top of the form, and replace PDF/CSV export with a native Save As dialog (File System Access API).

**Architecture:** All changes are in `invoice_generator.html`. Task 1 refactors `INV_PREFIX` into a reassignable variable so Task 2 can regenerate it on reset. Tasks 3 and 4 convert the two export functions to `async` and swap the trigger-download pattern for `showSaveFilePicker`. No new files, no new dependencies.

**Tech Stack:** Vanilla JS, jsPDF 2.5.1 (CDN), File System Access API (`window.showSaveFilePicker`)

---

### Task 1: Refactor `INV_PREFIX` into a reassignable variable

**Files:**
- Modify: `invoice_generator.html` — lines 499–510

- [ ] **Step 1: Replace the `const` with a `let` and extract a helper**

Find this block (around line 499):

```js
const INV_PREFIX = Array.from({length: 5}, () =>
  String.fromCharCode(65 + Math.floor(Math.random() * 26))
).join('');

function autoInvoiceNumber() {
  const val = document.getElementById('inv-date').value;
  if (val) {
    const [y, m, d] = val.split('-');
    document.getElementById('inv-number').value = INV_PREFIX + m + d + y.slice(2);
  }
  updatePreview();
}
```

Replace it with:

```js
function randomPrefix() {
  return Array.from({length: 5}, () =>
    String.fromCharCode(65 + Math.floor(Math.random() * 26))
  ).join('');
}

let invPrefix = randomPrefix();

function autoInvoiceNumber() {
  const val = document.getElementById('inv-date').value;
  if (val) {
    const [y, m, d] = val.split('-');
    document.getElementById('inv-number').value = invPrefix + m + d + y.slice(2);
  }
  updatePreview();
}
```

- [ ] **Step 2: Verify the invoice number still auto-generates on page load**

Open `invoice_generator.html` via a local server (e.g., `python3 -m http.server`). The Invoice # field should be pre-filled with a 5-letter prefix + today's date (e.g., `XWKQM062526`). Change the Invoice Date field — the number should update to reflect the new date.

- [ ] **Step 3: Commit**

```bash
git add invoice_generator.html
git commit -m "refactor: extract randomPrefix() so invPrefix can be reassigned on new invoice"
```

---

### Task 2: Add `newInvoice()` function and button

**Files:**
- Modify: `invoice_generator.html` — brand header HTML (~line 199), JS section

- [ ] **Step 1: Add the button to the brand header HTML**

Find this block (around line 199):

```html
<div class="brand">
  <div class="brand-label">Invoice Generator</div>
  <div class="brand-name">C&amp;C Royal Service</div>
</div>
```

Replace it with:

```html
<div class="brand" style="display:flex;justify-content:space-between;align-items:flex-end;">
  <div>
    <div class="brand-label">Invoice Generator</div>
    <div class="brand-name">C&amp;C Royal Service</div>
  </div>
  <button class="btn-small" onclick="newInvoice()" style="margin-bottom:4px;">+ New Invoice</button>
</div>
```

- [ ] **Step 2: Add the `newInvoice()` function**

Add the following function immediately after `autoInvoiceNumber()` (around line 511, before `toggleSection`):

```js
function newInvoice() {
  if (!confirm('Start a new invoice? Current data will be cleared.')) return;

  invPrefix = randomPrefix();

  const today = new Date();
  const y = today.getFullYear();
  const m = String(today.getMonth() + 1).padStart(2, '0');
  const d = String(today.getDate()).padStart(2, '0');
  document.getElementById('inv-date').value = `${y}-${m}-${d}`;

  autoInvoiceNumber();

  ['cust-name', 'cust-contact', 'cust-address', 'cust-phone', 'cust-email'].forEach(id => {
    document.getElementById(id).value = '';
  });
  document.getElementById('cust-saved').selectedIndex = 0;

  document.getElementById('line-items').innerHTML = '';
  addLineItem();

  updatePreview();
}
```

- [ ] **Step 3: Verify in the browser**

1. Fill in a customer name and add 2–3 line items.
2. Click `+ New Invoice`.
3. Confirm dialog appears. Click **Cancel** — nothing changes.
4. Click `+ New Invoice` again. Click **OK**.
5. Customer fields are blank. Line items show one empty row. Invoice # has a new prefix and today's date. Invoice date is today. Business info is unchanged.

- [ ] **Step 4: Commit**

```bash
git add invoice_generator.html
git commit -m "feat: add New Invoice button that resets customer and work items"
```

---

### Task 3: Update `exportPDF()` to use `showSaveFilePicker`

**Files:**
- Modify: `invoice_generator.html` — `exportPDF()` (~line 917)

- [ ] **Step 1: Replace `exportPDF()`**

Find this function (around line 917):

```js
function exportPDF() {
  const doc = buildPDF();
  const invNumber = document.getElementById('inv-number').value;
  doc.save((invNumber || 'invoice') + '.pdf');
}
```

Replace it with:

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

- [ ] **Step 2: Verify in Chrome/Edge**

1. Open the app via a local server.
2. Add a line item and click **Export PDF**.
3. A Save As dialog appears with the suggested filename pre-filled (e.g., `XWKQM062526.pdf`).
4. Choose a folder and save. Open the saved file — it should be a valid PDF.
5. Click **Cancel** in the dialog — no error, nothing happens.

- [ ] **Step 3: Commit**

```bash
git add invoice_generator.html
git commit -m "feat: use showSaveFilePicker for PDF export with auto-download fallback"
```

---

### Task 4: Update `exportCSV()` to use `showSaveFilePicker`

**Files:**
- Modify: `invoice_generator.html` — `exportCSV()` (~line 675)

- [ ] **Step 1: Make `exportCSV` async and replace the download block**

Find the end of `exportCSV()` (around line 712–718):

```js
  const csv = rows.map(r => r.map(q).join(',')).join('\n');
  const blob = new Blob([csv], { type: 'text/csv' });
  const a = document.createElement('a');
  a.href = URL.createObjectURL(blob);
  a.download = (document.getElementById('inv-number').value || 'invoice') + '.csv';
  a.click();
}
```

Replace it with:

```js
  const csv = rows.map(r => r.map(q).join(',')).join('\n');
  const invNumber = document.getElementById('inv-number').value;
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
}
```

Also change the function signature from `function exportCSV()` to `async function exportCSV()`.

- [ ] **Step 2: Verify in Chrome/Edge**

1. Add a line item with a date, description, and amount.
2. Click **Export CSV**.
3. A Save As dialog appears with the suggested filename pre-filled (e.g., `XWKQM062526.csv`).
4. Choose a folder and save. Open the file in Excel or a text editor — rows should be correct with all invoice fields.
5. Click **Cancel** in the dialog — no error, nothing happens.

- [ ] **Step 3: Commit**

```bash
git add invoice_generator.html
git commit -m "feat: use showSaveFilePicker for CSV export with auto-download fallback"
```
