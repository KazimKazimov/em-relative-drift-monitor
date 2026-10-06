# Relative Drift Monitor

Window-free, online detection of large relative moves (outperformance / underperformance) across a basket of assets, with a backtest and PnL attribution. Built around 30 EM 10y yields, designed to extend to FX, credit, rates, equities and CDS.

**Run it:** open `index.html` in any browser (single file, no dependencies, works offline). If GitHub Pages is enabled, it is also served at the repo's Pages URL.

> Status: all results shown are on **synthetic data**. Plug in real, carry-adjusted returns before drawing conclusions.

## Method

1. **Beta-adjusted residuals.** Recursive least squares (forgetting factor, half-life 90d) of each asset against the cross-sectional mean or a factor. Residuals are EWMA-standardised.
2. **Two-sided CUSUM** on the standardised residuals (parameters `h` alarm threshold, `k` drift allowance). Signed statistic `S = S+ - S-`. Onset = last zero crossing of `S`. No lookback window.
3. **Swing layer.** Zigzag on the cumulative residual; a turn confirms when it reverses by more than `ZZ(2) x 250d residual vol x sqrt(20)`. Reports bp since turn, peak, pullback, percentile of completed swings (the "max drawdown" view).
4. **Pair matrix.** Same CUSUM on `dy_i - b_ij dy_j` for every ordered pair; net score `g_i = mean_j (S_ij - S_ji)/2`.
5. **Analog episodes.** First crossing of an `|S|` bucket (0.5 step) in a direction; stores forward residuals at 5/10/20/40d, age bin and vol bin. `P(continue)` comes from pooled analogs: all, same region, similar rating, similar duration (widening order (attr,ctx) -> (*,ctx) -> (attr,*) -> (*,*)).
6. **Scanner.** Filter assets/pairs by P(mean reversion) / P(momentum), horizon and pooling.
7. **Backtest.** Point-in-time (only episodes whose horizon completed before day t), signal at t, PnL at t+1, 60d warm-up. Rules: daily top-K, alarm threshold (enter `|g|>h/2`, exit `<h/4`), analog probability (momentum and reversion books separate). Sizing: 1/beta (net beta zero ex ante) or equal weight.
8. **Attribution.** By entry `|S|`, age, with/against swing, vol regime, book; n, hit rate, mean bp, t-stat, average PnL path.

## Views

Ranked | Pair matrix | Scanner | Backtest. The "true dynamics" slider on synthetic data runs from reverting (-1) to trending (+1).

## Real-data needs

Carry-adjusted returns per asset (yield: carry + rolldown x duration; FX: forward points; credit/CDS: spread carry), per-class factors, a NaN policy, an asset-class adapter, and ideally Kalman betas. Static `META` table (region, rating bucket, duration) in the source is approximate and replaceable.

## Files

- `index.html` - the monitor (all code inline; engine functions: `simulate`, `runCusum`, `swings`, `computePairs`, `netScores`, `buildEpisodes`, `lookup`, `analogStats`, `backtest`, `backtestAnalog`, `renderAttribution`).
- `docs/hub.html` - documentation hub: method, parameters, findings, data spec, roadmap, MATLAB/Python port map, change log.

## Roadmap

Swing-aware analogs, Kalman betas, carry adjustment, multi-asset confirmation, walk-forward holdout, exposure diagnostics, Wilson intervals, daily-change view, event calendar, trade export, port of the engine to Python/MATLAB.
