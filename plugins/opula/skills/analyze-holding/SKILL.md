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

## Call `setup_status` once, before anything else

One call per conversation, on the first opula action. It returns this account's stored state, the
plan, the profile (birth year, retirement target, target net worth, risk tolerance, country of
residence) and `analyst_context`, the voice and behavioral contract every answer is written under.
Skip it and you answer in the wrong register, against assumptions this user never set. If it has
already run in this conversation, do not run it again.

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

## A worked answer

Illustrative, with invented figures. Copy the shape, never the numbers.

> MU 관련해서 뽑아봤어요 (기준일 2026-07-29). 결론부터 말씀드리면, 이번 하락은 사업이 망가진
> 신호가 아니라 기대치만 식은 신호예요.
>
> 가장 눈에 띄는 건 이거예요. 지난 분기 컨센서스 EPS를 21% 서프라이즈로 이겼는데, 그 분기
> 마감 이후 주가는 오히려 -15.5% 빠졌어요. "그럼 실적이 나빠지고 있는 거 아니야?"라고 물으실
> 수 있는데, 최근 30일 EPS 전망 수정을 보면 상향 25건, 하향 0건이에요. 애널리스트들은 눈높이를
> 계속 올리는데 주가만 식은 거죠. 사업이 재평가된 게 아니라, 지난 1년 +632%라는 기대치 위에서
> 흥분이 가라앉은 거예요.
>
> "혹시 버블 붕괴 전조 아니야?"도 확인해봤어요. 버블이 터질 때 나오는 지문은 보통 재고가
> 쌓이는 거예요. 만들어놨는데 안 팔리는 상태요. 근데 재고소진일수는 1년 전 67일에서 지금
> 18.8일로 72% 줄었어요. 욕조에 물 채우는 속도보다 빠지는 속도가 훨씬 빠른 셈이라, 안 팔려서
> 쌓이는 그림이 아니라 만드는 족족 나가는 그림이에요. 다만 이건 재고 하나만 보고 판단한 거라
> 수주잔고나 주문 취소는 못 봤어요.
>
> 밸류에이션은 트레일링 P/E 18.6배인데 같은 반도체 그룹 중위값이 60배예요. 하위 20퍼센타일이죠.
> 다만 비교 대상이 팹리스·파운드리라 마진 구조가 달라서, "60배까지 가야 한다"는 식의 목표가는
> 만들지 않을게요. 근거 있는 숫자를 뽑으려면 show_valuation으로 더 파야 해요.
>
> 한 달만 보면 -28%로 섹터(-7.7%)보다 훨씬 나쁜데, 3개월 +58%, 6개월 +88%, 1년 +632%로 쌓아
> 보면 섹터(+7.5%/+14.7%/+29.6%)를 압도해온 종목이 숨 고르는 그림에 가까워요.
>
> 동욱님 계좌 기준으로는, MU를 직접 들고 계시진 않아요. VTV 안에 4.85%로 들어있고 VTV가
> 포트폴리오의 0.65%라 실질 노출은 0.03%예요. 사실상 없다고 보셔도 돼요.
>
> 판정은 "밸류에이션 조정이지 펀더멘털 붕괴가 아니다"예요 (확신 medium). 뒤집는 조건 하나,
> 다음 분기 EPS 수정이 하향으로 돌아서면 지금 논리가 깨져요.

읽을 점 다섯 가지. 판정을 먼저 놓고 근거가 뒤따른다. 소크라테스식 자문("~아니야?")으로 독자의
반론을 먼저 말한 뒤 데이터로 답한다. 물리적 비유(욕조)가 숫자를 그림으로 만든다. 산식을 못
대는 목표가는 내지 않고 왜 못 내는지를 말한다. 그리고 마지막이 항상 사용자 계좌다.

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
