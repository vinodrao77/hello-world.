# SPX Weekly Short Strangle Backtester — Strategy Specification

This document defines the **inputs**, **rules**, and **outputs** for a deterministic weekly SPX options backtester. It is intentionally implementation-agnostic and contains no code.

## Inputs

### Market Data
- **Underlying**: SPX (S&P 500 Index).
- **Option chain data** for weekly expirations, including at minimum:
  - Strike prices.
  - Bid/ask (or mid) prices for options.
  - Expiration date and time to expiration.
  - Option type (call/put).
- **Underlying price** at entry and through the trade lifecycle.

### Expected Move
- A deterministic **expected move** value for each trade entry date.
- The expected move is required to define the 70–80% expected move strike region.

### Backtest Configuration
- **Backtest date range** (start/end).
- **Entry timing** for weekly trades (e.g., specific day/time of week). The backtester must use a consistent, deterministic entry timing.
- **Position sizing** rules (e.g., number of contracts) if applicable.
- **Pricing convention** (e.g., mid price) for determining credits, profits, and losses.

## Strategy Rules

### Core Structure: Weekly Short Strangle
- Enter a **short strangle** on SPX options with the same weekly expiration.
- The position consists of:
  - One short call option.
  - One short put option.

### Strike Selection
- Both short strikes must be **outside 70–80% expected move**.
- The 70–80% expected move region is defined relative to the expected move value for that entry date.
- The specific strike selection method within that region must be deterministic (e.g., nearest available strike outside the band), but must not violate the 70–80% constraint.

### Hedging (Optional, Max One)
- The position may include **at most one broken-wing butterfly hedge**.
- No additional hedges are permitted.
- The hedge must be constructed and attached at entry using deterministic rules.

### Exits
- **Profit target:** exit the entire position at **65% profit** relative to the initial credit.
- **Loss limit:** exit the entire position at **1.5× the initial credit loss**.
- **No rolling:** positions are never rolled or adjusted.

### Trade Lifecycle
- If neither exit condition is met, the position is held until expiration and closed according to deterministic end-of-trade handling.

## Outputs

For each trade, the backtester must output at minimum:

- **Entry details**:
  - Date/time.
  - Selected strikes (call/put), expiration.
  - Initial credit.
  - Expected move used for strike selection.
  - Whether a broken-wing butterfly hedge was used.
- **Exit details**:
  - Date/time.
  - Exit reason (profit target, loss limit, expiration).
  - Exit value and realized P/L.
- **Performance metrics**:
  - Per-trade P/L.
  - Aggregate P/L.
  - Win/loss counts and win rate.
  - Max drawdown.
  - Expectancy (if defined by the backtester).

## Determinism Requirements
- All inputs and rules must be applied **deterministically** (no randomness).
- For ambiguous choices (e.g., which strike to pick within a band), the implementation must document and consistently apply a deterministic selection rule.
