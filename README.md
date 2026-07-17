# Brimstone

Brimstone is the umbrella for all of our in-house trading and market bots — everything that trades with company capital lives here, under one dashboard and one naming family (cave/mineral themed). Both live bots are built for paper trading first, with a manual, deliberate path to real capital.

Source code is private. This README describes what the system does and how it was built, not the implementation itself.

## Active bots

- **Bedrock** (formerly Pilot) — a day-trading bot for commodities, forex, and stocks (oil, gold, silver, wheat, corn, EUR/USD, GBP/USD, USD/JPY, and equities like AAPL/SPY). Runs a momentum + mean-reversion hybrid strategy, picking the strategy per asset class, and sizes positions with a fixed risk-per-trade and a daily loss cap. Named for its role as the steady, foundational bot in the family.
- **Pyrite** (formerly Phoenix) — an aggressive crypto scaler aimed at growing a small starting balance ($1–$50) into a much larger one on Coinbase. Reuses Bedrock's signal generation but layers on its own conviction scoring, Kelly-criterion position sizing, and a dedicated guardrails module before anything gets executed. Named for its high-risk, glitters-like-gold profile.
- **Unified Launcher** — a single Flask dashboard that starts, stops, and monitors Bedrock and Pyrite from one terminal instead of running them separately, with shared logging and session state.

## Sibling scaffolds under the Brimstone umbrella

These three now live inside `Brimstone/` alongside Bedrock and Pyrite, organized under the same family, though each runs independently:

- **Vein** (formerly TokenOps) — a multi-agent pump.fun toolkit: Geode (token launch), Quartz (independent signal trading), Onyx (guarded position monitoring), and Aquamarine (two-sided market making/liquidity). Scaffold stage — paper trading by default.
- **Echo** (formerly PolyCopy / PolymarketAPI) — copies trades from top Polymarket international traders onto our own Polymarket US account.
- **Reverb** (formerly phantom-copytrader) — multichain (Solana/EVM) wallet copy-trading, generalized from Echo's pipeline across chains.

## How Bedrock and Pyrite were built

- **Language/runtime:** Python.
- **Market data:** yfinance for Bedrock's commodities/forex/stock feeds.
- **Signals:** classic technical indicators — EMA (9/21/50), RSI (14), MACD (12/26/9), Bollinger Bands (20), ATR (14) — combined into momentum and mean-reversion signals, with the strategy choice mapped per asset class.
- **Risk management:** ATR-based stop loss/take profit (1.5x/3x ATR, a 2:1 reward-to-risk target), a fixed max risk per trade, a cap on simultaneous open positions, and a daily loss circuit breaker.
- **Pyrite's execution layer:** a conviction-scoring model on top of Bedrock's raw signals, Kelly-criterion sizing to scale bet size with edge, a guardrails module enforcing hard limits before any order goes out, and direct Coinbase execution.
- **Backtesting:** both bots have their own historical backtesting engines and their own SQLite databases, so backtests and live/paper runs never contend with each other.
- **Execution modes:** both default to paper trading. Going live is a manual, explicit step — Bedrock connects to Interactive Brokers (TWS/IB Gateway) once paper results are reviewed, and Pyrite requires flipping `paper_trading` off only after reviewing an internal audit of its sizing/risk logic.
- **Dashboard:** a single Flask server (`unified_launcher.py`) manages Bedrock and Pyrite as background threads, streams logs, and serves a local web dashboard for start/stop/status control.

## Status

Bedrock and Pyrite are actively developed and run in paper mode by default; live trading is opt-in per bot after review. Vein, Echo, and Reverb are independent scaffolds/products at varying stages of completeness.

## Disclaimer

This project is for educational and personal use. Algorithmic trading involves significant financial risk. Past performance does not guarantee future results — always paper trade and backtest before risking real capital.
