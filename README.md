# 💻 Laptop Register — GitHub Pages + Google Sheet

A single static HTML page where learners enter **Name, Surname, Laptop name, Serial number**. Submissions are appended to a Google Sheet via a tiny Google Apps Script.

No server. No Python. 100% free.

---

## What you'll end up with
- A public URL like `https://<your-username>.github.io/laptop-register/`
- A Google Sheet that fills up automatically, one row per submission.

---

## Step 1 — Create the Google Sheet

1. Go to https://sheets.new (creates a new sheet).
2. Rename it e.g. **Laptop Register**.
3. In row 1, add these headers exactly (columns A–E):

   | Timestamp | Name | Surname | Laptop | Serial |
   |---|---|---|---|---|

---

## Step 2 — Add the Apps Script (the "backend")

1. In the sheet, click **Extensions → Apps Script**.
2. Delete any boilerplate and paste the code below.
3. Click 💾 **Save**, name the project "Laptop Register".

```javascript
function doPost(e) {
  try {
    const data = JSON.parse(e.postData.contents);
    const sheet = SpreadsheetApp.getActiveSpreadsheet().getSheets()[0];
    sheet.appendRow([
      new Date(),
      data.name || "",
      data.surname || "",
      data.laptop || "",
      data.serial || "",
    ]);
    return ContentService
      .createTextOutput(JSON.stringify({ ok: true }))
      .setMimeType(ContentService.MimeType.JSON);
  } catch (err) {
    return ContentService
      .createTextOutput(JSON.stringify({ ok: false, error: String(err) }))
      .setMimeType(ContentService.MimeType.JSON);
  }
}
```

---

## Step 3 — Deploy it as a Web App

1. Top-right: **Deploy → New deployment**.
2. Gear icon → choose **Web app**.
3. Settings:
   - **Description**: Laptop Register
   - **Execute as**: *Me*
   - **Who has access**: **Anyone** ← important
4. Click **Deploy**. Google will ask you to authorize — click **Authorize access**, pick your account, then **Advanced → Go to project (unsafe) → Allow**. (This is only because the script is yours, not verified publicly.)
5. Copy the **Web app URL** shown at the end. It looks like:
   `https://script.google.com/macros/s/AKfycby..../exec`

---

## Step 4 — Paste the URL into `index.html`

Open `index.html`, find this line near the bottom:

```js
const SCRIPT_URL = "PASTE_YOUR_APPS_SCRIPT_WEB_APP_URL_HERE";
```

Replace it with the URL you just copied. Save.

---

## Step 5 — Push to GitHub and enable Pages

1. Create a new repo on GitHub, e.g. `laptop-register`.
2. Upload `index.html` (drag-and-drop on the repo page is fine).
3. Go to **Settings → Pages**.
4. Under **Build and deployment → Source**, pick **Deploy from a branch**.
5. **Branch**: `main`, folder: `/ (root)`. Click **Save**.
6. Wait ~1 minute. Your site will be live at:
   `https://<your-username>.github.io/laptop-register/`

Share that link with your learners. ✅

---

## Testing

1. Open the published link.
2. Fill the form, click Submit — you should see ✅ "Recorded".
3. Open the Google Sheet — a new row appears.

If a submission fails:
- Make sure you deployed the Apps Script with **Who has access: Anyone**.
- Make sure the `SCRIPT_URL` in `index.html` is the **/exec** URL (not `/dev`).
- Re-deploy after any Apps Script change: **Deploy → Manage deployments → ✏️ → New version → Deploy**.

---

## Notes

- The form uses `mode: "no-cors"` so the browser won't show the Apps Script response, but the row is still written. That's why we rely on the server to be reachable and trust success — if you need stronger confirmation, switch to `mode: "cors"` and add `setHeader` handling via a `doOptions` function (advanced).
- To prevent duplicates, you can add a uniqueness rule in the Sheet on the Serial column, or extend the Apps Script to reject duplicates.
