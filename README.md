# allo's bot

One always-on service with three jobs:

- **Payouts.** Every hour it works out each $ALLO staker's share of every coin launched on allo,
  records it on-chain with the distributor key, and serves everyone's drops to allo.cash.
- **Launches on pump.fun** (once allo v2 is set up with `./pump-setup.sh`; until then it waits).
  It opens and seeds each new coin's stakers' pool within seconds of launch, harvests the pools
  and splits creator fees hourly, buys each staker's chosen payout token ($ALLO, $ALLO into their
  stake, USDC, or gold, which it buys through USDC) a small slice at a time, and pays everyone
  without them signing anything. It
  also keeps a stock of coin addresses ending in `aLLo` ready for the launch page.
- **stALLO** (once it's turned on with `./pump-setup.sh stallo`). stALLO's pooled stake is paid
  like a staker who chose $ALLO into their stake: its share is always bought into $ALLO and put
  back into the stake, so every stALLO is worth more $ALLO after each payout.
- **Paired coins** (when `PAIR_KEY` is set). For coins launched "paired with Hyperliquid", it
  claims their fees, sends allo's half to the deploy key, runs the rest as each coin's position on
  Hyperliquid, and buys back and burns the coin with the profit.

It runs allo's contract, `7NmEQXhcA2wULS2EzmU2MikWhdbTsZ4MfRuDW55QaLLo`, and keeps that
contract's memory in its own folder on the disk, so nothing recorded before is touched. Until
$ALLO and its own pool are out, launches still go through: each coin's slice for stakers waits in
allo's vault, and the bot seeds its pool as soon as allo's settings name $ALLO and its pool.

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

Fund the bot's keys: about **0.5 SOL** for the distributor key once launches on pump.fun are
open (it opens each new coin's pool and gets that rent back in the same launch, and pays network
fees), and **0.05 SOL** for the pair bot's Solana key.

## Optional settings

- `BACKLOG_SPREAD_SECONDS` (default 7 days): how long the stakers' share saved up before staking
  opened takes to pay out.
- `INTERVAL_SECONDS` (default 3600): how often it pays.
- `PAIR_INTERVAL_SECONDS` (default 600): how often it checks paired coins.
- `PAIR_MAX_LEVERAGE` (default 10) and `PAIR_TAKE_PROFIT_PCT` (default 20).
- `PAIR_TREASURY`: where allo's half of paired coins' fees goes. Defaults to the deploy key.
- `CONVERT_SWAPS_PER_RUN` (default 2), `CONVERT_MAX_IMPACT_BPS` (default 100): how many buys of a
  payout token each run makes, and how far each may move its price (100 = 1%).
- `CONVERT_MIN_LAMPORTS` (default 5000000, 0.005 SOL): the smallest buy of a payout token. At 1%
  a buy, a pool needs about 100 times this in SOL to take one.
- `CONVERT_MAX_PREMIUM_BPS` (default 1000): the bot doesn't buy a payout token while it costs more
  than this above its median price over the last six runs (1000 = 10%).
- `CONVERT_STALL_HOURS` (default 24): SOL that has waited this long to be bought into a token is
  paid as SOL instead.
- `MINT_STOCK` (default 30), `MINT_GRINDERS` (default 1): coin addresses ending in `aLLo` kept
  ready, and how many processor cores make them.

## What it serves

- `/health`: whether it's running, and the last run of each job.
- `/drops/<wallet>`, `/launches`: stakers' drops.
- `/pairs/markets`, `/pairs`, `/pairs/<coin>`: paired coins, for the launch page and coin pages.
- `/streams/...`: stakers' payouts from the reward streams. `/mint`: a coin address ending in
  `aLLo` for the launch page. `/seed/<coin>`: asks it to open a new coin's pool right away.

## Safety

The distributor key can't take anything from anyone's stake, or rewards that were already paid
out. It does decide where rewards not yet paid out go, and buys stakers' payout tokens with them
through pools allo approved. So a thief with it could misassign those rewards or sell them
cheaply. Keep it as private as the deploy key. `./new-bot-key.sh` swaps it.

The pair bot's keys hold paired coins' money while it's on its way and in their positions.
Keep `keys/pair-bot.json` and `keys/hyperliquid.json` as private as the deploy key. If a
Hyperliquid transfer is ever interrupted, the bot stops paired coins and says why on `/health`
instead of guessing: nothing is sent twice.
