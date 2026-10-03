# hummingbot-for-radix

Hummingbot setup for market making **XRD** on **MEXC** and **Gate.io** (spot).

> Run this on **your own machine or a VPS you control**, not in a Claude cloud session:
> it needs a long-lived process, a stable IP for API whitelisting, and your exchange API keys.
> **Never paste API keys into chat or commit them.**

## 1. What Hummingbot is
Open-source bot framework. You connect exchange API keys, pick a strategy, and it
places/refreshes limit orders around the mid price (market making) or runs other logic.
- Connectors: `mexc` and `gate_io` (spot). Trading pair format: `XRD-USDT`.
- Strategies: V2 controllers (e.g. `pmm_simple`, `pmm_dynamic`) via the `v2_with_controllers`
  script, or standalone scripts such as `simple_pmm`. Older "pure market making" v1 is legacy.
- Two independent bots (one per exchange) is the simplest start. Cross-exchange
  market making (`xemm`) is possible later but needs inventory on both venues.

## 2. Before you run anything
1. **Check both exchanges list XRD/USDT and that Hummingbot has XRD pairs live** (`connect`, then `ticker`/order book in-client).
2. **Make a trade-only API key** on each exchange: *no withdrawal permission*, IP-whitelisted to your host.
3. **Fees/volume:** check maker fee tiers; XRD liquidity is thin so spreads must cover fees + adverse selection.
4. **Fund small** (e.g. $200-500 USDT + equal XRD value). Paper-trade first (`paper_trade_exchanges` in `conf_client.yml`).
5. Compliance: market making your own token vs. someone else's, wash-trading rules and local regulation are your responsibility. Do not quote against yourself to fake volume - exchanges ban it and it may be unlawful.

## 3. Quick start
```sh
cp .env.example .env            # optional password
docker compose run --rm hummingbot
```
Inside the client:
```
connect mexc        # paste API key/secret when prompted (stored encrypted in conf/connectors)
connect gate_io
balance
create --controller-config     # pick pmm_simple, mexc, XRD-USDT; save as xrd_mexc_pmm
create --controller-config     # repeat for gate_io -> xrd_gate_pmm
start --script v2_with_controllers.py --conf <your-script-config>.yml
status
stop
```
`conf/controllers/*.yml.example` show sensible starting values (wide 0.5-1% spreads, small size,
stop-loss). Treat them as guidance - **I could not verify field names against the current
Hummingbot docs from my environment**, so generate real configs with the `create` wizard.

## 4. Operating rules
- Start with `stop_loss` set and low `total_amount_quote`; widen spreads when XRD is volatile.
- Watch `logs/` and `status`; run under `tmux`/Docker restart policy on a VPS once stable.
- Keep bots on separate API keys per exchange, rotate keys if the host is ever shared.
- Pin the Docker image tag after you validate a version.

## 5. Possible next steps
Inventory-skew parameters, dynamic spreads (`pmm_dynamic`), Hummingbot API + Dashboard for
multi-bot management, and an XRD price feed from the Radix Gateway/on-chain DEX (see the
sibling `radix-dev` repo) as a reference price for quoting.
