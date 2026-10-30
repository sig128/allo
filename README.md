# allo's bot

One always-on service with two jobs:

- **Payouts.** Every hour it works out each $ALLO staker's share of every coin launched on allo,
  records it on-chain with the distributor key, and serves everyone's drops to allo.cash.
- **Paired coins** (when `PAIR_KEY` is set). For coins launched "paired with Hyperliquid", it
  claims their fees, sends allo's half to the deploy key, runs the rest as each coin's position on
  Hyperliquid, and buys back and burns the coin with the profit.

Run **exactly one** copy, and never delete its disk: the disk is its memory of what it has paid
and what each coin owns. A fresh copy refuses to pay coins it doesn't recognise rather than pay
anything twice.

## Settings on Railway

In Terminal, from `~/CRYPTOOO/allo-mainnet`:
```
./railway.sh            # paired positions on Hyperliquid's test network
./railway.sh mainnet    # paired positions for real, once the test network round worked
```
It asks for your RPC link and copies every setting to your clipboard, keys included, without
showing them. In Railway: allo service → **Variables** → **Raw Editor** → select all → paste →
**Update Variables** → **Deploy**.

The service also needs a **Volume** mounted at `/data`, and a domain (**Settings → Networking →
Generate Domain**, port `8787`). `https://<your domain>/health` shows `"ok": true` when it runs.

Fund the bot's keys: about **0.1 SOL** for the distributor key and **0.05 SOL** for the pair
bot's Solana key. They pay tiny network fees.

## Optional settings

- `BACKLOG_SPREAD_SECONDS` (default 7 days): how long the stakers' share saved up before staking
  opened takes to pay out.
- `INTERVAL_SECONDS` (default 3600): how often it pays.
- `PAIR_INTERVAL_SECONDS` (default 600): how often it checks paired coins.
- `PAIR_MAX_LEVERAGE` (default 10) and `PAIR_TAKE_PROFIT_PCT` (default 20).
- `PAIR_TREASURY`: where allo's half of paired coins' fees goes. Defaults to the deploy key.

## What it serves

- `/health`: whether it's running, and the last run of each job.
- `/drops/<wallet>`, `/launches`: stakers' drops.
- `/pairs/markets`, `/pairs`, `/pairs/<coin>`: paired coins, for the launch page and coin pages.

## Safety

The distributor key can't take anything from allo's vault or from anyone's stake; at worst a
thief could misassign drops that haven't been claimed yet. `./new-bot-key.sh` swaps it.

The pair bot's keys hold paired coins' money while it's on its way and in their positions.
Keep `keys/pair-bot.json` and `keys/hyperliquid.json` as private as the deploy key. If a
Hyperliquid transfer is ever interrupted, the bot stops paired coins and says why on `/health`
instead of guessing: nothing is sent twice.
