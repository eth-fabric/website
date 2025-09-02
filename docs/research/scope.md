---
title: Scope
nav_order: 5.5
layout: default
parent: Research
permalink: /research/scope
---
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

{: .important-title }
> Info
>
> For the full post see Fabric's post on [ethresearch](https://ethresear.ch/t/scope-synchronous-composability-protocol-for-ethereum/22978).

# SCOPE: Synchronous Composability Protocol for Ethereum

## What it Does
SCOPE is an accounting framework that enables **synchronous cross-chain function calls with return values**.  

With SCOPE, a contract on one chain can call into a contract on another chain and *immediately* consume the result — as if both lived on the same chain. This restores the seamless developer experience Ethereum originally offered before execution fragmented across rollups.  

## Why it Matters
Today’s cross-chain communication is slow and one-directional: a rollup can “send a message” to another, but the sender cannot act on the result until minutes or hours later. This makes contracts across rollups feel like isolated islands.  

SCOPE makes them feel like a single mainland again: you can call, get a response, and act on it instantly — preserving Ethereum’s core property of composability.

## Background
SCOPE builds directly on two earlier designs:

- **Ultra Transactions**  
[Ultra TXs](../education/composability/ultra-txs.md) introduced the idea of bundling all L1 and L2 transactions into a single atomic L1 transaction. This enabled synchronous cross-chain execution by relying on account abstraction, top-of-block inclusion, and real-time proving. The power of Ultra TXs comes from collapsing multiple domains into one programmable “ultra transaction.”

- **CIRC (Coordinated Inter-Rollup Communication)**  
   [CIRC](https://espresso.discourse.group/t/circ-coordinated-inter-rollup-communication/43/3) introduced the idea of mailbox-style commitments. Each rollup maintains inboxes for incoming cross-chain messages and outboxes for outgoing ones. Because these mailboxes are Merkleized, it’s easy to verify that what one rollup sends matches what another receives.

   When you combine mailbox commitments with shared settlement, you get atomicity: if the inbox and outbox roots don’t match, settlement fails. This ensures no message can be skipped or tampered with.

   The limitation is that this model is pull-based. A message has to be written to the outbox on one chain first, and only later read and consumed from the inbox on the destination chain. That means you can’t act on the result instantly — you need two separate transactions.

**SCOPE combines these insights:**  
From CIRC it inherits efficient, verifiable accounting using rolling hashes.  
From Ultra TX it inherits the push-based execution model, bundling, and reliance on coordination with the L1 proposer (via top-of-block guarantees).

Together, these allow synchronous function calls between L1 and L2s.

## How it Works
SCOPE builds on [**Signal-Boost’s insight**](signal-boost.md#how-it-works) that synchronous composability does not strictly require rollups to be fully based. Instead, it requires **coordination between the L1 proposer and the L2 sequencer**, typically by giving the sequencer *top-of-block guarantees*.  

On top of this, SCOPE introduces a simple but powerful accounting model:

- Each chain tracks **rolling hashes** of all cross-chain requests and responses:
  - `requestsOutHash` / `requestsInHash`
  - `responsesOutHash` / `responsesInHash`
- When a contract calls `scopedCall()`, the source chain appends the request to its `requestsOutHash`.
- The shared sequencer injects a matching `handleScopedCall()` on the destination chain, executes it, and updates `responsesOutHash`.
- The sequencer then pre-fills the source chain’s `responsesIn` mapping so that when the `scopedCall()` executes, it reads and returns the result synchronously.
- At settlement, both chains prove equivalence:  
  `source.requestsOutHash == dest.requestsInHash` and  
  `source.responsesInHash == dest.responsesOutHash`.  

If any call is skipped, reordered, or tampered with, the hashes diverge and settlement fails.

## Why this is a Progression
Signal-Boost showed how to give rollups synchronous access to **read arbitrary L1 state** using delegated bundling and top-of-block execution.  
SCOPE generalizes this further: it makes synchronous function calls possible in *all directions* (L1↔L2 and L2↔L2), with verifiable return values and atomicity.  

This means developers can now:
- Call into a contract on another rollup and immediately use the result.  
- Build cross-rollup dapps that rely on instant coordination.  
- Compose logic across L1 and L2s without waiting for long settlement delays.  

## End-to-End Flow
1. **User call:** A contract on Rollup A invokes `scopedCall()` targeting Rollup B.  
2. **Request hash:** Rollup A updates its `requestsOutHash`.  
3. **Execution on B:** The sequencer inserts `handleScopedCall()` on Rollup B, executes the function, and updates `responsesOutHash`.  
4. **Response pre-fill:** The sequencer injects `fillResponsesIn()` on Rollup A, storing the response bytes.  
5. **Return:** When Rollup A’s `scopedCall()` executes, it reads the pre-filled response and continues execution as if the call was local.  
6. **Settlement:** At the end of the block, both rollups prove their rolling hashes match along with their state transition functions. If they don’t, the bundle reverts atomically.  

---

SCOPE builds on the accounting rigor of CIRC and the atomic bundling of Ultra TX, while applying the coordination guarantees pioneered by Signal-Boost. The result is a framework that pushes Ethereum closer to a unified blockspace where fragmentation disappears and composability is restored.