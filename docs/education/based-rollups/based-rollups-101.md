---
title: Based Rollups 101
nav_order: 3.1
layout: default
parent: Based Rollup Series
permalink: /education/based-rollups/based-rollups-101
---
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## A Brief History of Based Rollups

The concept of based rollups was formally introduced in an ETH Research post by Justin Drake in early 2023 (though Vitalik first introduced the idea conceptually in 2021). Since then, based rollups have continued to gain traction within the Ethereum community due to the following advantages they offer:

- Inherited Characteristics of Ethereum: Because based rollups use Ethereum validators for sequencing transactions, they inherit key decentralization properties from Ethereum.
- Enhanced Interoperability: When combined with standardization efforts, based rollups help Ethereum and its rollups function more like a unified chain.
- Rollup Bootstrapping: Based rollups can leverage existing infrastructure and liquidity from both Ethereum and other based rollups. For example, USDC on Ethereum can be used for a trade executed on a based rollup, eliminating the need for the rollup to establish separate partnerships to get USDC issued natively.
- Benefits for App Developers: Developers can create customized rollups tailored to their users while maintaining seamless connectivity with Ethereum L1 and other based rollups, reducing friction in the ecosystem.

A rollup team called Taiko developed the first version of a based rollup protocol with this [commit](https://github.com/taikoxyz/taiko-mono/commit/a0e58b840089fe84a475f3bbc43d06777d366224) on July 30, 2022, and launched Taiko on May 27, 2024.

## What Is a Based Rollup?

A based rollup is a rollup that relies on Ethereum’s validator set for sequencing and inclusion in the Ethereum L1 block (i.e., Ethereum blockspace and where a block proposal for the based rollup is represented as a L1 transaction and the canonical sequence of the based rollup is derived from these L1 transactions). For more details on a long discussion on definitions, see [here](https://x.com/mteamisloading/status/1887575011311095824).

## How Does a Based Rollup Function?

A based rollup structurally resembles a traditional rollup. However, instead of using a single dedicated sequencer, Ethereum’s L1 validators perform the sequencing function.

![Traditional Rollup vs Based Rollup](/website/assets/images/Roll-up-Comparison.png)

## The Role of Preconfirmations in Based Rollups

One drawback of based rollups is that they do not provide the near-instant transaction confirmations that most rollups offer today. Based rollups are limited by the block time of Ethereum. To address this, preconfirmations were introduced.

## A Brief History of Preconfirmations

Preconfirmations, in the context of based sequencing, were first discussed in an ETH Research post in 2023. Since then, various models have been explored to integrate them effectively.

## What Is a Preconfirmation?

A preconfirmation is a reliable and timely commitment from one party to another ahead of settlement. This is similar to what happens when you swipe a credit card in a store—the credit card company provides a preconfirmation to the merchant, assuring them that they will get paid, even though the actual funds take a few days to transfer.

In the context of based rollups, preconfirmations work similarly. An L1 validator offers a near-instant confirmation before a transaction is officially included in a block. This confirmation is backed by both economic and social capital. While not all based rollups require preconfirmations, most will find them beneficial. Until Ethereum achieves faster slot times, preconfirmations provide users with quick transaction confirmations while still leveraging the advantages of based rollups.

## How Does a Based Rollup with Preconfirmations Work?

A based rollup with preconfirmations introduces additional complexity but also enhances user experience or makes it on par with how users interact with L2s today.

![Based Rollup With Preconfs](/website/assets/images/BasedRollup-Preconf.png)

## More Details Around Based Rollups and L1 Block Construction
Below is a more detailed picture of how based rollups receive and sequence based rollup transactions and L1 transactions. 

### Today's Block Construction
Today, over 90% of blocks in Ethereum are built via validators running an extra piece of software called MEV-Boost. This software allows validators to tap into the proposer builder separation protocol (“PBS”). Putting aside technical details / specifics of how PBS works, this opt-in protocol essentially lets validators outsource block construction. Put another way, with the PBS protocol, instead of validators packing the block themselves, they ask another actor to pack it for them (i.e., block builders). 

![Based Rollup With Preconfs](/website/assets/images/Ethereum-Blocks.png)

### So How Is This Going to Work With Based Rollups?

![Based Rollup With Preconfs](/website/assets/images/Based-rollup-blocks.png)

## Do Based Rollups Solve Everything?

Based rollups are one tool to help improve composability across Ethereum and lower the barrier to launching a rollup. To really see the benefits of composability we also need social consensus around things like shared cross-chain standards, proof aggregation, shared bridging and many other technological developments.

One of Ethereum’s biggest challenges today is that rollups don’t always work well together. Each rollup operates like an isolated island, making movement between them slow and cumbersome. Based rollups help solve this by making rollups feel like they’re part of the same system rather than disconnected chains.

How? They allow Ethereum itself to handle transaction ordering. Normally, rollups have their own systems for determining transaction sequencing. But with based rollups, Ethereum’s validators take on this role, ensuring that rollups using based sequencing naturally synchronize with one another—like two apps running on the same operating system rather than separate devices.

This significantly improves composability—a fancy way of saying that apps and transactions can interact across rollups more easily. Instead of relying on complex coordination mechanisms between different chains, everything just works. Whether it’s transferring assets or seamlessly combining DeFi Legos across different rollups, the experience becomes as smooth as if they were all on the same chain.

## What’s Still Needed
However, based sequencing alone isn’t enough. For Ethereum’s rollup ecosystem to truly function as a single unified chain, several key improvements are required. Based sequencing is a foundational piece, but without additional infrastructure, rollups will remain asynchronous—fast in some cases but prone to delays, complexity, and failure points that hinder seamless cross-rollup interactions.

Here’s what else is needed:
- Cross-Rollup Standards – Currently, rollups are built using different architectures, making interoperability difficult. Establishing common standards for smart contract calls and messaging will allow rollups to communicate more easily, just as internet applications follow common protocols.
- Shared Bridging – Many rollup bridges create separate versions of the same asset, leading to fragmentation and liquidity issues. A shared bridge would ensure that rollups rely on the same source of truth for assets, preventing fungibility problems and making cross-rollup transfers as seamless as moving funds within a single chain.
- Shared Blobs – Until Ethereum has abundant blob space, rollups need a way to efficiently share a single blob to reduce costs. This also helps validators maintain a unified transaction history for the based rollups they sequence, improving composability.
- Proof Aggregation – Each rollup has its own proof system to ensure user funds are secure. By aggregating multiple rollup proofs, costs can be reduced while ensuring that all rollups settle simultaneously, further improving composability.
- Low-Latency Verification – For rollups to behave as a single chain, they need fast and reliable state verification. Advances in cryptography, such as zero-knowledge proofs and trusted execution environments, can help rollups verify each other in near real-time.

## The Role of Based Sequencing
Based sequencing is the foundation for all of these improvements. It ensures that rollups align naturally with Ethereum’s timeline, laying the groundwork for a future where rollups operate as a seamless, unified Ethereum experience.

## Who Is Working On Based Rollups

![Teams Currently Building in Based Rollup Ecosystem](/website/assets/images/Current-Ecosystem.png)

The latest for this image can be found [here](https://docs.google.com/presentation/d/1YckjQ1LEGs9E8lhSO3-qFvpdLAefEAzgx5018CL7O-M/edit#slide=id.g2e3075303b5_0_204)

This was just a based rollup 101, for more technical details please see the [Based Rollups 201](https://eth-fabric.github.io/website/education/based-rollups/Based-Rollups-Componets) document.

Special thank you to [Rex](https://x.com/LogarithmicRex) for your feedback on writing this and broader education around Etheruem!
