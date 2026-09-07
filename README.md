# FX Carry Trade

A long-short G10 carry strategy, backtested on daily spot and monthly interest rate data from 2002 to 2026. Nine currencies are ranked each month by expected Sharpe ratio, defined as the interest rate differential against USD divided by realised volatility. The strategy goes long the top three and short the bottom three, rebalances monthly, and charges three basis points per unit of turnover. Over the full sample it produces an annualised Sharpe ratio of 0.41 with a skew of −0.52 and a maximum drawdown of 26 per cent concentrated in the 2008 crisis.

The negative skew is the central finding. Carry behaves like selling insurance: the rate differential is collected in small regular increments, and the accumulated premium is returned in a small number of violent unwinds.

## Strategy

1. **Universe.** Nine G10 currencies against USD: AUD, CAD, CHF, EUR, GBP, JPY, NOK, NZD, SEK.
2. **Carry.** Each currency's three-month interbank rate minus the USD rate.
3. **Volatility.** Rolling 252-day standard deviation of daily spot returns, annualised by √252.
4. **Signal.** Expected Sharpe ratio, computed as carry divided by volatility.
5. **Weights.** Long the three highest-ranked currencies, short the three lowest. Two weighting schemes are constructed and compared: equal notional (±1/3 per leg) and inverse volatility (proportional to 1/σ, normalised so each leg sums to one).
6. **Lag.** Weights are shifted forward one period before being applied to returns, to prevent look-ahead bias.
7. **Costs.** Turnover is measured as the absolute change in weights between rebalance dates, summed across currencies, and charged at three basis points.

The portfolio is zero-net-investment, so returns are already excess and no risk-free rate is subtracted.

## Data

| Series | Source | Frequency | Coverage |
|---|---|---|---|
| Spot FX | FRED | Daily | 2000 onward |
| Short-term interest rates | OECD, IR3TIB (three-month interbank) | Monthly | 2000 onward |

Rates are forward-filled to daily frequency and reindexed onto the spot calendar. The sample ends 2026-04-30, the last month with complete rate data across all nine currencies.

## Results

Sample: 2002-05-31 to 2026-04-30, 288 monthly observations.

| Metric | Equal-weighted | Inverse-vol weighted |
|---|---|---|
| Annualised Sharpe | 0.41 | 0.38 |
| Maximum drawdown | −26.0% | −26.3% |
| Drawdown date | 2009-01-31 | 2009-01-31 |
| Hit rate | 0.59 | 0.59 |
| Skew | −0.52 | −0.45 |
| Annualised turnover | 1.58 | 1.96 |

<img width="821" height="618" alt="equity_curve" src="https://github.com/user-attachments/assets/5bb53f7a-fe56-43bd-ad07-bf809bf7d026" />

The curve compounds steadily from 2002 to 2007, falls sharply into early 2009, then moves broadly sideways for a decade as G10 rate differentials compressed under near-zero policy rates. It turns up again from around 2021 as the Federal Reserve raised rates ahead of the ECB and the Bank of Japan.

## Robustness

The 252-day volatility window and the monthly rebalance are both arbitrary choices. The grid below reports every metric across three volatility windows and two rebalance frequencies, rather than a single preferred specification.

| Vol window | Frequency | Sharpe (eq) | Sharpe (iv) | Max DD (eq) | Max DD (iv) | Skew (eq) | Skew (iv) | Turnover (eq) | Turnover (iv) |
|---|---|---|---|---|---|---|---|---|---|
| 60 | Monthly | 0.366 | 0.332 | −0.280 | −0.295 | −0.563 | −0.485 | 2.06 | 3.30 |
| 60 | Quarterly | 0.342 | 0.315 | −0.285 | −0.296 | −1.355 | −1.310 | 1.38 | 2.24 |
| 90 | Monthly | 0.379 | 0.337 | −0.274 | −0.291 | −0.583 | −0.504 | 1.69 | 2.55 |
| 90 | Quarterly | 0.387 | 0.360 | −0.251 | −0.262 | −1.317 | −1.140 | 1.24 | 1.89 |
| 252 | Monthly | 0.408 | 0.384 | −0.260 | −0.263 | −0.522 | −0.450 | 1.58 | 1.96 |
| 252 | Quarterly | 0.349 | 0.318 | −0.259 | −0.270 | −1.491 | −1.474 | 1.09 | 1.47 |

Three observations:

- **Sharpe is clustered between 0.34 and 0.41 across all six specifications.** The reported result is therefore a property of the strategy rather than an artefact of the parameter choices. The best cell exceeds the second-best by 0.02, which is not a meaningful margin.
- **Equal weighting beats inverse-volatility weighting in every cell**, and inverse-volatility produces better skew in every cell. Both findings from the headline specification hold across the grid.
- **Skew is substantially more negative at quarterly frequency**, at roughly −1.3 to −1.5 against −0.45 to −0.58 monthly. Two effects contribute: consecutive losing months compound into a single extreme quarterly observation, and skew is a third-moment estimate computed on roughly 95 quarterly observations rather than 288 monthly ones.

## Findings

**Negative skew is the defining feature of the strategy.** The hit rate of 0.59 confirms the same profile independently: the strategy wins in most months and loses more when it loses. This is the return distribution of a short volatility position.

**The Sharpe ratio alone is insufficient to describe it.** Sharpe measures risk as standard deviation, which squares deviations from the mean and therefore treats a large gain as exactly as much risk as a large loss. It cannot distinguish a strategy whose tail lies on the upside from one whose tail lies on the downside. Reporting skew and maximum drawdown alongside it shows that this strategy's risk arrives concentrated in short episodes.

**Inverse-volatility weighting underperforms, and transaction costs are not the reason.** The difference in turnover of 0.38 turns per year costs approximately one basis point annually at three basis points, an order of magnitude below the observed return gap. The cause is the weighting itself. In G10, high-yielding currencies are also the volatile ones, because they carry a premium precisely for selling off in risk-off episodes. Dividing by volatility therefore reduces the largest carry positions and increases the smallest, so expected return falls at least as much as risk does.

**The signal compounds this.** It is already carry divided by volatility, so volatility selects the currencies and then sizes them as well. Volatile names are penalised twice for the same characteristic. Inverse-volatility weighting is appropriate when the underlying signal is volatility-neutral, which this one is not.

**The Sharpe ratio of 0.41 is the expected result rather than a disappointing one.** Published estimates for G10 carry typically fall between 0.4 and 0.7. This sample includes the entire post-2008 period of compressed rate differentials, and returns are charged transaction costs throughout.

## Assumptions and limitations

- **Carry is computed as a rate differential, not from forward points.** The two are equivalent under covered interest parity, but the parity relationship has not held cleanly since 2008. This omits the cross-currency basis, which in practice means the funding leg costs slightly more than the interbank rate implies. The reported returns are therefore mildly flattered, particularly after 2008.
- **Transaction costs are assumed at three basis points per unit of turnover.** G10 spot is highly liquid and bid-offer at institutional size is typically below one basis point, but the assumption is intended to cover market impact and the cost of rolling forwards as well.
- **Annualisation assumes serially independent returns.** Mean returns are scaled by 12 and volatility by √12. Carry unwinds are momentum-like, as the consecutive losses through 2008 demonstrate, so true annual volatility is somewhat higher than this scaling implies and the annualised Sharpe is mildly overstated.
- **The Japanese rate series begins 2002-04-01.** This is a genuine gap in OECD coverage and sets the start of the sample, since the basket cannot be ranked across all nine currencies before then.
- **Four monthly observations are missing from the UK series** and have been forward-filled. These fall at the end of the sample rather than in the interior, so the fill carries the last known GBP rate forward through the final months rather than bridging a gap between two known values. GBP carry is therefore stale at precisely the point where the sample ends.
- **The final partial month is dropped.** OECD rate data runs to 2026-05, but the spot series covers only a single day of that month, so the corresponding return would measure one day's move rather than a month's. The sample therefore ends 2026-04-30.
- **Monthly rates are forward-filled to daily frequency.** Rates are therefore stale within each month, which is realistic for a monthly rebalance but would matter at higher frequency.
- **The volatility estimate is backward-looking.** The signal is a forward-looking numerator over a backward-looking denominator. A fully ex-ante version would use option-implied volatility.
- **No correlation structure is modelled.** Inverse-volatility weighting equalises standalone risk contribution but ignores the correlations between currencies, which are high within the G10 bloc.

## Outstanding work

- **Transaction cost sensitivity.** Costs are fixed at three basis points throughout. Repeating the analysis across a range, for instance three, five, ten and twenty basis points, would establish the level at which the strategy stops being viable. That threshold is a more informative result than the single reported figure.
- **Correlation-aware weighting.** Both schemes size positions on standalone characteristics. A risk-parity or minimum-variance construction would account for the high correlations within the G10 bloc.

## Repository contents

| File | Description |
|---|---|
| `FX_Carry_Trade.ipynb` | Full analysis, Stages 1 to 11 |
| `G10Rates_Data.csv` | OECD short-term interest rates, exported from the OECD Data Explorer |
| `equity_curve.png` | Cumulative return, both weighting schemes |

The notebook requires a FRED API key. It must be run from the top after a fresh restart, as several cells reassign variables in place and are not idempotent on re-run.
