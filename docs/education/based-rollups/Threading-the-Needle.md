---
title: Based Rollups Threading the Needle
nav_order: 3.3
layout: default
parent: Based Rollup Series
permalink: /education/based-rollups/Threading-the-Needle
---

## Tying It All Together

By now we assume you have read [Based Rollup 101](https://eth-fabric.github.io/website/education/based-rollups/based-rollups-101) and [Based Rollup Components](https://eth-fabric.github.io/website/education/based-rollups/Based-Rollups-Componets). Below is an attempt to tie this all together. The reader should keep in mind there are still a variety of ways to implement based rollups where the details vary by team / approach. We would highly encourage visiting the [awesome based rollup](/website/education/awesome-based-rollups) and [preconfs](/website/education/awesome-based-preconfs) pages for an exhustive list of articles, stacks, presentations and podcasts. 

### Many Boxes and Arrows
Below is a detailed diagram of a based rollup tying all the components and sub-components together. We note that while this picture may appear complex, some of the parts just the traditional L2 stack. We also want to again note that this is just one way to view based rollups and as teams work through research and implementations we may see shifts in this view. We also note that this image doesn’t include any proving-related components and is important to note that validity rollups (ZK/SGX), proving modules or markets are critical components that need to be thought about as they significantly impact the complexity of the DerivationHelper contract.

![Fabric Overview with Dido](/website/assets/images/dido-overview.png)

### Definitions
- **User:** An L2 user trying to submit transactions to one or more based rollups.
- **RPC Router:** A service that aggregates requests from users and makes them available to the L2 builders 
- **L2 Public Mempool:** A mempool for pending transactions across multiple based rollups.
- **L2 Full Node:** A standard full node to track the state of the L2.
- **L2 Builder A - C:** A group of rollup builders that build cross-rollup bundles.
- **Gateway:** The entity responsible for issuing preconfirmations. This can be an L1 validator directly or a party they delegate to.
- **Relay:** A trusted party from the PBS pipeline: "relays are a doubly-trusted data-availability layer and communication interface between builders and validators." [Source](https://docs.flashbots.net/flashbots-mev-boost/relay#:~:text=mev%2Dboost%20is%20effectively%20just,might%20connect%20to%20many%20relays.)
- **L1 Builder:** A party that builds L1 blocks on behalf of the L1 validator.
- **L1 Searcher:** A party that searches for transactions on behalf of the L1 builder.
- **Universal Registry Contract ("URC"):** A minimal smart contract that enables registrations and slashing to enhance the credibility of generic proposer commitments.
- **URC Indexer:** An indexer that tracks the state of the URC off-chain.
- **DerivationHelper Contract:** A smart contract that assists the L2 full node when deriving the L2 state (see [Dido](/website/research//Dido)).
- **Lookahead Window:** A smart contract that provides an on-chain view of the beacon chain lookahead to track the slots the validators will be proposing at.
- **L1 Proposer:** L1 block proposer AKA an Ethereum validator.
- **batchInbox Address:** The address that rollup blobs are sent to.
- **L2 Derivation:** The process of deriving the L2 state from the L1 state.
- **Execution Layer:** The virtual machine that executes transactions to determine state.
- **L2 State:** The current state of the L2.
- **Settlement Layer:** The proof system and L1 contracts that allow the L1 to update it's view on the L2's state.
- **Shared Bridge:** A bridge multiple L2s share to facilitate safer interoperability.

#### Notes on the Diagram
- The RPC Router and L2 Public Mempool are not required but can help with censorship resistance for non-contentious transactions. In practice there will likely be a mix of public mempool and private orderflow.
- To reduce Gateway sophistication in this design, L2 Builders can horizontally scale to service as many L2s as needed but the Gateway only acts as a clearing house that issues preconfs to the auction winner.
