# TradingPortal

Sarbjit Singh's self-hosted options & algo trading system for the Indian markets — options chain analytics, strategy automation, and unified connectivity to Zerodha, Upstox, Angel One, Fyers, IIFL, and other Indian brokers.

## What this is

TradingPortal is built directly on top of **[OpenAlgo](https://github.com/marketcalls/openalgo)**, the open-source algorithmic trading platform. The core application (broker adapters, options analytics suite, strategy hosting, order execution) in this repo is OpenAlgo's codebase; everything under `strategies/` and `docs/` beyond the upstream docs is TradingPortal-specific.

**Upstream project's own README is preserved at [`UPSTREAM_README.md`](./UPSTREAM_README.md)** — read that for the full feature tour (four trading surfaces: Unified Broker API, Python Strategy Host, no-code Flow builder, and the Options Trading Suite with twelve analytical tools).

## License — read this before you deploy

This project is licensed under the **GNU Affero General Public License v3.0** (see [`LICENSE.md`](./LICENSE.md)), inherited from OpenAlgo. The practical consequence: if you run a modified version of this code as a network service — including a private VPS, not just a public product — and let **anyone other than yourself** use it, you are legally required to make the complete corresponding source code (including your modifications) available to those users. Running it purely for yourself, locally or on a private server only you access, does not trigger that obligation. If you ever plan to let others use TradingPortal or sell access to it, revisit this before doing so.

## Getting started

Follow [`INSTALL.md`](./INSTALL.md) for the full setup (Python 3.12+, Node.js for the frontend build, VS Code recommended). Quick version:

1. Copy `.sample.env` to `.env` and fill in a `SECRET_KEY`/`API_KEY_PEPPER` (see comments in the file) — do **not** commit `.env`.
2. Install Python deps: `pip install -r requirements.txt`
3. Build the frontend: see `frontend/` and `INSTALL.md` for the Node build steps.
4. Run `python app.py` (or use `docker-compose.yaml` for a containerized setup).
5. Open the app and connect a broker under Settings — you'll need API credentials from that broker's developer portal (Zerodha Kite Connect, Upstox, Angel One SmartAPI, Fyers, IIFL's XTS API, etc.).

## Before you go live: SEBI compliance

India's retail algo-trading framework is now mandatory (in force since April 1, 2026). See [`docs/SEBI-COMPLIANCE.md`](./docs/SEBI-COMPLIANCE.md) before connecting a real broker account and placing live orders — it covers Algo-ID tagging, the 10 orders/second threshold, and static IP whitelisting.

## Repo layout

- Everything at the root (`app.py`, `broker/`, `blueprints/`, `frontend/`, `restx_api/`, etc.) — OpenAlgo's core platform, largely unmodified.
- `strategies/` — your own trading strategies go here (also where OpenAlgo's Python Strategy Host looks for hosted scripts).
- `docs/` — OpenAlgo's own docs, plus `SEBI-COMPLIANCE.md` and `RESEARCH.md` (TradingPortal-specific).
- `NOTICE.md` — attribution and AGPL compliance notes for this fork.

## Credits

Built on [OpenAlgo](https://github.com/marketcalls/openalgo) by [marketcalls](https://github.com/marketcalls) and contributors. See `NOTICE.md` for full attribution.
