# Lotus / Filecoin RPC and JavaScript Library Audit

**2021-04**

This report captures a condensed summary of a brief audit conducted of the current state of the JavaScript ecosystem surrounding Filecoin and the affordances provided to JavaScript developers via the Lotus RPC API.

## Lotus RPC API

### OpenRPC support
> JSON based API definition of the lotus json-rpc interfaces that are auto-generated from the code.

The Good: 
- it is generated from the go rpc implementation, so is always up to date. 
- It could be used to generate api docs and (almost complete) typescript and rust clients.

The Bad: 
- Not yet auto-discoverable or useable with existing tools like https://playground.open-rpc.org (the api definition doc should be discoverable from the api url, but currently isn't, probably due to auth, TBC)

By manually extracting the definition files from the repo you can generate api docs from it:
https://playground.open-rpc.org/?url=https://gist.githubusercontent.com/olizilla/fc6abf836023b6ee662955695f1cfd21/raw/a4cb5aa3c6fbbc47bcf4b92d2370b8b012a810cb/full.json

API client generation is possible for Typescript and rust, but [it's rough](https://github.com/olizilla/lotus-openrpc-client/blob/main/client/typescript/src/index.ts#L280). This could be improved by adding more detail to the lotus openrpc definition (we should do that) but may also require improving the open-roc/generator project

Notably, we cannot define the file import api with open-rpc, as it only specifies how to define JSON payloads, so we will always need a custom step to create a client or docs that includes the file import api.

It could be used to simplify the custom generation code in https://github.com/filecoin-shipyard/js-lotus-client-schema

Exploration: https://github.com/olizilla/lotus-openrpc-client

Last updated: 31 March 2021 _(auto genereated from rpc code changes)_
### lotus-gateway

https://github.com/filecoin-project/lotus/tree/master/cmd/lotus-gateway

## filecoin-shipyard JavaScript libraries

### [js-lotus-client](https://github.com/filecoin-shipyard/js-lotus-client)

Last updated: 20 Jan 2021

#### https://github.com/filecoin-shipyard/js-lotus-client-rpc

#### https://github.com/filecoin-shipyard/js-lotus-client-schema

#### https://github.com/filecoin-shipyard/js-lotus-provider-browser

#### https://github.com/filecoin-shipyard/js-lotus-provider-nodejs

### [Filecoin.js](https://github.com/filecoin-shipyard/filecoin.js)

#### Maturity

The current state of the library specified in the README is `ALPHA: things will not work or work incorectly, will break and the API will change! ⚠️`

Taking into account the activity of the repo, it has not been in actively development since November 2020 (Lotus version: ?), relying on an outdated fork of `filecoin-signing-tools-js`. There is a [Roadmap Issue](https://github.com/filecoin-shipyard/filecoin.js/issues/1) where we can see that general Filecoin operations are no yet implemented.

When installing the module via npm latest, I got `"0.0.5-alpha"`even though github master `package.json` shows `"0.0.3-alpha"`.

#### What it supports

- Wallet Mnemonics
- Interact with traditional wallet providers like metamask (could not make it work)

The roadmap indicates that more general filecoin operations might get implemented in the future.

#### Overlaps

1. Exported `WsJsonRpcConnector` and `HttpJsonRpcConnector` seem the same as [@filecoin-shipyard/lotus-client-provider-nodejs](@filecoin-shipyard/lotus-client-provider-nodejs) + [@filecoin-shipyard/lotus-client-provider-browser](@filecoin-shipyard/lotus-client-provider-browser), which abstracts the WS or HTTP, but also supports both.
  - It is worth pointing out that even in the browser, `node-fetch` seems to be bundled and every time the window existence is checked.

#### Testing

Tests are failing to run, even by trying to run a shell script as done in CI. In the latest commits, CI did also not run.

#### Experiments

1. [Guides example](https://filecoin-shipyard.github.io/filecoin.js/docs/guides-example)

`filecoin.js` supports both HTTP RPC and WS RPC. There are some inconsistencies and the examples do not work with a simple copy-paste, due to variable naming differences.

At first, it was possible fetch the lotus version using an updated lotus instance. But fails using `wss://api.chain.love/rpc/v0` with `Error: method 'Filecoin.Version' not found`, which means there are some version inconsistencies.

2. [Setup Wallet provider](https://filecoin-shipyard.github.io/filecoin.js/docs/setup-lotus-provider)

Docs outdated on [Setup Example](https://filecoin-shipyard.github.io/filecoin.js/docs/setup-lotus-provider#setup-example), even though the docs website [markdown](https://github.com/filecoin-shipyard/filecoin.js/blob/master/documentation/src/07-setup-lotus-provider.md#setup-example) in the repo file seems up to date. `LotusWalletProvider` should be the new `HttpJsonRpcWalletProvider` and `getDefaultAccount` should be `getDefaultAddress`.

With the updates above, it is possible to get the wallet address. 

3. [Setup Mnemonic Provider](https://filecoin-shipyard.github.io/filecoin.js/docs/setup-mnemonic-provider)

In this case, both the website and the repo docs are outdated and like above `getDefaultAccount` should be `getDefaultAddress`.

The Mnemonic Provider value is not clear as there is no information on the repo/docs about it. After researching a bit on other resources I got the following definition:

"The mnemonic is used to derive multiple private keys. It is a unique wallet phrase that gives you a human readable format of words to backup your wallet for recovery. It is typically a phrase that allows you to access an infinite number of accounts. At its purest form it is a pattern of letters, words or associations that help you remember the information easily. It is used by Ledger, TREZOR, MetaMask, Jaxx and others." from [support.token.im](https://support.token.im/hc/en-us/articles/360002074233-What-is-Mnemonic-Phrase)

4. [Setup Metamask Provider](https://filecoin-shipyard.github.io/filecoin.js/docs/setup-metamask-provider)

This seems one of the most interesting providers, as it should aim to provide the same type of wallet experiences as blockchain users are used to.

After attempting to use this example (fix same issues as before), it ran but no wallet address was returned, despite the promise being resolved. Trying to go to the [alternative route](https://filecoin-shipyard.github.io/filecoin.js/docs/setup-metamask-provider#setup-metamask-provider) of installing locally Filecoin Metamask Snap, there is basically no documentation. After downloading the browser release, there was no documentation on how to actually use it.

Also found https://filsnap.netlify.app/ which seems interesting to investigate.

5. [Using a provider to send/sign messages](https://filecoin-shipyard.github.io/filecoin.js/docs/send-message)

The wallet messages documentation is incomplete. Wallet addresses are strings [per typedef](https://github.com/filecoin-shipyard/filecoin.js/blob/520005204628674a8b5bb974d51d7398fa1981e5/dist/filecoin.js.d.ts#L2360).

6. [Examples](https://github.com/filecoin-shipyard/filecoin.js/tree/520005204628674a8b5bb974d51d7398fa1981e5/documentation/examples)

Before running the examples, it is necessary to run `npm run bundle` as the examples statically import the dist folder generated from the webpack bundle.

The 4 examples do not run out of the box as they rely on an outdated API. After updating the API calls according to the changes described above, there were still other problems with non existing API calls.

Finally, the examples contain forms to fill in but is complex to understand what is needed, as well as its formats. It would be great that these examples have a documentation journey to help developers go through their usage.

#### Working code snippet

<details>
<summary>code</summary>

```js
'use strict'

// const __LOTUS_RPC_ENDPOINT__ = 'wss://api.chain.love/rpc/v0'
// const __LOTUS_AUTH_TOKEN__ =

const {
  WsJsonRpcConnector,
  LotusClient,
  LotusWalletProvider,
  MnemonicWalletProvider,
  MetamaskSnapHelper,
  MetamaskWalletProvider
} = require('filecoin.js')

;(async function () {
  const connector = new WsJsonRpcConnector({ url: __LOTUS_RPC_ENDPOINT__, token: __LOTUS_AUTH_TOKEN__ });
  const jsonRpcProvider = new LotusClient(connector);

  // 0
  const version = await jsonRpcProvider.common.version();
  console.log('version', version);

  // 1
  const walletProvider = new LotusWalletProvider(jsonRpcProvider);
  const myAddress = await walletProvider.getDefaultAddress();
  console.log('myAddress', myAddress)

  // 2
  const hdWalletMnemonic = 'equip ... young';
  const hdDerivationPath = `m/44'/461'/0'/0/0`;

  const walletProvider = new MnemonicWalletProvider(
    jsonRpcProvider,
    hdWalletMnemonic,
    hdDerivationPath
  );

  const myAddress = await walletProvider.getDefaultAddress();
  console.log(myAddress);

  // 3
  const metamaskHelper = new MetamaskSnapHelper({ url: __LOTUS_RPC_ENDPOINT__, token: __LOTUS_AUTH_TOKEN__ });
  const err = await metamaskHelper.installFilecoinSnap();

  const metamaskWalletProvider = new MetamaskWalletProvider(jsonRpcProvider, metamaskHelper.filecoinApi)

  let metamaskAddress
  try {
    metamaskAddress = await metamaskWalletProvider.getDefaultAddress();
  } catch (e) {
    console.log('metamask error', e);
  }

  console.log(metamaskAddress);
})();
```

</details>


Last updated: 17 Nov 2020

### [browser-retrieval](https://github.com/filecoin-shipyard/browser-retrieval)

### https://github.com/filecoin-shipyard/tour-de-lotus

> A slideshow tour of Lotus using the Lotus JS Client

> A slideshow built with mdx-deck and mdx-deck-live-code that connects to a real or simulated Lotus "local net" and uses Lotus JS Client to perform various API actions.

Last updated: 10 May 2020

### https://github.com/filecoin-shipyard/dumbo-drop

> Massive Parallel Graph Builder

Used to build Filecoin Discover data in AWS. Very custom use-case but likely contains interesting workflows for users building large graphs of data for storage in Filecoin.

Last updated: 1 Jul 2020

## filecoin-shipyard JavaScript Example Apps

Three of the example apps are currently described in the official Filecoin documentation at <https://docs.filecoin.io/build/examples/>. However, they have not been actively maintained and rely on older components that are unlikely to still be operaitonal. In addition, the linked examples focus on small-deals, which are currently not optimal for Filecoin deal-making.

We should consider updating the documentation for these examples to suggest their state as operational against pre-mainnet Filecoin and relying on possibly outdated components. They should also be removed from docs.filecoin.io until we can update them and ensure they demonstrate ideal working conditions for Filecoin.

filecoin-shipyard/filecoin-network-inspector may be the example application most worth focusing on updating since it doesn't rely on small deals and exercises basic parts of the Lotus RPC.

### https://github.com/filecoin-shipyard/meme-marketplace

NFT example app using Textile Hub

https://docs.filecoin.io/build/examples/meme-marketplace/overview/

Relies on the `filecoin-shipyard/powergate-pinning-service` example app.

Forked from https://github.com/textileio/js-examples but just using that as the base.

Last updated: 15 Sep 2020

### https://github.com/filecoin-shipyard/meme-nft-token

ERC 721 token smart contract for meme-marketplace example app.

Last updated: 5 Sep 2020

### https://github.com/filecoin-shipyard/powergate-pinning-service

Example app pinning service backed by Powergate

https://docs.filecoin.io/build/examples/simple-pinning-service/overview/

Example app setup doc points to an 0.0.1-beta.10 powergate-docker ZIP for "Docker+ devnet". Does it still work?

Last updated: 16 Jul 2020

### https://github.com/filecoin-shipyard/NFT-Snapshot-Bot

> Filecoin and ethereum-based twitter thread tokenisation bot.

Last updated: 10 Aug 2020

### https://github.com/filecoin-shipyard/filecoin-network-inspector

Example app implementing a Filecoin network inspector

https://docs.filecoin.io/build/examples/network-inspector/overview/

Relies on https://github.com/textileio/lotus-devnet which is not updated since 13 Nov 2020. Does it still work?

Last updated: 13 Jul 2020

### https://github.com/filecoin-shipyard/js-lotus-client-workshop

A series of workshop exercises intended to demonstrate the use of lotus-client-rpc in the browser.

* 00-select-node
* 01-chain-height
* 02-miner-address
* 03-chain-notify
* 04-state-power-all
* 05-state-list-miners
* 06-state-power-miners
* 07-camera
* 08-deals
* 09-retrieve

Last updated: 3 Sep 2020

### https://github.com/filecoin-shipyard/filecoin-tipset-visualizer

??

Digital MOB grant project.

Last updated: 19 Nov 2020

## filecoin-shipyard JavaScript go-filecoin Projects

Mostly based on the unmaintained `filecoin-api-client` library. These libraries have not been touched since September 2019, based on an old version of go-filecoin and may not have any utility even with venus, the renamed go-filecoin, which has evolved significantly since September 2019.

These projects should be placed into the GitHub "archived" state, with clear notices in their README about their status as not up-to-date examples. Further development work should not be ruled out on these projects, and a path to re-starting them should be documented in their README (e.g. "Contact ... at ... if you would like to re-start this project and contribute fixes to make it usable").

### https://github.com/filecoin-shipyard/js-filecoin-api-client

API client for go-filecoin.

We should ask the maintainers of Venus if they would like to do anything with this in the short-term, if not, it should be archived.

Last updated: 15 Sep 2019

### https://github.com/filecoin-shipyard/filecoin-browse-asks

> A simple command line tool to view the available Filecoin asks on the network.

Last updated: 28 Sep 2019

### https://github.com/filecoin-shipyard/filecoin-big-head

> A simple command line tool to display the height of the Filecoin blockchain that your node has synced.

Last updated: 28 Sep 2019

### https://github.com/filecoin-shipyard/use-filecoin-network-info

> React hook to query network info via Filecoin API

Last updated: 28 Sep 2019

### https://github.com/filecoin-shipyard/use-filecoin-head

> React hook to query head blocks and chain height via Filecoin API

Last updated: 28 Sep 2019

### https://github.com/filecoin-shipyard/use-filecoin-config

> React hook to load config via Filecoin API

Last updated: 28 Sep 2019

### https://github.com/filecoin-shipyard/filecoin-pickaxe

> This is a command-line tool for managing "bundles" (groups of files) that you would like to import into Filecoin.

Last updated: 29 Sep 2019

#### https://github.com/filecoin-shipyard/filecoin-pickaxe-direct-deal

> This is a command-line tool for interactively storing "bundles" of files from filecoin-pickaxe into the Filecoin network.

Last updated: 28 Sep 2019

#### https://github.com/filecoin-shipyard/filecoin-pickaxe-agent

> Agent to perform requested Filecoin tasks

Last updated: 7 Sep 2019

### https://github.com/filecoin-shipyard/npm-go-filecoin-dep

> Download go-filecoin to your node_modules.

Last updated: 1 Sep 2019

## Ecosystem libraries

### Glif.io

https://www.glif.io/

#### https://github.com/glifio/wallet

#### https://github.com/glifio/modules/tree/primary/packages

Dependents of https://github.com/glifio/wallet, some are also useful by themselves.

##### filecoin-address
##### filecoin-message-confirmer
##### filecoin-message
##### filecoin-number

Was https://github.com/glifio/filecoin-number

##### filecoin-rpc-client
##### filecoin-wallet-provider
##### local-managed-provider
##### react-components

### https://github.com/CoinSummer/lotus-jsonrpc-provider

TypeScript Lotus RPC client. Not up to date, likely not used.

Last updated: 1 Nov 2020


### https://github.com/keyko-io/filecoin-verifier-tools

### https://github.com/Zondax/filecoin-signing-tools

### https://github.com/mgoelzer/zondax-pch-demo

### https://github.com/filecoin-project/starling

### https://github.com/spacegap/spacegap.github.io

### https://github.com/trufflesuite/ganache-filecoin-alpha-cli/

Work in progress Ganache flavored Truffle
