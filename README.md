# Jaipur Departure Explorer

A static, GitHub Pages-friendly dashboard to compare train options from Jaipur across multiple destinations and dates. The project includes an explicitly labelled demo mode and an optional Cloudflare Worker API proxy for RailCore.

## What's included

- 30 editable destination/railhead options (station code + city-level matching)
- Date filters (default: 25, 26 and 27 November 2026)
- Sleeper (SL) and AC 3 Tier (3A), General quota
- Destination comparison chart, sortable/searchable scorecard and CSV export
- Train detail modal with on-demand fresh class availability checks
- API key kept on the Worker, never in the GitHub Pages frontend

## 1. Preview without an API key

Open `index.html` in a browser, or publish the repository with GitHub Pages. The dashboard will run in **DEMO DATA** mode. All sample counts and train records are illustrative, not live railway availability.

## 2. Connect live route data

This project uses RailCore's independent Indian Railways API. RailCore is not IRCTC and is not affiliated with Indian Railways. Review its docs, terms and pricing before using it:

- https://railcore.tech/docs/api/route-trains
- https://railcore.tech/docs/api/seat-availability
- https://railcore.tech/docs/quickstart
- https://railcore.tech/pricing

Create a RailCore API key with the `journeys:read` and `availability:read` scopes. The Worker expects a Free or Starter key (these plans use key-only authentication). Higher plans may require HMAC signing and would need additional Worker code.

### Deploy the Worker

1. Create a Cloudflare account and install Wrangler locally (`npm install -g wrangler`).
2. From the project root, run `wrangler init railscout-api` or create a Worker named `railscout-api`.
3. Copy `api/worker.js` into the Worker entry file.
4. Set the secret:

   ```bash
   wrangler secret put RAILCORE_KEY
   ```

   Paste your RailCore key into the terminal prompt. Do not commit it to GitHub or put it in `config.js`.

5. Optionally restrict CORS to your GitHub Pages origin by setting a Worker variable named `ALLOWED_ORIGIN`, e.g. `https://YOUR-USERNAME.github.io` (include the repository path only if you configure that exact origin; browser Origin normally contains the host only).
6. Deploy:

   ```bash
   wrangler deploy
   ```

7. Copy the deployed Worker URL, e.g. `https://railscout-api.YOUR-SUBDOMAIN.workers.dev`.
8. In `config.js`, set:

   ```js
   window.RAILSCOUT_CONFIG = {
     API_BASE: 'https://railscout-api.YOUR-SUBDOMAIN.workers.dev'
   };
   ```

9. Commit and push the project to GitHub. In repository Settings → Pages, deploy from the `main` branch and `/ (root)`.

## 3. API usage and cost guardrails

The full scan makes one `trains between stations` request per destination per selected date. With 30 destinations and 3 dates, that is **90 successful route lookups**. RailCore's published Free plan currently lists 100 requests/day and 500/month; check its current pricing/limits before scanning. A full 30 × 3 scan therefore uses most of a Free daily allowance. Failed requests are documented as unmetered, but successful requests count.

A fresh availability check is deliberately on-demand for one train and selected class(es). Checking both SL and 3A uses two class-specific availability requests. This avoids automatically spending a large number of API credits on every train returned for every destination.

The route endpoint returns compact availability context that may be cached upstream. The UI therefore distinguishes the route scan from the on-demand **Fresh check**. Always look at the provider's returned update timestamp and verify before booking.

## 4. Important data notes

- The destination list uses a representative station code for each destination. `match=city` asks RailCore to include other terminals in the same city where supported. The actual boarding and destination station codes are shown per train.
- Kumbhalgarh is represented by Falna (FA), a railhead for onward road travel; it is not a station at Kumbhalgarh Fort itself.
- Train counts and train-class counts are different metrics. The chart uses train-class results (SL and 3A are separate checks). The table also shows train count as class results; unique train numbers are noted in the KPI subtext.
- Berth totals are only added when the API supplies a numeric available-count field or an `AVAILABLE-n` text. A blank means the provider did not supply a count, not zero.
- This is a planning aid, not a booking system. Availability changes quickly. Confirm final details on IRCTC or an authorised booking channel.

## 5. Customise destinations

Edit the `DESTINATIONS` array near the top of `app.js`. Each entry uses:

```js
{ name: 'Agra', code: 'AGC', region: 'Taj Mahal', tag: 'Heritage' }
```

Use valid RailCore station codes. `match=city` is used for route discovery.

## Files

- `index.html` – dashboard structure
- `styles.css` – responsive design
- `config.js` – public Worker URL only (never put secrets here)
- `app.js` – UI, demo mode, scan and detail logic
- `api/worker.js` – server-side RailCore proxy
