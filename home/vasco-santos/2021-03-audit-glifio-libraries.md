# glifio libraries

The repos does not provider a brief overview of what it has to offer, which makes the analysis more difficult.

## [rpc-client](https://github.com/glifio/modules/tree/primary/packages/filecoin-rpc-client)

The rpc client is a wrapper around `axios` to create JSON RPC requests with a simple API.

```js
const LotusRPCEngine = require('@glif/filecoin-rpc-client')

const config = {
  apiAddress: 'http://127.0.0.1:1234/rpc/v0',
  token: 'aaaaaaaa.bbbbbbbbbbbb.i_ZZZZZZ-3xYYYYYY',
}

const lotusRPC = new LotusRpcEngine(config)
const address = 't1jdlfl73voaiblrvn2yfivvn5ifucwwv5f26nfza'
const balance = await lotusRPC.request('WalletBalance', address)
```

```js
async request<A = any>(method: string, ...params: any[]): Promise<A>
```

This library is quite easy to maintain, but it requires users to fully look into lotus API when using it. In addition, the user is responsible for parsing the response.

## [filecoin-wallet-provider](https://github.com/glifio/modules/tree/primary/packages/filecoin-wallet-provider)

**State**: Active development

Wallet implementation inspired by Metamask. This seems to resolve some of the gaps (link them?)

## [filecoin-address](https://github.com/glifio/modules/tree/primary/packages/filecoin-address)

TODO: Understand use case

## [filecoin-message-confirmer](https://github.com/glifio/modules/tree/primary/packages/filecoin-message-confirmer)

TODO: Understand use case

## [filecoin-message](https://github.com/glifio/modules/tree/primary/packages/filecoin-message)

TODO: Understand use case

## [filecoin-number](https://github.com/glifio/modules/tree/primary/packages/filecoin-number)

A wrapper class built around javascript's bignumber.

TODO: Understand use case (I guess this is for deal negotiation with sizes?)
