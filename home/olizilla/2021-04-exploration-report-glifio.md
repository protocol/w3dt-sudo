# glif.io wallet & modules

> Exploration report by @olizilla and @vasco-santos

## Recommendations

- Point folks at the wallet as an example of a production app. The wallet app has been audited, gets regular use, and has a clear codebase with tests.
- Add an architecture over view diagram and docs to the readme. This would go a long way to making wallet an educational resource.
- Confirm that we're ok with it listing out PL signing addresses in a public repo https://github.com/glifio/wallet/blob/508ba48825246abf998db3f283771dba492a66d1/constants.js#L42-L65
  - @riba felt it was uncool, as it removes any plausible denyability of those addresses being PLs
  - @why felt it was nothing to worry about, as they are on the chain and would not be difficult to guess as belonging to PL
- Have the static site built in CI and published to IPFS.
  - https://github.com/glifio/wallet/issues/513
- Add a note to the Readme to explain why the site works only in Chrome.
- Investigate why the static site is 40MB !?
  - this doesn't appear to be an issue when using the app... the largest single asset is 2MB and the wasm module is 1.6MB
- Adopt `filecoin-address` in filecoin-project org to better promote it.
- Research and develop a blessed filecoin-number js lib, using `@glif/filecoin-number` as a starting point.
- Add `@glif/filecoin-rpc-client` to the [filecoin docs](https://docs.filecoin.io/build/lotus/api-client-libraries) as the minimalist's api client.


## Key Findings

- They've created some very approachable and useful rpc client docs with documenter.getpostman.com see: https://documenter.getpostman.com/view/4872192/SWLh5mUd?version=latest
  - Grouping the endpoints by concept and giving an intro to each section is helpful.
  - example requests and repsonse is good.
- Filecoin docs need to do a better job of explaining finality and how to approach the risk of working with chain data near the head that could be re-orged... the note on these docs implies that blocks >= 900 epochs old is ok, which is true, but it seems like awkward framing. https://documenter.getpostman.com/view/4872192/SWLh5mUd?version=latest#7c7d1a3f-c5d5-42ed-9b1d-b019d02f159d


## [glif.io wallet](https://github.com/glifio/wallet)

> how re-usable or educational is https://github.com/glifio/wallet as a resource we can recommend? ("Go check out this project if you want to learn how to build a filecoin-backed webapp")

It's a production app. It's not directly re-usable and it's not suffciently documented to be an introductory educational resource, but it is useful as a well built and tested example of an app that talks to Lotus, does something useful, and uses a popular UI stack (next.js, react, redux).


## [glif.io modules](https://github.com/glifio/modules)

> how useful are the individual components in their projects list independently. @glifio/filecoin-number seems quite useful, what about the rest?

It's a lerna repo, commits every month, and the project builds and works out of the box. Each module is explored below


## [@glif/filecoin-rpc-client](https://github.com/glifio/modules/tree/primary/packages/filecoin-rpc-client)

**TL;DR** Too bare-bones for general consumption, but a good example for folks who like it minimal.

A very light [(~70 LoC) wrapper](https://github.com/glifio/modules/blob/9b97f31054c20b39b8a75d30727fa33162715dab/packages/filecoin-rpc-client/src/index.ts
) around `axios` to create JSON RPC requests.

- 👍 lightweight, low maintenance, never out of date, easy to understand.
- 👍 lovely docs https://documenter.getpostman.com/view/4872192/SWLh5mUd?version=latest
- ❌ methods passed as string arg. No api, no types, no way to discover the api from your IDE.
- ❌ you've got to parse filecoin responses manually
- ❌ CID params are awkward. Must be passed as a ipld-json link `{ '/': '<cid string>' }`

Example setup and usage

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

Passing a CID param

```js
const cid = {
  '/': 'bafy2bzacedavr434vvbck3nkowrffk2x67lgl7kfujlw3lyj2zjvmihn5rf7i',
}
const message = await lotusRPC.request('ChainGetMessage', cid)
```

API signature

```js
async request<A = any>(method: string, ...params: any[]): Promise<A>
```


## [@glif/filecoin-address](https://github.com/glifio/modules/tree/primary/packages/filecoin-address)

**TL;DR** 👍 use it. Adopt it into the filecoin-project org to give more visibilty.

> This is a JS implementation of the Filecoin address type, inspired by go-address. It can create new address instances and encode addresses, and it takes care of decoding and validating checksums.

## [@glif/filecoin-number](https://github.com/glifio/modules/tree/primary/packages/filecoin-number)

**TL;DR** Review and potentially replace with a lighter / more ergonimic alternative. 

A wrapper class for bignumber.js to represent an amount of FIL which may be larger than `Number.MAX_SAFE_INTEGER` (2^53 - 1). Provides conversion from attoFIL to picoFIL to FIL, and a Currency Conversion api which uses a remote service to provide the exchange rate.

- We should explore the ideal API for this representing FIL amounts.
- Consider using a lighter lib for big number support https://github.com/MikeMcl/big.js/wiki
- Consider just using the built-in BigInt https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/BigInt


## [filecoin-message](https://github.com/glifio/modules/tree/primary/packages/filecoin-message)

TODO


## [filecoin-message-confirmer](https://github.com/glifio/modules/tree/primary/packages/filecoin-message-confirmer)

TODO


## [filecoin-wallet-provider](https://github.com/glifio/modules/tree/primary/packages/filecoin-wallet-provider)

TODO


## [@glifio/react-components](https://github.com/glifio/modules/tree/primary/packages/react-components)

**TL;DR** Not for general consumption. Has some nice UI components, but folks can explore the wallet app to see them, and get the most up to date versions.

UI components for the glif.io wallet app, with storybook integration to preview them. Folks could pick ideas out of here but it's not aiming at generic components for mass consumption.

It's not actually used in wallet and is slightly behind some identical components that are in https://github.com/glifio/wallet/tree/primary/components/Shared
see: https://github.com/glifio/wallet/pull/726


## Notes

wallet has Some PL specific parts. e.g:

⚠️ list of PL signing fil addresses in public repo https://github.com/glifio/wallet/blob/508ba48825246abf998db3f283771dba492a66d1/constants.js#L42-L65
 
Why is it not a static html/js build hosted on IPFS? 
- started the conversation here https://github.com/glifio/wallet/issues/513

why 40MB build size?
```
added bafybeifn4nptqj3heumxoxn3xwosyjegzzrqzuzjordqkqjz6fdjwtbysy out/vault
added bafybeicfdbc2drxcjjoktznm7kvdzimd2po65c2ywztnaq3rk6b7gh5inu out
 40.41 MiB
```
  - tried scanning the package.json on bundlephobia but it broke the site...
  - https://bundlephobia.com/scan-results?packages=@glif/filecoin-address@1.1.0,@glif/filecoin-message@1.1.0,@glif/filecoin-message-confirmer@1.1.0,@glif/filecoin-number@1.1.0,@glif/filecoin-wallet-provider@1.1.0,@ledgerhq/hw-transport-webusb@5.39.0,@zondax/filecoin-signing-tools@0.14.0,@zondax/ledger-filecoin@0.11.2,axios@0.21.1,bip39@3.0.3,bluebird@3.7.2,cbor@5.1.0,crypto@1.0.1,dayjs@1.8.22,jspdf@2.1.1,lodash.clonedeep@4.5.0,lodash.isequal@4.5.0,object-path@0.11.5,react@17.0.2,react-dom@17.0.2,react-redux@7.1.0,redux@4.0.4,redux-logger@3.0.6,redux-thunk@2.3.0,styled-components@5.0.1,styled-system@5.1.5


loads @zondax/filecoin-signing-tools as a wasm module.

Chrome only. (possible due to web-usb support)

a test fails (also in CI for latest merge.)

it's a classic Next.js React + redux... e.g https://github.com/glifio/wallet/blob/primary/store/store.js is a very familiar redux set up.

nice set of ui utilities, mostly with tests https://github.com/glifio/wallet/tree/primary/utils

```
Test Suites: 1 failed, 73 passed, 74 total
Tests:       7 skipped, 1 todo, 401 passed, 409 total
Snapshots:   99 passed, 99 total
Time:        27.405s
Ran all test suites.
```