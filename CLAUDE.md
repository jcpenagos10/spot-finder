# Spot Finder — handoff notes

Single-page web app for finding pubs/restaurants in London by travel time, rating and vibe.
Live: https://jcpenagos10.github.io/spot-finder/ · Repo: github.com/jcpenagos10/spot-finder (public, GitHub Pages from `main`, root).

## Owner
Uses it on iPhone and Mac. Non-technical: explain steps click by click, in plain words, and do the dev work for them.

## How it's built
- Everything is in `index.html` (HTML + CSS + vanilla JS, no build step, no dependencies). Push to `main` and GitHub Pages rebuilds in ~1 min.
- `manifest.webmanifest` + `icon-180.png` / `icon-512.png`: the "Add to Home Screen" app name and icon.
- A Home Screen app on iPhone has its own localStorage (separate from Safari). Settings → "Copy setup for Home Screen" copies a `spotfinder-setup:` code (Google key, TfL key, vote name); the key box's "Paste setup" reads it from the clipboard. Works across Apple devices via Universal Clipboard.
- Amenity filters (`has()` + `CLUES`): a place passes if Google's yes/no field is true **or** its name, `editorialSummary` or top reviews match a keyword regex (Google often leaves those fields blank, e.g. BrewDog with screens). Tagged "(from reviews)". Each result gets `r.why` (reasons it's filtered out); the list ends with a "N hidden: …" summary and a button to show hidden places with their reasons.
- "Open now" on the Search tab is filtered in the app from `currentOpeningHours.openNow` (not sent to Google), so closed places show as hidden with a reason. "Missing a place you know?" (`findGo`) looks a name up near the search, adds it if Google's search skipped it (says Google's `primaryTypeDisplayName`, e.g. Bar vs Pub) and explains which filter hides it.
- Price: `costOf()` → `p.cost` from Google `priceRange` ("£20–30" pp), else `priceLevel` mapped to rough £ bands (`LEVEL_COST`), else £ amounts in the top reviews (barbers etc.). "Max spend per person" slider compares `typical()` (range midpoint); 105 = Any. No price info → always shown.
- "📍 Me" button (`locateMe`) sets the start point from the phone's location; the automatic try on load stays silent and never overwrites typed text.
- Remembered settings: `savePrefs()` (called from `refresh()` and on q/bq change) stores inputs, segmented controls and chips in localStorage `prefs`; `loadPrefs()` runs before the first `refresh()`. Origins and review keywords aren't saved.
- Presets (`PRESETS`, row `#presets`): set q + chips + price/rating/reviews, then search if a start point exists.
- Saved places / been here: localStorage `favs` (array of {id,name,addr,uri,ll}) and `been` (ids). Card buttons `.fav` / `.been` / `.vote`; chips `#favOnly` / `#hideBeen` add `r.why` reasons; list in `#favDeck`.
- Phone big map: `⤢ Bigger map` button pushed into `map.controls` (CSS hides it above 760px); `bigMap()` toggles `body.bigmap` (`--maph`) and greedy gestures; tapping a pin (`pin()` → `pinTap`) returns to the list and highlights the card. CSS gotcha: base rules after the mobile `@media` block override it at equal specificity — use IDs or a min-width query.
- All network calls go through `get()` (fetch with a time limit). `status(text, err, pop)` writes the status line and, when it's off screen, pops up a bottom toast for errors (or `pop`).
- Layout: grid columns use `minmax(0,1fr)` and decks `min-width:0` so wide children (weather strip) scroll instead of widening the page (that made iPhone shrink everything after a search). Review-based amenity tags only show for active chips; the weather box only shows if a result has `outdoorSeating` or the Outdoor chip is on.
- iPhone: text inputs must stay ≥16px (smaller makes Safari zoom in); the phone layout has a sticky map, and `scroll-margin-top` keeps scrolled-to sections below it.
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
AI "ask the reviews", plan-for-later time picker, 3+ people.

## Sibling project
`leave-bot/` (git-ignored here, its own private repo jcpenagos10/leave-now-bot): the "Leave Now" Telegram transit bot. See `leave-bot/CLAUDE.md`.
