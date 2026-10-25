# Israel Post item-trace API

Captured from `https://doar.israelpost.co.il/en/deliverytracking` in Chrome 155 on 2026-10-25, with one real in-transit item and one invalid code. Values below are illustrative, not from the real item.

## Endpoint

```
GET https://apimftprd.israelpost.co.il/MyPost-itemtrace/items/{code}/{lang}
lang: eng | heb
```

- Azure API Management. Without the page's subscription key: `401 {"statusCode":401,"message":"Access denied due to missing subscription key..."}` (tested with curl, browser headers, Origin and Referer set). Call it only through the page.
- Unknown code: `404`, empty body.
- Case-insensitive code; the response echoes `ItemCode` upper-cased.

Other calls the page makes (same host): `mypost-content/serviceAvailability?key=ItemTraceNew` (feature switch for the tracking page), `mypost-contentcore/DeliveryTrackingPage?culture=en-US` (page copy), `mypost-ad/IsAuth`.

## Response (200)

| Field | Type | Observed / meaning |
|---|---|---|
| `ItemCode` | string | `LA123456789GB` |
| `SugSherut` | string | Service type, e.g. `Express from  Abroad` (double space as sent) |
| `SenderName` | string | e.g. `Overseas mail` |
| `ReceiverName`, `ReceiverAddress` | string | Personal data. Hebrew address with מיקוד |
| `CategoryCode` / `CategoryName` | int / Hebrew string | Current state; codes below |
| `CategoryIcon`, `DeliveryTypeIcon` | URL | Ignore |
| `DeliveryTypeDesc` | string | Collecting branch + city, e.g. `מרכזי ירושלים ירושלים` |
| `Madaf` | string | Shelf/slot code at the branch, e.g. `ג 221` |
| `PickupDaysLeft` | int | `0` seen on an item awaiting collection 20 days after arrival |
| `DeliveryDate`, `DeliveryTime`, `DeliveredDate`, `DeliveryAddress` | nullable | Null while undelivered |
| `CustomsRequired` (int), `CustomsAmount` (number), `CustomsPaid` (bool), `CustomsPaymentLink`, `CustomsPaymentLinkText` | | Customs due |
| `HasImage`, `HasSignedImage`, `HasHazmana`, `MobileTrace`, `MarkedDoNotDisplay` | bool | Not investigated |
| `ItemActions` | | Not investigated |
| `Status`, `StatusCode`, `StatusForDisplay`, `ItemType`, `ItemName` | | Null / 0 / empty on the sample |
| `DateLastStatusSort`, `DateLastStatusSortOriginal`, `CategoryGroupIndex` | | Sorting helpers |
| `Maslul` | array | Event history, newest first |

`Maslul[]` items: `StatusDate` (`DD/MM/YYYY`), `CategoryCode`, `CategoryName`, `BranchName` (Hebrew), `City` (English with `/eng`), `Status` (English sentence with `/eng`), `CategoryIcon`. Some rows are blank (`BranchName` whitespace, `Status` `"   ,   - ."`).

## Category codes seen

| Code | `CategoryName` | English |
|---|---|---|
| 9 | מיועד להישלח אליך | Due to be sent to you |
| 1 | הוזמן | Ordered / shipment notice received |
| 45 | המשלוח הגיע הרגע לישראל | Just arrived in Israel |
| 3 | בתהליך מיון | Sorting |
| 4 | ממתין לאיסוף שלך | Waiting for you to collect |

Expected but not yet seen: delivered, returned to sender, held at customs. Add rows when captured.
