# Spot Finder — handoff notes

Single-page web app for finding pubs/restaurants in London by travel time, rating and vibe.
Live: https://jcpenagos10.github.io/spot-finder/ · Repo: github.com/jcpenagos10/spot-finder (public, GitHub Pages from `main`, root).

## Owner
Uses it on iPhone and Mac. Non-technical: explain steps click by click, in plain words, and do the dev work for them.

## How it's built
- Everything is in `index.html` (HTML + CSS + vanilla JS, no build step, no dependencies). Push to `main` and GitHub Pages rebuilds in ~1 min.
- Check a deploy: `gh api repos/jcpenagos10/spot-finder/pages/builds/latest --jq .status` → `built`.
- Local preview: `python3 -m http.server 8765` (`.claude/launch.json` has it as `spot-finder`).

## Keys (never commit any)
- The Google Maps API key is pasted by each user in the app and saved only in their browser's localStorage (`gmapsKey`). It's restricted to `https://jcpenagos10.github.io/*`. Optional TfL key also in localStorage (`tflKey`).
- Don't type or paste API keys into fields on the user's behalf; ask them to.

## Data sources
- Google Maps JS: map + markers only (`AdvancedMarkerElement`, mapId `DEMO_MAP_ID`).
- Places API (New) REST `places:searchText` (up to 60 results via `nextPageToken`). REST field names: `outdoorSeating`, `goodForWatchingSports`, `servesBeer`… (the Maps JS Place class uses different names — don't mix them).
- Routes API `computeRouteMatrix` (proto3 JSON omits zero indexes → `?? 0`).
- TfL Journey Planner overrides Google's transit times in London; TfL line status for ⚠ warnings. No key needed.
- Open-Meteo (weather strip, no key), ESPN public scoreboard feed (match picker, no key), ntfy.sh (live group votes; topic `spotvote-<random>`, options in the URL hash, messages kept ~12h).

## Features
Search tab (two origins + fairness sort, filters, review keyword highlights, price, amenities, "show 20 more"), weather strip for outdoor seating, "⚽ Watch a match", "Best of" tab (trusted score = Bayesian average with 200 reviews at 4.0 prior; specific searches like "Basque cheesecake" put places whose name/top 5 reviews mention the rare words first (`dishWords`); walking crawl planner with brute-force order + Google Maps link), group vote (friends need no Google key).

## Testing without a Google key
In the browser, stub the globals `map`, `Marker`, `Pin`, `google.maps`, `textSearch`, `googleTimes`, set `origins[0].ll`, then click buttons. TfL / weather / fixtures / ntfy work for real.

## Ideas not built yet
AI "ask the reviews", plan-for-later time picker, 3+ people, a one-tap share link carrying the Google key (`#key=`) for friends.

## Sibling project
`leave-bot/` (git-ignored here, its own private repo jcpenagos10/leave-now-bot): the "Leave Now" Telegram transit bot. See `leave-bot/CLAUDE.md`.
