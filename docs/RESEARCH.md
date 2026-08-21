# Building TradingPortal: a stack survey for Indian options & algo trading

*Research pass across GitHub, Hugging Face, and trader communities — August 2026*

## Where this lands

Nobody needs to build a multi-broker order-routing layer from scratch anymore. The strongest open-source project for exactly what you described — options chain analytics, algo strategy execution, and unified connectivity to Zerodha, Upstox, IIFL, Angel One, Fyers and more — already exists and is actively maintained: **[OpenAlgo](https://github.com/marketcalls/openalgo)**. The realistic path for TradingPortal is to fork or build on top of it rather than re-implement broker adapters, options Greeks, and order management independently. The sections below lay out why, plus the pieces worth pulling in around it.

## Multi-broker connectivity

| Project | Brokers covered | What it actually gives you | License / activity |
|---|---|---|---|
| **[OpenAlgo](https://github.com/marketcalls/openalgo)** | 34+ brokers incl. Zerodha, Upstox, Angel One, Fyers, IIFL, Motilal Oswal, 5paisa, AliceBlue, Kotak Neo, HDFC Sky | Self-hosted platform, not just an adapter — normalized REST API (`/api/v1/`), visual strategy builder, Python strategy hosting, WebSocket streaming, TradingView webhook bridge, ₹1 Cr sandbox for paper trading | Actively developed, large contributor base, [docs](https://docs.openalgo.in) |
| **[Fenix](https://github.com/TheHardeep/fenix)** | 15 brokers incl. AliceBlue, Angel One, Dhan, Shoonya, 5paisa, Fyers, Groww, IIFL, Kotak Neo, Upstox, Zerodha | Thin, code-first unified Python API — swap broker by changing one class name, built-in paper-trading matching engine, rate limiting, secret redaction from logs | Lighter weight than OpenAlgo, better fit if you want a library rather than a hosted app |
| **[OpenBull](https://github.com/marketcalls/openbull)** | Upstox, Zerodha, Angel One, Dhan, Fyers | Options-first sibling to OpenAlgo — 30 multi-leg strategy templates, sandbox order-lifecycle simulation, unified WebSocket feed across brokers | Same author/ecosystem as OpenAlgo, FastAPI + PostgreSQL stack |
| Individual broker SDKs | One broker each | `pykiteconnect` (Zerodha), Upstox Python SDK, `SmartApi-python` (Angel One), `fyers-apiv3`, IIFL's XTS API | Use these only if you outgrow the unified layer for one specific broker's edge-case feature |

**Read:** if TradingPortal's core job is "one dashboard, many brokers, options-heavy," OpenAlgo is the closer match — it's already a full platform with a React frontend, not a library you assemble a UI around. If you'd rather keep full control of your own UI and just need broker plumbing, Fenix is the leaner dependency.

## Options chain & analytics

OpenAlgo and OpenBull both ship real options tooling out of the box — option chain views, live Greeks, IV smile/surface, max pain, OI tracking, GEX dashboards, straddle P&L simulators. That covers most of what a retail options desk needs without writing pricing math yourself.

If you want something narrower to study or embed standalone:

- [VarunS2002/Python-NSE-Option-Chain-Analyzer](https://github.com/VarunS2002/Python-NSE-Option-Chain-Analyzer) — pulls the NSE site's live option chain and renders trend indicators; good reference for how to scrape/parse NSE's option chain JSON correctly.
- [7715Karan/NSE-option_chain](https://github.com/7715Karan/NSE-option_chain) — simpler fetch-and-visualize script, useful as a minimal example.
- [sumitsainidev/OIAnalysis](https://github.com/sumitsainidev/OIAnalysis) — OI-focused analysis with Excel output, if you want a lightweight reporting path alongside the main app.

For the underlying math (Black-Scholes Greeks, IV solving) if you ever need it outside OpenAlgo/OpenBull's built-ins, `py_vollib` is the standard lightweight Python library traders reach for.

## Backtesting & strategy engines

None of the India-specific broker platforms above are primarily backtesting engines — they're built for live/paper execution. For strategy research and backtesting, pair them with a general-purpose engine:

- **vectorbt** — vectorized, very fast for parameter sweeps across large option/stock universes; steeper learning curve.
- **backtrader** — event-driven, easier mental model, huge community, slower on large sweeps.
- Both integrate cleanly with pandas data pulled from NSE or your broker's historical API, so you can backtest before wiring a strategy into OpenAlgo's live execution.

## Historical & reference market data

- **[jugaad-data](https://github.com/jugaad-py/jugaad-data)** — actively maintained, pulls live and historical NSE data (equities, indices, derivatives) without needing a broker key. Good for backtesting datasets and building your own indicator library.
- **nsepy** — older, less actively maintained now; jugaad-data has mostly superseded it, but it still turns up in a lot of tutorials.

## Where Hugging Face actually fits

Be honest with yourself about scope here: none of the trading-specific "stock prediction" models on Hugging Face (e.g. random community uploads like `stock-price-prediction` repos) are production-grade — they're mostly demo/coursework artifacts trained on tiny, stale datasets, and one of the more popular ones is literally flagged in its own discussion thread as not actually doing future prediction. Don't build execution logic on top of them.

The two genuinely useful categories:

- **[ProsusAI/finbert](https://huggingface.co/ProsusAI/finbert)** — a real, widely-cited BERT model fine-tuned for financial sentiment (positive/negative/neutral) on analyst reports and news. Reasonable as one signal feeding a news-sentiment overlay on top of your technical strategy — not as a standalone trading signal.
- General-purpose time-series forecasting models (e.g. Amazon's Chronos family) — worth exploring for volatility or price forecasting research, but treat outputs as a research input, not an order-generation signal, given how thin financial time series are relative to what these models are trained on.

If AI/ML is a "nice to have later" rather than day-one scope for TradingPortal, it's worth deferring — the connectivity, options analytics, and execution reliability layers below matter far more to a working system than a sentiment model.

## What the community actually says

Direct India-specific algo-trading threads are thinner on Reddit than GitHub stars would suggest — most serious discussion happens in project READMEs, Discord/Telegram communities tied to specific tools (OpenAlgo has an active one), and trading-education blogs like Marketcalls (which is the OpenAlgo author's own site and a good source for setup walkthroughs) rather than r/algotrading, which skews toward US/crypto brokers. Two practical takeaways that recur across both GitHub issues and broader quant-trading discussion:

1. Multi-broker "unified API" layers save real time but hide broker-specific quirks (margin calc differences, order-type support gaps, rate limits) — expect to still read each broker's raw API docs for anything beyond basic market/limit orders.
2. Paper-trade extensively before going live — both OpenAlgo and OpenBull ship sandbox/simulated execution specifically because retail algo failures are disproportionately caused by untested strategy code hitting real capital, not bad strategy ideas.

## A regulatory point that affects the architecture, not just the strategy

SEBI's retail algo-trading framework became mandatory for all brokers on **April 1, 2026** — already in force as of today. It matters for how you build TradingPortal, not just how you trade with it:

- If TradingPortal places more than **10 orders/second** through the API, it crosses into formal algo registration territory — each strategy needs an exchange-assigned Algo-ID tagged on every order.
- Brokers are now accountable for every algo running through their platform, so **static IP whitelisting** in your broker's developer console is a hard prerequisite for API trading, not an optional hardening step — worth building into TradingPortal's setup flow from day one.
- If you ever expose TradingPortal's strategies to other users (not just yourself), the white-box/black-box distinction matters: undisclosed ("black box") strategies legally require the provider to hold a SEBI Research Analyst license. Keeping strategy logic transparent to your own users sidesteps that entirely.

Worth designing the broker-connection settings screen around this from the start (IP whitelist status, Algo-ID field) rather than retrofitting it later.

## Suggested shape for TradingPortal

Given everything above, a pragmatic build order:

1. **Fork/vendor OpenAlgo as the execution and broker-connectivity core** rather than writing broker adapters from scratch — it already has Zerodha, Upstox, Angel One, Fyers, and IIFL wired up, plus the options analytics suite you asked for.
2. **Layer jugaad-data in for historical datasets** to backtest strategies before they ever touch OpenAlgo's live order path.
3. **Backtest with vectorbt or backtrader** against that historical data, iterate offline, and only promote a strategy to OpenAlgo's Python strategy runner once it's validated in paper mode.
4. **Treat FinBERT (or similar) as an optional signal input later**, not a load-bearing part of the initial system.
5. **Bake SEBI's Algo-ID and IP-whitelist requirements into the broker-connection UI** from the first version, since retrofitting compliance fields is more painful than including them at setup.

If you'd like, I can start by pulling OpenAlgo into the `TradingPortal` repo as a base and scaffolding the pieces above — say the word and I'll get going on that in your `Docker\GIT\TradingPortal` folder.

## Sources

- [OpenAlgo — GitHub](https://github.com/marketcalls/openalgo)
- [OpenAlgo — Documentation](https://docs.openalgo.in)
- [Introducing OpenAlgo V1.0 — Marketcalls](https://www.marketcalls.in/openalgo/introducing-openalgo-v1-0-the-ultimate-open-source-algorithmic-trading-framework-for-indian-markets.html)
- [Why Building Trading Systems in India Just Got 10x Easier — Marketcalls](https://www.marketcalls.in/algo-trading/why-building-trading-systems-in-india-just-got-10x-easier-inside-openalgos-unified-api-revolution.html)
- [OpenBull — GitHub](https://github.com/marketcalls/openbull)
- [Fenix — GitHub](https://github.com/TheHardeep/fenix)
- [buzzsubash/algo_trading_strategies_india — GitHub](https://github.com/buzzsubash/algo_trading_strategies_india)
- [VarunS2002/Python-NSE-Option-Chain-Analyzer — GitHub](https://github.com/VarunS2002/Python-NSE-Option-Chain-Analyzer)
- [7715Karan/NSE-option_chain — GitHub](https://github.com/7715Karan/NSE-option_chain)
- [sumitsainidev/OIAnalysis — GitHub](https://github.com/sumitsainidev/OIAnalysis)
- [jugaad-data — GitHub](https://github.com/jugaad-py/jugaad-data)
- [nsepy — GitHub](https://github.com/swapniljariwala/nsepy)
- [ProsusAI/finbert — Hugging Face](https://huggingface.co/ProsusAI/finbert)
- [ProsusAI/finBERT — GitHub](https://github.com/ProsusAI/finBERT)
- [foduucom/stockmarket-future-prediction discussion — Hugging Face](https://huggingface.co/foduucom/stockmarket-future-prediction/discussions/1)
- [SEBI Algo Trading Rules 2026 — Sahi](https://www.sahi.com/blogs/sebi-algo-trading-rules-2026-what-every-retail-trader-must-know-before-april)
- [SEBI Extends Retail Algo Trading Rollout — Angel One](https://www.angelone.in/news/market-updates/sebi-extends-retail-algo-trading-rollout-sets-new-deadlines-for-brokers)
- [Algorithmic Trading in India 2026 — QuantInsti](https://www.quantinsti.com/articles/algorithmic-trading-india/)
