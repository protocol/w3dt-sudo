# Lotus / Filecoin RPC and JavaScript Library Audit

**2021-04**

This report captures a condensed summary of a brief audit conducted of the current state of the JavaScript ecosystem surrounding Filecoin and the affordances provided to JavaScript developers via the Lotus RPC API.

## Lotus RPC API

### OpenRPC support

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