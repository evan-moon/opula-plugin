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

## Chain when the calendar demands it

`events_entered`, a high-impact economic event within 7 days, or a holding's earnings within 5 days
at over 5% weight all force a chain into `simulate_scenario`. Pull `beta_5y` from
`holdings[].fundamentals` and `daily_vol_pct` from `risk_summary`, then size the event in dollars.
"변동성이 있을 수 있어요" is not an answer when the numbers are one call away.

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
