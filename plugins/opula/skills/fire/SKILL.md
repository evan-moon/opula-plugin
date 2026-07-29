---
name: fire
description: >
  Project net worth forward with opula to answer retirement and financial-independence questions.
  Use when the user asks "언제 은퇴할 수 있어?", "FIRE 언제 돼?", "목표 달성 확률", "10년 뒤 얼마야?",
  "지금 페이스면 몇 살에 가능해?", "when can I retire", "will I hit my number", or names a target
  amount and a horizon.
---

# FIRE and net-worth projection

A forward-looking question, so it is a thought experiment, not a diagnostic. opula supplies no
default assumptions. You choose them, and you disclose them verbatim.

## Call `setup_status` once, before anything else

One call per conversation, on the first opula action. It returns this account's stored state, the
plan, the profile (birth year, retirement target, target net worth, risk tolerance, country of
residence) and `analyst_context`, the voice and behavioral contract every answer is written under.
Skip it and you answer in the wrong register, against assumptions this user never set. It matters
most here: the retirement target and target net worth stored on the profile are the numbers this
whole question is asked against. If it has already run in this conversation, do not run it again.

## Gather the starting point first

`get_wealth_brief` gives `net_worth.total` for `initial_value` and `cash_flow.net_flow` or
`savings_rate` for `monthly_contribution`. Use the user's own numbers, never a round guess.

If the user wants a "지금 이대로" scenario, `get_market_brief` → `risk_summary` carries their
measured `annualized_return_pct` and `annualized_vol_pct`. That makes an honest historical leg to
contrast against a bear and a bull overlay.

## Call

`project_net_worth` with at least two scenarios, three is better. Every scenario needs an explicit
`expected_annual_return` and `annual_volatility`.

**All of them go in ONE call, in the `scenarios[]` array.** Splitting them across two calls is the
mistake the array exists to prevent: the seeded shock sequence is shared only within a single call,
so scenarios computed in separate calls differ partly by random noise rather than purely by the
assumptions you set, and the comparison you are about to present stops being a clean one. Call the
tool twice only when the runs genuinely differ in something other than a scenario, such as a
with-target versus without-target question or two different horizons.

- `horizon_months` takes anything from 1 to 720. A FIRE question is usually 120 to 360.
- `target_value` when the user named a number. That adds probability of reaching it and median
  months to reach. Omit it and you only get a distribution.
- **`seed` whenever you compare scenarios.** Without it every scenario draws fresh shocks and the
  differences between them are partly noise rather than the assumptions you set.
- `currency` matches what the user thinks in. All inputs and outputs use it.
- `life_events` for a known one-time flow (down payment, tuition, an RSU vest), keyed by
  `month_offset` from today.

## Deliver

**The fan is the finding.** The width between P10 and P90 is the answer to "how sure is this",
and a list of median endpoints deletes exactly that. Give the band, not just the midpoint.

**State every assumption in the same breath as the result.** "연 7% 수익, 변동성 15% 가정으로"
belongs next to the number, not in a footnote. A projection whose assumptions are hidden reads as
a forecast, which it is not.

**Name the biggest lever.** Contribution, horizon, or return. One of the three moves the
probability most, and saying which one turns a chart into a decision.

opula has no display surface and emits no chart markup. Deliver the percentile series as an inline
table. Offer the visualization widget only if the user asks for a chart.

## Examples

**"지금 페이스면 언제 은퇴 가능해?"**
→ `get_wealth_brief` for current net worth and monthly net flow → `project_net_worth` with bear
(5%/18%), base (7%/15%) and bull (9%/16%) legs, shared `seed`, 300 months → lead with the band at
the horizon, then the probability of hitting their number, then the lever.

**"10억 모으려면 몇 년 걸려?"**
→ same setup with `target_value` supplied → answer with `median_months_to_target` per scenario and
the probability spread across them, assumptions quoted.

**"집 사면 은퇴가 얼마나 늦어져?"**
→ two runs, or one run with `life_events: [{month_offset: 24, amount: -400000000, label: "집
계약금"}]` → the delta between the two fans is the answer to the question actually asked.
