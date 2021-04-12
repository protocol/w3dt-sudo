# [Zondax/filecoin-signing-tools](https://github.com/Zondax/filecoin-signing-tools)

> Exploration report by @vasco-santos

The filecoin signing library used across several filecoin ecosystem libraries, such as:
- `filecoin.js`
- `ceramicnetwork/cli`
  - A CLI that allows you to interact with the Ceramic protocol.
- `@nodefactory/filsnap`
  - Metamask snap (plugin) to enable Metamask users interaction with filecoin dapps.
- `@glif/local-managed-provider`

## Maturity

Currently in `v.0.15.0`, this library is widely used, has been actively maintained over the last months and is the go to library for signing related operations in the ecosystem. It offers all the needed cryptographic primitives to interact with a Lotus node.

## Project

### Libraries

1. Rust Native Library
  - Secp256k1
  - BLS
  - Filecoin transactions (CBOR <> JSON serialization)
  - Multisig (in progress)
2. WASM Library (Browser and Node.js)
  - Secp256k1
  - BLS
  - Filecoin transactions (CBOR <> JSON serialization)
  - Multisig (in progress)
3. JSON RPC Server
  - Exposes most of the functions available in the signing library

### Cryptographic primitives

1. `generateMnemonic`
2. `keyDerive`
3. `keyDeriveFromSeed`
4. `keyRecover`
5. `transactionSerialize`
6. `transactionSerializeRaw`
7. `transactionParse`
8. `transactionSign`
9. `transactionSignLotus`
10. `transactionSignRaw`
11. `verifySignature`
12. `addressAsBytes`
13.  `bytesToAddress`

All available via Rust, Wasm (browser and Node.js) and JSON RPC Server.

## Key Findings

- Per [https://github.com/Zondax/filecoin-signing-tools/issues/373](https://github.com/Zondax/filecoin-signing-tools/issues/373), [Zondax/filecoin-signing-tools#358](https://github.com/Zondax/filecoin-signing-tools/issues/358), [Zondax/filecoin-signing-tools#356](https://github.com/Zondax/filecoin-signing-tools/issues/356) and [Zondax/filecoin-signing-tools#352](https://github.com/Zondax/filecoin-signing-tools/issues/352), there are a few issues with bundlers and browsers.
- Per [Zondax/filecoin-signing-tools#368](https://github.com/Zondax/filecoin-signing-tools/issues/368) and [Zondax/filecoin-signing-tools#343](https://github.com/Zondax/filecoin-signing-tools/issues/343), there are several issues with WASM.  The browser WASM example uses the pure JS implementation and an issue tracking how wasm is being loaded in the browser. Could not find any project leveraging the WASM library. Is it working?

## Overall Recommendations

- The [documentation site](https://zondax.ch/updating-documentation) as the main source for docs should be available. It has been on maintenance, which makes it difficult to understand what methods are supported unless we check the code/tests and this should be handled.
- Clarify WASM library status per above findings
- Friction onboarding
  - Several examples are provided, but some are not easy to run and do not have instructions. It would be extremely valuable for the community to have instructions for running this examples together with some educational content regarding what are the use cases for each and what happens under the hood.

## Potential Action Items for w3dt

- Run the WASM code in the browser and compare the bundle size with the pure JS implementation
