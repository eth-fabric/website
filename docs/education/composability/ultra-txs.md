---
title: Bonus - Ultra Transactions
nav_order: 4.4
layout: default
parent: Composability 101
permalink: /education/composability/ultra-txs
---
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

[Ultra Transactions](https://ethresear.ch/t/ultra-tx-programmable-blocks-one-transaction-is-all-you-need-for-a-unified-and-extendable-ethereum/21673) (ULTRA TX) is a model where a **single powerful L1 bundle** encodes all L1 and L2 transactions. This design enables both atomicity and synchrony, all wrapped up in a convenient abstraction for developers to compose with contracts on other chains. 

---

## The Big Picture
ULTRA TX reimagines an Ethereum block as one programmable unit. Instead of a collection of unrelated transactions, the block becomes a single atomic object that includes L1 transactions, L2 settlements, and cross-chain calls all at once.  
- **For developers:** this abstraction makes it possible to write contracts that call into other rollups or into L1 as if everything lived on one chain.  
- **For users:** deposits, trades, or complex multi-chain operations all finalize together without waiting minutes or hours for messages to relay.  

ULTRA TX leverages account abstraction to simplify bundling and ensure uniform handling of signatures and verification. This makes it possible for L1 and L2 calls to be composed together within a single unified bundle.

By collapsing execution into a single bundle, ULTRA TX makes Ethereum feel like one unified environment while still benefiting from the scalability of rollups.

---

## Achieving Atomicity
A key limitation of today’s design is that L1 and L2 interactions are fragmented: a deposit might succeed on L1 but fail to be processed on L2, leaving users stuck. ULTRA TX enforces atomicity by treating the **entire bundle as a single transaction**:
- If any component — an L1 transfer, a rollup settlement, or a cross-chain call — fails validation, the whole ULTRA TX reverts.  
- This guarantees “all-or-nothing” execution across chains, exactly like atomic function calls inside a single contract today.  

Atomicity also extends to proofs: one proof can cover the entire ULTRA TX, ensuring that any invalidity rolls back the whole bundle.

---

## Achieving Synchrony
Beyond atomicity, ULTRA TX enables synchrony. This means contracts on different chains can not only call each other, but also **receive and act on return values immediately**. 

The mechanism hinges on the `XCALLOPTIONS` precompile, which extends the EVM so that a contract can switch execution context to another rollup (or even back to L1) during a single ULTRA TX. From the developer’s perspective, it feels like making a normal external call — but under the hood, the builder is actually switching chain execution environments (i.e., their DB) to make the function call and returning the result.

Concretely:
1. A contract on L1 or L2 invokes `XCALLOPTIONS`, targeting another chain.  
2. The block builder executes the destination call and captures the return value.  
3. The result is passed back into the source contract’s execution so it can continue immediately, as if the call had been local.  
4. When the ULTRA TX is submitted onchain, the proof ensures the simulated return values exactly match the actual execution across all chains.  

This sequencing gives developers the illusion of synchronous, local composability even when logic spans multiple chains.

---

## The Extension Oracle
A challenge arises when L1 contracts want to access results that were computed on an L2. Normally, the L1 EVM cannot directly execute L2 code. ULTRA TX introduces the **Extension Oracle** to solve this:
- During simulation, the builder executes the L2 function call and records its output.  
- This output is then placed into the Extension Oracle, a simple contract that serves the result back when the L1 transaction requests it.  
- From the L1 contract’s perspective, it simply calls into the Extension Oracle and gets the return value — no special logic or proofs required.  

This mechanism bridges L1 and L2 execution without breaking determinism: the Extension Oracle is populated with the same data that will be validated in the ULTRA TX proof, so any attempt to falsify results causes the entire bundle to revert.

## Execution Flow

The lifecycle of an ULTRA TX can be understood as five distinct stages, each ensuring that L1 and L2 logic execute atomically and synchronously:

1. **Block Construction (Offchain)**  
   - The master builder aggregates all pending L1 and L2 transactions into a single bundle.  
   - For any cross-chain calls, the builder simulates execution across chains, invoking the `XCALLOPTIONS` precompile to gather return values.  
   - Outputs of these calls are stored in an `ExtensionOracle`, so that L1 contracts can later “read” results computed offchain.

2. **Local Execution & Proof Generation**  
   - The builder executes the entire bundle locally, as if it were one big rollup block spanning L1 and all participating L2s.  
   - Any invalid or inconsistent cross-chain calls cause the entire simulation to revert.  
   - Once execution succeeds, a single zk proof (or proof aggregation) is generated for the ULTRA TX.

3. **Onchain Proposal**  
   - The builder submits the ULTRA TX as the **first transaction** of the L1 block.  
   - This transaction contains all batched L1 operations, L2 blobs, extension oracle inputs, and the proof inputs necessary to verify correctness.

4. **Onchain Verification**  
   - L1 contract verifies the zk proof for the ULTRA TX.  
   - If the proof is valid, the ULTRA TX commits all included state changes atomically across L1 and the referenced L2s.  
   - If the proof fails, the entire ULTRA TX reverts — ensuring no partial execution leaks through.

5. **Finalization & Settlement**  
   - Once accepted, the ULTRA TX acts as the canonical record for both L1 and the included L2s.  
   - Rollups inherit the L1’s finality, and applications can trust that all cross-chain calls, responses, and return values were executed in a single atomic scope.  

This flow ensures that from the perspective of both developers and users, an ULTRA TX behaves like one massive transaction that spans Ethereum and its rollups, yet settles and proves as cleanly as a normal L1 block.