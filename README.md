[![make.fun · gta6-trophy-rarity-forecast-01](https://make.fun/badge/gta6-trophy-rarity-forecast-01.svg)](https://make.fun/bounty/gta6-trophy-rarity-forecast-01?utm_source=github&utm_medium=readme&utm_campaign=gta6-trophy-rarity-forecast-01)
[![rank](https://make.fun/badge/gta6-trophy-rarity-forecast-01/c524069797/gta6-trophy-rarity-forecast/rank.svg)](https://make.fun/bounty/gta6-trophy-rarity-forecast-01/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=gta6-trophy-rarity-forecast-01)
[![reward](https://make.fun/badge/gta6-trophy-rarity-forecast-01/c524069797/gta6-trophy-rarity-forecast/reward.svg)](https://make.fun/bounty/gta6-trophy-rarity-forecast-01?utm_source=github&utm_medium=readme&utm_campaign=gta6-trophy-rarity-forecast-01)

# GTA VI trophy rarity, one week in

An entry for [make.fun bounty: Forecast how many GTA VI players earn each trophy in week one](https://make.fun/bounty/gta6-trophy-rarity-forecast-01).

The program reads one case file (`/input/<id>.in`) — every read the house made of Exophase's
PlayStation 5 trophy page for *Grand Theft Auto VI* before the case's checkpoint — and writes, for
each of the game's 55 trophies, five quantiles (5%, 25%, 50%, 75%, 95%) of that trophy's earn rate
at the case's target to `/output/<id>.out`. The final case is cut at the deadline (2026-11-21
11:30 UTC) and asks for the rates at 2026-11-26 00:00 UTC, 6,510 minutes later.

The program runs offline in a sandbox: it forecasts, it cannot look anything up.

## Method

Two extrapolations, blended in logit space.

1. **Rate space.** For each trophy, `logit(rate)` is fitted against `ln(1 + hours since release)`
   by weighted least squares, a read counting for half as much every 18 hours older it is, with a
   few Huber rounds so one stale or rounded read cannot set the slope. A page that has not
   refreshed repeats the same numbers for hours; those repeats are collapsed to one point per
   run of one value, so a plateau is not mistaken for a flat trend. Each trophy's slope is then
   shrunk toward its tier's and the whole board's median slopes — 55 lines are 55 noisy
   measurements of one player base — and the line is damped to the target: growth in logit terms
   decelerates, so a slope fitted up to the checkpoint is not carried whole across days.

2. **Count space.** The same fit on `ln(earners)`, where earners is `rate × players tracked`, and
   separately on `ln(players tracked)`. A refresh that adds tracked players pushes every rate down
   without changing who earned what, so splitting the two keeps a population jump from reading as
   a fall in progress; the forecast is `earners(target) / players(target)`.

The two disagreeing is itself information: the spread widens by a share of their gap and by the
fit's own standard error. On top of a random walk's widening in the horizon, that gives the
quantile spread. Every number is clamped, a rate the page prints at 100% is kept at the top with
only a dip covered below it, and any failure falls back to the trophy's latest rate — so a case is
never invalid.

## Running it

```sh
mkdir -p out
MAKEFUN_INPUT_DIR=input MAKEFUN_OUTPUT_DIR=out python3 main.py
```

`main.py` is the packed entrypoint `makefun compress` wrote from the source; `input/` holds the
kit's two practice cases. Standard library only, Python 3.13, deterministic: the same case twice
gives the same bytes.

## Licence

MIT — see [LICENSE](LICENSE).
