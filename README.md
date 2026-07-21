# Brimstone

Brimstone is the umbrella for all of our in-house trading and market bots — everything that trades with company capital lives here, under one dashboard and one naming family (cave/mineral themed). Every bot is built for paper trading first, with a manual, deliberate path to real capital.

Source code is private. This README describes what the system does and how it was built, not the implementation itself.

## The five bots

All five are now managed from one launcher and share the same risk, conviction, logging, and ML framework.

| Bot | Market | Approach | State |
|---|---|---|---|
| **Bedrock** (formerly Pilot, formerly `Motherlode/`) | Commodities, forex, stocks | Momentum + mean-reversion hybrid, strategy chosen per asset class | Most mature by a wide margin — the bulk of the codebase |
| **Pyrite** (formerly Phoenix) | Crypto (Coinbase spot) | Bedrock's signals + conviction scoring, Kelly sizing, dedicated guardrails | Actively developed; long-only, shorts blocked in both paper and live |
| **Echo** | Polymarket event markets | Bedrock-style TA on each market's YES-token probability series | Working bot, paper mode |
| **Reverb** | On-chain crypto (Solana / EVM DEXs) | Bedrock's TA engine on on-chain OHLCV; pools trades with the other crypto bots for shared training history | Working bot, paper mode |
| **Vein** (formerly TokenOps) | pump.fun | Four agents: Geode (launch), Quartz (signal trading), Onyx (guarded position monitoring), Aquamarine (market making/liquidity) | Earliest stage; `DRY_RUN=true` everywhere |

**Correction to an earlier description of this project:** Echo and Reverb were
originally built as copy-trading bots (PolyCopy and phantom-copytrader) and were
described that way. They are not copy-traders anymore. Both were rebuilt as
**autonomous, signal-driven** bots that take their own positions from technical
analysis — Echo explicitly copies no trader, Reverb explicitly copies no wallet.

**Unified Launcher** — a single Flask server that starts, stops, and monitors all
five bots as background threads from one dashboard, with shared logging and session
state.

## How the bots were built

- **Language/runtime:** Python.
- **Market data:** yfinance for Bedrock's commodities/forex/stock feeds; on-chain OHLCV for Reverb; Polymarket odds series for Echo.
- **Signals:** classic technical indicators — EMA (9/21/50), RSI (14), MACD (12/26/9), Bollinger Bands (20), ATR (14) — combined into momentum and mean-reversion signals, with the strategy choice mapped per asset class. Bedrock's engine is the shared basis; Echo and Reverb apply it to non-traditional price series.
- **Risk management:** ATR-based stop loss/take profit (1.5x/3x ATR, a 2:1 reward-to-risk target), a fixed max risk per trade, a cap on simultaneous open positions, and a daily loss circuit breaker. A kill switch and a reconciliation job sit above all of it.
- **Pyrite's execution layer:** a conviction-scoring model on top of Bedrock's raw signals, Kelly-criterion sizing to scale bet size with edge, a guardrails module enforcing hard limits before any order goes out, and direct Coinbase execution.
- **Backtesting:** each bot has its own historical backtesting engine and its own SQLite database, so backtests and live/paper runs never contend with each other. Backtest fill economics deliberately mirror the live executor's assumptions (slippage and taker fees on both sides) so results aren't optimistic by construction.
- **Execution modes:** every bot defaults to paper. Going live is a manual, explicit, per-bot step. Bedrock's broker layer is configurable (`paper`, `oanda`, `ibkr`, `tradelocker`) and currently points at **OANDA practice**; Pyrite requires flipping `paper_trading` off only after reviewing an internal audit of its sizing and risk logic.
- **Secrets:** moved out of source into environment configuration; the dashboard is rate-limited.

## Status

No bot is trading real capital. Bedrock and Pyrite are actively developed and have
been through a MarkII audit pass (OANDA unit-sizing fix, kill switch, reconciliation
job, slippage modeling, macro-regime wiring). Echo and Reverb run but have no
meaningful live-equivalent track record yet. Vein is a scaffold.

## Next steps

1. **Produce a real paper track record.** The bots run, but there is no published
   multi-week paper-performance record to justify moving any of them toward live
   capital. This gates everything else.
2. **Finish the Bedrock → OANDA path.** The broker layer is configurable and set to
   OANDA practice; it needs sustained runtime against the practice endpoint before
   the live flag is worth discussing.
3. **Decide whether Vein continues.** It is the least developed and the most
   regulatorily exposed (pump.fun launches and market making are a different risk
   category from TA on liquid markets). Either invest in it or archive it.
4. **Consolidate the shared engine.** Bedrock's TA code is reused by Pyrite, Echo,
   and Reverb by copy rather than as a shared library, so fixes have to be applied
   in several places.
5. **Backtest parity checks across bots.** Bedrock and Pyrite have engines whose
   assumptions are aligned; Echo and Reverb need the same treatment before their
   results can be compared to the others.

## Disclaimer

This project is for educational and personal use. Algorithmic trading involves significant financial risk. Past performance does not guarantee future results — always paper trade and backtest before risking real capital.
