# Lotus / Filecoin RPC and JavaScript Library Audit

**2021-04/05** by [Protocol Labs W3DT Team Sudo](https://github.com/protocol/w3dt-sudo).

This report captures a condensed summary of a brief audit conducted of the current state of the JavaScript ecosystem surrounding Filecoin and the affordances provided to JavaScript developers via the Lotus RPC API. The audit was conducted to increase familiarity with the current Filecoin JavaScript ecosystem and to inform possible future work in this space to increase value for web3 developers building on Filecoin.

Action items and recommendations in this report can be identified by the following labels:

* ***(TODO TASK)*** - A small task that is less than project-sized but recommended.
* ***(POSSIBLE PROJECT)*** - A larger set of work that could be scoped into a project for Protocol Labs' W3DT or a Devgrant.
* ***(NEEDS INVESTIGATION)*** - Further investigation work necessary or recommended.

## Contents

* [Contents](#contents)
* [High-level Summary of Findings](#high-level-summary-of-findings)
* [Lotus RPC API](#lotus-rpc-api)
  * [General RPC API Findings](#general-rpc-api-findings)
  * [OpenRPC support](#openrpc-support)
  * [lotus-gateway](#lotus-gateway)
* [filecoin-shipyard JavaScript libraries](#filecoin-shipyard-javascript-libraries)
  * [js-lotus-client](#js-lotus-client)
  * [Filecoin.js](#filecoinjs)
  * [browser-retrieval](#browser-retrieval)
  * [https://github.com/filecoin-shipyard/tour-de-lotus](#httpsgithubcomfilecoin-shipyardtour-de-lotus)
  * [https://github.com/filecoin-shipyard/dumbo-drop](#httpsgithubcomfilecoin-shipyarddumbo-drop)
* [filecoin-shipyard JavaScript Example Apps](#filecoin-shipyard-javascript-example-apps)
  * [Recommendations](#recommendations)
  * [https://github.com/filecoin-shipyard/meme-marketplace](#httpsgithubcomfilecoin-shipyardmeme-marketplace)
  * [https://github.com/filecoin-shipyard/meme-nft-token](#httpsgithubcomfilecoin-shipyardmeme-nft-token)
  * [https://github.com/filecoin-shipyard/powergate-pinning-service](#httpsgithubcomfilecoin-shipyardpowergate-pinning-service)
  * [https://github.com/filecoin-shipyard/NFT-Snapshot-Bot](#httpsgithubcomfilecoin-shipyardnft-snapshot-bot)
  * [https://github.com/filecoin-shipyard/filecoin-network-inspector](#httpsgithubcomfilecoin-shipyardfilecoin-network-inspector)
  * [https://github.com/filecoin-shipyard/js-lotus-client-workshop](#httpsgithubcomfilecoin-shipyardjs-lotus-client-workshop)
  * [https://github.com/filecoin-shipyard/filecoin-tipset-visualizer](#httpsgithubcomfilecoin-shipyardfilecoin-tipset-visualizer)
* [filecoin-shipyard JavaScript go-filecoin Projects](#filecoin-shipyard-javascript-go-filecoin-projects)
  * [Recommendations](#recommendations-1)
  * [https://github.com/filecoin-shipyard/js-filecoin-api-client](#httpsgithubcomfilecoin-shipyardjs-filecoin-api-client)
  * [https://github.com/filecoin-shipyard/filecoin-browse-asks](#httpsgithubcomfilecoin-shipyardfilecoin-browse-asks)
  * [https://github.com/filecoin-shipyard/filecoin-big-head](#httpsgithubcomfilecoin-shipyardfilecoin-big-head)
  * [https://github.com/filecoin-shipyard/use-filecoin-network-info](#httpsgithubcomfilecoin-shipyarduse-filecoin-network-info)
  * [https://github.com/filecoin-shipyard/use-filecoin-head](#httpsgithubcomfilecoin-shipyarduse-filecoin-head)
  * [https://github.com/filecoin-shipyard/use-filecoin-config](#httpsgithubcomfilecoin-shipyarduse-filecoin-config)
  * [https://github.com/filecoin-shipyard/filecoin-pickaxe](#httpsgithubcomfilecoin-shipyardfilecoin-pickaxe)
  * [https://github.com/filecoin-shipyard/npm-go-filecoin-dep](#httpsgithubcomfilecoin-shipyardnpm-go-filecoin-dep)
* [Ecosystem libraries](#ecosystem-libraries)
  * [Glif.io](#glifio)
  * [https://github.com/CoinSummer/lotus-jsonrpc-provider](#httpsgithubcomcoinsummerlotus-jsonrpc-provider)
  * [https://github.com/keyko-io/filecoin-verifier-tools](#httpsgithubcomkeyko-iofilecoin-verifier-tools)
  * [Zondax/filecoin-signing-tools](#zondaxfilecoin-signing-tools)
  * [https://github.com/mgoelzer/zondax-pch-demo](#httpsgithubcommgoelzerzondax-pch-demo)
  * [https://github.com/filecoin-project/starling](#httpsgithubcomfilecoin-projectstarling)
  * [https://github.com/spacegap/spacegap.github.io](#httpsgithubcomspacegapspacegapgithubio)
  * [https://github.com/trufflesuite/ganache-filecoin-alpha-cli/](#httpsgithubcomtrufflesuiteganache-filecoin-alpha-cli)


## High-level Summary of Findings

**[OpenRPC support](https://github.com/filecoin-project/lotus/tree/master/build/openrpc)** support was grant-funded and landed in Lotus. It should allow clearer and auto-generated documentation of the API and auto-generation of clients in a variety of languages. Current status: it's basic and needs more work to be useful.
  * Code documentation extracted for API docs could be greatly improved.
  * Optional parameters are not identified, causing problems for typed language clients.
  * Lotus auth currently prevents auto doc and code generation from a live node.
  * The file import API is separate and difficult to integrate, so we have two APIs.
  * We should turn the js-lotus-client into an OpenRPC consumer to dogfood this and simplify our codegen paths.

**[js-lotus-client](https://github.com/filecoin-shipyard/js-lotus-client/)** is the most mature JavaScript API client for Lotus and can be used in Node.js and the browser. It uses a custom codegen solution to extract the API, docs and TypeScript definitions.
  * We need to invest in keep this updated, perhaps automatic releases and switch to OpenRPC codegen.
  * File import API poses usability challenges and is unclear to users.
  * Building out a suite of clear and practical example applications would be an excellent investment.
  * Optional parameters are not identified, causing problems for typed language clients.

**[Filecoin.js](https://github.com/filecoin-shipyard/filecoin.js)** is a grant-funded attempt at a higher-level, wallet-focused API aiming toward users familiar with Ethers.js. It is very early-stage and has been put on hold due to lack of adoption, interest and practical hurdles regarding the wallet & API interaction.
  * This project should not be used as it is and we need to make this clear in documentation.
  * It needs more planning and scoping before the work is continued—better integration with other tooling, including js-lotus-client, would be preferable.
  * The Ethers.js user-market is an important one and we should revisit prioritisation when we have improved some of the Filecoin fundamentals that are of interest to the JavaScript Dapp audience.

**filecoin-shipyard JavaScript Example Apps** include a suite of grant-funded example applications built on pre-launch Filecoin, using pre-launch assumptions about what will be possible, practical and reasonable. They include **two separate NFT marketplace example applications**. Most of the examples are not what we want to be emphasising as ideal workflows for Filecoin with our post-launch understanding, yet.
  * The example applications are all out of date and use old libraries and toolchains (including beta versions of Textile dev tools).
  * As part of this project we had them removed from docs.filecoin.io where they were still listed as go-to examples of how to build on Filecoin.
  * [filecoin-network-inspector](https://github.com/filecoin-shipyard/filecoin-network-inspector) has merit as a project to revive and demonstrate js-lotus-client's capabilities.
  * [js-lotus-client-workshop](https://github.com/filecoin-shipyard/js-lotus-client-workshop) was Jim Pick's Workshopper / Protoschool style teaching workshop for js-lotus-client. It has some excellent material that should be revived, finished, kept updated and promoted.

**go-filecoin Projects** in filecoin-shipyard include at least 10 exaple apps and JavaScript API libraries built for go-filecoin - now Venus. They have not been kept up to date, and their documentation does not make clear what they are or that they should not be used!
  * We should ask the Venus team whether they would like to take responsibility for any of these projects, and for those that are abandonware we should archive them and make their status clear in their READMEs.

**[Glif.io and its libraries](https://github.com/glifio)** provide some excellent example code for interacting with Filecoin and the Lotus API. It's actively used and kept updated because it's critical infrastructure for most FIL holders. It would be worth exploring how we can highlight, improve, or even replace some of the highest value libraries:
  * [@glif/filecoin-number](https://github.com/glifio/modules/tree/primary/packages/filecoin-number) - a FIL units converter
  * [@glif/filecoin-address](https://github.com/glifio/modules/tree/primary/packages/filecoin-address) - a JavaScript sister to go-address.
  * [@glif/filecoin-message-confirmer](https://github.com/glifio/modules/tree/primary/packages/filecoin-message-confirmer) - confirms messages are on-chain, has some API concerns around the duplicate message problems exchanges had recently with `StateSearchMsg` and friends.
  * <https://github.com/glifio/wallet> has potential to be a great example app as it uses a popular JavaScript application stack. It would need investment in documentation and approachability to make this so.

**[Zondax/filecoin-signing-tools](https://github.com/zondax/filecoin-signing-tools)** is one of the most heavily used JavaScript libraries dedicated to Filecoin and are used to sign messages prior to submission to chain. Users include Glif.io, Ceramic, MetaMask FilSnap and Filecoin.js. It's both an important tool for our ecosystem and deserves investment and improvement but also a potentially excellent example library for understanding the basics of dealing with Filecoin in JavaScript.
  * We should explore ways to invest in improving this project for use and to potentially make it an example library we highlight in our documentation.
  * The state of WASM vs pure-JS implementation of crypto appears incomplete, there may be work to do here to streamline it for the various use-cases.


## Lotus RPC API

### General RPC API Findings

The majority of investigation into the Lotus RPC API was conducted via the [js-lotus-client](#js-lotus-client) library and the [OpenRPC support](#openrpc-support). Action items relating to Lotus can be found in the sections below.

### [OpenRPC support](https://github.com/filecoin-project/lotus/tree/master/build/openrpc)

> JSON based API definition of the lotus json-rpc interfaces that are auto-generated from the code.

[Developed](https://github.com/filecoin-project/lotus/pull/4711) under a [Devgrant](https://github.com/filecoin-project/devgrants/pull/165) and [merged in Lotus](https://github.com/filecoin-project/lotus/pull/5843) in late March, 2021.

#### Maturity

The Good:
- It is generated from the Go RPC implementation, so is always up to date.
- It could be used to generate API docs and (almost complete) TypeScript and Rust clients.

The Bad:
- TypeScript declarations do not take optional parameters into consideration, which makes all parameters required in a TypeScript project.
- Not yet auto-discoverable or usable with existing tools like https://playground.open-rpc.org (the API definition doc should be discoverable from the API, but currently isn't, perhaps due to auth, TBC).

Last updated: 31 March 2021 _(auto genereated from RPC code changes)_

https://github.com/protocol/w3dt-sudo/pull/16#issuecomment-820864315

#### Key Findings

N/A

#### Recommendations

* ***(NEEDS INVESTIGATION)*** We should investigate and enable auto-discovery and doc generation from a Lotus endpoint URL. As a workaround, it can be done by manually extracting the definition files, e.g.:
https://playground.open-rpc.org/?url=https://gist.githubusercontent.com/olizilla/fc6abf836023b6ee662955695f1cfd21/raw/a4cb5aa3c6fbbc47bcf4b92d2370b8b012a810cb/full.json
* API client generation is possible for TypeScript and Rust, but [it's rough](https://github.com/olizilla/lotus-openrpc-client/blob/main/client/typescript/src/index.ts#L280). This could be improved by adding more detail to the Lotus OpenRPC definition (we should do that) but may also require improving the OpenRPC/generator project.
  * Notably, we cannot define the Lotus file import API with OpenRPC, as it only specifies how to define JSON payloads and the file import API uses a separate workflow, so we will always need a custom step to create a client or docs that includes the file import API.
  * Some [input on this has been provided by an OpenRPC maintainer](https://github.com/protocol/w3dt-sudo/pull/16#issuecomment-820864315) in response to our investivation and we may see improvements upstream that unlock more potential here.
* ***(POSSIBLE PROJECT)*** We should simplify the custom code generation for https://github.com/filecoin-shipyard/js-lotus-client-schema and using a single method would be ideal, since we have OpenRPC it may be the best approach to reduce the number of toolchains we need and use.

#### Experiments

Exploration: https://github.com/olizilla/lotus-openrpc-client

### lotus-gateway

https://github.com/filecoin-project/lotus/tree/master/cmd/lotus-gateway

*We did not cover the lotus-gateway during our investigation but it remains relevant and should be included in any further investigation.*

## filecoin-shipyard JavaScript libraries

Many of the libraries in filecoin-shipyard were not investigated in depth as they are largely unmaintained and are primarily designed for narrow feature-set and/or example use.

***(TODO TASK)*** Due to the fast-changing nature of the Filecoin protocol and Lotus, as well as the fact that many of these libraries are built using premature assumptions about usability of the network (e.g. deal size), it is likely that any project listed below that has not been updated during 2021 is out of date and should be marked as archived. We should be careful to set expectations to users about what they can expect from these projects.

### [js-lotus-client](https://github.com/filecoin-shipyard/js-lotus-client)

> The [lotus JS client](https://github.com/filecoin-shipyard/js-lotus-client) is a collection of small JS libraries, which enables developers to interact with Lotus via it’s JSON-RPC API.

It is decoupled in:

- [Schema](https://github.com/filecoin-shipyard/js-lotus-client-schema): contains schemas for JSON data representation compatible with Lotus JSON-RPC API.
- Provider libraries ([Node.js](https://github.com/filecoin-shipyard/js-lotus-client-provider-nodejs) and [browser](https://github.com/filecoin-shipyard/js-lotus-client-provider-browser)): inspired by “providers” in the Ethereum ecossystem, a provider links to a running node exposing interface that connects to a Lotus JSON-RPC API endpoint using WebSockets or HTTP.
- [RPC](https://github.com/filecoin-shipyard/js-lotus-client-rpc): used together with a schema for creating the RPC messages from function parameters.

#### Maturity

The Good:
- It is generated from the Go RPC implementation, so is always up to date (as long as the generation is re-run). See the two [`go-`](https://github.com/filecoin-shipyard/js-lotus-client-schema) directories in [js-lotus-client-schem](https://github.com/filecoin-shipyard/js-lotus-client-schema) for the custom codegen.
- It could be used to generate API docs and (almost complete) TypeScript declarations.

The Bad:
- TypeScript declarations do not have into consideration optional parameters, which makes them required in a TypeScript project.
- A large number of unanswered open issues.

#### Key Findings

* The `client.clientDealSize` seems problematic. The expectation per the naming would be to get the actual pieceSize to provide, but getting: `unexpected deal status while waiting for data request: 11 (StorageDealFailing). Provider message: deal rejected: proposal piece size is invalid: padded piece size must be a power of 2`
* List miners takes a really long time.
* The different workflow required for the file import API is not clear, especially to those not familiar with Lotus mechanics.
* Authentication tokens requirements seem too strict.
  * Some methods, including `listDeals` need write permissions. `ClientStartDeal` needs admin permissions.
  * The RPC API seems to target users who own a lotus instance, instead of a general user. But, in the client this is not clear at all.

#### Recommendations

* ***(POSSIBLE PROJECT)*** Build guided examples where it is clear to the user what needs to be changed and what happens behind the scenes
  * A simple example of creating a deal should exist, including how to import a file using the Provider which is not clear unless one asks someone with knowledge on creating deals.
* ***(POSSIBLE PROJECT)*** We should identify optional parameters and default values in the docs and TypeScript definitions. It is not clear which parameters are required and optional. Moreover, TypeScript use is impractical without this clarification.
* ***(POSSIBLE PROJECT)*** Custom codegen could be completely (or mostly?) replaced with OpenRPC codegen. See above.
* ***(POSSIBLE PROJECT)*** Codegen in https://github.com/filecoin-shipyard/js-lotus-client-schema should be performed on each major Lotus release, with a new npm version published. A GitHub action to PR changes seems like a reasonable step, but it also needs a clear *owner* to ensure versions get published.

#### Experiments

Explorations:

- [connect-lotus-client-with-deployed-lotus-instance.md](https://github.com/protocol/w3dt-sudo/blob/master/home/vasco-santos/2021-02-exploration-report-connect-lotus-client-with-deployed-lotus-instance.md)
- [js-lotus-client.md](https://github.com/protocol/w3dt-sudo/blob/master/home/vasco-santos/2021-03-exploration-report-1-on-js-lotus-client.md)

### [Filecoin.js](https://github.com/filecoin-shipyard/filecoin.js)

> A JavaScript (and TypeScript) library for interacting with the Filecoin's Lotus node, with support for external signers.

This work was funded by a Devgrant, intending to be familiar to [ethers.js](https://github.com/ethers-io/ethers.js/) users, but development was paused due to lack of adoption and some difficulties with the message handling/signing workflow.

#### Maturity

The current state of the library specified in the README is `ALPHA: things will not work or work incorectly, will break and the API will change! ⚠️`

Taking into account the activity of the repo, it has not been in actively development since November 2020, relying on an outdated fork of `filecoin-signing-tools-js`. There is a [Roadmap Issue](https://github.com/filecoin-shipyard/filecoin.js/issues/1) where we can see that general Filecoin operations are no yet implemented.

The latest version in npm is `"0.0.5-alpha"`, however the `package.json` in GitHub shows `"0.0.3-alpha"`, so there may be more recent code not pushed to GitHub.

Last updated: 17 Nov 2020

#### Key Findings

- Supports Wallet Mnemonics
- The roadmap indicates that more general Filecoin operations might get implemented in the future.
- Tests are failing to run, even by trying to run a shell script as done in CI. In the latest commits, CI did also not run.

#### Recommendations

***(NEEDS INVESTIGATION)*** Look into what is happening with the versioning and whether there's code not pushed to GitHub.

***(POSSIBLE PROJECT)*** Consider re-starting this work to make a more complete JavaScript API library for dapp developers. Needs planning and scoping.

Minor items arising out of investigation that could be followed up within a project:

* Exported `WsJsonRpcConnector` and `HttpJsonRpcConnector` seem the same as [@filecoin-shipyard/lotus-client-provider-nodejs](https://github.com/filecoin-shipyard/js-lotus-client-provider-nodejs) + [@filecoin-shipyard/lotus-client-provider-browser]([@filecoin-shipyard/lotus-client-provider](https://github.com/filecoin-shipyard/js-lotus-client-provider)-browser), which abstracts the WS or HTTP, but also supports both.
  * We should converge the providers/connectors into a single library
* An isomorphic fetch implementation should be used
* Interact with traditional wallet providers like metamask (could not get it up and running)
  * We should work towards a frictionless experience and/or a general improvement on the documentation for this core functionality provided

#### Experiments

See https://github.com/protocol/w3dt-sudo/blob/master/home/vasco-santos/2021-03-audit-filecoinjs.md for an in-depth exploration and experiments in attempting to use Filecoin.js

### [browser-retrieval](https://github.com/filecoin-shipyard/browser-retrieval)

> The Filecoin Browser Retrieval Market is a browser-based p2p network that functions like a CDN for content that was originally retrieved from the Filecoin storage network.

Last updated: 23 Dec 2020

### https://github.com/filecoin-shipyard/tour-de-lotus

> A slideshow tour of Lotus using the Lotus JS Client

> A slideshow built with mdx-deck and mdx-deck-live-code that connects to a real or simulated Lotus "local net" and uses Lotus JS Client to perform various API actions.

Last updated: 10 May 2020

### https://github.com/filecoin-shipyard/dumbo-drop

> Massive Parallel Graph Builder

Used to build Filecoin Discover data in AWS. Very custom use-case but likely contains interesting workflows for users building large graphs of data for storage in Filecoin.

Last updated: 1 Jul 2020

## filecoin-shipyard JavaScript Example Apps

> Three of the example apps are currently described in the official Filecoin documentation at <https://docs.filecoin.io/build/examples/>. However, they have not been actively maintained and rely on older components that are unlikely to still be operaitonal. In addition, the linked examples focus on small-deals, which are currently not optimal for Filecoin deal-making.

### Recommendations

We should consider updating the documentation for these examples to suggest their state as operational against pre-mainnet Filecoin and relying on possibly outdated components. ~~They should also be removed from docs.filecoin.io until we can update them and ensure they demonstrate ideal working conditions for Filecoin.~~ (They were removed during the compilation of this report).

### https://github.com/filecoin-shipyard/meme-marketplace

> NFT example app using Textile Hub

https://docs.filecoin.io/build/examples/meme-marketplace/overview/

Relies on the `filecoin-shipyard/powergate-pinning-service` example app.

Forked from https://github.com/textileio/js-examples but just using that as the base.

Last updated: 15 Sep 2020

### https://github.com/filecoin-shipyard/meme-nft-token

> ERC 721 token smart contract for meme-marketplace example app.

Last updated: 5 Sep 2020

### https://github.com/filecoin-shipyard/powergate-pinning-service

> Example app pinning service backed by Powergate

https://docs.filecoin.io/build/examples/simple-pinning-service/overview/

Example app setup doc points to an 0.0.1-beta.10 powergate-docker ZIP for "Docker+ devnet". Does it still work?

Last updated: 16 Jul 2020

### https://github.com/filecoin-shipyard/NFT-Snapshot-Bot

> Filecoin and ethereum-based twitter thread tokenisation bot.

Last updated: 10 Aug 2020

### https://github.com/filecoin-shipyard/filecoin-network-inspector

> Example app implementing a Filecoin network inspector

https://docs.filecoin.io/build/examples/network-inspector/overview/

Relies on https://github.com/textileio/lotus-devnet which is not updated since 13 Nov 2020. Does it still work?

Last updated: 13 Jul 2020

***(POSSIBLE PROJECT)*** This may be the example application most worth focusing on updating since it doesn't rely on small deals and exercises basic parts of the Lotus RPC.

### https://github.com/filecoin-shipyard/js-lotus-client-workshop

> A series of workshop exercises intended to demonstrate the use of lotus-client-rpc in the browser.

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

***(POSSIBLE PROJECT)*** This series of workshops is still valid for the [js-lotus-client](#js-lotus-client) library even though it is now out of date. The lessons are useful for demonstrating the feature-set and may be worth investing in to update and keep maintained.

### https://github.com/filecoin-shipyard/filecoin-tipset-visualizer

Digital MOB grant project.

Last updated: 19 Nov 2020

## filecoin-shipyard JavaScript go-filecoin Projects

Mostly based on the unmaintained `filecoin-api-client` library. These libraries have not been touched since September 2019, based on an old version of go-filecoin and may not have any utility even with venus, the renamed go-filecoin, which has evolved significantly since September 2019.

### Recommendations

***(TODO TASK)*** These projects should be placed into the GitHub "archived" state, with clear notices in their README about their status as not up-to-date examples. Further development work should not be ruled out on these projects, and a path to re-starting them should be documented in their README (e.g. "Contact ... at ... if you would like to re-start this project and contribute fixes to make it usable").

### https://github.com/filecoin-shipyard/js-filecoin-api-client

> API client for go-filecoin.

#### Recommendations

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

### [Glif.io](https://www.glif.io/)

> _interoperable tools for the Filecoin network_

Last updated 19 April 2021 

#### https://github.com/glifio/wallet

> A lightweight web interface to send and receive Filecoin via your Ledger device

##### Key Findings

It's a production app. It's not directly re-usable and it's not yet suffciently documented to be an introductory educational resource, but it is useful as a well built and tested example of an app that talks to Lotus, does something useful, and uses a popular UI stack (next.js, react, redux).

##### Recommendations

* ***(POSSIBLE PROJECT)*** Adding an architecture diagram and an explanation of how the parts fit together would go along way to making wallet an educational resource for web3 app developers.

#### https://github.com/glifio/modules/tree/primary/packages

> Dependents of https://github.com/glifio/wallet, some are also useful by themselves. It's a lerna repo, commits every month, and the project builds and works out of the box. Each module is explored below

##### [@glif/filecoin-address](https://github.com/glifio/modules/tree/primary/packages/filecoin-address)

> This is a JS implementation of the Filecoin address type, inspired by go-address. It can create new address instances and encode addresses, and it takes care of decoding and validating checksums.

This is a good library, and serves an important purpose. We could consider adopting it into the filecoin-project org to give more visibilty, or at a minimum enhancing our documentation to make it obvious for web3 app developers ***(TODO TASK)***.

##### [@glif/filecoin-message-confirmer](https://github.com/glifio/modules/tree/primary/packages/filecoin-message-confirmer)

Poll the StateSearchMsg API method to verify that a specfied message CID appears on-chain with a 0 exit-code reciept. Uses `@glif/filecoin-rpc-client`.

A reasonable solution given the current API. 

###### Recommendations

* ***(POSSIBLE PROJECT)*** We should explore a better API for waiting for a message to succesfully execute.
- ***(TODO TASK)*** Need to fix https://github.com/glifio/modules/issues/65 ("filecoin-message-confirmer should verify the response message CID")

##### [@glif/filecoin-message](https://github.com/glifio/modules/tree/primary/packages/filecoin-message)

Normalises the casing of message object keys between Lotus and Zondax. If you need to talk to both, you are going to want to avoid dealing with the fact that lotus uses upper-camel case `GasFeeCap` and zondax uses all lower case `gasfeecap`.

##### [@glif/filecoin-number](https://github.com/glifio/modules/tree/primary/packages/filecoin-number)

A wrapper class for bignumber.js to represent an amount of FIL which may be larger than `Number.MAX_SAFE_INTEGER` (2^53 - 1). Provides conversion from attoFIL to picoFIL to FIL, and a Currency Conversion api which uses a remote service to provide the exchange rate.

Dealing with FIL numbers and their various fractional forms is a very common operation for dealing with Filecoin so this library, or some incarnation of the basic concepts, has potential to be critical ot the JavaScript Filecoin ecosystem.

###### Recommendations

***(POSSIBLE PROJECT)***

- Review and potentially replace with a lighter / more ergonimic alternative. 
- We should explore the ideal API for this representing FIL amounts.
- Consider using a lighter lib for big number support https://github.com/MikeMcl/big.js/wiki
- Consider just using the built-in BigInt https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/BigInt
- Currency conversion should be a seperate utility as many use-cases will not need it.

##### [@glif/filecoin-rpc-client](https://github.com/glifio/modules/tree/primary/packages/filecoin-rpc-client)

**TL;DR** Too bare-bones for general consumption, but a good example for folks who like it minimal.

A very light [(~70 LoC) wrapper](https://github.com/glifio/modules/blob/9b97f31054c20b39b8a75d30727fa33162715dab/packages/filecoin-rpc-client/src/index.ts) around `axios` to create JSON RPC requests.

- 👍 lightweight, low maintenance, never out of date, easy to understand.
- 👍 lovely docs https://documenter.getpostman.com/view/4872192/SWLh5mUd?version=latest
- ❌ methods passed as string arg. No API, no types, no way to discover the API from your IDE.
- ❌ you've got to parse filecoin responses manually
- ❌ CID params are awkward. Must be passed as a ipld-json link `{ '/': '<cid string>' }`

##### [@glif/filecoin-wallet-provider](https://github.com/glifio/modules/tree/primary/packages/filecoin-wallet-provider)

Abstraction layer for filecoin wallet implementations. A good example to explore, but needs more iterations before it's something we should recommend to folks, and it's unlikely to get that attention in the near future.

Aims at being a facade over different filecoin impl providers, to provide a common API and re-use common / computable methods where possible. Ideally a povider impl adds implementations of sensitive functions / primatives while non-senstive / high-level ones are provided by this module.

##### [@glif/react-components](https://github.com/glifio/modules/tree/primary/packages/react-components)

Not for general consumption. Has some nice UI components, but folks can explore the wallet app to see them, and get the most up to date versions.

#### Others

A full exploration report: https://github.com/protocol/w3dt-sudo/pull/19

### https://github.com/CoinSummer/lotus-jsonrpc-provider

> TypeScript Lotus RPC client. Not up to date, likely not used.

Last updated: 1 Nov 2020

### https://github.com/keyko-io/filecoin-verifier-tools

### [Zondax/filecoin-signing-tools](https://github.com/Zondax/filecoin-signing-tools)

> The filecoin signing library used across several filecoin ecosystem libraries

#### Maturity

Currently in `v.0.15.0`, this library is widely used, has been actively maintained over the last months and is the go to library for signing related operations in the ecosystem. It offers all the needed cryptographic primitives to interact with a Lotus node.

Main users include:
- `filecoin.js`
- `ceramicnetwork/cli`
  - A CLI that allows you to interact with the Ceramic protocol.
- `@nodefactory/filsnap`
  - Metamask snap (plugin) to enable Metamask users interaction with filecoin dapps.
- `@glif/local-managed-provider`

Last updated: 29 Apr 2021

#### Key Findings

- Per [https://github.com/Zondax/filecoin-signing-tools/issues/373](https://github.com/Zondax/filecoin-signing-tools/issues/373), [Zondax/filecoin-signing-tools#358](https://github.com/Zondax/filecoin-signing-tools/issues/358), [Zondax/filecoin-signing-tools#356](https://github.com/Zondax/filecoin-signing-tools/issues/356) and [Zondax/filecoin-signing-tools#352](https://github.com/Zondax/filecoin-signing-tools/issues/352), there are a few issues with bundlers and browsers.
- Per [Zondax/filecoin-signing-tools#368](https://github.com/Zondax/filecoin-signing-tools/issues/368) and [Zondax/filecoin-signing-tools#343](https://github.com/Zondax/filecoin-signing-tools/issues/343), there are several issues with WASM.  The browser WASM example uses the pure JS implementation and an issue tracking how wasm is being loaded in the browser. Could not find any project leveraging the WASM library. Is it working?

It includes libraries in `Rust Native Library`, `WASM Library` and `JSON RPC` with the main cryptographic primitives to use in filecoin. More details in the [Exploration Report](https://github.com/protocol/w3dt-sudo/blob/master/home/vasco-santos/2021-04-exploration-report-fil-signing-tools.md)

#### Overall Recommendations

**(POSSIBLE PROJECT)** Recommendations below would require investment with Zondax, or adoption by filecoin-shipyard, or other. Improvement of this project would be worthwhile as an example library but also because it serves an important purpose for the JavaScript library ecosystem.

- The [documentation site](https://zondax.ch/updating-documentation) linked as the main source for docs should be available. It has been on maintenance, which makes it difficult to understand what methods are supported unless we check the code/tests and this should be handled.
- Clarify WASM library status per above findings
- Friction onboarding
  - Several examples are provided, but some are not easy to run and do not have instructions. It would be extremely valuable for the community to have instructions for running this examples together with some educational content regarding what are the use cases for each and what happens under the hood.
- **(TODO TASK)** Run the WASM code in the browser and compare the bundle size with the pure JS implementation, learnings would be helpful for our own crypto stack and may suggest improvements for Zondax.

### https://github.com/mgoelzer/zondax-pch-demo

> demo code to show off the features of zondax/filecoin-signing-tools

Last updated: 9 October 2020

### https://github.com/filecoin-project/starling

> A command-line interface for simplified, coordinated, decentralized storage on the Filecoin network. This is a work in progress and is not yet production-ready. Use at your own risk.

Last updated: 31 October 2020

### https://github.com/spacegap/spacegap.github.io

https://spacegap.github.io/#/

Last updated: 9 October 2020

### https://github.com/trufflesuite/ganache-filecoin-alpha-cli/

> Alpha CLI for Ganache's Filecoin integration. Will be replaced by ganache-cli once integration is stable.

Work in progress Ganache flavored Truffle.

This appears to have been integrated into Truffle as of publication of this report. See https://www.trufflesuite.com/docs/filecoin/ganache/getting-started/get-started-with-the-gui

Last updated: 9 October 2020
