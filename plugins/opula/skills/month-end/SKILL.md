---
name: month-end
description: >
  Settle a month in the opula ledger: record every balance and cash-flow category for the period
  in one batch, then confirm what changed. Use when the user says "이번 달 결산", "월말 정리",
  "가계부 마감", "이번 달 자산 기록해줘", "月末", "close out the month", "record my month-end", or
  supplies a month's balances and spending to be written down.
---

# Month-end settlement

The month-end record is the spine of the defense side of opula. Net worth trend, savings rate,
cash-flow attribution and the settlement streak all read from it. A partially recorded month is
worse than an unrecorded one, because the figures derived from it look complete and are not.

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
