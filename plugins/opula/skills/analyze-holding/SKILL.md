---
name: analyze-holding
description: >
  Answer an opinion, thesis or deep-dive question about a specific ticker or theme using opula.
  Use when the user asks "MU 어때?", "NVDA 지금 사도 돼?", "이거 고평가야?", "메모리 사이클 죽었어?",
  "what do you think of TSLA", "is X overvalued", "should I be worried about my semi exposure",
  or asks for a report or a verdict on any listed name, whether or not they hold it.
  Applies to EVERY ticker question, including a follow-up naming a different ticker later in the
  same conversation ("TSLA는?", "그럼 AMD는?"). A second ticker is a second report, not a
  continuation of the previous one.
---

# Analyze a holding or theme

Turns a ticker opinion question into this user's answer rather than a generic stock take.
It does not size an entry or time one: that is `show_technical`, and only after the verdict.

## The one rule that matters

**Call `get_insight_report` first. Every ticker question, every time.**

This includes the second and third ticker in the same conversation. "TSLA는 어때?" following a
report on MU is a new question about a different company, and answering it from the drill-down
tools you happened to use last is the single most common way this goes wrong. Having already run
the chain once is a reason to reset to the report, not a reason to skip it.

Opening with a `show_valuation` / `show_financials` / `show_technical` chain is the failure this
skill exists to prevent. Those tools return numbers about the ticker. `get_insight_report` returns
which analysis frames actually fired on current data plus `portfolio_exposure`, the user's real
stake in the name through direct weight, ETF look-through and correlation proxies. That last part
is the only thing a general-purpose answer cannot produce, and it is what the user is paying for.

Pass 1 to 4 tickers. For a theme, pass its most representative tickers and put the question in
`topic`. Non-US symbols take a market prefix (`KRX:005930`).

## After the report

1. Lead with the fired models, each presented through its `binary_frame` pair. Models that did not
   fire are not findings, do not list them.
2. Follow a `drill_down` pointer only when a specific claim in your answer cannot stand without it.
   **Budget: 2 drill-down calls, and often the right number is zero.** The report already carries
   the evidence figures. Running `show_valuation`, `show_financials` and `show_technical` as a set
   is the improvised chain wearing a different hat, and it teaches the next turn to skip the report
   entirely. Pick the one weakest link in the argument and check that.
3. Read the multi-horizon return matrices (1M/3M/6M/1Y) stacked on top of each other. A single
   window is how a rotation gets mistaken for a trend.
4. Close on `portfolio_exposure`. The answer lands on the user's money or it did not land.

## Non-negotiables

- **Give a verdict.** "결정은 당신 몫" is not an answer. The one binding constraint is that any
  price level or judgment must come with the arithmetic behind it ("섹터 중앙값 P/E 22 × 선행 EPS
  $12.4 = $273"). If you cannot show the arithmetic, say the evidence is thin instead of producing
  a number.
- **Cite only numbers that appear in `evidence` or in another opula response**, with their `as_of`
  dates. Web figures are allowed beside opula figures, labelled as such, and never overwrite them.
- **Disclose every `data_gaps` entry that touches the argument.** A gap is a stated limit, never a
  silently skipped step.
- **Read the return matrices stacked, and mind two traps carried over from the brief.** opula's
  cost basis is `weighted_average`, so the average price will differ from a brokerage running FIFO
  or specific-lot: disclose the method whenever the user reconciles against their broker screen.
  And `portfolio_exposure` weights come from the live transaction log, while the month-end book in
  `get_wealth_brief` can legitimately differ, so name the basis if both appear.
- `narrative_models` have no computable discriminant here. Apply them from your own knowledge and
  research if they fit, and say the numbers behind them are unverified.

## Examples

**"MU 어때?"**
→ `get_insight_report(tickers: ["MU"])` → open on the fired models by their binary frames, follow
one or two `drill_down` pointers for the weakest link in the argument, close on how much of the
user's account actually rides on it.

**"내 반도체 익스포저 괜찮아?"**
→ a theme ask, not a single name. `get_insight_report(tickers: ["NVDA", "MU", "AVGO"], topic:
"is my semiconductor exposure over-concentrated")` → the report's `portfolio_exposure` already
carries ETF look-through, so a fund holding counts even when the user never bought the ticker.

**"NVDA 지금 사도 돼?"**
→ still starts here. The verdict comes from the report. Entry sizing and timing come after, from
`show_technical` and the brief's `size_hint`, and only once the verdict says the name is worth
owning.
