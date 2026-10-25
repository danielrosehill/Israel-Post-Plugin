---
name: israel-post-tracking
description: Track an Israel Post (דואר ישראל) parcel or registered item by tracking code — status, event history, pickup branch, shelf number, customs due, days left to collect.
---

# Israel Post tracking

Look up a tracking code (e.g. `RR123456789IL`, `LA123456789GB`, `CP…`, `UA…`) and report where the item is, what the user must do next, and the full event history.

API and field reference: `reference/itemtrace-api.md` (plugin root). Read it when parsing the JSON.

## Facts (confirmed in a real Chrome session, 2026-10-25)

- Tracking page: `https://doar.israelpost.co.il/en/deliverytracking?itemcode={code}` (Hebrew: `/deliverytracking`). The old `mypost.israelpost.co.il/itemtrace` redirects here. No login needed.
- The page calls `GET https://apimftprd.israelpost.co.il/MyPost-itemtrace/items/{code}/{eng|heb}`. That is Azure API Management: without the page's subscription key it returns `401 missing subscription key`. Do **not** extract that key and call the API from outside the page. Use the page itself.
- Not found: the API returns **404 with an empty body**; the page shows "No results found for this item".
- The page caches the last result per code in client state. Searching the same code again in the same tab sends no request. Change the case (`la123…` instead of `LA123…`) to force a fresh call; the API accepts either.
- Some event-history rows come back blank (`"   ,   - ."`). Drop them.

## Flow (Claude in Chrome)

1. Open the page with no code: `https://doar.israelpost.co.il/en/deliverytracking`.
2. Install a capture hook with `javascript_tool`:
   ```js
   window.__cap=[];const o=XMLHttpRequest.prototype.open,s=XMLHttpRequest.prototype.send;
   XMLHttpRequest.prototype.open=function(m,u){this.__u=u;return o.apply(this,arguments)};
   XMLHttpRequest.prototype.send=function(){this.addEventListener('load',()=>{if(/itemtrace\/items/i.test(this.__u))window.__cap.push({u:this.__u,status:this.status,body:this.responseText})});return s.apply(this,arguments)};
   ```
3. `find` the textbox "Locating packages by tracking number", type the code, press Return, wait ~3 s.
4. Read `window.__cap.at(-1)`: `status` 404 = not found; 200 = parse `body` as JSON.
5. Return only the fields you need from JS (see below). Claude in Chrome blocks tool output that looks like tokens or cookies, and the raw body has long icon URLs, so `JSON.stringify` of the whole object is often withheld.
6. Several codes: repeat step 3 in the same tab. The hook stays installed until the page navigates.
7. Close the tab when done.

**Fallback** if the hook captures nothing: `get_page_text` on `?itemcode={code}`. The page text has the same status, branch, shelf number and history, but not `PickupDaysLeft` or the customs fields.

## What to report

From the JSON (names in `reference/itemtrace-api.md`):

- **Current state:** `CategoryName` (Hebrew, e.g. `ממתין לאיסוף שלך` = waiting for you to collect) and the newest `Maslul[0].Status` (English when the call ends `/eng`).
- **Where to collect:** `DeliveryTypeDesc` (branch + city) and `Madaf` (shelf/slot code, e.g. `ג 221`, which the counter asks for).
- **Deadline:** `PickupDaysLeft`. When it is `0` and the item is still awaiting collection, say the pickup window may have run out and suggest calling the branch. That reading is inferred, not documented by Israel Post.
- **Customs:** `CustomsRequired`, `CustomsAmount`, `CustomsPaid`, `CustomsPaymentLink`. If an amount is due and unpaid, give the link.
- **History:** `Maslul` newest first, as `date | category | branch, city | status`, blank rows dropped.
- **Next actions:** the branch appointment skill `israel-post-appointment` books a package-pickup slot (`package_pickup`) at the collecting branch. Offer it when the item awaits collection.

`ReceiverName` and `ReceiverAddress` are personal data. Use them only to confirm the item is the user's when asked; don't write them to files or logs.

## Not yet verified

- Whether headless Playwright gets past the site's bot protection (Radware-style scripts load on the page). Only a real, signed-in-or-not Chrome profile has been tested.
- `HasHazmana`, `ItemActions`, `HasSignedImage` meanings beyond their names.
