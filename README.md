# The Barracks Court — Live Availability Dashboard

A public, auto-refreshing calendar showing which hourly slots are **Open** or
**Booked** for the next 14 days. Customers check it before booking.

- **No backend of its own** — it reads live data from the same Google Apps Script
  that powers the booking form (`court-booking-form`), which stores bookings in
  your Google Sheet.
- **One static file** (`index.html`), so it runs free forever on Vercel's Hobby
  plan (or GitHub Pages, or Netlify).
- Auto-refreshes every 60 seconds; day-by-day view on mobile, full 14-day grid
  on demand; prices shown per slot; past hours today are grayed out (Manila time).

## How it fits together

```
Customer checks ─> This dashboard (Vercel) ──GET──> Apps Script /exec ?api=availability
Customer books ──> selects slots, "Book my slot" popup ──POST──> same /exec (doPost)
                   └─> Apps Script re-checks availability, saves the GCash screenshot
                       to Drive, appends Bookings rows, emails the receipt
```

Both endpoints live in `Code.gs` in the `court-booking-form` project. The
availability API exposes **only** dates, time slots, and open/booked status —
never customer names, emails, or phone numbers. The classic Apps Script form
still works at the same URL as a fallback.

## Changing the admin email

Where **new-booking notifications** go (not customer receipts — those always go
to the address the customer typed into the booking form).

1. Spreadsheet → **Barracks Court → Change the admin email…**
2. Type the new address. It takes effect immediately — no redeploy.
3. **Barracks Court → Send a test email** to confirm it actually arrives.

The address is stored in Script Properties, so it overrides the `ADMIN_EMAIL`
constant in `Code.gs`. That constant is now only the fallback for when no
property has ever been set — editing it has no effect once you've used the menu
once.

## Closing individual dates

For a holiday, maintenance day or private event — the rest of the calendar
stays open and bookable.

1. Spreadsheet → **Barracks Court → Close a specific date…**
2. Type the date as `yyyy-mm-dd` (e.g. `2026-07-27`).
3. Optionally add a note shown to customers ("Closed for court resurfacing").

That day's 24 slots all show as **Closed** in red, the column header is tagged,
and the note appears under the grid. Bookings for that day are rejected server
side too, so a stale tab can't slip one through.

To undo: **Barracks Court → Reopen a specific date…**

Closed days are listed on the **Closed Dates** tab, one per row — you can also
add/remove rows there by hand instead of using the menu. The tab is created
automatically the first time you close a date.

> **Closing a date does not cancel existing bookings.** Rows already on the
> Bookings sheet stay there and still count toward the Summary. The menu warns
> you how many there are; contact those customers yourself, then set their rows
> to `Cancelled`.

## Turning reservations off temporarily

No code change and no redeploy needed — it's a switch in the Google Sheet:

1. Open the booking spreadsheet → menu **Barracks Court → Turn reservations OFF…**
2. Type a custom notice, or leave it blank for the default
   ("Reservations for The Barracks Court are temporarily unavailable until
   further notice.") → **OK**.
3. Within a minute the dashboard hides the calendar and shows the notice
   instead. Pages already open on someone's phone switch over by themselves.

To reopen: **Barracks Court → Turn reservations back ON**. Not sure which state
it's in? **Barracks Court → Reservations: are they on or off?**

While it's off, the Apps Script also *rejects* booking submissions, so a stale
page that someone left open before the switch was flipped can't sneak one
through. Existing bookings on the sheet are untouched either way.

## Setup (~10 minutes)

### 1. Redeploy the booking form (one time)

The updated `Code.gs` (with the JSON API) must be live first:

1. Open your booking spreadsheet → **Extensions → Apps Script**.
2. Replace `Code.gs` with the updated version from `court-booking-form/Code.gs`.
3. **Deploy → Manage deployments → ✏️ → Version: New version → Deploy.**
   (The public `/exec` URL stays the same.)
4. Test it: open `YOUR_EXEC_URL?api=availability&days=3` in a browser —
   you should see JSON, not the form.

### 2. Point the dashboard at your form

Open `index.html`, scroll to the CONFIG block near the bottom, and paste your
`/exec` URL:

```js
var BOOKING_APP_URL = "https://script.google.com/macros/s/XXXXX/exec";
```

(That single URL powers both the "Book a slot" button and the live data.)

### 3. Push to GitHub (free account is fine)

```bash
cd barracks-court-availability
git init
git add .
git commit -m "Barracks Court availability dashboard"
gh repo create barracks-court-availability --public --source=. --push
# — or create the repo on github.com and:
# git remote add origin https://github.com/YOUR_USERNAME/barracks-court-availability.git
# git push -u origin main
```

### 4. Deploy on Vercel (free Hobby plan)

1. Go to [vercel.com](https://vercel.com) → sign up / log in **with GitHub**.
2. **Add New → Project** → Import `barracks-court-availability`.
3. Framework preset: **Other**. No build command, no output directory — defaults
   are fine for a static site. → **Deploy**.
4. You get a URL like `barracks-court-availability.vercel.app`. Share that with
   customers (and paste it into `slotsSheetUrl` in `Code.gs` if you want the
   booking form's "View All Available Slots" button to link here).

Every future `git push` redeploys automatically.

## Free-tier limits (you won't hit them)

| Service | Free tier | This project uses |
|---|---|---|
| GitHub | Unlimited public/private repos | 1 tiny repo |
| Vercel Hobby | 100 GB bandwidth/mo, unlimited static requests | a ~15 KB page |
| Apps Script | 20k+ URL fetches/day quotas | 1 request/min per open viewer |

## Customizing

Everything is in the CONFIG block at the bottom of `index.html`:

- `VISIBLE_DAYS` — columns shown at once (default 7)
- `FETCH_DAYS` — how far ahead customers can navigate (max 30, matches the API cap)
- `REFRESH_SECONDS` — auto-refresh interval
- Slot times and prices come from the API automatically — change them once in
  `Code.gs` and the dashboard follows.
