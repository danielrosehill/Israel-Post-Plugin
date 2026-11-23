---
name: israel-post-registered-letter
description: Send one or more letters by Israel Post registered mail (דואר רשום) without leaving home, through the "מכתב דיגיטלי" web service - upload a PDF, Israel Post prints and posts it and returns a tracking number. Covers checking each recipient's address against the official company/partnership registry, filling sender and per-letter recipient details, cover note, PDF attachment (done by the user), payment (done by the user), and recording tracking numbers. Trigger phrases - "send this as registered mail", "doar rashum online", "send a registered letter to X", "mail these letters registered".
---

# Israel Post registered letter (מכתב דיגיטלי)

Israel Post prints your PDF in black and white and posts it as registered mail. You never
handle an envelope. One order can hold several letters to different recipients, each with
its own tracking number.

**Use this, not דואר בקליק.** דואר בקליק is prepaid postage for an envelope you still
take to a branch yourself, or have collected by courier for 30 ₪ plus 2.50 ₪ per extra
item. Users asking to "send registered mail online" almost always mean מכתב דיגיטלי.

Verified end to end in Claude in Chrome on 2026-11-23: two letters, registered, paid,
tracking numbers returned.

## Inputs

- One PDF per letter. Limits from the site: PDF only, ≤ 1.5 MB, ≤ 10 pages per file,
  ≤ 15 pages across all files on one letter. Printed black on white.
- Per letter: recipient (person or company), postal address, shipping type, short cover note.
- Sender: from `~/.config/israel-post/preferences.yaml` → `israel_post.sender`
  (first/last name, phone, address with apartment). If it is missing, ask once and offer to save it.

Write the batch as a JSON file before starting. It is both the run sheet and the record.
Format: `reference/registered-letter-batch.example.json`. Keep government ID numbers out of
it. If an ID must appear, it goes in the PDF only.

## 1. Check every address against the registry first

A registered letter to a company should go to its **registered office**, not a marketing
address. Look the entity up in the Israeli registrar's open data on data.gov.il (CKAN,
no key needed):

| Entity type | `resource_id` |
|---|---|
| Companies (בע"מ, ח.פ. 51…) | `f004176c-b85f-4542-8901-7b3176f9a054` |
| Partnerships (שותפות מוגבלת, 55…) | `139aa193-fabb-4f6b-a71b-0bb40fd73eb2` |

```bash
curl -s -G https://data.gov.il/api/3/action/datastore_search \
  --data-urlencode resource_id=<id> --data-urlencode q=<name-or-number>
```

Use the city, street, house, postcode and **"אצל"** (care-of) fields. Check the status is
`פעילה` (active). Put the care-of line in the letter's address block, and the registry
number on the letter if you have it. Business directories (Dun's, checkid) are mirrors:
use them to find the number, then confirm it in the registry. Later, compare the site's
postcode lookup with the registry postcode.

## 2. Price (as of 2026-11-23, per letter)

| Option | ₪ |
|---|---|
| דואר רשום (registered: tracking + delivery confirmation) | 18.50 |
| רשום מהיר (express registered) | 29.00 |
| דואר 24 | 10.50 |
| דואר 72 | 4.70 |
| דואר רגיל | 4.40 |

Default to דואר רשום for anything legal. Tell the user the total and get a yes before step 3.

## 3. Drive the form

URL: `https://israelpost.co.il/שירותים/מכתב-דיגיטלי/`. The user should already be signed
in to their Israel Post account (header shows "שלום <name>"). Signing in is theirs to do.

**Step "פרטי השולח" (sender)**
- `משתמש פרטי` (private user). First/last name are pre-filled from the account.
- `מספר טלפון נייד` is **required and must be a mobile**. Leave the account's mobile there.
  If the user wants a landline given, put it in `מספר טלפון`.
- Sender address: pick it from `בחר כתובת`, or use `להוספת כתובת לחץ כאן`. In the modal,
  city and street are **autocomplete**: type, then click the suggestion. Fill house and
  apartment, then click the 🔍 next to `מיקוד` to look up the postcode. Leave "save to my
  addresses" unticked unless the user asks. Then `המשך`.

**Step "פרטי המכתב", one block per letter (`מכתב 1`, `מכתב 2`, …)**
1. Click `בחירה` on the shipping card. It changes to `נבחר`, and the block header shows the price.
2. Recipient: `הוספת נמען חדש` opens a modal with the same autocomplete pattern. For a
   company, put the legal name in שם פרטי and the rest, e.g. `בעמ - לידי המנכל`, in שם משפחה.
   The form **drops quote marks** (בע"מ → בעמ) and **truncates שם משפחה at ~25 chars**, so
   abbreviate (שותפות מוגבלת → ש.מ.). Look up the postcode, compare it with the registry, then `שמור`.
   Saved recipients do not carry over between דואר בקליק and מכתב דיגיטלי.
3. `תוכן` (Quill editor): this is printed as a **cover note**. Keep it to four lines:
   addressee, subject (הנדון), "מצורף מכתבי מיום <date> (<n> עמודים)", sender name +
   address + phone. Don't paste the whole letter, or it prints twice. Enter it with
   real clicks and keystrokes. Setting the editor's content from JavaScript doesn't register.
4. Attachment: **the user does this.** They click `צירוף קובץ` and pick the PDF. The
   site's uploader ignores files set by automation: Claude in Chrome's `file_upload` fills
   `input#newfile<N>` but the app never registers the file (tested 2026-11-23, including
   firing change/input events). Before this step:
   - copy each PDF into one folder, **named by recipient**
     (`Letter 1 - <recipient>.pdf`) and also as a plain-ASCII copy (`letter1-<slug>.pdf`);
   - open the folder for them (`xdg-open <dir>`) and list which file goes on which `מכתב N`.
5. **Upload one file at a time.** Each file shows `קובץ בבדיקה` (being checked) for a while.
   A second upload sent while another is still being checked failed twice with
   `קובץ נכשל / חלה שגיאה בטעינת הקובץ`, and worked once the first had finished. Fix: delete
   (🗑) the failed file, wait for the other to finish, and attach again.
6. `הוספת מכתב` adds the next letter block.

Finally: the terms checkbox (accepting terms needs the user's explicit OK), continue, and
**the user pays**. Never enter card details.

## 4. Record the result

The confirmation page shows `שולם סכום של`, `מספר עסקה` (transaction) and `דבר דואר`: the
tracking numbers (`RR…H`), comma-separated, **in letter order**. Map them by order and
mark that as inferred until the tracking page confirms it. Save a screenshot of the page.
Update the batch JSON (`status`, `sent`, `tracking`, `payment`) and the project's
correspondence log. Delivery can be followed with `israel-post-tracking`. **Verified 2026-11-23:** the `RR…H` codes resolve on the tracking page within the hour, with status `הוזמן` (ordered) and one event ("Notification has been received regarding a parcel being shipped"), `DeliveryTypeDesc` `איסוף עצמי`, and an **empty `ReceiverAddress`**. So the code→letter mapping can't be confirmed from tracking until the item is printed and dispatched.
Israel Post support: 171, doartek@postil.com.

## Not yet verified

- Whether `חזקת מסירה` (deemed-delivery flag) changes anything for a private sender.
- What the printed cover note looks like (font, page order relative to the PDF).
