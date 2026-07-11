# Gridiron GM — NIL Trio Builder

A free-play college-football game. You get a **$10M NIL budget** to sign one QB,
one RB, and one WR from real player-seasons (2015–2025). Spin each position in
order — QB, then RB, then WR — and each spin deals a fresh board of real players
at different NIL prices. Sign one, bank the rest, and you can't see the next
spin until you commit. Your finished trio is ranked against **every possible
QB-RB-WR trio** in the pool.

Built on the same 11-year CFBD player dataset, deployed to Netlify like CFB Scordle.

## How it plays
- **Value = production percentile within position.** A top-1% RB is worth the
  same 1000 points as a top-1% QB, so every pick matters equally — the QB doesn't
  drown out the skill positions.
- **Trio score** = QB value + RB value + WR value (0–3000).
- **Rank** = your score's exact percentile against the full distribution of all
  trios, precomputed by convolution and baked into `gamedata.js` (an 11-entry-per-
  point lookup, ~11 KB). Instant, no backend.
- **The catch:** the three highest-value players cost ~$12M combined — more than
  your $10M. You literally cannot afford the perfect team, so the game is about
  finding value per dollar, not just buying stars. Random play lands ~top 75%;
  sharp value-hunting can reach top 99.9% on the same budget.

## Files
- `index.html` — the entire game (HTML + CSS + JS).
- `gamedata.js` — 11,100 embedded player-seasons + the ranking CDF (~590 KB).
- `netlify.toml` — publish dir (`.`). No build step, no functions.

## Deploy — same as Scordle, but simpler (no leaderboard = pure static)

This game has **no backend**, so unlike Scordle you have three easy options.

### Option A — Connect a Git repo (recommended, matches your workflow)
1. Put this folder in a GitHub repo and push it.
2. In Netlify: **Add new site → Import an existing project → GitHub**, pick the repo.
3. Leave build command blank; publish directory `.`. Deploy.
4. Every push auto-deploys. Edit `gamedata.js` or `index.html` in GitHub's web
   editor (works fine on your locked Windows machine) and it redeploys itself.

### Option B — Drag and drop (works here since there's no function)
Just drag this folder onto the Netlify dashboard. Done. (This is the thing that
*couldn't* work for Scordle because of its leaderboard dependency.)

### Option C — Netlify CLI
```bash
npm i -g netlify-cli
netlify deploy --prod
```

## After it's live
Rename the site subdomain in **Site settings** (e.g. `gridiron-gm`). The share
button auto-uses the deployed URL — nothing to configure.

## Tuning knobs (top of the `<script>` in index.html)
- `BUDGET` — the NIL cap (default `10_000_000`).
- `CANDIDATES` — players dealt per spin (default `4`).
- Price bands live in `drawCandidates()` and `tierOf()` if you want to reshape
  the star/impact/starter/sleeper tiers.

## Refreshing / expanding the data later
Re-run the data build against your CSV to regenerate `gamedata.js`, then commit
it. The pipeline: pivot the long-format CSV to player-seasons, infer position
from dominant production, score = yards + TD×25, convert to positional percentile
(value 0–1000), price on a steep percentile curve ($50K–$4M), then convolve the
three value histograms to rebuild the ranking CDF.
