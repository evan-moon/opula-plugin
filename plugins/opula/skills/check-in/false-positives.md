# False positives

Nine ways an opula figure reads as a finding when it is not. Each one names the field that tells
you, because every one of these is disclosed in the payload rather than left to guesswork.

## 1. Concentration coverage

**Misread**: a high HHI means the whole portfolio is concentrated.
**Field**: `concentration.hhi_coverage_pct`. Below roughly 60% the index describes a minority of
the account. An `Unknown` row in `top` is missing sector or country metadata, not a real cluster.
**Say**: quote the coverage alongside the number, or scope the claim to what it covers.

## 2. Annualization

**Misread**: annualizing a short window to make it comparable.
**Field**: `annualization_supported`. When false, the period is under a year and the geometric
annualization would overstate it.
**Say**: answer with `total_pct` and name the actual window. Never annualize past the gate.

## 3. Sharpe and Sortino sample gate

**Misread**: a null ratio means the data is broken or absent.
**Field**: `risk_summary.sample_days` against `ratio_min_days`. Null means not enough history yet.
**Say**: how many days remain, as a number. "데이터 부족" is the phrasing to avoid. Answer the
underlying question with `total_return_pct` in the meantime.

## 4. Time-weighted against money-weighted

**Misread**: one of `gap_pct` and `gap_value` is wrong because they disagree.
**Field**: both. `gap_pct` isolates selection, `gap_value` includes the timing of deposits.
**Say**: the divergence is the story. A user who picked well but funded badly is a real and common
result. Do not use one to refute the other.

## 5. Book against live

**Misread**: `get_wealth_brief` and `get_market_brief` disagree, so one is stale.
**Field**: they measure different things. Wealth allocation is the month-end book. Market portfolio
is live, derived from the transaction log. Wealth never fuses the live value.
**Say**: name which basis you are quoting whenever both appear in one answer.

## 6. Cost basis method

**Misread**: opula's average price is wrong because the brokerage screen shows another.
**Field**: opula uses `weighted_average`. A brokerage on FIFO or specific-lot produces a different
average and a different unrealized number for the same position.
**Say**: disclose the method whenever the user is reconciling against a brokerage screen. Neither
number is an error.

## 7. Two price clocks

**Misread**: prices look stale because the timestamp is old.
**Field**: `prices_synced_at` is when opula fetched. `prices_as_of` is the exchange's last print.
**Say**: Friday's close showing on a Sunday is correct, not stale. Quote the clock that answers the
question the user actually asked.

## 8. Inception baseline

**Misread**: a contribution or performance window is year-to-date.
**Field**: `baseline_is_inception`. When true the window starts at the first record, not January.
**Say**: name the real start date rather than calling it YTD.

## 9. Manually priced assets

**Misread**: crypto, gold, real estate and other assets are quoted live.
**Field**: these come from `add_price` and carry whatever date they were last set. They are
deliberately excluded from `portfolio_snapshots`, which is why `risk_summary.coverage_pct` exists.
**Say**: flag the as-of date when one of these is material, and offer to refresh it.
