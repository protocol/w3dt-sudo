# [filecoin.js](https://github.com/filecoin-shipyard/filecoin.js)

## Maturity

The current state of the library specified in the README is `ALPHA: things will not work or work incorectly, will break and the API will change! ⚠️`

Taking into account the activity of the repo, it has not been in actively development since November 2020 (Lotus version: ?), relying on an outdated fork of `filecoin-signing-tools-js`. There is a [Roadmap Issue](https://github.com/filecoin-shipyard/filecoin.js/issues/1) where we can see that general Filecoin operations are no yet implemented.

When installing the module via npm latest, I got `"0.0.5-alpha"`even though github master `package.json` shows `"0.0.3-alpha"`.

## What it enables

- Wallet Mnemonics
- Interact with traditional wallet providers like metamask (could not make it work)

The roadmap indicates that more general filecoin operations might get implemented in the future.

## Same Features/Modules

1. Exported `WsJsonRpcConnector` and `HttpJsonRpcConnector` seem the same as [@filecoin-shipyard/lotus-client-provider-nodejs](@filecoin-shipyard/lotus-client-provider-nodejs) + [@filecoin-shipyard/lotus-client-provider-browser](@filecoin-shipyard/lotus-client-provider-browser), which abstracts the WS or HTTP, but also supports both.
  - It is worth pointing out that even in the browser, `node-fetch` seems to be bundled and every time the window existence is checked.

## Experiments

### Testing

Tests are failing to run, even by trying to run a shell script as done in CI. In the latest commits, CI did also not run.

### [Guides example](https://filecoin-shipyard.github.io/filecoin.js/docs/guides-example)

`filecoin.js` supports both HTTP RPC and WS RPC. There are some inconsistencies and the examples do not work with a simple copy-paste, due to variable naming differences.

At first, I tried to use our own lotus instance, which worked to fetch the lotus version. But then tried `wss://api.chain.love/rpc/v0` with `Error: method 'Filecoin.Version' not found`, which means there are some version inconsistencies.

### [Setup Wallet provider](https://filecoin-shipyard.github.io/filecoin.js/docs/setup-lotus-provider)

Docs outdated on [Setup Example](https://filecoin-shipyard.github.io/filecoin.js/docs/setup-lotus-provider#setup-example), even though the docs website [markdown](https://github.com/filecoin-shipyard/filecoin.js/blob/master/documentation/src/07-setup-lotus-provider.md#setup-example) in the repo file seems up to date. `LotusWalletProvider` should be the new `HttpJsonRpcWalletProvider` and `getDefaultAccount` should be `getDefaultAddress`.

With the updates above, I could get this to work and get the wallet address. 

### [Setup Mnemonic Provider](https://filecoin-shipyard.github.io/filecoin.js/docs/setup-mnemonic-provider)

In this case, both the website and the repo docs are outdated and like above `getDefaultAccount` should be `getDefaultAddress`.

The Mnemonic Provider value is not clear as there is no information on the repo/docs about it. After researching a bit on other resources I got the following definition:

"The mnemonic is used to derive multiple private keys. It is a unique wallet phrase that gives you a human readable format of words to backup your wallet for recovery. It is typically a phrase that allows you to access an infinite number of accounts. At its purest form it is a pattern of letters, words or associations that help you remember the information easily. It is used by Ledger, TREZOR, MetaMask, Jaxx and others." from [support.token.im](https://support.token.im/hc/en-us/articles/360002074233-What-is-Mnemonic-Phrase)

### [Setup Metamask Provider](https://filecoin-shipyard.github.io/filecoin.js/docs/setup-metamask-provider)

This seems one of the most interesting providers, as it should aim to provide the same type of wallet experiences as blockchain users are used to.

After attempting to use this example (fix same issues as before), it ran but no wallet address was returned, despite the promise being resolved. Trying to go to the [alternative route](https://filecoin-shipyard.github.io/filecoin.js/docs/setup-metamask-provider#setup-metamask-provider) of installing locally Filecoin Metamask Snap, there is basically no documentation. After downloading the browser release, I could not do anything with it.

Also found https://filsnap.netlify.app/ which seems interesting to investigate.

### [Using a provider to send/sign messages](https://filecoin-shipyard.github.io/filecoin.js/docs/send-message)

The wallet messages documentation is incomplete. Wallet addresses are strings [per typedef](https://github.com/filecoin-shipyard/filecoin.js/blob/520005204628674a8b5bb974d51d7398fa1981e5/dist/filecoin.js.d.ts#L2360). The [examples](https://github.com/filecoin-shipyard/filecoin.js/tree/master/documentation/examples) should help more, even though there are no instructions on how to use them.

### [Examples](https://github.com/filecoin-shipyard/filecoin.js/tree/520005204628674a8b5bb974d51d7398fa1981e5/documentation/examples)

Before running the examples, it is necessary to run `npm run bundle` as the examples statically import the dist folder generated from the webpack bundle.

The 4 examples do not run out of the box as they rely on an outdated API. After updating the API calls according to the changes also made on the docs above, there were still other problems with non existing API calls.

Finally, the examples contain forms to fill in but is complex to understand what is needed, as well as its formats. It would be great that these examples have a documentation journey to help developers go through their usage.

## Code used

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

## Fixes upstream / Issues

- [filecoin-shipyard/filecoin.js#32](https://github.com/filecoin-shipyard/filecoin.js/pull/32)
- [filecoin-shipyard/filecoin.js#33](https://github.com/filecoin-shipyard/filecoin.js/pull/33)
- [filecoin-shipyard/filecoin.js#35](https://github.com/filecoin-shipyard/filecoin.js/issues/35)
