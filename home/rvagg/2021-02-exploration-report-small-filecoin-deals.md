# @rvagg Exploration Report: Small Filecoin Deals (2021-02)

## Background

### Filecoin storage for the uninitiated

* Typical miners are continually "sealing" new sectors, containing either 32 GiB or 64 GiB of data
* Sealed sectors have a lifetime, pre-specified by the miner, up to a maximum of 18 months and a minimum of 6 months
* Sealed sectors contribute to miner "power", power increases a miner's chances of winning block rewards
* Sectors may, or may not contain data from clients in the form of paid "deals"
* Sectors not containing deals are called "committed capacity" (CC) sectors, they simply contribute to the total network capacity without storing useful data
   * CC sectors remove the incentive to self-deal (faux-deal) where miners pretend to have deal data to increase power
   * Total network capacity, including CC, signals to users that the network has real hardware ready to store data even if those sectors currently don't
* Sectors that contain deal data are called regular sectors
* There is no requirement for a mix of data types in sealed sectors (deal / non-deal)
* Miners have to prove that they retain the entirety of the sealed sectors as-is in regular invervals during each day of the sector's lifetime and are penalised if they can't prove or miss a proving deadline
* Sectors not proven over a fixed number of consecutive deadlines are terminated and a penalty applied
* Clients who have deals in terminated sectors have unpaid funds returned to them
* Miners lock up collateral when they seal sectors, which is returned when the sector expires without fault, as an incentive to keep them active during the sector lifetime
* Deals can be made in the live market by a client contacting a miner directly and:
    * Requesting deal conditions (price per GiB per epoch (30 seconds), min size and max size)
    * Proposing a specific deal
    * Having a deal accepted by a miner
    * Transferring the data to the miner
    * The miner including the deal data into a sector they are committing (currently or queued)
    * Having deal confirmation placed on the chain pointing to its location in the sealed sector
* Deals can also be made "offline" where the deal is made but the data is shipped (perhaps manually) to the miner who loads it into a sector
* Deal funds pay out to the miner over the course of the lifetime of the deal to ensure they continue proving retention of data
* Deals can either be regular deals or "verified deals"
    * Verified deals come from an additional social layer outside of Filecoin, called Filecoin Plus (FIL+)
    * Notaries are organisations that can hand out "DataCap" to clients
    * Notaries are selected by the Filecoin community, distributed around the world
    * "DataCap" is a quota that can be spent, along with FIL, to make verified deals
    * Verified deals contribute 10x to miner power, proportional to how much space they occupy in a sector
    * Therefore, verified deals should be attractive to miners who may charge less FIL to get them

### Deal incentives & challenges

* Miners bear responsibility for publishing deals to the chain, with high gas costs this can be in the order of $5
* Recent changes allow for batch publishing deals, but anecdotally this hasn't changed the economics for miners much (need more context here - is it because the batch publish is still quite expensive, or because there are not enough deals to batch?)
* Accepting deals can offset the cost of sealing and proving, but this biases toward larger deals
* Verified deals have the 10x increase in power, but they are scarce because FIL+ is so new, currently unknown the scale of incentive impact for miners for large and small deals
* Bugs in Lotus have made miners gun-shy on doing anything more than CC:
  * Transferring deal data is buggy and transfers often fail (anecdotal)
  * Retrieval is buggy and can cause crashes, which are problematic when you need 100% uptime to prove sectors (anecdotal)
* A lot of large miners don't seem to have transitioned from CC sealing to accepting deals - could be immaturity of market
* Many miners don't respond to `query-ask` at all. Of those that do, the majority appear to reject deals even when those deals fit within the ask parameters. So the `query-ask` mechanism seems of limited use in its current form.

Filecoin currently only stores ~5 Pb of deal data and growth rate is not large.

## Installing and Running Lotus

Followed instructions at https://docs.filecoin.io/get-started/lotus/installation/ for an AWS instance

 * Large AMD EPYC instance, Ubuntu 20.04
 * Installed dependencies (used Go Snap rather than Go tarball)
 * Cloned and build Lotus with CPU SHA support
 * Installed systemd service to run as user `lotus` and put logs into that home directory
 * Used minimal stateroots snapshot @ https://fil-chain-snapshots-fallback.s3.amazonaws.com/mainnet/minimal_finality_stateroots_latest.car
 * Started manually with `lotus daemon --import-snapshot`
 * Stopped, then started via systemd, let it sync, watching the log output with `tail -f`
 * Can interact with the daemon via the CLI with `lotus client` and `lotus wallet`, etc. as the `lotus` user (appears to be user-restricted IPC)
 * Made a wallet and transferred some FIL in it for experimenting

Interesting commands:

 * `lotus sync wait` will run until the chain is synced, zero exit status on success
 * `lotus wallet` - `lotus wallet new`, `lotus wallet new`, `lotus wallet balance`, `lotus wallet export` (export privkey for backup)
 * `lotus client`
   * `lotus client import <path>` to import file(s) into the client (somewhere?), presumably as UnixFS (?), gets a CID
   * `lotus client local` shows you what you have imported and their CIDs
   * `lotus client balances` shows you funds that you have active in the "market" (where you have to have funds in order to make deals etc. i.e. out of your wallet)
   * `lotus client find <CID>` lets you query the network for a CID

Deals:

   * `lotus client query-ask <minerID>` ask a miner for their deal terms - returns FIL asking price per GiB per epoch (30 seconds) for both regular and verified deals, along with minimum and maximum deal sizes.
   * `lotus client list-asks` will do a scan of the network for miners with power and do a `query-ask` on each of them, reporting the output to stdout, one miner per line (~450)
   * `lotus client deal` without additional arguments takes you through a prompted deal-making process that asks for your data CID, a miner ID, whether it's a verified deal; it queries the miner, asks you to confirm the asking price and starts the deal process if you say `yes`.
   * `lotus client list-deals` to show you deals in progress and their status (Note: the status identifiers are not self-explainatory, the best source of documentation seems to be the markets code: https://github.com/filecoin-project/go-fil-markets/blob/master/storagemarket/dealstatus.go)
   * `lotus client deal-stats` seems to be the best way to view the outcome of your deals once they disappear off `list-deals`
   * The Lotus daemon log also has output related to deals, you can grep for the status, e.g. `grep StorageDealError ~/var/lotus-daemon.log` (mostly just `"error reading Response message: EOF"`)

### Making deals

* Imported a 9.6 Mb (`10016091` b) JPG file to Lotus with `lotus client import`. Some notes about sizes:
  * Note that data is _padded_ when making deals. First 2 empty bits per 254 bytes are added (called "fr32 padding", `Math.ceil(10016091 * (256/254))`), then it's rounded up to the next nearest power-of-two value as "pieces" stored in Filecoin have to fit neatly into a proving tree (`2**Math.ceil(Math.log2(Math.ceil(10016091*(256/254))))`).
  * The additional padding is done transparently, but it leads to wonky numbers in the deal process that aren't explained well (and online documentation is inconsistent on this and doesn't explain it well - TODO some PRs)
  * The most consistent number shown is power-of-two _minus_ the fr32 padding. (`2**Math.ceil(Math.log2(Math.ceil(10016091*(256/254))))/(256/254)/1024/1024` as `MiB`, or `15.875`, presented as `15.88 MiB` by Lotus - clear as mud).
  * Aside: an implication of padding is that when packing data in for making efficient deals, you should get as close to the "power-of-two - fr32 padding" size as possible. The larger the data, the more waste you'll have. We had to do this for Filecoin Discover data to edge up as close as possible to 1 GiB pieces.
* Found some miners
  * Used the MinerX list (link to spreadsheet omitted)
  * Ran `query-ask` against all of them (`for i in $(cat minerx-fellows.txt); do echo "querying $i"; lotus client query-ask $i >> minerx-fellows.txt.query-ask; done`); got 34 out of 36 responses.
  * Filtered by minimum deal size, only 7 had miners accepting deals of 16 MiB or less
  * Slate did some work on finding miners (not sure how up to date this is), collated @ https://sentinel.slate.host/api/mapped-static-global-miners and includes some miners that accept small deals too.
* Ran `lotus client deal` to step through some deals interactively with miners I'd narrowed down. Got the following results for my file and submitted deals with most of these:
  - f019104 ~0.0093312 FIL (0.000000018 FIL per epoch) ~$0.32
  - f019041 ~0.0002592 FIL (0.0000000005 FIL per epoch) ~$0.01
  - f066596 ~0.010368 FIL (0.00000002 FIL per epoch) ~$0.36
  - f03134 ~8100 FIL (0.015625 FIL per epoch) ~$283,500 _(obviously I said `no` to this one)_
  - f023565 ~0.0000000000005184 FIL (0.000000000000000001 FIL per epoch) $0
  - f022352 ~0.0001458 FIL (0.00000000028125 FIL per epoch) ~$0.01 (0.005103)
* Deal process is very slow and involves:
  * Moving funds from the local wallet into the "StorageMarket" actor, to setup for a kind of escrow so the miner knows they're going to get paid
  * The miner deciding whether to accept the funds. Miners set default deal parameters but can set additional configuration options and even insert an "deal acceptance script" to their deal flow to accept and reject deals based on all of the parameters. See https://docs.filecoin.io/mine/lotus/miner-configuration/#using-filters-for-fine-grained-storage-and-retrieval-deal-acceptance
    * This can be slow too, I had deals stuck in `StorageDealCheckForAcceptance` for extended periods, I'm not sure what this is but maybe it's waiting for manual acceptance by the miner?
  * Transferring the data via graphsync to the miner
  * The miner setting up the deal with the chain (does this come before or after the next step?)
  * The miner queueing the piece for sealing (can be slow, miners can choose to batch deals, which defer deals until enough have been selected, and can set maximum number of deals per sector too)
  * Sealing the sector that includes the piece (slow)
* Successfully made a deal with **only one miner**, [`f022352`](https://filfox.info/en/address/f022352) in Norway! As of writing, still waiting for PreCommit, ~5 hours later.

### Verified deals

* FIL+ details are at https://github.com/filecoin-project/filecoin-plus-client-onboarding/ - information about notaries and clients asking for DataCap for making verified deals
* https://verify.glif.io/ will give you 8 GiB of DataCap every month if you have an established GitHub account, without going through a Notary.
* https://verify.glif.io/ will also let you look at DataCap balance for a wallet/ID.
* `lotus-shed` (comes with the Lotus source, `make lotus-shed`) has a `verifreg` command that's being built out that you can use to check a client's balance `lotus-shed verifreg check-client <address/ID>`
  * `lotus-shed` has other interesting commands where you can look up notaries and see all of the clients with DataCap.
* I got some DataCap to try it out through glif.
* Using `lotus client deal` to attempt some deals again, I was able to say `yes` to "verified deal", this yields new prices, some of which are `0 FIL`:
  - f019104 0 FIL $0
  - f019041 ~0.000000405 FIL (0.00000000000078125 FIL per epoch)
  - f066596 0 FIL $0
  - f03134 ~8100 FIL (0.015625 FIL per epoch) ~$283,500 _(obviously I said `no` to this one)_
  - f023565 0 FIL $0
  - f022352 0 FIL $0
  - f071624 ~0.000000405 FIL (0.00000000000078125 FIL per epoch)
  - f0135078 ~0.000000405 FIL (0.00000000000078125 FIL per epoch)
  - f058369 ~0.00081 FIL (0.0000000015625 FIL per epoch)
  - f010479 ~0 FIL (0 FIL per epoch)
  - f01247 ~0 FIL (0 FIL per epoch)
* Successfully made a deal with the same miner as for my unverified deal, [`f022352`](https://filfox.info/en/address/f022352). As of writing, still waiting for PreCommit, ~5 hours later. 0 FIL instead of 0.0001458 FIL.
* Also had success with [`f058369`](https://filfox.info/en/address/f058369) which is also waiting for PreCommit. Costing 0.00081599375 FIL ($0.03).
