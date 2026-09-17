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

## Known quirks (accepted, unchanged in v1.7)

- `ta.change(time("D"))` rolls at futures session start (18:00 ET), not midnight.
- `keyBullish` persists overnight until the next 10:00 candle, so pre-10:00
  color reflects the prior day's direction.

## Section 4 — Entry Triggers (settled in v1.7)

Directed by Brandon (collaborator). The script is now `strategy()` (was
`indicator()`), with `process_orders_on_close = true` so market entries fill at
the signal bar's close.

1. Signal = rejection of the ten line on a confirmed 5m candle (v1.9):
   - Short: `high >= line` and `close < line` — wick reached the line, closed
     on the rejected side. Includes break-and-fail candles that OPEN above.
   - Long: mirror.
   - "Strict Rejection" input (default OFF) additionally requires the candle
     to open on the origin side (the v1.7/v1.8 behavior, which produced far
     fewer signals and made whole sessions look empty).
   - REMINDER: the line dies at the 18:00 ET session roll (3:00 PM on the
     user's Pacific axis). Evening retests of the level never signal. Open
     question for Brandon: should the line stay tradeable past the roll?
2. Bias gate (input, default ON): shorts only on a bearish line, longs only on
   a bullish line.
3. Stop = far edge of the most recently CREATED unmitigated same-side rejection
   block (top of bearish block for shorts, bottom of bullish for longs).
   Price is SNAPSHOTTED at entry — the block object may be deleted mid-trade.
   v1.8: if no valid block exists (or its edge is on the wrong side of entry),
   the stop FALLS BACK to the rejection candle's own wick extreme (input,
   default ON). With fallback OFF, no block -> trade skipped.
   v1.8 also marks every raw line rejection with a tiny gray x (input, default
   ON) so skipped setups are visible across history — the v1.7 no-block rule
   was silently eating most historical trades.
4. Target = Reward:Risk input (default **5.0** — supersedes the earlier 6:1
   discussion) measured from the actual fill.
5. Max trades per day input (default 1); counter rolls with `time("D")`
   (18:00 ET session start, same quirk as the line).
6. Open trade is flattened at day roll (input, default ON).
7. Visuals kept: entry triangles + stop (orange) / target (teal) lines that
   extend while the trade is open.

### Known deliberate compromises (v1.7)

- Brandon described the WICK itself as the entry ("that wick would then enter
  the trade"). A resting limit at the line would also fill on candles that
  break THROUGH, so v1.7 enters at market on the close of the confirmed
  rejection candle instead. Entry price = signal close, not the line. Revisit
  if fills look too far from the line.
- Risk is measured entry->stop from the actual fill, so the 5:1 distance moves
  with the fill, not the line.
- Brandon's transcript said "red candle" for the short setup (red = bullish on
  this chart). Interpreted as: any candle that wicks in and closes back on its
  original side. Confirm with him if shorts should additionally require a
  down-closing candle.

## Active experiment: 1m vs 5m execution (Brandon, Sep 16)

Same v1.9 script loaded on a 1m chart vs a 5m chart; compare Strategy Tester
reports. No code change needed — the ten line is pinned to the 5m feed, all
other logic follows the chart timeframe. Caveats: on 1m, rejection blocks and
pivots become 1m structures (Swing Strength 3 = 3 min/side) so stop placement
shifts, and far more rejections fire against the daily trade cap. If 1m looks
promising, planned v2.0 = 1m execution with blocks still anchored to the 5m
feed for a cleaner apples-to-apples.

## Working agreement

- Iterate on ONE complete script; deliver the FULL file every time.
- Bump the version in the indicator() title on every change.
- Scope changes down; make exactly the change requested, nothing extra.
- When the user reports something looks wrong, check the actual code/history
  before proposing fixes.
