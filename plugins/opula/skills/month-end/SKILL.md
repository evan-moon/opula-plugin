---
name: month-end
description: >
  Settle a month in the opula ledger: record every balance and cash-flow category for the period
  in one batch, then confirm what changed. Use when the user says "이번 달 결산", "월말 정리",
  "가계부 마감", "이번 달 자산 기록해줘", "月末", "close out the month", "record my month-end", or
  supplies a month's balances and spending to be written down.
---

# Month-end settlement

## Precondition

opula 툴(`setup_status` 등)을 호출할 수 없으면, 답변을 시도하지 말고 아래를 그대로 안내한 뒤
중단한다.

> 플러그인은 설치됐는데 opula 커넥터가 아직 연결되지 않았습니다.
> 설정 → 커넥터 → opula → 연결 을 누르시면 바로 동작합니다.

The month-end record is the spine of the defense side of opula. Net worth trend, savings rate,
cash-flow attribution and the settlement streak all read from it. A partially recorded month is
worse than an unrecorded one, because the figures derived from it look complete and are not.

## Phase 0: Call `setup_status` once, before anything else

One call per conversation, on the first opula action. It returns this account's stored state, the
plan, the profile (birth year, retirement target, target net worth, risk tolerance, country of
residence) and `analyst_context`, the voice and behavioral contract every answer is written under.
Skip it and you answer in the wrong register, against assumptions this user never set. If it has
already run in this conversation, do not run it again.

## Phase 1: Know what a complete period requires

Call `show_categories`. It returns the canonical catalog of every valid
`(type, sub_type, category)` combination with Korean labels. A pair outside it is rejected on
write, so never invent one.

The catalog exists because certain categories are routinely forgotten when a month is recorded
from memory: 연금, 주택청약, 예수금, 달러 현금, 차량, 배당·이자, 보험·통신·주거비. Walk it, do not
work from what the user happens to mention.

## Phase 2: Diff against the last settled period

Call `show_balance` and `show_flow` for **both** the previous period and the period you are
settling. Two reads each, four calls. The previous period tells you which categories this user
actually keeps; the current one tells you which of them are already filled, because a period is
often settled in pieces or re-run to fix a partial record. Skip the current period and you will
ask the user to re-enter rows they already gave you.

Whatever had a row last month and has no value this month is a question to ask, not a zero to
assume. A category that genuinely went to
zero and a category the user forgot look identical in the data, and only one of them is correct.

Ask about the gaps in one batch. Do not interrogate category by category.

## Phase 3: Record stocks too

Wealth net worth is book-based. At settlement, record holdings value under `domestic_stock` and
`overseas_stock` alongside cash, savings, usd_cash, deposits, real estate, vehicle, pension and
physical assets. The whole ledger.

This does not double-count. The book ledger is separate from the live intraday portfolio value in
`get_market_brief`, which comes from the transaction log, and wealth never fuses the live value.
The two can legitimately differ. Record `usd_cash` with `sub_type: "cash"` so it counts as liquid.

## Phase 4: Write it in one call

`add_monthly` with `period` (YYYY-MM), `date` (usually month-end), and the full `balance[]` and
`flow[]` arrays. One call writes the entire period, both sides.

Ask which currency each figure is in before writing. opula converts to USD at the entry date using
historical FX, so a value entered in the wrong currency is a silently wrong cost basis.

## Phase 5: Confirm

The `add_monthly` response carries `consistency`, so the streak is already in hand. Then call
`get_wealth_brief` and report what actually moved: net worth against last month, the `attribution`
split between what they saved and what the market did, savings rate, and runway.

Close on the streak when it is worth closing on. "N개월 연속" is the habit signal, and a settlement
that lands with no acknowledgement is a chore rather than a ritual.

## A worked question

Phase 2 ends in one batched question, never a category-by-category interrogation. Illustrative,
with invented figures.

> 지난달(2026-06) 결산 내역을 확인했어요. 이번 달(2026-07) 결산을 위해 아래 숫자들을 알려주시면
> 한 번에 기록할게요. 지난달 값을 참고로 같이 적었으니, 변동이 없으면 "동일"이라고만 하셔도 돼요.
>
> **자산 (7/31 기준 잔액)**
> 현금 $87,992 / 달러현금 $1,295 / 주택청약 $4,127 / 국내주식 $4,713 / 해외주식 $282,197 /
> 연금 $7,424 / 금현물 $15,505 / 전세보증금 $65,084 / 차량 $44,068
> 증권사 예수금: 지난달엔 기록이 없었어요. 대기 중인 현금이 있으면 알려주세요 (없으면 넘어갈게요)
>
> **부채**
> 신용카드 미결제 $6,684. 이번 달 카드값은요? 대출은 계속 없는 걸로 알고 있어요, 맞나요?
>
> **현금흐름 (7월 한 달)**
> 수입: 급여, 사업소득, 배당, 이자, 기타. 지난달은 결혼 축의금 같은 일회성이 컸는데 이번 달도
> 특이 수입 있었나요?
> 지출: 식비, 여행, 기타생활비, 보험, 통신, 관리비, 월세, 경조사. 카드값 총액으로 주셔도 되고
> 항목별로 나눠주셔도 좋아요.
>
> **통화**
> 각 금액이 원화인지 달러인지도 같이 알려주세요. 환율 오차가 생기지 않게 확인차 여쭤봐요.

읽을 점 넷. 카탈로그를 걸으면서도 질문은 한 덩어리다. 전월 값을 참고로 붙여 사용자가 "동일"만
쳐도 되게 만든다. 지난달에 없던 항목은 빠뜨린 게 아니라 "없으면 넘어간다"고 명시해 부담을
없앤다. 그리고 쓰기 전에 통화를 반드시 확인한다.

## Examples

**"이번 달 결산하자"**
→ `show_categories` → `show_balance` and `show_flow` for last month → one batched question about
the gaps → `add_monthly` with the full period → `get_wealth_brief` → net worth delta split into
agency and market, then the streak.

**"7월 카드값 320만원 나왔어" (mid-settlement)**
→ that is one flow row, not a settlement. Confirm whether they want the whole month closed. If yes
run the full sequence. If not, `add_flow` alone and say the period is still open.

**"지난달 결산 빼먹었어"**
→ same sequence with the older `period`. `add_monthly` is an upsert, so re-running a period is
safe and fixes a partial record in place.
