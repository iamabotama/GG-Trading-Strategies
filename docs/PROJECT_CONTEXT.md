# Project Context — GG Key Open Strategy

Read this first when picking up the project in a new chat.

## Chart environment

- TradingView, Pine Script **v6**
- Symbol: S&P 500 E-mini Futures (CME), primarily 5-minute chart
- User's chart time axis displays **Pacific time** (user is in Washington state)
- **Candle colors are non-default: RED = bullish, BLACK = bearish**
  (this has caused confusion before — do not assume green/red)

## The "Ten Line" (key open level) — settled behavior

1. Key candle = the **10:00 AM Eastern** 5-minute candle
   (appears as 07:00 on the user's Pacific axis).
2. Line level (as of v1.6):
   - **Bearish** key candle → **top of body** (= open)
   - **Bullish** key candle → **bottom of body** (= open)
   - Both cases equal the candle's OPEN price. Body edges only, never wicks.
3. Line color matches the candle: red bullish, black bearish.
4. One independent line per day. Days must never connect to each other.
5. Implemented via `request.security(…, "5", …)` + `plot(…, style_linebr)`,
   so it renders on any chart timeframe.

## History of corrections (avoid re-introducing)

- v1 used `keyBullish ? low : high` (wick-based) — wrong.
- Mid-project versions used close / bottom-of-body-bearish — wrong direction, flipped in v1.6.
- An earlier line.new() version drew a VERTICAL line when created with
  x1 == x2 plus extend.right (degenerate line). If using line objects,
  always create with `bar_index + 1` as x2.
- A box drawn at the key candle (high-to-low) was explicitly REMOVED — user does not want it.
- Lime/green colors clash with the user's scheme — use red/black.

## Known quirks in current v1.6 (accepted for now, not yet fixed)

- `ta.change(time("D"))` rolls at futures session start (18:00 ET), not midnight.
- `keyBullish` persists overnight until the next 10:00 candle, so pre-10:00
  color reflects the prior day's direction.

## Open / planned work (Section 4)

Position box based on the key open, discussed but NOT yet built:

- Direction: bearish key open → short setup; bullish → long setup.
- The **entry line** of the position tool sits exactly ON the ten line.
- Ratio: **6 : 1** (reward : risk). Long = reward above / risk below; short mirrored.
- UNRESOLVED: what defines 1 unit (the stop distance). Options discussed:
  fixed points, candle-derived, ATR-based, fixed dollar risk (~$750/trade
  appeared in the user's manual position tools with floating quantity).
- Pine cannot draw TradingView's native long/short position tool; will be
  replicated with boxes + lines, and/or real strategy orders.

## Working agreement

- Iterate on ONE complete script; deliver the FULL file every time.
- Bump the version in the indicator() title on every change.
- Scope changes down; make exactly the change requested, nothing extra.
- When the user reports something looks wrong, check the actual code/history
  before proposing fixes.
