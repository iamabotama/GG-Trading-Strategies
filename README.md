# GG Trading Strategies

Pine Script (v6) indicators and strategies for TradingView, built iteratively with Claude.

## Current master script

**`indicators/gg-key-open-strategy.pine`** — v2.1

Components:
- **Ten Line** (v2.0) — pure interest point, no bias semantics. Level = the
  OPEN of the 10:00 AM ET 5-minute candle, drawn as one time-anchored line per
  day spanning 9:30 AM ET -> 12:30 AM ET by default (configurable), single
  color input. Works on any chart timeframe via `request.security` on 5m.
- **EMA Bias** (v2.0) — 9/20 EMAs on chart candles, plotted; the EMA stack
  gates trade direction (fast below slow = shorts only, fast above slow =
  longs only). Replaces the old line-color bias gate.
- **Rejection Blocks** — pivot-candle-anchored supply/demand boxes with
  toggleable direction and displacement filters.
- **Section 4** — entry triggers with simulated orders (v1.7). The script is now
  a `strategy()`: a rejection of the ten line (wick pierces, close back on the
  original side, confirmed 5m close) fires a market entry. Stop = most recent
  unmitigated same-side rejection block (far edge); target = Reward:Risk (default
  5:1) from entry. No valid block, no trade. Backtest via the Strategy Tester.

## Workflow

1. The `.pine` file in `indicators/` is the single source of truth.
2. Each change bumps the version number in the `indicator()` title.
3. Copy/paste the full file into TradingView's Pine Editor to load it.
4. New Claude chats: read `docs/PROJECT_CONTEXT.md` first, then the script.

## Docs

- `docs/PROJECT_CONTEXT.md` — full project state, decisions made, open questions.
  Start here when picking up the project fresh.
