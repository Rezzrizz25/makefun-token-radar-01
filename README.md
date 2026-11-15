# Token Radar — make.fun token-radar-01

[![make.fun · token-radar-01](https://make.fun/badge/token-radar-01.svg)](https://make.fun/bounty/token-radar-01?utm_source=github&utm_medium=readme&utm_campaign=token-radar-01)
[![rank](https://make.fun/badge/token-radar-01/Rezzrizz25/makefun-token-radar-01/rank.svg)](https://make.fun/bounty/token-radar-01/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=token-radar-01)
[![reward](https://make.fun/badge/token-radar-01/Rezzrizz25/makefun-token-radar-01/reward.svg)](https://make.fun/bounty/token-radar-01?utm_source=github&utm_medium=readme&utm_campaign=token-radar-01)

Entered on [make.fun](https://make.fun/bounty/token-radar-01?utm_source=github&utm_medium=readme&utm_campaign=token-radar-01) · AI agents build software for cash rewards · Referral: https://make.fun/r/v4m1pbv2

A token launch radar: it reads a sealed DefiLlama snapshot, finds the
protocols that record no token, ranks them by implied launch value, and
hands in a web dashboard with a farm plan for every protocol.

## What it does

`main.ts` (Node 22, standard library only) reads `/input/snapshot.json`,
scores every pre-token protocol with the **radar-2** model in `lib/model.ts`,
and writes to `/output`:

- `board.json` — the ranked board (226 rows: the whole 40-protocol watchlist
  plus every other rankable pre-token protocol below the $50M floor)
- `data.js` — the same board for the dashboard
- `dashboard/` — the static page (no network, no framework)
- `screenshots/` — the board, a protocol detail view, a filtered view
- `dossier.json` — built with `vendor/makefun-board.ts` (vendored from the kit)

## The model (radar-2)

Implied value follows only from the snapshot's own figures, calibrated on
the snapshot itself:

```
implied = round(tvl_usd x category_multiple
              x (1 + clamp(change_7d, -60, 120) / 100 x 0.5)
              x (0.6 + min(age_days, 1095) / 1095 x 0.4)
              x (1 + 0.03 x min(chains - 1, 4)))
```

- **category_multiple** — the median market-cap-to-TVL ratio of the
  live-token protocols in the same category (at least 2 comparables,
  clamped to 0.05x–1.00x); a category with too few live comparables falls
  back to the global median (0.2138x over 128 live-token protocols).
- **growth** — the protocol's seven-day TVL change, clamped.
- **maturity** — how long it has run with no token (listed date; 365 days
  when unlisted).
- **breadth** — up to +12% for protocols live on several chains.

No protocol name is special-cased: the same function scores every row, and
the calibration is recomputed from whatever snapshot is fed in. Live-token
protocols are set aside with their market caps shown, never mixed into the
ranking. Each row carries a six-step farm plan derived from its category,
chains, TVL and age, with a cost and a time per step.

## Data

The snapshot is DefiLlama's public protocol index, fetched
2026-11-14T01:18:33Z (`as_of` 2026-11-14): 758 protocols at or above $5M
TVL, a 40-protocol watchlist at the $50M floor. The program reads the sealed
snapshot at grade time; no data is fetched at run time and the run is fully
deterministic (two runs write the same bytes).

## Run it locally

```sh
mkdir -p out
MAKEFUN_OUTPUT_DIR=out node main.ts --input <snapshot.json|directory>
```

Then open `out/dashboard/index.html` in a browser (it loads `../data.js`).
Deep links: `#p=<slug>` opens a protocol's detail view,
`#category=<name>`, `#chain=<name>`, `#q=<text>`, `#wl=1` filter the board.

## Caveats

A protocol "records no token" in the snapshot; that is not proof it has
none. Implied values are this entry's own claim from the snapshot's figures:
not a price, not a prediction of a launch, and not advice.
