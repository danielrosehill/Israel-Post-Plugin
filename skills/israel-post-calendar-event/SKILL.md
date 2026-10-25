---
name: israel-post-calendar-event
description: Add a booked Israel Post appointment to the user's calendar, with the branch address and the parcels, shelf numbers and customs to collect.
---

# Israel Post appointment → calendar event

Run after `israel-post-appointment` books a slot, when the user wants it in their calendar. Offer it once after every successful booking ("Add this to your calendar?"); don't create an event unasked.

## Inputs

From the booking confirmation (`israel-post-appointment`):
- Branch name and `branchnumber`, service (Hebrew label), date (`DD-MM-YYYY`), time (`HH:MM`).

From the branch dataset (`skills/israel-post-appointment/data/branches.json`, match on `branchnumber`):
- `branchname`, `street`, `house`, `city`, `telephone`, `geocode_latitude`, `geocode_longitude`.

The items to collect, for package-pickup (`מסירת דואר ללקוח`) appointments:
- Tracking codes the user mentioned in the conversation, or ask "Which items are you collecting? (tracking codes, or skip)".
- For each code, run `israel-post-tracking` and take `ItemCode`, `SugSherut`, `Madaf` (shelf), `DeliveryTypeDesc` (collecting branch), customs fields, `PickupDaysLeft`.
- If an item's collecting branch differs from the booked branch, tell the user before creating the event. They probably booked the wrong branch.

## Event

| Field | Value |
|---|---|
| Title | `Israel Post: <service in English> (<branchname>)`, e.g. `Israel Post: parcel pickup (מרכזי ירושלים)` |
| Start | booking date + time, timezone `Asia/Jerusalem` (local wall-clock time, no fixed offset) |
| End | start + 30 min |
| Location | `דואר ישראל <branchname>, <street> <house>, <city>` (maps resolves the Hebrew address) |
| Reminders | the calendar's defaults |
| Guests | none, unless the user names someone (e.g. the person collecting) |

Description (plain text, one item per line):

```
Israel Post appointment: <service Hebrew label> (<service English>), <HH:MM>.
At the branch, type your mobile number (<phone, masked: 05X-XXX-XX67>) at the queue terminal. The slot is held for 10 minutes.
Branch phone: <telephone>

To collect:
- <ItemCode> (<SugSherut>), shelf <Madaf><; customs due ₪<CustomsAmount>: <CustomsPaymentLink> if unpaid>
- ...

Bring ID. Collecting someone else's item: bring their ID or a signed authorisation (ייפוי כוח) form.
```

- Drop the "To collect" block for services other than package pickup, or when the user skips it.
- Flag `PickupDaysLeft` of `0` or `1` on its item line ("pickup window may be ending").
- Mask the phone in the description; calendars get shared.

## Creating it

Use whatever calendar the session can write to:

1. **A calendar tool is available** (Google Calendar, Microsoft 365/Outlook, CalDAV MCP): create the event on the user's primary calendar with notifications to guests off unless the user named guests. Report the event link.
2. **No calendar tool:** write an `.ics` file (one `VEVENT`, `DTSTART;TZID=Asia/Jerusalem:<YYYYMMDDTHHMMSS>`, `DTEND` +30 min, `SUMMARY`, `LOCATION`, `DESCRIPTION` with `\n` line breaks, a `VTIMEZONE` for Asia/Jerusalem or a UTC time computed for that date) to the user's working directory, named `israel-post-<YYYY-MM-DD>-<HHMM>.ics`, and give the path so they can open it.

Confirm the title, time and location in one line before writing, unless the user already asked for the event in so many words ("put it in my calendar").

## After

Tell the user the event's time, location and link or file path. If they cancel the appointment later, offer to delete the event too.
