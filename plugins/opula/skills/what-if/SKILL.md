---
name: what-if
description: >
  Size a hypothetical market event against the user's actual portfolio with opula.
  Use when the user asks "내일 장 어떨 것 같아?", "다음 주 조심할 거 있어?", "FOMC 매파적으로 나오면?",
  "실적 발표 앞두고 얼마나 위험해?", "S&P 3% 빠지면 내 계좌 어떻게 돼?", "what if the Fed hikes",
  "how bad could tomorrow be", or names any short-horizon event and wants its impact.
---

# What-if scenario sizing

A deterministic conditional, not a forecast. The caller supplies the shock, opula computes the
point-estimate impact per holding. Horizon is one day to one week. Anything measured in months or
years belongs in the `fire` skill instead.

## Phase 0: Call `setup_status` once, before anything else

One call per conversation, on the first opula action. It returns this account's stored state, the
plan, the profile (birth year, retirement target, target net worth, risk tolerance, country of
residence) and `analyst_context`, the voice and behavioral contract every answer is written under.
Skip it and you answer in the wrong register, against assumptions this user never set. If it has
already run in this conversation, do not run it again.

## Phase 1: Pull the real exposure

`get_market_brief` first, always. Two fields matter:

- `holdings[].fundamentals.beta_5y` → `beta_overrides`. Without an override a stock defaults to
  beta 1.0, which understates a high-beta name and overstates a defensive one.
- `risk_summary.daily_vol_pct` → `daily_vol_pct`. Without it `one_sigma_range_usd` comes back null
  and the scenario has nothing to be compared against.

## Phase 2: Frame two or three scenarios

`simulate_scenario` takes up to eight, but two or three is what a person can hold in their head.
Conventional shapes:

- directional triplet: bear / base / bull at `equity_pct` -0.02 / 0 / +0.02
- conditional pair: the event happens / it does not
- idiosyncratic single: one ticker via `per_ticker_pct`, for an earnings event

`per_ticker_pct` overrides `equity_pct` for that ticker. Use it when the event is about one name
rather than the market.

## Phase 3: Deliver against the one-sigma range

The dollar impact alone means nothing. "-$2,900" is frightening or trivial depending on what a
normal day looks like, and `one_sigma_range_usd` is what tells the user which. Always present the
scenario next to it: "평소 하루 출렁임의 두 배쯤 되는 사건이에요".

**Disclose every assumption.** The shock size, the betas you used, and any ticker override. The
number is only as good as the assumption, and an undisclosed assumption turns a conditional into a
prediction.

**Non-stock holdings default to beta 0**, meaning gold, real estate, crypto and other are untouched
by `equity_pct`. Say so when they are a meaningful share of the portfolio, and pass an override if
you want to assert a linkage.

## When this fires without being asked

A short-horizon question answered from the brief alone is a failure. These force the chain even
when the user never said "what if":

- tomorrow, next week, or a specific weekday
- a high-impact economic calendar event within 7 days
- a holding's earnings within 5 days at over 5% weight

## Examples

**"다음 주 FOMC 어떻게 봐야 해?"**
→ `get_market_brief` → betas and daily vol → `simulate_scenario` with hawkish (`equity_pct`
-0.03) and dovish (+0.02) legs → dollar impact each way, sized against the one-sigma daily range,
assumptions quoted.

**"내일 NVDA 실적인데 얼마나 위험해?"**
→ idiosyncratic: `per_ticker_pct: {"NVDA": -0.10}` and a +0.08 upside leg, plus the weight NVDA
carries → the answer is the dollar range on their account, not a view on the print.

**"S&P 10% 빠지면 나 어떻게 돼?"**
→ single scenario at `equity_pct` -0.10 with real betas → per-holding breakdown led by the largest
contributors, and the note that non-stock holdings sat this one out.
