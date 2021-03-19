# Experiment with lotus client

## Deal Flow

Requisites:
- A wallet id with FIL

Flow:
1. import file
  * `provider.importFile(data: String): Promise<CID>`
2. fetch file info for deal
  * `client.clientDealSize(cid: Cid): Promise<DataSize>`
3. list miners
  * `client.stateListMiners(tipSetKey: Cid[]): Promise<Array<string>>`
4. get miner info for each miner
  * `client.stateMinerInfo(address: string, tipSetKey: Cid[]): Promise<MinerInfo>`
5. find the price and conditions of each miner
  * `client.clientQueryAsk(id: string, address: string): Promise<StorageAsk>`
6. start a deal with the intended miners
  * `clientStartDeal (startDealParams: StartDealParams): Promise<Cid>`


```js
// const wallet = ...
const cidString = await provider.importFile('this is random text!')

const cid = {
  '/': cidString
}

const dataSize = await client.clientDealSize(cid)

const head = await client.chainHead()
const miners = await client.stateListMiners(head.Cids)

for (const miner in miners) {
  let minerInfo, storageAsk
  try {
    minerInfo = await client.stateMinerInfo(miner, head.Cids)
    storageAsk = await client.clientQueryAsk(minerInfo.PeerId, miner)
  } catch (err) {
    console.log('error trying to get conditions for miner', miner)
  }
  
  // TODO: add your logic for: should we try this minor?

  const dealCid = await client.clientStartDeal({
    Data: {
      PieceCid: cid,
      PieceSize: dataSize.PieceSize,
      Root: cid,
      TransferType: ''
    },
    DealStartEpoch: -1, // default
    EpochPrice: storageAsk.Price,
    FastRetrieval: true,
    MinBlocksDuration: undefined,
    Miner: selectedMiner,
    ProviderCollateral: undefined,
    VerifiedDeal: false,
    Wallet: wallet
  })
}
```

Notes:
- the expectations here would be that `client.clientDealSize` returned the actual pieceSize to provide, but I am getting: `unexpected deal status while waiting for data request: 11 (StorageDealFailing). Provider message: deal rejected: proposal piece size is invalid: padded piece size must be a power of 2`
- this is the [recommended flow](https://docs.filecoin.io/store/lotus/store-data), but for the lotus client we should probably leave the file import for the end, once we have a miner ready to receive the file. We need to compute the size first though.
- list miners takes a long time
- for majority of the minders found, they are not responsive

## Issues / Inconsistencies

We should identify optional parameters and default values in the docs. It is not clear, which parameters are required and optional. Moreover, if we use typescript, we really need to provide all parameters in order to make typescript running.

### Authentication

What is the logic behind client auth permissions? Some methods, including `listDeals` need write permissions. In addition, `ClientStartDeal` needs admin permissions.

Providers have support for providing a token, or a token callback function for security purposes in the instance options.

### Util commands

#### commP

`lotus client commP asks.txt`
ERROR: open asks.txt: no such file or directory

Related to? https://github.com/filecoin-project/lotus/issues/3574

### clientQueryAsk

- address + peerId params
  - docs only mention https://docs.filecoin.io/store/lotus/store-data/#find-the-price-and-conditions
  - we should have similar APIs: CLI, Client, ...
    - CLI abstracts complexity that needs to be done in the client
- mostly fails with the miners fetched from listMiners

### import

Importing files remotely is strange. The first impressions by the docs is to use [`clientImport`](https://filecoin-shipyard.github.io/js-lotus-client/api/full-node-api/client.html#clientimport). However, this is only possible when using a client locally, as we need to provide the path to the file.
While the RPC library does not enable this, there is an import functionality to send a file into the server, by using the `provider` instead.
