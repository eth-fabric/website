---
title: SCOPE
nav_order: 4.42
layout: katex
parent: Composability 101
permalink: /education/composability/scope
---
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

# Motivation: one powerful transaction

Before diving into SCOPE, it helps to look at the design it generalizes: [Ultra Transactions](https://ethresear.ch/t/ultra-tx-programmable-blocks-one-transaction-is-all-you-need-for-a-unified-and-extendable-ethereum/21673) (ULTRA TX) from the Gwyneth research.

ULTRA TX reimagines an Ethereum block as one programmable unit. Instead of a collection of unrelated transactions, the block becomes a single atomic object that includes L1 transactions, L2 settlements, and cross-chain calls all at once.

- **For developers:** this abstraction makes it possible to write contracts that call into other rollups or into L1 as if everything lived on one chain.
- **For users:** deposits, trades, or complex multi-chain operations all finalize together without waiting minutes or hours for messages to relay.

## Atomicity via a single bundle

ULTRA TX enforces atomicity by treating the **entire bundle as a single transaction**:
- If any component — an L1 transfer, a rollup settlement, or a cross-chain call — fails validation, the whole ULTRA TX reverts.
- This guarantees "all-or-nothing" execution across chains, exactly like atomic function calls inside a single contract today.

## Synchrony via `XCALLOPTIONS`

The mechanism hinges on the `XCALLOPTIONS` precompile, which extends the EVM so that a contract can switch execution context to another rollup (or even back to L1) during a single ULTRA TX. From the developer's perspective, it feels like making a normal external call — under the hood, the builder is switching chain execution environments to make the function call and returning the result.

1. A contract on L1 or L2 invokes `XCALLOPTIONS`, targeting another chain.
2. The block builder executes the destination call and captures the return value.
3. The result is passed back into the source contract's execution so it can continue immediately.
4. When the ULTRA TX is submitted on-chain, the proof ensures the simulated return values exactly match the actual execution across all chains.

## The Extension Oracle

When L1 contracts want results computed on an L2, the **Extension Oracle** bridges them:
- During simulation, the builder executes the L2 function call and records its output.
- The output is placed into the Extension Oracle, a simple contract that serves the result back when the L1 transaction requests it.
- From the L1 contract's perspective, it simply calls into the Extension Oracle and gets the return value — no special logic or proofs required.

## What ULTRA TX gets right — and where it strains

ULTRA TX proves the concept: if one entity builds both the L1 block *and* the L2 blocks inside it, and bundles everything into a single proof-gated L1 transaction, you get USC. But this requires a *master builder* that operates every chain's sequencing inside a single pipeline. That's a strong coordination assumption and doesn't generalize cleanly to rollups with their own sequencer sets.

This is where SCOPE comes in: it keeps the top-of-block "programmable unit" idea but factors it into a minimal coordination protocol that any L2 sequencer can plug into — without ceding their sequencing role.

---

# SCOPE

[SCOPE](https://ethresear.ch/t/scope-synchronous-composability-protocol-for-ethereum/22978) (Synchronous Composability Protocol for Ethereum) is a minimal accounting framework that lets an L2 sequencer compose with L1 on demand while remaining in control of their rollup the rest of the time.

The core idea: the sequencer doesn't need the L1 write-lock *all* the time — just for the moments they're composing. SCOPE gives them a way to *rent* that lock from the L1 proposer, slot by slot, when they need it.

## How it solves dual prestate

Recall the [dual prestate problem](/website/education/composability/achieving-synchrony#the-dual-prestate-problem): a sequencer composing synchronously with L1 needs a stable view of both L1 and L2 state at execution time. The L2 prestate is trivially controlled (the sequencer has a monopoly). The L1 prestate is the hard part.

SCOPE resolves this through **proposer coordination**:

1. When the sequencer wants to compose with L1, they coordinate with the upcoming L1 proposer — typically by purchasing a top-of-block preconfirmation.
2. The proposer commits to a specific L1 prestate (e.g., "the first transaction of my block will be the sequencer's cross-chain bundle").
3. The sequencer now has a stable L1 prestate, builds the synchronous transaction, and hands it back for inclusion.
4. When composition isn't needed, users get the familiar dedicated-sequencer UX.

This is why SCOPE is sometimes called a "push-based" protocol: the L2 sequencer pushes synchronous transactions up to L1 when demand arises.

## Comparison to fully based

| | Fully based | SCOPE |
|---|---|---|
| Who sequences the L2? | L1 proposer (rotating) | Dedicated L2 sequencer |
| When does L1-L2 sync happen? | Every slot (by construction) | On demand |
| Preconf infra required? | Yes (for UX across every slot) | Yes (for composition slots) |
| Default UX | Based preconf-driven | Classical rollup |

Fully based rollups pay the coordination cost *every* slot just in case someone wants to compose synchronously. SCOPE only pays the coordination cost when a synchronous composition is actually requested.

## Tradeoffs

- **Preconf market required:** SCOPE depends on a functional preconf market where L2 sequencers can acquire top-of-block slots from L1 proposers. The reliability and price of that market directly impact how practical synchronous composition is.
- **Latency spike on compose:** composing synchronously adds coordination latency compared to a simple L2 transaction. Users pay for the sync path; they don't pay for it when not composing.
- **Proposer participation:** if the next L1 proposer hasn't opted into the preconf system, synchronous composition is unavailable for that slot — users fall back to async composability.

## When SCOPE fits

SCOPE is a natural fit for rollups that:
- Want to keep their dedicated sequencer and existing UX.
- Expect synchronous L1 composition to be a *feature* rather than the default.
- Are willing to rely on preconf infrastructure for the moments they need it.

It's a lighter-weight commitment than going fully based, at the cost of added complexity during the compose path.

<span class="fs-8">
[> Slot-end Handoff](/website/education/composability/slot-end-handoff){: .btn }
</span>
