# REC Bus Live Tracker

No login anywhere. **Anyone** can open the site, view buses, track them on the map, and share a bus's live location.

One phone on the bus shares GPS → Supabase (one row per bus) → Supabase Realtime → every viewer's Leaflet map moves live.

**Stack:** React + Vite + TypeScript, Tailwind v4, Supabase (Postgres + Realtime), Leaflet + OpenStreetMap, hosted on Vercel.

## What changed in this version
- **Map flicker fixed.** Map is created once; route/stops redraw only when stops change; the bus marker is moved with `setLatLng`.
- **Blank/grey map fixed.** `invalidateSize()` on resize; tile URL is `https://tile.openstreetmap.org/{z}/{x}/{y}.png`.
- **One realtime path** (`postgres_changes`), with re-sync after reconnect / when the tab becomes visible.
- **No login, no admin page, no accounts.** Everyone can share.
- **One sharer per bus.** A database trigger blocks a second device from taking over a bus updated in the last 60 s. The lock releases when the sharer taps Stop or after 60 s of silence.
- **Sharing stays alive:** screen wake lock + a 15 s heartbeat while the bus is standing still.
- The old vanilla `index.html` was replaced by a normal Vite entry; its logos are in `public/`.

## Setup
1. **Supabase** → SQL Editor → run `supabase/schema.sql`, then `supabase/seed_rec_routes.sql`.
2. Copy `.env.example` to `.env` and paste your **anon/publishable** key (Project Settings → API). Never use the `service_role` / secret key in this app.
3. `npm install` then `npm run dev`.
4. **Edit buses and stops** in Supabase → Table Editor (`buses`, `bus_stops`). The seed coordinates are approximate, so correct them there.

## Deploy on Vercel
Push to GitHub → import the repo in Vercel (Framework: Vite) → add `VITE_SUPABASE_URL` and `VITE_SUPABASE_ANON_KEY` under Settings → Environment Variables → deploy. `vercel.json` rewrites all routes to `index.html`, so `/track/3` works on refresh.

## Testing GPS
- GPS only works on **HTTPS** or `localhost` (Vercel gives you HTTPS).
- Laptop: Chrome DevTools → ⋮ → More tools → Sensors → custom location.
- No GPS: use **Start Demo** on the Share page.
- Two phones: phone 1 → Share Location → pick a bus → Start. Phone 2 → Track → the marker moves. Stop Sharing → phone 2 shows Offline.

## Because there is no login (please read)
Anyone who opens the site can share a location, so someone could deliberately send a wrong position. The database limits the damage but can't fully prevent it:
- Only coordinates inside a box around Chennai are accepted (edit in `schema.sql`).
- One device holds a bus at a time (60 s lock), and updates faster than 1/second are ignored.
- Nobody can delete data from the app, and buses/stops can only be changed from the Supabase dashboard.

If misuse ever happens, the easiest upgrades are a **secret share code** per bus (one extra column and field) or turning login back on for sharing only.

## Troubleshooting
| Problem | Fix |
|---|---|
| "Supabase is not configured" banner | Fill `.env` (or Vercel env vars) and restart/redeploy |
| "new row violates row-level security" | Location is outside the allowed box in `schema.sql`, or the schema wasn't run |
| "currently being shared by another device" | The lock works. Track instead, or wait 60 s after they stop |
| Marker never moves on viewers | Check Realtime is enabled for `bus_locations` (Database → Replication) |
| Map is grey/blank | Check the browser console / network tab for blocked tile requests |
| Sharing stops when phone locks | Keep the screen on; the wake lock helps on most phones but not all |
