---
title: Based Rollups 101
nav_order: 3.1
layout: default
parent: Based Rollup Series
permalink: /education/based-rollups/based-rollups-101
---


## A Brief History of Based Rollups

The concept of based rollups was formally introduced in an ETH Research post in early 2023 by Justin Drake (previously this conceptually had been introduced by Vitalik in 2021). Based rollups have continued to be pursed by teams across the Ethereum community due to the following features based rollups provide:

- **Inherited Characteristics of Ethereum:** Because based rollups use Ethereum validators for sequencing transactions, they inherit some of the decentralization characteristics of Ethereum.
- **Enhanced Interoperability:** Based rollups are a solution that when combined with standardization efforts will make Ethereum and all of the rollups feel like one chain.
- **Rollup Bootstrapping:** Based rollups can leverage infrastructure and liquidity in other based rollups and Ethereum. For instance, USDC on Ethereum can be used for a trade executed on a based rollup (i.e., the rollup does not need to do BD to get USDC to be issued on its rollup).
- **Benefits for App Developers:** Developers can create customized rollups for their users while maintaining connectivity with other based rollups and Ethereum L1, reducing existing friction

## What Is a Based Rollup?

A based rollup is a rollup that relies on Ethereum’s validator set for sequencing its blocks.

## How Does a Based Rollup Function?

A based rollup structurally resembles a traditional rollup. However, instead of using a single dedicated sequencer, Ethereum’s L1 validators perform the sequencing function.

![Traditional Rollup vs Based Rollup](/website/assets/images/Roll-up-Comparison.png)

## The Role of Preconfirmations in Based Rollups

One drawback of based rollups is that they do not provide the near-instant transaction confirmations that most rollups offer today. Based rollups are limited by the block time of Ethereum. To address this, preconfirmations were introduced.

## A Brief History of Preconfirmations

Preconfirmations, in the context of based sequencing, were first discussed in an ETH Research post in 2023. Since then, various models have been explored to integrate them effectively.

## What Is a Preconfirmation?

A preconfirmation is a reliable and timely commitment from one party to another ahead of settlement. This is similar to what happens today when you swipe a credit card in a store, the credit card company issues a preconfirmation to the company accepting the credit card that the company will get paid even though it takes a few days for the actual money paid to move across accounts. Preconfirmations in the context of based rollups function similarly where by the L1 validator offers a near-instant confirmation before the transaction is actually included in a block. This confirmation is backed by economic and social capital. While not all based rollups require preconfirmations, most will find them beneficial and until Ethereum achieves faster slot times, preconfirmations serve as a mechanism to provide users with quick transaction confirmations while still benefiting from the advantages of based rollups.

## How Does a Based Rollup with Preconfirmations Work?

A based rollup with preconfirmations introduces additional complexity but also enhances user experience or makes it on par with how users interact with L2s today.

![Based Rollup With Preconfs](/website/assets/images/BasedRollup-Preconf.png)

## More Details Around Based Rollups and L1 Block Construction
Below is a more detailed picture of how based rollups receive and sequence based rollup transactions and L1 transactions. 

![Based Rollup With Preconfs](/website/assets/images/Based-rollup-blocks.png)

Important items to note:
- The based transaction pool is used as a generic term of where transactions are entering a mempool
- Similar to the market structure of Ethereum today, where validators delegate block stuffing to a sophisticated entity, we expect validators to remain "dumb pipes" and not sophisticated, delegating / leveraging another party to perform the actual sequencing of transactions and block stuffing

## Based Rollups Solve Everything

Based rollups are one tool to help improve composability across Ethereum and lower the barrier to launching a rollup. To really see the benefits of composability we also need social consensus around things like shared cross-chain standards, proof aggregation, shared bridging and many other technological developments.

## What Else Is Potentially Needed

One of the biggest challenges in Ethereum today is that rollups don’t always **work well together**. Each rollup is like its own island, and moving between them can be slow and clunky. Based rollups help  **fix this by making rollups feel like they’re part of the same system**, rather than disconnected chains.

How? They let Ethereum itself handle transaction ordering. Normally, rollups have their own systems for deciding which transactions go first. But with based rollups, Ethereum’s validators do this job instead. This means that rollups using based sequencing can naturally **sync up with each other**—like two apps running on the same phone instead of separate devices.

This **supercharges composability**, which is just a fancy way of saying that apps and transactions can interact across rollups more easily. Instead of needing complicated systems to coordinate things across different chains, everything just works—whether it’s transferring assets or seamlessly combining **DeFi Legos** across different rollups, just like they were on the same chain.

However, **based sequencing alone isn’t enough**. For Ethereum’s rollup ecosystem to truly feel like a single unified chain, a lot of work still needs to be done. But based sequencing is a requirement—without it, rollups will always be asynchronous, which can still feel fast in some cases but **introduces delays, added complexity, and potential failure points** that make cross-rollup interactions less seamless and more fragile. Here’s what else is needed:

- **Cross-Rollup Standards** – Right now, rollups are built in different ways, making it harder for them to interact. **Common standards** for things like smart contract calls and messaging will allow rollups to communicate more easily, just like apps on the internet follow common protocols.
- **Shared Bridging** – Today, bridges between rollups often create separate versions of the same asset, leading to **fragmentation and liquidity issues**. A shared bridge ensures that rollups use the same source of truth for assets, preventing **fungibility problems** and making transfers between rollups as seamless as moving funds within a single chain.
- **Shared Blobs** - Until there are abundant blobs, rollups need a way to share a single blob efficiently to reduce transaction costs. This also helps validators create a single global history for the based rollups they sequence, helping with composability.
- **Proof Aggregation** – Every rollup has a proof system in place to guarantee their users funds are safe. By **aggregating proofs**, we can bundle multiple rollups’ proofs together, reducing costs. Also, ensuring they settle at the same time helps with composability.
- **Low-Latency Verification** – For rollups to act like one chain, they need **fast and reliable verification** of each other’s state. Advances in cryptography, like zero-knowledge proofs and trusted execution environments, can help rollups verify each other in near real-time.

Based sequencing is the foundation for all of this. It ensures that rollups naturally align with Ethereum’s timeline, **setting the stage for a future where rollups feel like a single, unified Ethereum experience**.

## Who Is Working On Based Rollups

![Teams Currently Building in Based Rollup Ecosystem](/website/assets/images/Current-Ecosystem.png)

The latest for this image can be found [here](https://docs.google.com/presentation/d/1YckjQ1LEGs9E8lhSO3-qFvpdLAefEAzgx5018CL7O-M/edit#slide=id.g2e3075303b5_0_204)

This was just a based rollup 101, for more technical details please see the [Based Rollups 201](/website/education/Based-Rollups-201) document.

