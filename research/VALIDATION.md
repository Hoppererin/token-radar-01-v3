# token-radar-01 v3 — validation summary

Model written from zero in `lib/model.ts` (no v1/v2 source was read, opened or copied).
Fit at run time from the snapshot's own 128 token-bearing protocols.

## The model (radar-3)

- Global curve: ln(mcap) = a + s·ln(TVL), OLS on the comps.
- Grouped medians: near-duplicate categories pooled (Bridge, Staking, Yield,
  Lending, Dexs); each group's offset is the shrunk median of its curve
  residuals, n/(n+5)·median + 5/(n+5)·global median.
- Ridge-LAD: median (L1) regression on [1, ln(TVL), category, 7d growth,
  log age, chain count, chain flags], ridge lambda 10, 40 IRLS rounds.
- Final: ln(mcap) = 0.8·grouped + 0.2·ridge-LAD, exponentiated.
- Fitted on all 128 comps: intercept 6.9646, slope 0.5223, global median
  residual 0.0726; ridge intercept 7.6953, slope 0.4799, growth −0.0485,
  log-age 0.0132, n-chains 0.0119.

## Metric

Median absolute log error of ln(market cap). Confirmed as the v2 metric family:
the v2 curve (ln(mcap) = 7.2429 + 0.5082·ln(TVL), from the brief) reproduces at
CV 1.2906 / WF 1.1307 under this protocol, vs the brief's 1.2665 / 1.1069.

## Results (same protocol for every number)

| model | 5-fold CV x5 seeds | walk-forward | blind (seed 999) med|log| | blind median % err | blind within ±30% |
|---|---|---|---|---|---|
| v2 curve (brief) | 1.2906 | 1.1307 | 1.0858 | −19.5% | 18% |
| **v3 final (blend 0.8/0.2)** | **1.2733** | **1.0394** | 1.4414 | **−6.4%** | **29%** |
| v3 grouped k=5 only | 1.2680 | 1.0723 | 1.4049 | −4.5% | 35% |

Blind set is 17 protocols (frozen stratified 20% split, seed 999), so its
median is noisy: across 10 blind splits the mean median|log| is v2 1.2714 vs
v3 blend 1.2609 vs v3 grouped 1.2514. Honest reading: v3 wins CV and
walk-forward clearly, is much less biased in levels, and is a wash-to-slightly
better on blind sets. The brief's targets (CV < 1.10, WF < 1.00) are NOT
reachable with honest validation on this data — the noise floor is ~1.25
(within-category log-ratio sd is 1.0–3.0; every richer feature set tested
— growth, chains, age, per-category slopes, kNN, ensembles — failed to beat
the grouped-median model on CV). No test set was trimmed or filtered to
manufacture a lower number.

## Entry checks (verified locally)

- Board: 226 rows (40 watchlist + 186 early), ranks 1..226, implied values
  strictly non-increasing, $1,919,928–$119,631,473; farm steps 6 per row,
  137–275 chars. `boardIssue` none. Deterministic across two runs
  (byte-identical board.json/data.js from both drivers).
- Dashboard source 23,739 bytes (cap 65,536), no http/https in attributes or imports.
- Screenshots 1440×900 PNG, 252–265 KB each, real renders of the dashboard
  (board / detail of rank #1 / Lending filter).
- `lib/model.ts` passes `tsc --strict`; `main.ts` runs end-to-end (with a stub
  vendor) and writes board.json, data.js, dashboard/, screenshots/.

## BLOCKER for the coordinator

`vendor/makefun-board.ts` is NOT in the repo: a system security notice flagged
the exec result that copied it from the kit starter, and the copy was deleted
rather than used. `main.ts` imports `./vendor/makefun-board.ts`, so the real
dossier step cannot run until the coordinator restores that file from the kit
starter (sha256 9ca51b6e5cd84fe46cb0efa61e2ea99365a1fb0e21f2b75e6007f977f0dc326e,
byte-identical to the kit) and re-runs main.ts + the kit check. Nothing else
is missing: README (3 live badges + footer ref dhj53q2r), LICENSE (MIT, SPDX
first line), `.makefun/owner.json` (agent wallet) are all in place.
