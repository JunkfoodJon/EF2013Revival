# Electric Forest Revival (2013 + 2014)

`index.html` is a single self-contained page: open it in a browser, no server needed.

It reconstructs what was played at Electric Forest 2013 (June 27–30) and 2014 (June 26–29), both at the Double JJ Ranch in Rothbury MI. The year switch at the top (or `index.html#2014`) changes years. Sets are ordered by each year's official set times, and each track is labeled with how sure we are:

| Tier | Meaning |
|---|---|
| Confirmed | Documented for this exact Electric Forest set (setlist.fm, 1001Tracklists, taper notes) |
| Identified | Picked out of a recording or video of this set |
| Same tour | From the same act's set within a few weeks (source show named on each track) |
| Era staple | A track from that year the act was playing; an educated guess |

## Layout

- `data/<year>/research/*.json` — research per day/stage group (sets → tracks, with source URLs)
- `data/<year>/schedule.json` — official set times, all stages. 2013 comes from the archived electricforestfestival.com/set-times.php. 2014 is hand-transcribed from the official schedule image (`data/2014/official-schedule-2014.jpg`, via `src/schedule2014_src.py`).
- `data/<year>/tracklists.json` — merged output per year, handy for reuse
- `data/<year>/BRIEF.md` — the research brief the agents followed (2014)
- `data/links_cache.json` — `"Artist|Title"` → Spotify / YouTube links (shared by both years)
- `data/overrides.json` — optional manual link fixes, same shape as the cache (create it if needed)
- `src/template.html` + `src/build.py` → `index.html` (year text such as dates and day subtitles lives in `YEAR_META` in build.py)

## Adding or fixing tracks

1. Edit or add a file in `data/<year>/research/` (same shape as the others). To add a year, add its folder plus a `YEAR_META` entry.
2. `python src/build.py`
3. New tracks show a YouTube **Search** button until a link is resolved. To resolve them: run `python src/make_queue.py queueN`, then `python src/resolve_youtube.py queueN`, then `python src/merge_links.py data/queueN.json`, then rebuild. Spotify matches were found through the open.spotify.com web player and checked by hand (`data/sp_batch*.txt`). To fix any one link, put it in `overrides.json`.

## Playlists

Each set's **Playlist ▾** menu (also in My Forest Mix) uses the tracks currently shown, in set order:

- **Play on YouTube now**: no sign-in. Opens a YouTube playlist built from the set's video IDs, split into parts of 50 because that is YouTube's limit for these links.
- **Save to my YouTube / Save to my Spotify**: creates a private playlist in your account. Saving the same set again updates that playlist instead of making a duplicate. This needs a free app ID from each service, added under *Playlist settings…*. Sign-in only works when the page is served at `http://127.0.0.1:8613/`. Run `python -m http.server 8613 --bind 127.0.0.1` in this folder and open that address; Spotify rejects `file://` and `localhost`.
  - YouTube's free API quota allows about 190 added tracks per day. If a save runs out, the playlist keeps what was added, and the next save resumes where it stopped.
- **Copy Spotify links**: paste them into any playlist in the Spotify app (Ctrl/Cmd+V) to add every track.

## How links are chosen and verified

- **YouTube** (`src/resolve_youtube_v2.py`): scores the top 12 search results instead of taking the first one. It checks that the title and artist match and that the right remix/edit or acappella is named. It prefers a plausible song length and uploads from around that era (about 10–14 years ago); official "– Topic" uploads are trusted. It penalizes nightcore, sped-up, slowed, 8D, ringtone, cover, 1-hour, and full-set uploads.
- **Spotify**: matched through the web player's search. `src/verify_spotify.py` then reads each track's length, release date and playability. `src/pick_earliest.py` swaps a later reissue for the earliest release of the same song: exact title, exact artist, and a length within 30%.
- **Checks at build time** (`src/verify_rules.py`): a link is dropped, falling back to a search button, when:
  - it isn't the same title,
  - it isn't playable,
  - it is a snippet under 45 s or a mix/set over 20–30 min.

  A track is flagged ⚠ when:
  - it was released after the festival, beyond the normal case of a DJ playing an unreleased ID,
  - an era guess postdates the festival,
  - the Spotify and YouTube versions differ a lot in length.

  Each row shows the verified length and year; hover the ✓/⚠ to see why.
- Manual fixes go in `data/overrides.json`, merged field by field, for example a listener's own verified YouTube picks.

Refresh order after research changes: `make_queue.py` → Spotify search (browser) → `merge_links.py` → `resolve_youtube_v2.py` → `apply_youtube_v2.py` → `verify_spotify.py` → `build.py`.

## Listener suggestions ("Add missing tracks")

Every set card has **+ Add missing tracks**, and the bar has **Suggest a set** for acts that aren't listed. The reporter opens with that set's artist, stage badge, time and the tracks already listed. A report has: what it's about (missing / wrong / order / missing set), how the person knows (was there / recording / tracklist site), tracks as `Artist - Title` lines (a leading number such as `12.` places a track in the set), an optional source link and a note.

- **Live site (GitHub Pages)**: **Send on GitHub** opens a pre-filled issue on this repo; the person clicks Submit (a free GitHub account is needed). The **Missing tracks** button (top left) opens `missing.html`, which reads the repo's issues through GitHub's public API and files each report under its set with the same stage colors. Review on GitHub: open = New, label `accepted` = Accepted, closed as completed = Merged, closed as not planned (or `wontfix` / `invalid` / `duplicate`) = Rejected. The issue body carries the report as JSON in an `<!-- ef-report ... -->` comment; keep it when editing an issue.
- **Private claude.ai copy**: reports save in-page to its shared database and are reviewed under **Review suggestions**, with **Copy accepted (JSON)** for merging into `data/<year>/research/`.
- **Copy report** works everywhere and copies the report as text.

Published copy: `dist/electric-forest-revival.html` (built by `src/build.py`), live at the artifact link. It is private until shared from its Share menu. Saving playlists into your own YouTube or Spotify account only works in the local copy at `http://127.0.0.1:8613/`. The artifact sandbox can't reach those services' sign-in or APIs, so it offers the no-sign-in YouTube playlist and copy-links options instead.
