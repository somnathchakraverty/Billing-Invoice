# Billing Invoice

**Live Demo:** [https://somnathchakraverty.github.io/Billing-Invoice/](https://somnathchakraverty.github.io/Billing-Invoice/)

A lightweight, browser-based invoice/receipt generator — no backend, no dependencies, no build step. Open the HTML file in any modern browser, fill in your details, and print or save as PDF.

---

## Files

| File | Purpose |
|---|---|
| `index.html` | **Main file.** Interactive editable receipt in thermal-printer style (280 px wide). |
| `invoice_print_ready.html` | A4 professional invoice layout — same data, full-page format. |
| `invoice_receipt.pdf` | Sample printed output generated from the main file. |

---

## Features

### invoice_html_replica.html

#### Editable fields (click any field to edit)
| Field | Location in receipt |
|---|---|
| Merchant name | Bold header — wraps to next line if long |
| Address lines (3) | Lines below merchant name |
| GSTIN | Below address |
| Mobile number | Below GSTIN |
| Customer name | After "Name:" label |
| Date | Left of the meta row |
| Order type | Right of the meta row (e.g. "Pick Up", "Dine In") |
| Time | Second meta row, left |
| Bill number | Second meta row, right |
| Cashier name | Third meta row |
| Item names | Inside the items table — wraps to two lines if long |
| Item quantity | Inside the items table |
| Item unit price | Inside the items table |
| Tax label (CGST / SGST) | Totals section — rename to anything |
| Tax percentage | Totals section — change rate, amounts recalculate instantly |

#### Dynamic calculations
- **Row amount** = Qty × Price — updates on every keystroke
- **Sub Total** = sum of all row amounts
- **CGST amount** = Sub Total × CGST %
- **SGST amount** = Sub Total × SGST %
- **Grand Total** = Sub Total + CGST + SGST

#### Item management
- **Add item** — click `+ Add Item` button above the receipt; a new blank row appears and the cursor jumps to its name field
- **Delete item** — click the `x` button on any row; totals recalculate immediately

#### Print / PDF
- Click **PRINT / SAVE AS PDF** button
- In the browser print dialog set **Destination → Save as PDF**
- All edit controls, delete buttons, and input borders are hidden automatically via `@media print`

---

## How to Use

1. Download or clone this repository
2. Open `invoice_html_replica.html` in Chrome or Edge
3. Click any field in the receipt and type to edit it
4. Use `+ Add Item` to add rows; `x` to remove them
5. Adjust tax labels and percentages as needed
6. Click **PRINT / SAVE AS PDF** → Save as PDF

No internet connection, server, or installation required.

---

## Code Structure — invoice_html_replica.html

The file is a single self-contained HTML file with three sections: CSS, HTML markup, and JavaScript.

### CSS

```
@page { size: auto; margin: 10mm; }
```
Sets the paper margins when the browser prints. `size: auto` lets the browser use its default paper size (A4 or Letter depending on OS settings).

**Key CSS classes:**

| Class | Role |
|---|---|
| `.receipt` | The 280 px white card that mimics a thermal receipt |
| `.fi` | Base style for all inline `<input>` editable fields — transparent background, no border by default, dashed underline on hover/focus |
| `.fi-center` | Makes an `.fi` input span the full width and center its text (used for address lines) |
| `.ce-field` | Applied to `contenteditable` divs (merchant name) — same hover/focus behaviour as `.fi` but allows text wrapping across lines |
| `.tbl-input` | Styles for qty and price `<input>` elements inside the items table — bold, transparent, borderless |
| `.tbl-ce` | `contenteditable` block for item names — `max-width: 110px` + `word-break: break-word` so long names wrap within the column |
| `.del-btn` | Small `x` delete button per row |
| `.divider` | 1 px dashed horizontal rule between sections |
| `.row` | Flexbox row with `justify-content: space-between` for left/right label pairs |

**Print rules (`@media print`):**
- Hides `.print-btn-wrap`, `.controls`, and `.delete-col` entirely
- Removes box-shadow from `.receipt`
- Strips borders and backgrounds from all editable inputs so they look like plain text on paper

---

### HTML Structure

```
body
 ├── .print-btn-wrap          ← "PRINT / SAVE AS PDF" button (hidden on print)
 ├── .controls                ← "+ Add Item" button (hidden on print)
 └── .receipt
      ├── .ce-field           ← Merchant name (contenteditable, wrapping)
      ├── .fi inputs ×5       ← Address / GSTIN / Mobile (centered inputs)
      ├── divider
      ├── Name field          ← "Name:" label + .fi input
      ├── divider
      ├── Meta rows           ← Date, Order Type, Time, Bill No., Cashier (.fi inputs)
      ├── divider
      ├── <table>
      │    ├── <thead>        ← Static column headers: Item / Qty / Price / Amt
      │    └── <tbody id="itemsTbody">   ← Rows injected by renderTable()
      ├── divider
      ├── Sub Total row       ← #totalQty, #subTotal (updated by recalc)
      ├── divider
      ├── CGST row            ← #cgstName input, #cgstPct input, #cgstAmt span
      ├── SGST row            ← #sgstName input, #sgstPct input, #sgstAmt span
      ├── divider
      ├── Grand Total row     ← #grandTotal span (updated by recalc)
      ├── divider
      └── "Thanks"
```

---

### JavaScript

Three functions handle all interactivity.

#### `renderTable()`
Rebuilds the entire `<tbody>` from the `items` array. Called on page load, after adding an item, and after deleting an item.

Each row contains:
- A `contenteditable` `<span class="tbl-ce">` for the item name — `oninput` writes back to `items[i].name`
- A `<input type="number">` for qty — `oninput` calls `recalc()`
- A `<input type="number">` for price — `oninput` calls `recalc()`
- A plain `<td class="amount-cell">` for the calculated amount (read-only, updated by `recalc`)
- A `<button class="del-btn">` that calls `deleteItem(i)`

Item indices are embedded as literals in the `oninput` / `onclick` strings at render time, so they are always accurate for the current array state.

```js
const items = [
  { name: 'Rasam Papad', qty: 1, price: 100.00 },
  ...
];

function renderTable() {
  // clears tbody, loops items[], writes one <tr> per item
}
```

#### `recalc()`
Reads the current DOM values of every qty/price input, computes all derived values, and writes them back to the DOM. Does **not** re-render the table, so the user's cursor position is preserved while typing.

```
subtotal  = Σ (qty × price)  for each row
cgst      = subtotal × (cgstPct / 100)
sgst      = subtotal × (sgstPct / 100)
grandTotal = subtotal + cgst + sgst
```

Tax percentages are read live from `#cgstPct` and `#sgstPct` inputs, so changing a tax rate instantly updates the grand total.

#### `deleteItem(i)`
Removes the item at index `i` from the array using `splice`, then calls `renderTable()` to redraw. Because the table is fully re-rendered, all indices in `oninput`/`onclick` handlers are reassigned correctly.

#### `addItem()`
Pushes a blank `{ name: 'New Item', qty: 1, price: 0.00 }` object onto the array, calls `renderTable()`, then focuses the name field of the new last row.

#### `esc(str)`
HTML-escapes special characters (`&`, `"`, `'`, `<`, `>`) before injecting item names into `innerHTML` strings, preventing XSS and broken markup when names contain quotes or ampersands.

---

## Browser Compatibility

Works in all modern browsers. Tested in:
- Chrome 120+
- Microsoft Edge 120+
- Firefox 121+

**Recommended for PDF generation: Chrome or Edge** — they produce the cleanest output via the built-in Save as PDF printer.

---

## License

Free to use and modify for personal or commercial billing purposes.
