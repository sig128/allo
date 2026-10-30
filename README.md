# allo's payout bot

Every hour it works out each $ALLO staker's share of every coin launched on allo, records it
on-chain with the distributor key, and serves everyone's drops to allo.cash.

It runs as one always-on service with a small disk. Run **exactly one** copy, and never delete
its disk: the disk is its memory of what it has paid. A fresh copy refuses to pay coins it
doesn't recognise rather than pay anything twice.

## Put it on Railway

1. Make an account at railway.com and add a payment card (the Hobby plan is enough).
2. In Terminal, once:
   ```
   npm install -g @railway/cli
   railway login
   ```
3. From this folder:
   ```
   cd ~/CRYPTOOO/allo-bot   # or deploy this GitHub repository from the Railway dashboard
   railway init          # name the project allo-bot
   railway up            # uploads and starts it (it stops at first: settings come next)
   ```
4. In the Railway dashboard, open the allo-bot service:
   - **Variables**, add:
     - `RPC_URL`: a QuickNode link just for the bot (make a new endpoint in QuickNode)
     - `DISTRIBUTOR_KEY`: the contents of `allo-mainnet/keys/distributor.json`. Copy it
       without showing it: `pbcopy < ~/CRYPTOOO/allo-mainnet/keys/distributor.json`, then paste.
     - `STATE_DIR`: `/data`
     - `PRIORITY_MICRO_LAMPORTS`: `100000`
   - **Volume**: add one, mounted at `/data`.
   - **Settings → Networking → Generate Domain**. Send that address to Claude: it goes into the site.
5. Send about **0.1 SOL** to the distributor key `Fjzz5RAKs84K3xiDK44A5h44WahYFqzauU8V7dfoHxuN`.
   It pays a tiny fee for each hourly record.

Check it at `https://<your domain>/health`: `"ok": true` means it's running.

## Settings

- `BACKLOG_SPREAD_SECONDS` (default 7 days): how long the stakers' share saved up before staking
  opened takes to pay out.
- `INTERVAL_SECONDS` (default 3600): how often it pays.

## Safety

The distributor key can't take anything from allo's vault or from anyone's stake. The worst a
thief could do is misassign drops that haven't been claimed yet. If it ever leaks, your deploy
key can swap it for a new one.
