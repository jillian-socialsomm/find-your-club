# Find Your Club — SocialSomm club-matching quiz

A self-contained, dependency-free quiz that matches visitors to a SocialSomm wine club.
Live at **https://jillian-socialsomm.github.io/find-your-club/** (GitHub Pages, deploys
automatically on push to `main`).

## Files

| File | What it is |
|---|---|
| `index.html` | The entire app: markup, styles, quiz logic, club data, embedded media. No build step, no dependencies. |
| `weights.json` | The scoring rubric. Fetched at page load; edit it here on GitHub to change live scoring without touching code. If the fetch fails, fallback weights embedded in `index.html` apply. |

## How it works

1. **Five questions** (single-choice, choose-3, a 5-stop slider, and two more single-choice).
   Each answer awards points to one or more of 8 clubs per `weights.json`
   (club keys: `sarah, madison, jason, vince, tps, hillary, grapechic, sss`; negatives subtract).
2. **Email gate** before results.
3. **Results = podium of top 3 clubs** (winner center), each tile built from club data
   (pulled from the SocialSomm Bubble DB and baked into `index.html`), with a card-flip
   to that club's box-overview video.

### Hard rules (in code, not in weights.json — search `showResults` in index.html)

- **SocialSomm Selections (`sss`) always appears** in results: keeps rank 1–2 if earned, else takes the No. 3 slot.
- **"<$20" budget answer forces the No. 1 match to TPS** — unless the natty answer was
  "natty funk" or "good ones occasionally", in which case it forces **Hillary**.
- **Sippers ($25–75) or cellar ($40–75) budget answers exclude TPS** from results entirely.

## Data flow (results storage)

Each completion POSTs to a Google Form (see the `GFORM` constant: action URL + `entry.*`
field IDs), which feeds the "Test Quiz Reponses" Google Sheet (owner: Jill). Two row types,
distinguished by `answers_json`:

- **Completion**: email, `answers_json` (all 5 answers), `scores_json` (all 8 club totals), `matches_json` (top 3).
- **Select click**: `answers_json = {"event":"select_click","club":"..."}` — fired when a
  results tile's "Select This Club" button is clicked.

The Sheet's "Merged view" tab joins the two by email. A Bubble backend-workflow endpoint
(`RESULTS_ENDPOINT` constant, `record_quiz_result`) is pre-wired for the future Bubble
migration; it currently 404s harmlessly.

## Videos

Card backs play each club's latest box-overview video via the **Vimeo player**
(IDs + unlisted hashes in the `CLUBS` object). The videos are domain-whitelisted for
`socialsomm.com`, `bubble.io`, `jillian-socialsomm.github.io`, and `localhost` — a new
host must be added to each video's whitelist (Vimeo API: `PUT /videos/{id}/privacy/domains/{domain}`).
Where the Vimeo player can't run (e.g. sandboxed hosting), the page falls back to embedded
2-second preview clips (base64 data URIs in `CLUBS`, which is also why `index.html` is ~1MB —
somm photos are embedded the same way).

## Editing

- **Scoring weights:** edit `weights.json` (answers are in on-screen order). Live ~1 min after commit.
- **Copy, club data, videos, rules:** edit `index.html`. Club tile data (prices, cadence,
  descriptions, Vimeo IDs) lives in the `CLUBS` object; questions/answers in `QUESTIONS`.
- **Local preview:** open `index.html` in a browser, or serve the folder
  (`python3 -m http.server`) for weights.json loading to work.

## Planned next phase

Embed in the SocialSomm Bubble app (paste into an HTML element): results POST to a Bubble
backend workflow instead of the Google Form, weights move to a Bubble table exposed via a
public workflow, and club tile data reads live from Bubble instead of being baked in.
