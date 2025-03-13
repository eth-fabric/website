---
title: Based Rollups Threading the Needle
nav_order: 3.3
layout: default
parent: Based Rollup Series
permalink: /education/based-rollups/Threading-the-Needle
---

## Tying It All Together

By now we assume you have ready Based Rollup 101 and Based Rollup 201. Below is an attempt to tie this all together. The reader should keep in mind there are still a variety of ways to implement based rollups where the details vary by team / approach. We would highly ecourage visiting awesome based rollup (NEED TO ADD LINK) and [preconfs](https://github.com/eth-fabric/awesome-based-preconfs) for an exhustive list of articles, stacks, presentations and podcasts. Further, if you are not familer with traditional rollup design we would encourage teams to learn more here (INSERT LINK TO BEST PLACE TO LEARN)

### Many Boxes and Arrows
Below is a detailed diagram of a based rollup tying all the componenetns and sub-components together. We note that while this picture may appear complex, many of the parts are actually part of current L2 stacks. .

![Fabric Overview with Dido](/website/assets/images/dido-overview.png)

### Definitions
- User: Party requesting a preconf for a transaction/data in a based rollup or a party submitting a transaction/data. 
- RPC Router:
- L2 Public Mempool: This is a generalized description of a mempool where transactions from L2s will be submitted to be preconfed or sequenced by the L1 validator. 
- L2 Full Node: Exactly as the name suggest, a full node tracking the state of the rollup. 
- L2 Builder A - C:
- Gateway: The entity responsible for issuing preconfirmations and sequencing the L2. This can be a L1 validator directly or a party they delegate certain rights to.
- Relay: Role most are familar with in the PBS pipeline, "relays are a doubly-trusted data-availability layer and communication interface between builders and validators." [Source](https://docs.flashbots.net/flashbots-mev-boost/relay#:~:text=mev%2Dboost%20is%20effectively%20just,might%20connect%20to%20many%20relays.)
- L1 Builder: 
- L1 Searcher:
- Wallets: 
- URC Indexer: 
- Universal Registry Contract ("URC"): A minimal smart contract that enables registrations and slashing to enhance the credibility of generic proposer commitments.
- Derivation Contract:
- Lookahead Window: A smart contract that provides an on-chain view of the beacon chain lookahead. This is essential for most based rollups to move beyond “total anarchy mode” by granting sequencing rights ahead-of-time—such as for enforcing state locks for L2 execution preconfs.
- L1 Prpopser: Proposer that is part of Ethereum's validator set.
- batchIbox Address:
- L2 Derivation:
- Execution Layer:
- L2 State:
- Settlement Layer:
- Shared Bridge:
