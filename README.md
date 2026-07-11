# duek-walkforward-validation

**The validation protocol that falsified the entry-signal hypothesis — and the negative result, published in full.**

Part of the [Duek research program](https://github.com/vrajpanchal-04/duek). This repository documents the out-of-sample validation methodology and its central finding: across five GA-optimized entry-strategy families on BTC/USDT perpetual futures, **no configuration produced a durable out-of-sample edge** under fixed φ reward-risk geometry and realistic taker execution.

---

## Why this repository exists

The turning point was a reproduction failure. A GA run reported a high-win-rate genome; a standalone engine rebuilt from that exact genome took **zero trades** on the same data. The "edge" lived partly in implementation details of the fitness evaluation, not in the market. That failure forced a rule that now governs the entire program:

> **No result counts until it is (a) reproduced by an independent implementation and (b) evaluated on data the optimizer never saw.**

This is the repository where that rule was applied at scale.

## The constraint set under test

| Constraint | Value |
|---|---|
| Reward-risk ratio | Fixed 233/144 = 1.61805556 (φ) |
| Target win rate | ≥ 57% |
| Trade frequency | 120–200 trades/year |
| Fees | KuCoin taker, 0.06% per side |
| Slippage | 2 bps per side, adverse |
| Intrabar resolution | 1-minute exits, **adverse-first** when stop and TP touch in the same minute |
| Funding | Excluded (noted as an upward bias on results) |
| Data | BTC/USDT PERP 1m OHLCV, 2020–2026, 3.4M+ bars; signals on 15m, bias on 1h |

## Protocol

1. **GA search** across five strategy families (breakout, pullback, three-candle, candle-run, hybrid; ICT-sweep tested in a later run), single-split, with indicator banks precomputed for tractability.
2. **Holdout feasibility scan** — random sampling of the parameter space evaluated directly on the holdout window, to test whether *any* region of the space meets the constraints (independent of GA selection bias).
3. **Rolling walk-forward** — optimize on 2 years, test on the next untouched year, roll forward.
4. **Bootstrap 95% confidence intervals** on trade-level outcomes for every surviving candidate.
5. **Multiple-testing awareness** — thousands of evaluated configurations mean the best in-sample result is expected to be a false discovery (White 2000; Bailey & López de Prado 2014). Selection was therefore never based on in-sample rank alone.

## Results

### Holdout window: best candidates meeting the trade-frequency band

Out-of-sample win rates for candidates inside 120–200 trades/year clustered at **0.44–0.47** (best observed ≈ 0.472), after fees and slippage. Bootstrap 95% CIs (representative range 0.388–0.511) do not plausibly reach the 0.57 target at realized sample sizes.

### Walk-forward folds (rolling 2y train / 1y test) — representative runs

| Train window | Test year | Family | Train WR | Test WR | Train net R | Test net R | Test max DD |
|---|---|---|---|---|---|---|---|
| 2020–2021 | 2022 | hybrid | 0.549 | **0.319** | +0.365 | **−0.223** | 20.6% |
| 2021–2022 | 2023 | hybrid | 0.500 | **0.312** | +0.203 | **−0.289** | 4.8% |
| 2022–2023 | 2024 | pullback | 0.513 | **0.428** | +0.249 | +0.032 | 8.0% |
| 2023–2024 | 2025 | pullback | 0.538 | **0.337** | +0.340 | **−0.191** | 27.2% |

**Reading:** train windows look strong — sometimes near target. The held-out year collapses in win rate, expectancy, or both, in three of four folds. The single non-negative fold (2024) is marginal. This is the textbook signature of backtest overfitting, observed directly.

### Full-sample check (2020–2026)

The same candidates frequently fail to remain profitable across the full period once all regimes and costs are included — a regime-dependence + selection-bias signature.

## Conclusions

1. **The entry-signal hypothesis is falsified in this search space.** With OHLCV-only signals on 15m/1h and taker execution, {WR ≥ 57%, 120–200 t/yr, RR = φ} do not hold simultaneously out-of-sample.
2. **What survived:** the execution model, the risk geometry, and the validation protocol itself.
3. **What the folds revealed:** catastrophic test years are not random — they cluster by market regime. That observation is the foundation of the program's current direction ([`duek-regime-analysis`](https://github.com/vrajpanchal-04/duek-regime-analysis)).
4. **Structural paths that could change the answer** (untested claims, listed as hypotheses): maker-leaning execution with realistic fill modeling; information beyond bar OHLCV (order-book imbalance, liquidations, funding regime); relaxing one constraint; diversification across uncorrelated sub-strategies.

## Gene-level diagnosis

Across the search, win rate and trade frequency were controlled by identifiable, interpretable levers: selectivity gates (HTF strength minimums, trigger buffers, cooldowns) trade frequency against win rate; stop geometry dominates because RR is fixed — tighter stops raise trade count and lower win rate, and simultaneously raise per-trade cost in R-units, which is where small edges die.

## References

- H. White, "A Reality Check for Data Snooping," *Econometrica* 68(5), 2000.
- D. H. Bailey, J. Borwein, M. López de Prado, Q. J. Zhu, "Pseudo-Mathematics and Financial Charlatanism: The Effects of Backtest Overfitting on Out-of-Sample Performance," *Notices of the AMS* 61(5), 2014.
- M. López de Prado, *Advances in Financial Machine Learning*, Wiley, 2018.

---

*Research only. Nothing here is investment advice. No forward-looking performance claims are made.*
