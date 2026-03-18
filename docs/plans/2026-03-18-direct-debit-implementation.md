# Direct Debit Support Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Add Direct Debit mode (MTS `DD`, tx code `00`) to `nz_westpac_generator.html` via a top-level DC/DD mode switcher.

**Architecture:** Single state variable `currentMode` drives MTS source, tx code, and all label changes. A new `setMode()` function updates the UI reactively. No new files — all changes in `nz_westpac_generator.html`.

**Tech Stack:** Vanilla HTML + CSS + JavaScript. No build step. Verify by opening the file in a browser.

**Branch:** `DDFile` (already created, already checked out)

**Key file:** `c:\WorkSpace\BankFileGenerator\nz_westpac_generator.html`

---

### Task 1: Add `currentMode` state variable and `setMode()` function

**Files:**
- Modify: `nz_westpac_generator.html` — JavaScript state block (near top of `<script>`)

**Step 1: Add state variable**

Find the existing state block (currently ~line 610):
```javascript
let transactions   = [];
let generatedContent = '';
let currentTxCode  = '52';
```

Add one line after `currentTxCode`:
```javascript
let currentMode = 'DC';  // 'DC' or 'DD'
```

**Step 2: Add `setMode()` function**

Add this function directly after `setTxCode()`:
```javascript
function setMode(mode, btn) {
  currentMode = mode;

  // Update mode toggle active state
  document.querySelectorAll('#modeTabs .tab-btn').forEach(b => b.classList.remove('active'));
  if (btn) btn.classList.add('active');

  const isDd = mode === 'DD';

  // Show/hide Payment Type sub-toggle row
  document.getElementById('paymentTypeRow').style.display = isDd ? 'none' : '';

  // Update header badge
  document.getElementById('modeBadge').textContent = isDd ? 'DIRECT DEBITS' : 'DIRECT CREDITS';

  // Update Step 2 card title
  document.getElementById('step2Title').textContent = isDd ? 'Default Creditor Account' : 'Default Payer Account';

  // Update Payer Name label
  document.getElementById('payerNameLabel').textContent = isDd ? 'Creditor Name' : 'Payer Name';

  // Update transaction table column header
  document.getElementById('thPayer').textContent = isDd ? 'Creditor' : 'Payer';

  // Lock tx code to 00 for DD
  if (isDd) currentTxCode = '00';
  else currentTxCode = '52'; // restore DC default

  refreshPreview();
}
```

**Step 3: Verify — open browser console, run:**
```javascript
setMode('DD'); // should not throw
setMode('DC'); // should not throw
```

**Step 4: Commit**
```bash
git add nz_westpac_generator.html
git commit -m "feat: add currentMode state and setMode() function"
```

---

### Task 2: Add DC / DD mode toggle HTML above the Payment Type row

**Files:**
- Modify: `nz_westpac_generator.html` — Step 1 form body

**Step 1: Add `id="modeTabs"` toggle before the Payment Type row**

Find the Step 1 body opening (currently ~line 435):
```html
      <div class="step-body">

        <div class="form-row">
          <label>Payment Type</label>
          <div class="tab-group" id="txCodeTabs">
```

Replace with:
```html
      <div class="step-body">

        <div class="form-row">
          <label>Transaction Mode</label>
          <div class="tab-group" id="modeTabs">
            <button class="tab-btn active" onclick="setMode('DC', this)">Direct Credits</button>
            <button class="tab-btn" onclick="setMode('DD', this)">Direct Debits</button>
          </div>
        </div>

        <div class="form-row" id="paymentTypeRow">
          <label>Payment Type</label>
          <div class="tab-group" id="txCodeTabs">
```

**Step 2: Close the new `paymentTypeRow` div**

Find the closing of the Payment Type tab group (currently ~line 443):
```html
          </div>
        </div>

        <div class="form-row">
          <label for="processingDate">Processing Date</label>
```

Replace with:
```html
          </div>
        </div>
        </div><!-- /paymentTypeRow -->

        <div class="form-row">
          <label for="processingDate">Processing Date</label>
```

**Step 3: Add `id` attributes to elements that `setMode()` will update**

a) Header badge — find:
```html
    <span class="badge badge-amber">DIRECT CREDITS</span>
```
Replace with:
```html
    <span class="badge badge-amber" id="modeBadge">DIRECT CREDITS</span>
```

b) Step 2 card title — find:
```html
        <div class="step-title">Default Payer Account</div>
```
Replace with:
```html
        <div class="step-title" id="step2Title">Default Payer Account</div>
```

c) Payer Name label — find:
```html
          <label for="payerName">Payer Name</label>
```
Replace with:
```html
          <label for="payerName" id="payerNameLabel">Payer Name</label>
```

d) Transaction table Payer column header — find:
```html
                <th>Payer</th>
```
Replace with:
```html
                <th id="thPayer">Payer</th>
```

**Step 4: Browser verify**
1. Open `nz_westpac_generator.html` in browser
2. Click **Direct Debits** → Payment Type sub-toggle disappears, badge shows "DIRECT DEBITS", Step 2 title shows "Default Creditor Account", Payer Name label shows "Creditor Name", table header shows "Creditor"
3. Click **Direct Credits** → everything reverts

**Step 5: Commit**
```bash
git add nz_westpac_generator.html
git commit -m "feat: add DC/DD mode toggle UI with reactive label updates"
```

---

### Task 3: Update `buildDetailRecord` to use dynamic MTS and tx code

**Files:**
- Modify: `nz_westpac_generator.html` — `buildDetailRecord()` function

**Step 1: Find the hardcoded MTS line in `buildDetailRecord` (~line 658):**
```javascript
  const txCode = tx.tx_code || currentTxCode;
  const mts    = 'DC';
```

**Step 2: Replace with dynamic values:**
```javascript
  const txCode = currentMode === 'DD' ? '00' : (tx.tx_code || currentTxCode);
  const mts    = currentMode === 'DD' ? 'DD' : 'DC';
```

**Step 3: Browser verify**
1. Fill in Steps 1–2 with any values, import a CSV, click **Direct Credits** → Generate & Preview → raw line positions 26–29 should show `52DC` (or `50DC`)
2. Switch to **Direct Debits** → Generate & Preview → raw line positions 26–29 should show `00DD`
3. Count characters: position 26 is index 25 in 0-based. Verify `line.substring(25,29)` in browser console on a generated D record line.

**Step 4: Commit**
```bash
git add nz_westpac_generator.html
git commit -m "feat: write DD tx code 00 and MTS DD in detail records"
```

---

### Task 4: End-to-end verification and push

**Step 1: Full browser test — Direct Credits (existing behaviour)**
1. Open file, ensure **Direct Credits** is active
2. Fill Step 1: date=today, description="April Payroll", branch=0123
3. Fill Step 2: name=ACME LTD, bank=03, branch=0123, account=00123456, suffix=0000
4. Import the template CSV (download it first and add a second row)
5. Generate & Preview → confirm A record + D records render, MTS shows `DC`, tx code `52`
6. Download → open in text editor, confirm each line is exactly 180 chars + CRLF

**Step 2: Full browser test — Direct Debits**
1. Switch to **Direct Debits**
2. Confirm sub-toggle hidden, badge = "DIRECT DEBITS", Step 2 = "Default Creditor Account"
3. Fill same fields, import same CSV
4. Generate & Preview → D records show `00` / `DD` in Tx Code / MTS fields
5. Download → open in text editor, confirm positions 26–29 on each D line = `00DD`
6. Switch back to Direct Credits → generate again → confirms `52DC` restored

**Step 3: Push branch**
```bash
git push -u origin DDFile
```

---

## Summary of all changes to `nz_westpac_generator.html`

| What | Where | Change |
|---|---|---|
| State variable | JS state block | Add `let currentMode = 'DC'` |
| `setMode()` function | JS, after `setTxCode()` | New function |
| Mode toggle HTML | Step 1 body, top | New `#modeTabs` tab-group |
| Payment Type wrapper | Step 1 body | Wrap in `#paymentTypeRow` div |
| Header badge | `<header>` | Add `id="modeBadge"` |
| Step 2 title | Step 2 card | Add `id="step2Title"` |
| Payer Name label | Step 2 body | Add `id="payerNameLabel"` |
| Table Payer header | Transaction table | Add `id="thPayer"` |
| `buildDetailRecord` | JS | Dynamic `txCode` and `mts` |
