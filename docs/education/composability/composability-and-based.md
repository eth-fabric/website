---
title: Step 4 - Based Rollups
nav_order: 4.4
layout: default
parent: Composability 101
permalink: /education/composability/composability-and-based
---
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

# Based sequencing recap
So far everything in this series has applied to *shared sequencing*. In this section we're going to try and answer how any of this relate to *based sequencing*. 

But first what are based rollups?

> A rollup is said to be based, or L1-sequenced, when its sequencing is driven by the base L1. More concretely, a based rollup is one where the next L1 proposer may, in collaboration with L1 searchers and builders, permissionlessly include the next rollup block as part of the next L1 block. [- Justin Drake](https://ethresear.ch/t/based-rollups-superpowers-from-l1-sequencing/15016)

In other words, a based rollup is just a normal rollup that rotates between the L1 validators as their sequencer.

![Based Sequencing](/website/assets/images/based-sequencing.png)

Technically a based rollup shares a sequencer set (a subset of the Ethereum validators) but they are not necessarily doing *shared sequencing* by default. This means it’s entirely possible (though undesirable) for a based rollup to lack any of the properties of shared sequencing, in which case it would look like the left-side of the image. 

Superpowers are unlocked when the validators have the right to sequence **multiple** based rollups as shown on the right-side, in which case they function as a shared sequencer. For this to happen it's extremely important for different based rollups to agree on standards and adopt shared infrastructure so they can be shared, which is Fabric's objective.

## Sequencer selection

Let’s first cover the different “modes” of based sequencing as they exist along a spectrum. Note that this list is non-exhaustive but is meant to highlight the most common approaches.

### Total anarchy
Most known from Taiko’s implementation, the default form of based sequencing is “total anarchy mode” where the rollup allows *anyone* to submit blobs to sequence transactions. Importantly, since the L1 validator has the exclusive right to sequence the L1 block, they can guarantee that their blob is what lands despite it being a free-for-all to submit them. While this is the most decentralized mode of sequencing, it suffers from UX issues due to the 12s slot times of the L1. Rollup transactions will sit in the L2’s mempool until someone decides it’s profitable to sequence them or a charitable entity (i.e., the foundation) pays the costs to sequence them at a loss to themselves.

### Lookahead based 
Classical rollups get around the UX issue of "total anarchy mode" by having a dedicated sequencer to send transactions to and receive confirmations from *before* they are committed to the DA layer. Since based rollups cannot assign a dedicated sequencer, they rely on [preconfs](https://ethresear.ch/t/based-preconfirmations/17353) which allow proposers to give users cryptoeconomically-backed credible commitments about user transactions, e.g., “I will include your rollup transaction in a blob when it’s my turn to propose in three slots.” 

Using preconfs, a based rollup can give users confirmations ahead of them being committed to the DA layer, providing the same UX as a classical rollup. In practice this requires a registry of validators that are opted in to be preconfers for the rollup and a view of the beacon chain lookahead window which signals which validators are up next to propose. With just this knowledge, a rollup determine which L1 validators can sequence them and when.

### Execution Tickets 
Spire has been a proponent of using [tickets](https://docs.spire.dev/based-stack/based-sequencing#based-sequencer-election) and a lottery system to select which L1 proposer is allowed to sequence the rollup. L1 proposers in possession of a ticket (purchased from an L1 contract via descending dutch auction) are selected by lottery to become the shared sequencer for the opted-in based rollups during their slot. If no eligible L1 proposers exist, sequencing will fallback to "total anarchy mode."

Note that while this approach still requires a view of the lookahead as described previously, the ticketing system a more refined way of selecting proposers that allows MEV to be recaptured by the based rollups in the form of ticket-sale revenue.

## Gateways
Issuing preconfs to the users of potentially many different rollups adds many requirements to L1 proposers. To mitigate proposer sophistication, often a delegation step is assumed, similar to how most proposers today delegate their block building responsibilities to builders. 

In these designs, the proposer delegates sequencing and preconf rights to a more sophisticated entity referred to as a *gateway*: 
> The entity responsible for issuing preconfirmations and sequencing the L2. Sophisticated L1 proposers may choose to operate their own gateways, while others can delegate sequencing responsibilities to third-party operators. - [Kubi Mensah](https://ethresear.ch/t/becoming-based-a-path-towards-decentralised-sequencing/21733#p-52878-terminology-6)

Note that there are [concerns](https://ethresear.ch/t/preconfirmation-fair-exchange/21891#p-53230-extended-gateway-discussion-20) around gateway centralization and this topic is still under R&D, but for simplicity, the rest of this document we will assume that a gateway is acting on behalf the validator to both build their L1 blocks and serve as a shared sequencer for based rollups.

# L1 🔁 L2 atomic inclusion

{: .important-title }
> Remember
>
> Shared sequencing is just a subset of *based sequencing*, so all of the same guarantees we’ve previously covered apply to based sequencing. 

Given based sequencing can do everything shared sequencing can, what does based sequencing offer that shared sequencing cannot?

> There’s no way around this – if you want a credible commitment to the ordering of an Ethereum block, it’s gotta come from Ethereum validators. […] they can provide the same atomic inclusion guarantees with the [L1] as you can get between any rollups sharing the sequencer. - [Jon Charbonneau](https://dba.xyz/were-all-building-the-same-thing/#non-based-rollups-based-fallback)

The prerequisite for *atomic inclusion* was that the rollups shared the same global history and the way to guarantee this was by giving a single entity the sole authority to write it. Rollups can easily change the rules for who is allowed to sequence them but the L1 *cannot*: only Ethereum validators have proposing rights. 

This is what makes them so unique - Ethereum validators are the *only* entity that can have a write-lock on both the L1 and based rollups which is the prerequisite for L1-L2 atomic inclusion guarantees. As we saw previously, atomic inclusion was the first step for synchrony!

## How to bundle

So, assuming we have a gateway, how can we extend atomic inclusion to apply to L1 transactions? One way to accomplish this is by acknowledging that blob transactions are just a special type of L1 transaction. In the L2 atomic inclusion case, we included cross-chain bundles inside a shared blob, submitted it as a single blob transaction, then expected the rollups to derive their state from the shared blob. 

To extend atomicity to the L1, we simply need to bundle in sub-transactions as part of our blob transaction (e.g., via [multicall](https://solidity-by-example.org/app/multi-call/)). This way either the whole L1 bundle lands (blob transaction plus sub-transactions) or it all will revert. 

For example, a blob transaction $$TX_{blob}$$ may contain the sub-transaction $$TX_{L1_{deposit}}$$, attempting to atomically deposit from the L1 to the L2 and then process it. You can see that having the write-lock across the L1 and L2s is the only way to *guarantee* this can happen synchronously. 

While the contents of the blob needs to follow the same technique [described before](/website/education/composability/atomic-inclusion#preventing-unbundling) to prevent unbundling, it is simpler in the L1 case. If anyone attempted to extract out $$TX_{L1_{deposit}}$$, the standard L1 multicall contract would prevent unbundling because the transaction is nested in $$TX_{blob}$$ rather than being an individual transaction. This is like trying to observe someone’s public SAFE transaction and trying to extract and excute one sub-transaction from their batch - it's not possible or many things would break!

# L1 🔁 L2 atomic execution
Atomic execution follows naturally from atomic inclusion. To understand this case, we can look to the famous flashloan example that leverages AggLayer for cryptographically-safe atomic execution from the [previous section](/website/education/composability/atomic-execution#the-agglayer-approach-cryptographic-safety) and [real-time proving](/website/education/composability/atomic-composability#real-time-proving).

## Flashloan example

Assume Alice wants to perform an L1 to L2 arbitrage that uses liquidity from the L1. The Gateway will help facilitate this via synchronous atomic execution. The following steps will be ordered *temporally* so bare with me - we'll revist them in a [more comfortable order later](/website/education/composability/composability-and-based#synchronous-composability-with-the-l1)! 

1. Gateway receives Alice's request
2. Gateway simulates receiving a flashloan on L1 Aave contract ($$TX_{borrow}$$) and creates a deposit transaction ($$TX_{deposit_{L1}}$$) on AggLayer's shared bridge destined for $$R_A$$
3. Gateway sequences the deposit on $$R_A$$ ($$TX_{deposit_{L2}}$$), performs the arbitrage ($$TX_{swap_{L2}}$$), then sequences a withdrawal transaction ($$TX_{withdraw_{L2}}$$)
4. Gateway creates $$blob_{R_A}$$, proves the state transition function of $$R_A$$ via validity proof $$\pi_A$$, and proves $$R_A$$ can settle to the shared bridge via pessimistic proof $$\pi_{PP}$$
5. Gateway simulates publishing the blob ($$TX_{publish}$$), verifying $$\pi_A$$ via $$TX_{validity}$$, and verifying $$\pi_{PP}$$ via ($$TX_{pessimistic}$$)
6. Gateway simulates withdrawing ($$TX_{withdraw_{L1}}$$) from the shared bridge 
7. Gateway simulates repaying the flashloan ($$TX_{repay}$$) to the L1 Aave contract and returning assets to Alice
8. Gateway aggregates all sub-transactions ($$TX_{borrow}$$, $$TX_{deposit_{L1}}$$, $$TX_{publish}$$, $$TX_{validity}$$, $$TX_{pessimistic}$$, $$TX_{withdraw_{L1}}$$, $$TX_{repay}$$) and proofs ($$\pi_A$$, $$\pi_{PP}$$) as calldata and wraps it in a blob transaction (containing $$blob_{R_A}$$) called $$TX_{Alice}$$
9. $$TX_{Alice}$$ is included in the L1 block proposed by the L1 validator that delegated to the gateway
10. $$TX_{Alice}$$ executes by first initiating the flashloan, then executing each sub-transaction within the callback

Assuming we have access to real-time proving, this entire interaction can happen synchronously. Atomicity is guaranteed because any failures of the sub-transactions cause the $$TX_{Alice}$$ to revert (meaning the blob doesn't land on-chain) and invalid L2 state is caught by the validity / pessimistic proof.

😲 - so complicated but so capital efficient!

# L1 🔁 L2 composability
Now to the finale!  How can we get composability so L2s start to feel like they're just part of the L1?

## L1 composing with L2
The L1 always has a view (but not necessarily the latest) of the L2’s state because the L1 hosts the L2’s bridge contract. This makes it easy for the L1 to compose with the L2, and in fact this is what happens with withdrawals: the L1 bridge contract conditions the withdrawal on the user having the correct L2 balance and verifies this against the latest L2 state. With real-time proving, the L1 can access the most up to date L2 state during block construction.

## L2 composing with L1

For the L2 to compose with the L1, it needs to be able to read the latest L1 state. Rollups typically provide a read-only view of the L1 in order to process new deposits, e.g., in the OP stack each L2 block is rooted in an L1 block. The limitation is that the L1 state limited to the *last block* since the current L1 state root is not known at execution time.

Given the proposer has a write-lock on the L1, it’s possible to do even better by providing the rollup with the L1’s current state during the block’s construction (i.e., via [EIP-7814](https://github.com/ethereum/EIPs/blob/1676c9451a75fd0740c65e7d1d5f18296d68a9a0/EIPS/eip-7814.md) which allows this introspection). Alternatively, Nethermind has proposed a design where messages (e.g., an oracle price feed) can be passed from L1 to L2 during the [same L1 slot](https://ethresear.ch/t/same-slot-l1-l2-message-passing/21186).


## Synchronous composability with the L1

If we modify our previous example, composability allows the interaction to rely less on the gateway.

During step 2, assuming the same-slot-messaging approach is used, the gateway would store a message in the `MessagingService` L1 contract and include the message hashes in $$TX_{publish}$$. During L2 execution, the rollup contract would consume the message from the `MessagingService` L2 contract to know that the L1 deposit actually took place before proceeding. 

Now to summarize at a high level in a more intuitive ordering:

1. Alice initiates a flashloan to borrow tokens 
2. Alice deposits the borrowed tokens to the L2 and passes a message via the L2's inbox
3. The L2 synchronously reads the message from L1, processes the deposit, performs the arbitrage, then initiates the withdrawal back to the L1
4. The L2's blob and proofs are submitted and verified on the L1, settling the L2's state 
5. The L1 contract synchronously reads the L2's state and uses it to complete the withdrawal
6. Alice finally repays the flashloan
7. Profit

# Summary
Synchronous composability allows rollups to cross boundaries and feel like one chain. Universal synchronous composability is when everything, L1 and L2s, feel like one chain. This is **only possible** with based sequencing, but getting there is not as simple as adding a based sequencer. 

Throughout this series we covered the ingredients needed for USC. As a recap:
- [*Atomic inclusion*](/website/education/composability/atomic-inclusion#atomic-inclusion) is a necessary prequisite that can be guaranteed by shared or based sequencers. Based rollups should [take measures](/website/education/composability/atomic-inclusion#preventing-unbundling) to prevent unbundling and strive to share blobs. However, atomic inclusion is not enough to guarantee *safe* cross-chain interoperability on its own. 
- The [*Open Intents Framework*](/website/education/composability/atomic-execution#an-aside----open-intents-framework-cryptoeconomic-safety) is a pragmatic way to achieve cross-chain interoperability that does not require shared sequencers or even rollup stacks to be aware of it (at the cost of capital efficiency). When [combined with shared or based sequencing](/website/education/composability/atomic-execution#oif--atomic-inclusion), it can improve efficiency.
- [Atomic Execution](/website/education/composability/atomic-execution#atomic-execution) guarantees that cross-chain transactions either all succeed or do not execute at all. Protocols like [AggLayer](/website/education/composability/atomic-execution#the-agglayer-approach-cryptographic-safety) guarantee safety via cryptography. When combined with a shared or based sequencer, the execution can happen both synchronously and atomically. Based rollups should opt in to [shared bridges](/website/education/composability/atomic-execution#shared-bridging) and settle together to unlock atomic execution. 
- [Composability](/website/education/composability/atomic-composability#composability) allows contracts to access state on other rollups. To do this synchronously, rollups need to prove their state is valid to other rollups in real time, motivating the need for [real-time provers](/website/education/composability/atomic-composability#real-time-proving).
- **Universal Synchronous Composability** requires atomic inclusion, atomic execution, shared bridges, composability facilitated by real-time provers, and importantly L1 proposers with write-locks on the L1. These ingredients all come together to allow the L1 and all based rollups to feel like one *unified* Ethereum.


{: .important-title }
> Goal
>
> Fabric's goal is help shepard the adoption of based rollups. We're using these learnings to motivate standards and public good infrastructure to help accelerate the based rollup ecosystem towards USC!

---

In the next bonus section we'll learn about *ultra transactions* and how Gwyneth is doing composability.

<span class="fs-8">
[> Bonus - Ultra Transactions](/website/education/composability/ultra-txs){: .btn }
</span>