# BRRC — Break · Range · Retracement · Continuation

A TradingView **Pine Script v6 `strategy()`** implementing a futures continuation
setup as an explicit state machine:

```
HTF Bias → Break of Structure → Range → Retracement → Confirmation → Entry
```

Built as a working prototype of the BRRC concept: discretionary continuation
logic translated into objective, backtestable rules.

## Design decisions

### 1. No repainting, no look-ahead
- Higher-timeframe bias reads **only the last closed HTF bar** — the documented
  non-repainting form of `request.security` (`expr[1]` + `lookahead_on`).
- Swing structure uses `ta.pivothigh/pivotlow`, which confirm `pivotRight` bars
  after the extreme. Lagging by design: once a swing level is set, it never moves.
- Orders are processed on bar close; nothing is decided intrabar.

### 2. BOS = body close, never a wick
A Break of Structure requires the bar's **close** to cross the confirmed swing
level (`close > swingHigh` for longs, crossing-event form). A wick through the
level does not qualify and cannot fire a setup.

### 3. One setup, one state machine
Each setup lives in a typed object (`Setup`) with an explicit phase
(`Idle → Break → Range → Retrace`), its own invalidation rules (failed break =
body close back through the level; too-deep retracement; stale timeout) and a
hard reset on entry or invalidation. The pullback extreme is tracked from the
first pause bar, so a retracement that begins while the range is still forming
is not missed; confirmation is a close reclaiming the prior bar's extreme. A stale setup can never leak into a trade,
and phases cannot be skipped.

### 4. Execution stays out of Pine
Entries/exits emit a **JSON `alert_message`** meant for a webhook → broker
bridge (Tradovate, Binance, etc.). Order deduplication, position
synchronization and stop/target placement belong server-side — Pine only
signals. This prototype is built to feed such a bridge; it does not include one.

## Usage

1. Open TradingView → Pine Editor → paste `brrc.pine` → *Add to chart*.
2. Suggested demo context: **MES / MNQ, 5–15 min chart**, bias timeframe `60`.
3. The on-chart HUD shows the live setup state plus funnel counters
   (`BOS → Range → Retrace → Entries`, and how setups died) — the
   instrumentation used to debug the setup pipeline end to end.
4. Strategy Tester shows the backtest; all thresholds are inputs (pivot width,
   range pause bars, retracement zone measured over the breakout leg — from
   38.2% up to a full retest of the broken level —, ATR stop buffer,
   R-multiple target) and are starting points, not tuned values.

## Disclaimer

Educational prototype for evaluating setup logic. Not financial advice; futures
carry substantial risk. Backtest results do not guarantee future performance.
