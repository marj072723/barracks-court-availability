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
Customer books ──> Booking form (Apps Script /exec URL) ──> Google Sheet "Bookings"
Customer checks ─> This dashboard (Vercel) ──fetches──> same /exec URL ?api=availability
```

The `?api=availability` JSON endpoint was added to `Code.gs` in the
`court-booking-form` project. It exposes **only** dates, time slots, and
open/booked status — never customer names, emails, or phone numbers.

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

- `DAYS_TO_SHOW` — days ahead to display (max 30, matches the API cap)
- `REFRESH_SECONDS` — auto-refresh interval
- Slot times and prices come from the API automatically — change them once in
  `Code.gs` and the dashboard follows.
