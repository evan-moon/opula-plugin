---
name: import-ledger
description: >
  Import existing holdings or transaction history into opula from a paste, a CSV, a brokerage
  export or a screenshot. Use when the user says "거래내역 넣어줘", "내 보유종목 등록해줘",
  "증권사에서 뽑은 거 올릴게", "이거 좀 넣어줘" with a table or image attached, "import my
  portfolio", "here's my CSV", or is starting out and needs their existing positions recorded.
---

# Import an existing ledger

## Precondition

opula 툴(`setup_status` 등)을 호출할 수 없으면, 답변을 시도하지 말고 아래를 그대로 안내한 뒤
중단한다.

> 플러그인은 설치됐는데 opula 커넥터가 아직 연결되지 않았습니다.
> 설정 → 커넥터 → opula → 연결 을 누르시면 바로 동작합니다.

The single biggest drop-off in opula is between connecting and having any data. Everything the
product does reads from the transaction log, so this is the step that decides whether the account
is worth opening again.

Anything the user can put in front of you works: a pasted table, a CSV, a brokerage export, a
screenshot of a holdings screen, or a spoken list. Read it, do not ask them to reformat it.

## Phase 0: Call `setup_status` once, before anything else

One call per conversation, on the first opula action. It returns this account's stored state, the
plan, the profile (birth year, retirement target, target net worth, risk tolerance, country of
residence) and `analyst_context`, the voice and behavioral contract every answer is written under.
Skip it and you answer in the wrong register, against assumptions this user never set. Here it also
tells you whether this is a first run, which changes whether the import is an onboarding moment or
an addition to an existing book. If it has already run in this conversation, do not run it again.

## Phase 1: Parse and show your work

Extract ticker, date, type, shares and price. Then **show the user your detected column mapping
and the total row count, and wait for confirmation before writing anything.** A wrong mapping
applied to 200 rows is far more expensive to undo than one round trip.

For a screenshot, list what you read back as a table. Image extraction is where a misread digit
becomes a permanent cost basis.

## Phase 2: Get the shape right per row

**Non-US listings** set `market` to the listing exchange and keep `ticker` as the bare local code:
`{market: "KRX", ticker: "005930"}`, not "005930.KS". Price is in the market's native currency,
determined by `market`, and passing a `currency` that disagrees is rejected.

**Non-stock assets** set `asset_type` to crypto, commodity, real_estate or other. There is no
price provider for these, so `market` is ignored, `ticker` is a free-form name ("Bitcoin", "금
현물", "Seoul apartment"), and `price` is in the `currency` param. Keep them fresh afterwards with
`add_price`.

**Types**: buy and sell move shares. deposit adds shares, with `price: 0` for a grant or transfer
or the actual basis if known. dividend and tax are cash events, so `price` is the total amount and
`shares` is 1.

**Currency**: ask which currency the figures are in before writing. opula converts to USD at each
transaction's own date using historical FX. A past-dated row with no cached rate fails and asks
for `sync fx` rather than silently converting at today's rate.

## Phase 3: Write once

One `add_txn` call with all rows in `transactions[]`. Rows are sorted by date ascending before
insertion because average cost depends on order, so you do not need to pre-sort.

Do not loop one call per row. A 200-row import is one call.

## Phase 4: The reason field, and when to skip it

`reason` is the highest-leverage thing you can capture on a new trade, because it is capturable
only at entry time and it is what `show_thesis_track` scores the outcome against later. One short
sentence carries it.

**This does not apply to a historical import.** Migrating past data is exactly the case where
asking "why did you buy this in 2023" wastes the user's time for a thesis record that would be
reconstructed rather than remembered. Skip it here. Capture it on the next live trade instead.
It also never applies to deposit, dividend or tax rows: no investment decision was made.

## Phase 5: Land it

Call `get_market_brief` right after. The import is abstract until the user sees their own portfolio
come back with real prices, weights and a stance. That is the moment the account becomes theirs.

## A worked confirmation

Phase 1 ends here, before any write. Illustrative.

> 3건 읽었어요. 이렇게 매핑했는데 맞는지 봐주세요.
>
> | # | 종목 | 시장 | 티커 | 날짜 | 구분 | 수량 | 단가 |
> |---|---|---|---|---|---|---|---|
> | 1 | 애플 | US | AAPL | 2026-03-14 | 매수 | 10주 | $172.50 |
> | 2 | 엔비디아 | US | NVDA | 2026-04-02 | 매수 | 5주 | $118.20 |
> | 3 | 삼성전자 | KRX | 005930 | 2026-05-20 | 매수 | 20주 | ₩71,300 |
>
> 두 가지만 짚을게요.
>
> - 삼성전자는 원화로 들어가요. 한국 상장 종목이라 단가가 자동으로 KRW 기준이고, 2026-05-20
>   당시 환율로 달러 환산돼서 저장돼요. "달러 기준"이라고 하셨지만 이 줄만 원화로 적혀 있어서
>   그대로 읽었어요.
> - 매수 사유는 안 물어볼게요. 과거 내역 정리라 지금 와서 "왜 샀었지"를 재구성하는 건 시간
>   낭비예요. 사유 기록은 다음 실시간 매매 때 남기시면 돼요.
>
> 위 3건 이대로 기록할까요?

읽을 점 넷. 행 수를 먼저 말해서 누락을 사용자가 잡을 수 있게 한다. 티커를 사용자가 쓴 이름이
아니라 저장될 형태(`005930`, `KRX`)로 보여준다. 사용자 말과 데이터가 어긋나는 지점("달러
기준"이라 했는데 원화 줄이 있음)을 덮지 않고 짚는다. 그리고 안 하는 것(사유 질문)도 왜 안 하는지
말한다. 침묵한 생략은 사용자가 검증할 수 없다.

## Examples

**A pasted brokerage table**
→ read it, show "8 rows detected, columns mapped as 종목/매수일/수량/단가, currency KRW" → confirm
→ one `add_txn` with all 8, KRX rows carrying `market: "KRX"` → `get_market_brief`.

**A screenshot of a holdings screen**
→ transcribe into a table and show it back for verification → note that a holdings screen carries
current positions, not transaction history, so dates and cost basis may need asking → write as
buys at the stated average price.

**"비트코인이랑 금도 있어"**
→ `asset_type: "crypto"` with `ticker: "Bitcoin"`, `asset_type: "commodity"` with `ticker: "금
현물"`, priced in the user's currency → mention that these have no price feed and will need
`add_price` to stay current.
