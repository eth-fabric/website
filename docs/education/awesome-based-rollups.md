---
title: Awesome Based Rollups
nav_order: 2.3
layout: default
parent: Education
permalink: /education/awesome-based-rollups
---
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---
# Awesome Based Rollups [![Awesome](https://awesome.re/badge.svg)](https://awesome.re)

A curated list of resources, research, and implementations related to based rollups in the Ethereum ecosystem. Special thanks to [Swapnil](https://x.com/swp0x0) of Nethermind for starting this months ago. During aggregation we also want to thank the Spire and [mteam](https://x.com/mteamisloading) for also working on similar types of collections. 

## Introduction
Based rollups the future of Ethereum. Follow the Ethereum Sequencing and Preconfirmations calls [here](https://www.youtube.com/watch?v=jrm4ZUoj9xY&list=PLJqWcTqh_zKHDFarAcF29QfdMlUpReZrR).

## Research and Discussion
- [Based rollups—superpowers from L1 sequencing](https://ethresear.ch/t/based-rollups-superpowers-from-l1-sequencing/15016) - Justin Drake's post formalzing the based sequencing concept
- [Vanilla Based Sequencing](https://ethresear.ch/t/vanilla-based-sequencing/19379) - Early exploration around a "vanilla" implementation of based rollups.
- [Alternate PBS: A PBS Proposal for Based Rollups](https://collective.flashbots.net/t/alternate-pbs-a-pbs-proposal-for-based-rollups/2672) - This article introduces Alternate PBS, a new PBS proposal for based rollups.
- [Fabric - Fabric to Accelerate Based Rollup Infrastructure & Connectivity](https://ethresear.ch/t/fabric-fabric-to-accelerate-based-rollup-infrastructure-connectivity/21640) - Propsal for a coordination and development effort to establish standards for based rollups.
- [Becoming Based: A Path towards Decentralised Sequencing](https://ethresear.ch/t/becoming-based-a-path-towards-decentralised-sequencing/21733) - A proposal to work with current L2s that progresses towards based sequencing.
- [Understanding Based Rollups: PGA Challenges, Total Anarchy, and Potential Solutions](https://ethresear.ch/t/understanding-based-rollups-pga-challenges-total-anarchy-and-potential-solutions/21320) - Paper analyzes the economics of based rollups using total anarchy as a method of sequencing blocks.
- [Based Rollups with Stronger Finality & Revenue Share (WIP)](https://hackmd.io/2HHg2t-gSbyJX3M170Nigw) - Describes a version of Ethereum shared sequencing for "based rollups" that retains the benefits of vanilla based sequencing (liveness, censorship resistance, L1 security and composability etc) while offering rollups stronger finality and a share of sequencing revenue.

## Transaction Flow
- [Transaction Submission on Based Rollups](https://ethresear.ch/t/transaction-submission-on-based-rollups/18631) - This post explores the trade-offs between different ways to submit transactions to the rollup.

## Inbox / Withdraws
- [Custard - Improving UX for Super TXs](https://ethresear.ch/t/custard-improving-ux-for-super-txs/21273) - Proposes a technique for enabling atomically composable super transactions on based rollups through state management.
- [Fast (and Slow) L2→L1 Withdrawals](https://ethresear.ch/t/fast-and-slow-l2-l1-withdrawals/21161) - Introduces a new fast path for L2 withdrawals to L1 within the same L1 slot, enabled by solvers.
- [Same-Slot L1→L2 Message Passing](https://ethresear.ch/t/same-slot-l1-l2-message-passing/21186) - Post introduces a protocol that enables the L2 proposer to selectively inject L1 messages emitted in the same slot directly into L2.

## Composability
- [Generalized Synchronous Composability](https://capricious-firefly-0c5.notion.site/Generalized-Synchronous-Composability-132d07a3da30809aa801e26077a49b60) - Aims to sketch out a way to make it possible to do synchronous composability between L1 and all L2s in an efficient way.
- [The Open Intents Framework: Intents As A Public Good](https://www.openintents.xyz) - Important effort that can benefit based rollup composability.
- [ULTRA TX - Programmable blocks: One transaction is all you need for a unified and extendable Ethereum](https://ethresear.ch/t/ultra-tx-programmable-blocks-one-transaction-is-all-you-need-for-a-unified-and-extendable-ethereum/21673) - This post focuses on a novel mechanism to bring programmable L1 blocks and what this means for L2s and the interoperability between L1 and L2s.
- Embedded Rollups, [Part 1](https://ethresear.ch/t/embedded-rollups-part-1-introduction/21460) and [Part 2](https://ethresear.ch/t/embedded-rollups-part-2-shared-bridging/21461) - Posts exploring fast and efficient cross-chain interoperability by implementing an embedded shared-bridge rollup.

## Blobs
- [Blob Aggregation - Step Towards More Efficient Blobs](https://ethresear.ch/t/blob-aggregation-step-towards-more-efficient-blobs/21624) - Article explores efficiency gains for blob sharing.
- [Blob Sharing for Based Rollups](https://hackmd.io/@linoscope/blob-sharing-for-based-rollups) - Introduces a protocol for based rollups to share blobs with each other so they can fill the blobs more efficiently and reduce L1 gas cost.
- [Blob sharing protocol](https://hackmd.io/@dapplion/blob_sharing) - Details idea for blob sharing across various rollup stacks.
- [Nethermind Blob Sharing POC](https://github.com/NethermindEth/blob-sharing-poc) and [Blobs Sharing Presentation on Fabric Call](https://docs.google.com/presentation/d/1uPvwOYvuoAQAXg38pa0OT_qzVxsecCow0bve0gsZsDE/edit#slide=id.p)
- [Potential impact of blob sharing for rollups](https://ethresear.ch/t/potential-impact-of-blob-sharing-for-rollups/20619) - Explores the impact of blob sharing by rollups.
- [Shared Blob Compression](https://paragraph.xyz/@spire/shared-blob-compression) - Explores an implementation to help with the increasing need for chains to opt-into having their blobs aggregated and compressed with blobs from other appchains.
- [Will Blob Sharing Solve Dilemma for Small Rollups](https://arxiv.org/abs/2410.04111) - This paper examines the effectiveness of blob sharing based on real-world data collected approximately six months after the implementation of EIP-4844.

## Economics
 [Based Rollup Economics](https://taiko.mirror.xyz/PhlvGdIaY3-ZQ1DqI9uM5LxrWGWLAzLI84rkxhvPKmM) - Post that summarizes the current economic landscape of rollups, and explore based rollups economy.
- [Based Rollups can reward Proposers first come first serve](https://ethresear.ch/t/based-rollups-can-reward-proposers-first-come-first-serve/18317) - Articles explores how a simple FCFS system should be sufficient for Based Rollups.
- [Based Ticketing Rollup](https://hackmd.io/LRQPSItESPuMhUSwrB71rQ) - Presents a “Based Ticketing Rollup” that starts from the concept of a Based rollup and adds Execution Tickets to address its weak points.
- [MEV for “Based Rollup”](https://ethresear.ch/t/mev-for-based-rollup/15636) - Article exploring how MEV can work with based rollups.

## Provers
- [Prover Incentives](https://github.com/OpenZeppelin/minimal-rollup/blob/main/documentation/prover-incentives.md), [reference implmeentation](https://github.com/OpenZeppelin/minimal-rollup/blob/main/src/protocol/taiko_alethia/ProverManager.sol), and [presentation on prover incentives](https://docs.google.com/presentation/d/1QCPtfh057ikO1iqJFf5lFny5VZ9LduV2bzZ1wBIbjnQ/edit#slide=id.p)- OZ research on prover incentive 

## Stacks-and-Implmentations
- [Facet Based Stack](https://github.com/0xFacet)
- [Fabric Reference Implementation](https://github.com/eth-fabric)
- [Nethermind's Surge](https://github.com/NethermindEth/surge-taiko-mono)
- [Puffer Based Stack](https://github.com/PufferFinance/unifi-mono )
- [Rise Based Stack](https://github.com/risechain)
- [Spire Based Stack](https://github.com/spire-labs/based-stack)
- [Taiko Based Stack](https://github.com/taikoxyz/taiko-mono)
- [Taiko's Gwyneth Based Stack](https://github.com/taikoxyz/gwyneth) 

## Articles
- [A Based Thesis](https://hackmd.io/@sacha/based-rollup-thesis) - Early thesis outlining why based rollups are the future and important for Ethereum.
- [Based Rollups and Decentralized Sequencing (Twitter Spaces wrap-up](https://community.taiko.xyz/t/based-rollups-and-decentralized-sequencing-twitter-spaces-wrap-up/1220) - Summary of a twitter spaces with the EF, Nethermind / Flashbots, Espresso, and Taiko.
- [Based Booster Rollup (BBR): A new major milestone in Taiko’s roadmap](https://taiko.mirror.xyz/anPjF35Mrc_xzYgOTbUmfjr_MlhE3L8ZBZIxqmz9GZ8)
- [Examining the Based Sequencing Spectrum](https://research.chainbound.io/examining-the-based-sequencing-spectrum) - Explores the spectrum of based sequencing.
- [Based Rollup FAQ](https://taiko.mirror.xyz/7dfMydX1FqEx9_sOvhRt3V8hJksKSIWjzhCVu7FyMZU) - FAQ for based rollups.
- [Booster rollups - scaling L1 directly](https://ethresear.ch/t/booster-rollups-scaling-l1-directly/17125) - Outlines an approach where the L2 directly extends the blockspace of the L1 for all applications deployed on L1 by sharding the execution of transactions and the storage.
- [Facet: An Ethereum Rollup Built for Hard Times](https://facet.org/whitepaper) - Facet whitepaper.
- [Gwyneth Technical Design](https://capricious-firefly-0c5.notion.site/Gwyneth-Technical-Design-86a8d1a151954f559f8124301bed1d46) - Early document outlining how Gwyneth works.
- [How do based rollups actually work?](https://x.com/Spire_Labs/status/1869724672784572776) - Simple explainer of how based rollups work.
- [Linea as a based rollup - a theoretical architecture](https://community.linea.build/t/linea-as-a-based-rollup-a-theoretical-architecture/9882) - Linea explores what it might take to go based.
- [Puffer Docs](https://docs-unifi.puffer.fi/)
- [Spectrum of Based Rollups](https://taiko.mirror.xyz/a2cfOjLTY0T9RwxKczP6xs0q1piQR0i9c9JuhH5iY4U) - Explores the spectrum of based rollups.
- [Surge Documentation](https://docs.surge.wtf/docs/intro) - Documentation on a based rollup effort from Nethermind.
- [Spire Light Paper](https://github.com/spire-labs/litepaper) - As the name implies.
- [Taiko Protocol Overview](https://taiko.mirror.xyz/y_47kIOL5kavvBmG0zVujD2TRztMZt-xgM5d4oqp4_Y) - Early article outlining how Taiko worked.
- [Unified Endgame Rollup Requirements](https://ethresear.ch/t/unified-endgame-rollup-requirements/18733) - The list is created with the proposed milestones for rollup decentralization.
- [Unpacking The Next Generation Of Ethereum L2s (I): Based Rollups](https://x.com/2077Research/status/1879976056750502327) - Explainer of based rollups and some background / history.
- [We're All Building the Same Thing](https://dba.xyz/were-all-building-the-same-thing/) - Detailed perspective on based rollups + other subjects.

## Presentations
- [A Revenue Model for Based Rollups](https://www.youtube.com/watch?v=JFCfnhFL9Mc&t=1s)
- [Are Based Rollups Really Based](https://podcasts.apple.com/us/podcast/are-based-rollups-really-based/id1523220564?i=1000671822720&l=pt-BR)
- [Based: Sequencing, Preconfs, Ideology](https://www.youtube.com/watch?v=0_51Pqt39Rs)
- [Based Rollups: The Next Frontier of Ethereum Scaling](https://www.youtube.com/watch?v=thPIc-_h2ms)
- [Based Rollups w/ M-Team from Spire labs](https://www.youtube.com/watch?v=ZiG24GlsdDk)
- [Beam Me Up!](https://www.youtube.com/watch?v=3ve8H54VFb8)
- [Ethereum Sequencing and Preconf calls](https://www.youtube.com/playlist?list=PLJqWcTqh_zKHDFarAcF29QfdMlUpReZrR)
- [Fixing Fragmentation](https://www.youtube.com/watch?v=MnsjUZo7RRI)
- [Justin Drake & Federico Carrone on Ethereum’s Native Rollup Roadmap](https://www.youtube.com/watch?v=lutSYwknNjQ)
- [Next Generation Based Rollups: A Practical Approach to Unifying Ethereum](https://www.youtube.com/watch?v=Ier_f5V4_ow)
- [Open based sequencing protocol](https://www.youtube.com/watch?v=2IiScdmXO6Q&list=PLCjVy6JjB1u7dL6cGJgs3RZH4rDgJdGW9&index=20)
- [Scaling Ethereum With Based Rollups](https://www.youtube.com/watch?v=uI2KSEXhtZc)
- [The Future of Rollups is Based](https://x.com/BanklessHQ/status/1859237063008174248)
- [Unpacking based rollups with Taiko and Fabric](https://www.youtube.com/watch?v=G4D7yavSgg0)
- [Unleashing the Full Potential of Eth Validators](https://www.youtube.com/watch?v=AZqK2Wq7cEg)
- [Wtf are based rollups and preconfs?](https://www.youtube.com/watch?v=WiKPlNGrUzU)
- [Why Based Rollups Are Ethereum's Best Bet](https://www.youtube.com/watch?v=UfoG4cRk9Z0)
- [You’re Not Bullish Enough on Based Rollups!](https://www.youtube.com/watch?v=_iee7LkcUeQ)

## Contributing
Contributions to this awesome list are welcome! Please read the [contribution guidelines](CONTRIBUTING.md) before submitting a pull request.
