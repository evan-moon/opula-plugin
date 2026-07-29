---
name: what-if
description: >
  Size a hypothetical market event against the user's actual portfolio with opula.
  Use when the user asks "내일 장 어떨 것 같아?", "다음 주 조심할 거 있어?", "FOMC 매파적으로 나오면?",
  "실적 발표 앞두고 얼마나 위험해?", "S&P 3% 빠지면 내 계좌 어떻게 돼?", "what if the Fed hikes",
  "how bad could tomorrow be", or names any short-horizon event and wants its impact.
---

# What-if scenario sizing

## Precondition

opula 툴(`setup_status` 등)을 호출할 수 없으면, 답변을 시도하지 말고 아래를 그대로 안내한 뒤
중단한다.

> 플러그인은 설치됐는데 opula 커넥터가 아직 연결되지 않았습니다.
> 설정 → 커넥터 → opula → 연결 을 누르시면 바로 동작합니다.

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

## A worked answer

Illustrative, with invented figures. Copy the shape, never the numbers.

> 다음 주 자체보다 이번 주 목요일까지가 진짜 고비예요.
>
> 지난 한 주에 포트폴리오가 -8.7%, -4,476만원 빠졌어요. 그중 TSLA 한 종목이 -4,133만원,
> 사실상 전부예요. 그 결과 비중이 43%에서 37.7%로 줄었는데, 좋아서 줄어든 게 아니라 맞아서
> 줄어든 거예요. 시장 자체는 아직 위험선호 쪽이에요. VIX 18.7, 장단기 금리차 정상,
> 하이일드 스프레드 2.81%로 타이트. 문제는 시장이 아니라 쏠림이에요.
>
> 다가오는 일정은 오늘 MSFT 실적(비중 14.8%), 내일 AAPL 실적(4.3%)과 미국 GDP예요.
> 다음 주 캘린더는 오히려 비어 있어요. 이번 주에 다 몰려 있습니다.
>
> 자산 5억 기준 평소 하루 출렁임은 ±1,125만원(일간 변동성 2.24%)이에요. 이걸 기준선으로 두고
> 보세요.
>
> ```
> TSLA 단독 -12%     ████████████████████  -2,274만원   평소 하루의 2.0배
> GDP 하방 서프라이즈  ██████████████        -1,568만원   평소 하루의 1.4배
> MSFT/AAPL 실적 실망  ████████              -910만원     평소 하루보다 작음
> ─ 평소 하루 출렁임 ─ ██████████            ±1,125만원
> GDP 호조            ███████████ (+)       +1,254만원
> ```
>
> 가정 공개: 주식 충격은 종목별 시장민감도(beta_5y)로 가중했어요. TSLA 1.8, NVDA 2.21,
> MSFT 1.13. SGOV는 초단기 국채라 0.02, 원자재 헤지는 주식과 무관해서 0으로 뒀어요.
> 충격 크기(-12%, -2.5% 등)는 제가 정한 가정이지 예측이 아니에요.
>
> 읽는 법이 중요해요. 실적 두 개보다 TSLA 하나가 두 배 이상 무서워요. MSFT는 비중이 14.8%나
> 되는데도 -7%면 -521만원인데, TSLA는 -12%면 혼자 -2,274만원이에요. 다음 주에 조심할 건
> 이벤트가 아니라 한 종목입니다.

읽을 점 넷. 답을 이벤트 목록이 아니라 "무엇이 제일 무서운가" 한 문장으로 닫는다. 모든 시나리오를
평소 하루 출렁임과 나란히 놓는다. 가정을 각주가 아니라 본문에서 밝히고 예측이 아니라고 못 박는다.
그리고 비중이 큰 것과 위험한 것이 다르다는 걸 두 숫자를 나란히 놓아 보여준다.

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
