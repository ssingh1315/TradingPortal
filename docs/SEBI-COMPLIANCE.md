# SEBI algo-trading compliance checklist

India's SEBI retail algo-trading framework became mandatory for all brokers on **April 1, 2026** — it's in force now. This affects how TradingPortal should be configured before you connect a live (non-sandbox) broker account, regardless of which broker you use.

## Does this apply to you?

It applies to anyone placing orders through a broker API rather than clicking buttons in the broker's own app — which is exactly what TradingPortal does. The specific obligations scale with order rate:

- **Under 10 orders/second**: classified as a regular API user, lighter-touch requirements.
- **10 orders/second or more**: formal algo registration becomes mandatory, including an exchange-assigned Algo-ID tagged on every order from that strategy.

## Checklist before going live

- [ ] **Static IP whitelisting** — confirm with your broker (Zerodha, Upstox, Angel One, Fyers, IIFL, etc.) that the IP address TradingPortal will trade from is whitelisted in their API developer console. This is now a hard prerequisite, not optional hardening.
- [ ] **Algo-ID tagging** — if any strategy you run through TradingPortal can hit 10+ orders/second (most retail strategies won't, but check your order-frequency logic, especially anything with tight retry loops), register it with your broker/exchange and make sure the Algo-ID is tagged on outgoing orders.
- [ ] **White box vs. black box** — if TradingPortal ever runs a strategy whose logic isn't disclosed to whoever is using it (relevant only if you let someone else use your instance — see `NOTICE.md` on the AGPL implications of that), the provider needs a SEBI Research Analyst (RA) license. Keeping strategy code visible to whoever runs it sidesteps this entirely, and is the sane default for a personal system anyway.
- [ ] **Broker accountability** — brokers are now responsible for every algo running through their platform. Expect your broker to ask questions or require sign-off before enabling full API trading access if you haven't already gone through this.

## Where to configure this in TradingPortal

Broker connections are configured under the app's Settings → Broker Configuration screen (see `UPSTREAM_README.md` / `INSTALL.md`). When you add a broker's API credentials there, that's the point to also confirm IP whitelisting is done on the broker's side — TradingPortal doesn't currently have a dedicated field for tracking Algo-ID/whitelist status, so track it manually (e.g. in this file, or a private notes doc) until/unless that's added.

## Sources

- [SEBI Algo Trading Rules 2026 — Sahi](https://www.sahi.com/blogs/sebi-algo-trading-rules-2026-what-every-retail-trader-must-know-before-april)
- [SEBI Extends Retail Algo Trading Rollout — Angel One](https://www.angelone.in/news/market-updates/sebi-extends-retail-algo-trading-rollout-sets-new-deadlines-for-brokers)
- [Algorithmic Trading in India 2026 — QuantInsti](https://www.quantinsti.com/articles/algorithmic-trading-india/)

This is a plain-language summary, not legal advice — confirm current requirements with your broker and, for anything ambiguous, a professional familiar with SEBI's current circulars.
