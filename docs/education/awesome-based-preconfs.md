---
title: Awesome Based Preconfirmations
nav_order: 2.3
layout: default
parent: Education
permalink: /education/awesome-based-preconfs
---

# Awesome Based Preconfirmations [![Awesome](https://awesome.re/badge.svg)](https://awesome.re)

A curated list of resources, research, and implementations related to preconfirmations in the Ethereum ecosystem. Special thanks to [Swapnil](https://x.com/swp0x0) of Nethermind for starting this months ago. During aggregation we also want to thank the Spire and mteam for also working on similar types of collections. 

## Table of Contents
- [Introduction](#introduction)
- [Research and Discussion](#research-and-discussion)
- [Fair Exchange](https://github.com/eth-fabric/awesome-based-preconfs/edit/main/README.md#fair-exchange)
- [Sidecars](https://github.com/eth-fabric/awesome-based-preconfs/blob/main/README.md#sidecars)
- [Validators](https://github.com/eth-fabric/awesome-based-preconfs/blob/main/README.md#validators)
- [Universal-Registry-for-Commitments](https://github.com/eth-fabric/awesome-based-preconfs/blob/main/README.md#universal-registry-for-commitments)
- [Pricing-and-Incentive-Mechanisms](https://github.com/eth-fabric/awesome-based-preconfs/blob/main/README.md#pricing-and-incentive-mechanisms)
- [Gateways-and-Delegation](https://github.com/eth-fabric/awesome-based-preconfs/blob/main/README.md#gateways-and-delegation)
- [Stacks-and-Implmentations](https://github.com/eth-fabric/awesome-based-preconfs/blob/main/README.md#stacks-and-implmentations)
- [Articles](https://github.com/eth-fabric/awesome-based-preconfs/blob/main/README.md#articles)
- [Presentations](https://github.com/eth-fabric/awesome-based-preconfs/blob/main/README.md#presentations)
- [Full-Day-Events](https://github.com/eth-fabric/awesome-based-preconfs/blob/main/README.md#full-day-events)
- [Related Concepts](#related-concepts)
- [Contributing](#contributing)

## Introduction
Preconfirmations are a mechanism that allows block proposers to commit to a block's contents before it is included in the blockchain. Follow the Ethereum Sequencing and Preconfirmations calls [here](https://www.youtube.com/watch?v=jrm4ZUoj9xY&list=PLJqWcTqh_zKHDFarAcF29QfdMlUpReZrR).

## Research and Discussion
- [Based Preconfirmations](https://ethresear.ch/t/based-preconfirmations/17353) - The original post introduces the concept of based preconfirmations.
- [Strawmanning Based Preconfirmations](https://ethresear.ch/t/strawmanning-based-preconfirmations/19695) - Analysis of a simple “strawman” preconfirmation setup.
- [Leaderless and Leader-Based Preconfirmations](https://ethresear.ch/t/leaderless-and-leader-based-preconfirmations) - Discussion on leader-based and leaderless preconfs.
- [Analyzing BFT & Proposer-Promised Preconfirmations](https://ethresear.ch/t/analyzing-bft-proposer-promised-preconfirmations/17963) - Analysis of BFT preconfirmations and proposer-promised preconfirmations.
- [Rollup-Centric Considerations of Based Preconfimations](https://ethresear.ch/t/rollup-centric-considerations-of-based-preconfimations)- Explores how implementations of preconfirmations can configure blocktime and more efficient data publishing
- [Integrating Account Abstraction and Inclusion Preconfirmations](https://research.chainbound.io/integrating-account-abstraction-and-inclusion-preconfirmations) - Proposal to directly leverage EIP-7702 for adoption of preconfs benefiting from batching, gas sponsorship, and secure delegations.
- [Preconfirmations for Vanilla Based Rollups](https://github.com/LimeChain/based-preconfirmations-research/blob/732cb92474554c2529aabc61e83b8f0934ce6adf/docs/preconfirmations-for-vanilla-based-rollups.md) - Document outlines the design and mechanics to support preconfirmations in Vanilla Based Rollups.
- [Blob Preconfirmations with Inclusion Lists to Mitigate Blob Contention and Censorship](https://ethresear.ch/t/blob-preconfirmations-with-inclusion-lists-to-mitigate-blob-contention-and-censorship/19150)
- [RFC Preconfirmations Flow Exploration](https://limechain.notion.site/RFC-Preconfirmations-Flow-Exploration-30fe218a0ea0443fb6bc213da969a47d) - Early exploration of preconfirmations flows.

  
## Fair-Exchange
- [Preconfirmation Fair Exchange](https://ethresear.ch/t/preconfirmation-fair-exchange/21891) - Framework for analyzing protocols that seek to enforce timely-fair exchange of preconfirmations.
- [Solutions to the Preconf Fair Exchange Problem](https://ethresear.ch/t/solutions-to-the-preconf-fair-exchange-problem/19779) - Solutions for dealing with the fair exchange problem in leader-based preconfirmation setups.

## Sidecars
- [A simple, small, mev-boost compatible preconfirmation idea](https://ethresear.ch/t/a-simple-small-mev-boost-compatible-preconfirmation-idea/19800) - Adjusting MEV-Boost to handle preconfs.
- [Based Preconfirmations with Multi-round MEV-Boost](https://ethresear.ch/t/based-preconfirmations-with-multi-round-mev-boost/20091) - Propose multi-round MEV-Boost, a modification of MEV-Boost that enables based preconfirmations by running multiple rounds of MEV-Boost auctions within a single slot.
- [Commit-Boost: Proposer Platform to Safely Make Commitments](https://ethresear.ch/t/commit-boost-proposer-platform-to-safely-make-commitments/20107) - New sidecar that allows proposers to make commitments to preconf protocols.
- [Based proposer commitments - Ethereum’s marketplace for proposer commitments](https://ethresear.ch/t/based-proposer-commitments-ethereum-s-marketplace-for-proposer-commitments) - New sidecar that allows proposers to make commitments to preconf protocols.
- [Block Building Pipelines for Vanilla Based Rollups](https://github.com/LimeChain/based-preconfirmations-research/blob/732cb92474554c2529aabc61e83b8f0934ce6adf/docs/pipelines.md)
- [Commit-Boost Community Calls](https://github.com/Commit-Boost/pm/tree/main/Community_Calls)

## Validators
- [Delegation in Bolt: Outsourcing Sophistication While Preserving Decentralization](https://research.chainbound.io/delegation-in-bolt-outsourcing-sophistication-while-preserving-decentralization) - Article explores various “levels” of delegation from proposers for preconfs. 
- [Preconfirmations under the NO lens](https://ethresear.ch/t/preconfirmations-under-the-no-lens/19975) - Research analyzing preconfs from a validator's perspective. 

## Universal-Registry-for-Commitments
- [Credibly Neutral Preconfirmation Collateral: The Preconfirmation Registry](https://ethresear.ch/t/credibly-neutral-preconfirmation-collateral-the-preconfirmation-registry/19634) - Introduces a design for a credibly neutral preconfirmations registry.
- [Based Sequencer Selection](https://ethresear.ch/t/based-sequencer-selection/19747) - Exploratory proposal for a method of deterministically identifying sequencers on L2s to route transactions to.
- [Sequencer Opt-In, Discovery and Communication](https://github.com/LimeChain/based-preconfirmations-research/blob/732cb92474554c2529aabc61e83b8f0934ce6adf/docs/optin-mechanics.md) - Inital exploration for validators to register / opt in to based sequecning.
- [Universial Registry Contract](https://github.com/eth-fabric/urc) - Developed by over a dozen teams and currently being used by multiple preconf teams. Documents can be found in the Github.

## Pricing-and-Incentive-Mechanisms
- [Avoiding Accidental Liveness Faults for Based Preconfs](https://ethresear.ch/t/avoiding-accidental-liveness-faults-for-based-preconfs) - Proposal to solve accidental liveness slashing for proposers offering preconfs.
- [User-Defined Penalties: Ensuring Honest Preconf Behavior](https://ethresear.ch/t/user-defined-penalties-ensuring-honest-preconf-behavior/19545) - Post outlining how users should specify their preferred penalty when requesting a preconf.
- [Pre-confirmation Liveness Slashing Penalties from the Proposer’s Perspective](https://ethresear.ch/t/pre-confirmation-liveness-slashing-penalties-from-the-proposers-perspective/19884) - Explores the liveness penalty from the point of view of proposers from an economical perspective.
- [Pricing Transactions for Preconfirmation](https://ethresear.ch/t/pricing-transactions-for-preconfirmation/21802) - Proposes a framework for pricing preconfirmations but also to initiate a constructive discussion within the Ethereum community.
- [Pricing Ethereum Blocks with Vol Markets with Implications for Preconfirmations](https://ethresear.ch/t/pricing-ethereum-blocks-with-vol-markets-with-implications-for-preconfirmations/20419) - A discussion and illustrate approach for the pricing of preconfs that responds in real-time to current market conditions.
- [Estimating the Revenue from Independent Sub-Slot Auction Preconfirmations](https://research.lido.fi/t/estimating-the-revenue-from-independent-sub-slot-auction-preconfirmations/8801) - Analysis towards understanding the economic feasibility of preconfirmations.
- [A Pricing Model for Inclusion Preconfirmations](https://research.lido.fi/t/a-pricing-model-for-inclusion-preconfirmations/9136) - Article focuses on the pricing of inclusion preconfirmations.
- [Pricing Future Blockspace: A Data-driven Approach](https://www.luban.wtf/taiyi-pricing-1) - Introduce a pricing model developed for hedged preconfirmations with a focus on preserving underwriter funds and generating steady yields.
- [Analysing Expected Proposer Revenue from Preconfirmations](https://research.lido.fi/t/analysing-expected-proposer-revenue-from-preconfirmations/8954) - Article exploring expected revenue from preconfirmations vs the current MEV-Boost acution.
- [Value-Capturing Based Rollups with Based Preconfirmations](https://collective.flashbots.net/t/value-capturing-based-rollups-with-based-preconfirmations/2884) - Outlines a protocol which allows for based preconfirmations, while ensuring the based rollup captures much of the value generated from block building.

## Gateways-and-Delegation
- [The Preconfirmation Gateway ~ Unlocking Preconfirmations: From User to Preconfer](https://ethresear.ch/t/the-preconfirmation-gateway-unlocking-preconfirmations-from-user-to-preconfer/18812) - Introduces The Preconfirmation Gateway to completely abstract preconfirmations from users.
- [Ahead-of-Time Block Auctions To Enable Execution Preconfirmations](https://ethresear.ch/t/ahead-of-time-block-auctions-to-enable-execution-preconfirmations/21345) - Article analyzing a gateway as an unsophisticated entity and the other as a sophisticated builder.
- [Proposer Commitments - A Validator’s Case For Delegation](https://mirror.xyz/ladislaus.eth/aZWM5O_gjqj56w0lvCGhRMwYfoAF_jVdMuHWO3cq-JE) - Outlines perspective on gateways and the current market structure of Ethereum. 
- [Based Ultrachain](https://x.com/sam_battenally/status/1889247872035754321) - Outlines approaches to gateways and delegation and why be based.

## Stacks-and-Implmentations
- [Bolt](https://chainbound.github.io/bolt-docs/) - Bolt enables Ethereum block proposers to provide credible commitments about the contents of their blocks.
- [mev-commit](https://docs.primev.xyz/) - A credible commitment network used for preconfirmations & more.
- XGA [here](https://docs.xga.com) and [here](https://research.lido.fi/t/xga-extensible-gas-auctions-for-enabling-preconfirmations-without-restaking-or-epbs/7584) - L2 Gas Auction platform.
- [UniFi](https://docs.puffer.fi/unifi-avs-intro/) - preconf protocol.
- [Interstate](https://docs.interstate.so/intro) - preconf protocol.
- [Luban Taiyi](https://docs.luban.wtf/taiyi_overview) - preconf protocol.
- [ETHGas](https://docs.ethgas.com) - preconf protocol.
- [Cairo](https://github.com/cairoeth/preconfirmations) - Early implementation, also ETHResearch post [here](https://ethresear.ch/t/towards-an-implementation-of-based-preconfirmations-leveraging-restaking/19211).
- [ZuBerlin - Preconfs Devnet]([https://www.notion.so/ZuBerlin-Preconfs-Devnet-b693047f41e7407cadac0170a6711dea](https://twisty-wednesday-4be.notion.site/ZuBerlin-Preconfs-Devnet-b693047f41e7407cadac0170a6711dea)) - Early devnets with various teams working on preconfs. 

## Articles
- [Preconfirmation for the Average Joe](https://x.com/ceciliaz030/status/1875558701324759392) - Simple explnation on preconfs.
- [What Based Rollups Need from the L1?](https://taiko.mirror.xyz/mg8H0IuGaG0S7aWHG1bPygYpaEDxqYmW-vyfmL9LeFY) - Article explaining why based rollups need faster block times.
- [Preconfirmations: Credible Promise of Future Execution](https://www.longhash.vc/post/preconfirmations-credible-promise-of-future-execution) - Overview of research around preconfs.
- [Preconfirmations: The Fulfillment-Delivery Paradigm](https://mirror.xyz/preconf.eth/sgcuSbd1jgaRXj9odSJW-_OlWIg6jcDREw1hUJnXtgI) - Proposes a formal definition for a preconfirmation, outlined how they’re relevant in the context of decentralized systems and mev, and analyzed considerations to enable them efficiently.
- [Preconfirmations: Explained](https://www.luganodes.com/blog/preconfirmations-explained/) - Preconfs 101.
- [Preconfirmations: On splitting the block, mev-boost compatibility and relays](https://ethresear.ch/t/preconfirmations-on-splitting-the-block-mev-boost-compatibility-and-relays/19837) - Article discussing XGA style preconfs.
- [Proposal: Onboard Bolt to the Lido Alliance](https://research.lido.fi/t/proposal-onboard-bolt-to-the-lido-alliance/7724) - Proposal for Bolt (preconfs protocol) to join Lido Alliance.
- [Introducing ETHGas and Realtime Proposer Commitments to the Lido Community](https://research.lido.fi/t/introducing-ethgas-and-realtime-proposer-commitments-to-the-lido-community/9018) - ETHGas introduction to Lido Community.
- [The Preconfirmation Sauna](https://ethresear.ch/t/the-preconfirmation-sauna/19762) - Outlines Switchboard’s vision for how this preconfirmation competition should play out.
- [Grounded Relay: Superpowers from Relay Coordination](https://ethresear.ch/t/grounded-relay-superpowers-from-relay-coordination/18601) - Using the relay as a coordinator for various services.
- [State Lock Auctions: Towards Collaborative Block Building](https://ethresear.ch/t/state-lock-auctions-towards-collaborative-block-building/18558) - Idea on how to implement state locks on Ethereum.
- [Preconfirmations Glossary & Requirements](https://hackmd.io/@Perseverance/Sy4a_BX2p) - Early work to define terms around preconfs flows.
- [Conditions API - Block Conditions Extension to the Builder API](https://hackmd.io/@Perseverance/H1d95Jf4C) - Early idea around APIs to enable commitments / preconfs.
- [Road to Real-time: Preconfirmation Shreds](https://blog.riselabs.xyz/incremental-block-construction/) - This article proposes an approach to achieving faster transaction (pre)confirmations in Layer 2 blockchain networks through incremental block construction.
- [Proposer-Commitment Infrastructure in Ethereum](https://simbro.medium.com/proposer-commitment-infrastructure-in-ethereum-61ad3b31f05f) - Describes the out-of-protocol solutions that are in development, and explore what sort of solution space they open up for different types of Ethereum users.
- [Uncrowdable Inclusion Lists: The Tension between Chain Neutrality, Preconfirmations and Proposer Commitments](https://ethresear.ch/t/uncrowdable-inclusion-lists-the-tension-between-chain-neutrality-preconfirmations-and-proposer-commitments/19372) - Discussion to not crowd in protocol inclusion lists with preconfs / other commitments.
- [Based Preconfs FAQ](https://hackmd.io/@samlaf/based-preconfs-faq) - Based preconfs FAQ.

## Presentations
- [Get Ready for Preconfs: Are They the Future of Ethereum?](https://www.youtube.com/watch?v=89-S4IvbAwg)
- [Based: Sequencing, Preconfs, Ideology](https://www.youtube.com/watch?v=0_51Pqt39Rs)
- [Preconf Vision and Its Place in Ethereum](https://www.youtube.com/watch?v=qeg9Eyr5W8Y)
- [Designing an End to End Solution for Based Preconfirmations](https://www.youtube.com/watch?v=70xIIrGXDSo)
- [A proposer's perspective on preconfirmations: a new game in town?](https://www.youtube.com/watch?v=Wa5O4TMEdwE&t=1s)
- [Based Preconfirmations with MR-MEV-Boost](https://www.youtube.com/watch?v=fo2xDLSst_M)
- [How to price your Preconfirmation](https://ethcc.io/archives/how-to-price-your-preconfirmation)
- [Preconfirmations on Ethereum with Bolt: Impact on Node Operators](https://www.youtube.com/watch?v=m_JzE-7lwM8&list=PLhvXP1-8VKZRol0ZQHabpm2vmMlq1yG2E&index=27&t=6s)
- [Credible Commitments: How Pre-Confirmations Unlock L2 Interoperability](https://blockworks.co/podcast/bellcurve/0345b516-eb15-11ee-854f-cbb5ba1433a1)
- [Learnings From Building The Universal Preconf Registry](https://www.youtube.com/watch?v=j0dIwo-ACr4&list=PLCjVy6JjB1u7dL6cGJgs3RZH4rDgJdGW9&index=9)
- [To Delay or Not Delay Preconfs](https://www.youtube.com/watch?v=e_49f42Cno0&list=PLCjVy6JjB1u7dL6cGJgs3RZH4rDgJdGW9&index=14)
- [Sidecar standardization with Commit-Boost](https://www.youtube.com/watch?v=-wWLcwz08iI&list=PLCjVy6JjB1u7dL6cGJgs3RZH4rDgJdGW9&index=17)
- [How Preconfs & Blockspace Products Make Ethereum Relevant](https://www.youtube.com/watch?v=3QW1XSbBmvQ&list=PLCjVy6JjB1u7dL6cGJgs3RZH4rDgJdGW9&index=20)
- [Supercharging Ethereum w/ Fast Preconfs & Decentralized Yield Distribution](https://www.youtube.com/watch?v=61qbQTbu5KM&list=PLCjVy6JjB1u7dL6cGJgs3RZH4rDgJdGW9&index=25)
- [Keynote : Rollup Preconfirmations](https://www.youtube.com/watch?v=boxGqp9mGJ4)
- [Soft Confirmations and other builder services](https://docs.google.com/presentation/d/1sXJA9vCzJ3sT-3FyrH6CBrOnkjBERiWnvc8gQxilnoM/edit#slide=id.p) - Presentation from Alex Stokes on builder services.
- [Preconfs in a minute](https://x.com/Agglayer/status/1890427606022984171)
- [Designing an End to End Solution for Based Preconfirmations](https://app.devcon.org/schedule/CRWBCC)

## Full-Day-Events
- ZuBerlin - 2024
   - [Drew, Kubi, Lorenzo - Commit-Boost](https://streameth.org/zuberlin/watch?session=66681afef9b8e98b1ec95fdd) - Introducing the Commit-Boost effort, including a walkthrough of the code.
   - [Daniel - LimeChain](https://streameth.org/zuberlin/watch?session=66684f7006eda795c8925909) - Dicusses how preconfirmations interact with the L1 PBS pipeline.
   - [Conor - Switchboard](https://streameth.org/zuberlin/watch?session=66682e25f9b8e98b1ec98882) - Introduces Preconfirmations Sauna, a credibly neutral effort to standardise preconfirmations.
   - [Harry - Luban](https://streameth.org/zuberlin/watch?session=6668652806eda795c89291b2) - Shares a lottery mechanism for pricing preconfirmations.
   - [Jonas - Chainbound](https://streameth.org/zuberlin/watch?session=666828e8f9b8e98b1ec97e79) - Shares how Bolt enables L1 preconfirmations.
   - [Christian - Primev](https://streameth.org/zuberlin/watch?session=66685ecd06eda795c8928664) - Shares how mev-commit enables L1 preconfirmations.
   - [Tariz - Radius](https://streameth.org/zuberlin/watch?session=66686a3306eda795c892964e) - Shares how Radius integrates based sequencing.
- [Preconf.erence Devcon 7](https://www.youtube.com/playlist?list=PLoHFINX7DFU7q5MTJIEha1Ll2DWVG-zSb)
- [Preconf.erence ETH Denver](https://www.youtube.com/playlist?list=PLoHFINX7DFU5W6x2b-u_KpbEsbsmJ8qb5)
- [ETHGas ETHDenver Event on Preconfs](https://www.youtube.com/playlist?list=PLW6BSM3-lmsXF9XeWTWfb8Wu-psRHL-4Q)

## Related Concepts
- [Proposer-Builder Separation (PBS)](https://github.com/ethereum/builder-specs) - A related concept that separates the roles of proposers and builders in the Ethereum consensus layer.
- [MEV-Boost](https://boost.flashbots.net/) - A middleware that outsources block building to a network of builders, enabling proposer commitments.
- Embedded Rollups, [Part 1](https://ethresear.ch/t/embedded-rollups-part-1-introduction/21460) and [Part 2](https://ethresear.ch/t/embedded-rollups-part-2-shared-bridging/21461) - Posts exploring fast and efficient cross-chain interoperability by implementing an embedded shared-bridge rollup.

## Contributing
Contributions to this awesome list are welcome! Please read the [contribution guidelines](CONTRIBUTING.md) before submitting a pull request.
