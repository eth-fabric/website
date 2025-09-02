---
title: Signal-Boost
nav_order: 5.3
layout: default
parent: Research
permalink: /research/signal-boost
---
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

![Signal-Boost](/website/assets/images/signal-boost.png)

{: .important-title }
> Info
>
> For the full post see Fabric's post on [ethresearch](https://ethresear.ch/t/signal-boost-l1-interop-plugin-for-rollups/22354/1).

# Signal-Boost: L1 Interop Plugin for Rollups

## What it Does
Signal-Boost allows rollups to ingest and react to *generic L1 state* in real time.  
Unlike today’s bridges or async messaging, this means an L2 contract can synchronously read fresh L1 values — such as an oracle price or vault status — and act on them in the same L1 slot.  

## Why it Matters
Current same-slot messaging (e.g. [Nethermind’s Same-Slot Message Passing design](https://ethresear.ch/t/same-slot-l1-l2-message-passing/21186)) only works if upstream L1 contracts explicitly push data into a special `SignalService` contract. That severely limits adoption since existing L1 protocols would need to change their logic.  

Signal-Boost generalizes the idea: anyone can query arbitrary L1 view functions and commit the results as verifiable signals. L2 contracts can then consume these signals immediately.

## How it Works
To accomplish this, Signal-Boost borrows key techniques from [**Ultra Transactions**](https://ethresear.ch/t/ultra-tx-programmable-blocks-one-transaction-is-all-you-need-for-a-unified-and-extendable-ethereum/21673):

- **Bundling via Account Abstraction (EIP-7702):** The L2 sequencer can combine user transactions with their own signal-writing transaction into a single L1 bundle.

- **Top-of-Block Execution:** By placing the bundle at the very start of the L1 block, the signals reflect a clean, known post-state from the previous block. This avoids inconsistencies that arise if mid-block L1 transactions shift the state.

The biggest insight is that achieving synchronous L1↔L2 composability **does not require the L2 sequencer to also be the L1 proposer**. Instead, it only requires coordination: the sequencer ensures their bundle is given top-of-block execution guarantees by the L1 proposer. This relaxes the “based sequencer” assumption while still delivering the UX benefits.

## End-to-End Flow
Here’s how a synchronous transaction looks with Signal-Boost:

1. **User intent:** An L2 user submits a transaction that depends on live L1 data (e.g. “trade if Chainlink price < X”).  
2. **Sequencer prep:** The L2 sequencer collects the user’s transaction and a list of L1 view queries (`SignalRequests`).  
3. **L1 bundle:** The sequencer bundles:
   - A call to `SignalBoost.writeSignals()` to query L1 contracts and publish a `signalRequestsRoot`.  
   - The L2 batch that consumes those signals.  
4. **Simulation:** The sequencer simulates execution locally, learns the `signalRequestsRoot`, and builds Merkle proofs.  
5. **Injection:** The bundle is placed at the **top of the L1 block**, ensuring the signals are fresh and consistent.  
6. **Execution:**  
   - On L1: `SignalBoost` commits the root to `SignalService`.  
   - On L2: the batch verifies the proofs and executes user logic using the signals.  
7. **Settlement:** If using real-time proving, the sequencer can include a rollup validity proof in the same bundle, making the L1 and L2 state updates atomic.

The result is that an L2 user can synchronously consume arbitrary L1 state in their contract calls.