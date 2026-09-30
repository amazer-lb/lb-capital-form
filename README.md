# Partner Brand Asset Order Form

A single-page web form that LB Capital partner roofing companies use to order branded apparel, collateral and promo items. Each submission is emailed to Kerry at LB Capital. There is no database or spreadsheet: **the email is the only record of an order.**

Originally built by an intern (jpatrick@lbcap.com, no longer with the company). Ownership was moved to Andrew Mazer (amazer@lbcap.com) on 2026-09-30.

---

## How it works

```
Partner fills in form (this repo's HTML page)
        │  GET request (JSONP): ?callback=…&data={rows:[…]}
        ▼
Google Apps Script web app  "Order form"
        │  handleOrder() builds a plain-text email
        ▼
GmailApp.sendEmail → kerry@lbcap.com
        │  subject: "New LB Capital Order — <Company>"
        ▼
Script replies { status: "success" } → form shows "Order submitted"
```

1. The partner picks a phase tab and enters quantities. Only the **active** tab is submitted; anything typed on other tabs is ignored.
2. The page builds one row per item that has a quantity > 0:
   `[timestamp, phase, company, branch, requestedBy, shipTo, orderDate, orderNum, SKU, itemLabel, qty]`
   Apparel sizes are folded into the label, e.g. `PM hoodies - Size Women's M`.
3. The rows are sent as JSON in the URL of a `<script>` tag (JSONP) to the Apps Script web app.
4. The script emails the order, then calls back with `status: success` or `status: error` + message.
5. The page shows success only on `status: success`. On an error, a network failure or a 15-second timeout, it shows an error message.

## Where things live

| Piece | Location | Owner |
|---|---|---|
| Form (HTML/CSS/JS, one file) | This GitHub repo | Andrew (transfer pending / done — update when confirmed) |
| Backend | Apps Script project **"Order form"** — `script.google.com/d/13imWrWymc1VaDQRD053T1-WHMjdGq63rsbwT08787tFUG9Y-2A18qbov/edit` | amazer@lbcap.com (transferred from jpatrick@lbcap.com) |
| Live web app deployment | Deployment ID `AKfycby-Pd8Hb6xIq3LfYaVCY8o_ZJiLeNH5dNr-_L88IUUpxHLwqrfWIGWpCQI-On29AbEJ` | Must be deployed from Andrew's account (see below) |
| Order recipient | kerry@lbcap.com (hard-coded in the script) | — |
| Order records | Kerry's inbox, plus the sender's Gmail **Sent** folder | — |

The form calls the backend here, near the top of the `<script>` block:

```js
const APPS_SCRIPT_URL = 'https://script.google.com/macros/s/AKfycby-…AbEJ/exec';
```

If the script is ever redeployed as a **new** deployment, that URL changes and this line must be updated.

## Apps Script settings

From the project's `appsscript.json`:

- **Execute as:** User deploying. Emails are sent from the Gmail of whoever deployed the web app.
- **Who has access:** Anyone, anonymous. Partners submit without signing in, and anyone with the URL can call it.
- **Runtime:** V8, time zone America/New_York.

Functions in `Code.gs`:

- `doGet(e)` — entry point the form uses (JSONP). Wraps `handleOrder` and returns `callback({...})`.
- `doPost(e)` — same logic for a JSON POST body. Not used by the current form.
- `handleOrder(raw)` — parses `{rows}`, validates that rows exist, builds the email and sends it. Reads fields by position (`row[8]` SKU, `row[9]` label, `row[10]` qty), so **the column order in the form's `collectRows()` must not change** without updating the script.
- `testEmail()` / `testHandleOrder()` — manual tests to run from the editor.

### Keeping the deployment tied to the right account

The live deployment sends mail as the account that deployed it. After taking ownership:

1. **Deploy → Manage deployments** → select the deployment above → **Edit (pencil)**.
2. **Version: New version → Deploy**, and approve the Gmail permission as yourself.
3. This keeps the same URL, so no HTML change is needed.

Verify by submitting a test order and checking that Kerry's email is **from** the current owner.

## Editing the form

- **Sizes:** edit the `SIZES` array in the script block. All apparel dropdowns are filled from it. Men's/Unisex values are sent plain (`M`); Women's are sent as `Women's M`.
- **Items:** each item is either a `.qty` input (plain count) or an `.apparel-item` block (size + qty lines). The SKU and name are read from the `data-sku` and `data-item` attributes.
- **Phases:** three tabs, labelled by `PHASE_LABELS`; sections are `#phase-0`, `#phase-1` and `#phase-2`.
- The email text is built by `handleOrder` in the Apps Script project, not in this repo.

## Known limitations

- **No order log** beyond email. The Order number field is collected but never included in the email.
- A quantity entered with no size selected is submitted without a size.
- Women's sizes share the men's SKU; fulfillment has to read the label.
- The whole order travels in the request URL, so very large orders could hit URL length limits.
- The endpoint is public and anonymous, so spam submissions are possible.

## Change log

- **2026-09-30** — Added Women's sizes (XS–3XL) to all apparel dropdowns; the size list is now defined once in the `SIZES` array. The Apps Script project was transferred from jpatrick@lbcap.com to amazer@lbcap.com. No backend code changes.
