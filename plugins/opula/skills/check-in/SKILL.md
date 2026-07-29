---
name: check-in
description: >
  Daily or weekly portfolio check-in through opula, led by what changed since the last visit.
  Use when the user asks "오늘 포트폴리오 어때?", "내 자산 어때?", "체크인", "오늘 시장 어때?",
  "뭐 달라진 거 있어?", "how's my portfolio", "anything I should know today", or opens a session
  with no specific question but clearly wants their account read.
---

# Portfolio check-in

The second visit reading identically to the first is what kills this product. A static portfolio
returns the same facts every time, so a state-first answer feels like the same wall every day.
Lead with the delta.

## Call `setup_status` once, before anything else

One call per conversation, on the first opula action. It returns this account's stored state, the
plan, the profile (birth year, retirement target, target net worth, risk tolerance, country of
residence) and `analyst_context`, the voice and behavioral contract every answer is written under.
Skip it and you answer in the wrong register, against assumptions this user never set. If it has
already run in this conversation, do not run it again.

## Call

`get_market_brief`. One call carries everything below. Do not fan out into `show_*` tools to
assemble what the brief already returned.

## Read `delta_since_last.material` first, then branch

**material true** - open on the change, in this order:

1. `portfolio_move` in the user's own money, then `moved_holdings` led by the biggest
   `portfolio_impact_pct`, then `rec_flips` (a holding whose verdict changed, highest signal
   there is, surface it with the action word), then `events_entered`.
2. `stance` plus the non-HOLD entries of `recommendations[]`, each led by its ACTION WORD.
   When nothing new is actionable, one line ("나머지는 그대로예요") beats re-dumping every holding.
3. One forward lead the user can execute now.

**material false** - do not manufacture a change narrative and do not dump the state wall. Say it
plainly in one line, give the single thing worth knowing (a shifted stance, or the nearest
catalyst), and close forward. A quiet delta is information.

On a first visit (`last_seen_at` null) close with the memory promise in one line: next time opens
on what changed. That promise is the reason to come back.

## Deliver the verdicts, do not soften them

`recommendations[]` is opula's rule-computed verdict per holding, not an opinion to hedge.
Turning a computed TRIM into "고려해보세요" is a failure, not caution. Lead with the action word,
the conviction, one thesis line, and the tripwire.

A BUY or ADD carrying a `capacity_guard` conflict holds two true facts at once: the name is worth
owning, and the household cannot fund it this month. Deliver both, name the specific reason the
guard gives, and point at the funding path when `funded_by_rec_ids` points at a TRIM.

A recommendation carrying `data_gaps` was not judged. Say "이건 판단을 못 했어요" plus the missing
input in plain words. Never let it pad the answer as if it were a considered HOLD.

## Before you call a number a finding

Read `false-positives.md` next to this file when the answer leans on a risk, concentration or
performance figure. It lists the nine cases where an opula number reads as a finding and is not,
with the field that discloses each one. The two that come up most: a null Sharpe means the history
is short, so say how many days remain rather than "데이터 부족", and a high HHI means little when
`hhi_coverage_pct` is low.

## Chain when the calendar demands it

`events_entered`, a high-impact economic event within 7 days, or a holding's earnings within 5 days
at over 5% weight all force a chain into `simulate_scenario`. Pull `beta_5y` from
`holdings[].fundamentals` and `daily_vol_pct` from `risk_summary`, then size the event in dollars.
"변동성이 있을 수 있어요" is not an answer when the numbers are one call away.

## A worked answer

Illustrative, with invented figures. Copy the shape, never the numbers.

> 지난번 보신 이후로 계좌가 -3.2%, -1,610만원 움직였어요. 거의 다 한 종목이에요. TSLA가
> -1,240만원(전체의 77%), NVDA가 -290만원이고 나머지는 잔물결이에요. TSLA는 어제 실적에서
> 매출은 맞췄는데 마진 가이던스를 낮춘 게 컸어요 (웹 기준).
>
> 새로 생긴 건 판정 하나예요. NVDA가 HOLD에서 TRIM으로 바뀌었어요 (확신 medium). 비중이
> 26.4%로 이 카테고리 한도 25%를 넘었거든요. 2회 분할로 계산돼 있고, 24% 아래로 내려오면
> 다시 HOLD예요.
>
> TSLA는 판정 그대로 TRIM이에요 (확신 high, 37.7% 대 한도 15%). 지난주에 말씀드린 것과
> 같은 이유라 다시 늘어놓지 않을게요. 대신 이번 주에 달라진 건 안 팔고 맞추는 길이 조금
> 열렸다는 거예요. 놀고 있는 현금 7,301만원을 다른 종목에 넣으면 32.9%까지 내려와요.
>
> 나머지는 그대로예요. 그리고 목요일에 MSFT 실적이 있는데 비중이 14.8%라 그냥 넘기기엔
> 커요. 그날 얼마나 흔들릴 수 있는지 숫자로 뽑아볼까요?

읽을 점 넷. 상태가 아니라 변화로 연다. 판정이 바뀐 것(`rec_flips`)을 액션 워드와 함께 가장
먼저 올린다. 안 바뀐 판정은 근거를 재탕하지 않고 "이번 주에 달라진 것"만 말한다. 그리고
실행 가능한 다음 한 걸음으로 닫는다. 메뉴가 아니라 하나다.

조용한 주간이면 이렇게 짧아진다.

> 지난번 이후 눈에 띄게 움직인 건 없어요. 계좌는 +0.4%, 판정도 그대로예요. 대신 다음 주
> 화요일에 CPI가 있어요. 그게 이번 달 남은 기간의 방향을 정할 가능성이 커요. 미리 사이즈를
> 재볼까요?

없는 변화를 지어내지 않는다. 조용한 것도 정보다.

## Examples

**"오늘 포트폴리오 어때?"**
→ `get_market_brief` → `delta_since_last.material` true → open on the move and the two names that
drove it, surface a HOLD→TRIM flip with the action word, close on one executable lead.

**"뭐 달라진 거 있어?" (quiet week)**
→ `delta_since_last.material` false → "지난번 이후 눈에 띄게 움직인 건 없어요" → the nearest
catalyst → one forward question.

**"오늘 시장 어때?" with FOMC in three days**
→ `get_market_brief` → the delta first → then chain `simulate_scenario` with hawkish and dovish
legs, sized against the portfolio's one-sigma daily range, assumptions disclosed.
