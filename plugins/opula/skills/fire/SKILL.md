---
name: fire
description: >
  Project net worth forward with opula to answer retirement and financial-independence questions.
  Use when the user asks "언제 은퇴할 수 있어?", "FIRE 언제 돼?", "목표 달성 확률", "10년 뒤 얼마야?",
  "지금 페이스면 몇 살에 가능해?", "when can I retire", "will I hit my number", or names a target
  amount and a horizon.
---

# FIRE and net-worth projection

## Precondition

opula 툴(`setup_status` 등)을 호출할 수 없으면, 답변을 시도하지 말고 아래를 그대로 안내한 뒤
중단한다.

> 플러그인은 설치됐는데 opula 커넥터가 아직 연결되지 않았습니다.
> 설정 → 커넥터 → opula → 연결 을 누르시면 바로 동작합니다.

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

## A worked answer

Illustrative, with invented figures. Copy the shape, never the numbers.

> 지금 순자산 5억 200만원, 최근 3개월 평균 순저축 월 310만원으로 돌렸어요. 목표는 프로필에
> 적어두신 15억이고요. 20년(240개월) 뒤 이렇게 나와요.
>
> | 시나리오 | 가정 (수익률/변동성) | P10 | 중앙값 | P90 | 15억 도달 확률 |
> |---|---|---|---|---|---|
> | 보수 | 5% / 18% | 9.8억 | 15.1억 | 24.2억 | 51% |
> | 기본 | 7% / 15% | 13.1억 | 19.4억 | 29.0억 | 78% |
> | 낙관 | 9% / 16% | 16.8억 | 25.7억 | 40.1억 | 91% |
>
> 가정은 제가 정한 거예요. opula는 기본값을 안 갖고 있어서 셋 다 제가 넣은 숫자고, 같은 시드를
> 써서 세 시나리오가 같은 충격 순서를 공유해요. 그래서 표의 차이는 난수가 아니라 수익률과
> 변동성 차이예요.
>
> 밴드를 보셔야 해요. 기본 시나리오 중앙값은 19.4억인데, 밑으로 13.1억까지 열려 있어요.
> "20년 뒤 19억"이 아니라 "13억에서 29억 사이 어딘가, 중간이 19억"이 정확한 문장이에요.
>
> 가장 큰 레버는 기여금이에요. 월 310만원을 400만원으로 올리면 기본 시나리오 도달 확률이
> 78%에서 89%로 올라가요. 같은 폭을 수익률로 만들려면 연 7%를 9%로 올려야 하는데, 그건
> 동욱님이 통제할 수 있는 변수가 아니에요.

읽을 점 넷. 시작값과 기여금의 출처를 먼저 밝힌다. 가정을 표 바로 옆에서 공개하고 opula가 기본값을
안 준다는 것도 말한다. 중앙값 하나가 아니라 밴드가 답이라고 명시한다. 그리고 레버 하나를 골라
통제 가능한 것과 아닌 것을 갈라준다.

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
