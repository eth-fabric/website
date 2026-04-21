---
title: Step 4 - Achieving Synchrony
nav_order: 4.4
layout: default
parent: Composability 101
permalink: /education/composability/achieving-synchrony
---
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

# From async to sync

In the previous sections we built up the ingredients for cross-chain composability: [atomic inclusion](/website/education/composability/atomic-inclusion), [atomic execution](/website/education/composability/atomic-execution), and [composability](/website/education/composability/atomic-composability) itself. Together, these allow contracts on different rollups to access each other's state and execute atomically — but so far, this can happen *asynchronously*, meaning the composing rollups don't need to settle in the same L1 slot.

For L2-to-L2, synchronous composability is achievable with a shared sequencer (or any L2-L2 coordinator) as we covered in [Step 3](/website/education/composability/atomic-composability). The remaining challenge is **L1-to-L2 synchrony**: can we make composability between the L1 and L2s happen within a single slot, so that everything feels like one unified chain? This is what we've been calling Universal Synchronous Composability (USC).

# The dual prestate problem

At the heart of every approach to L1-L2 synchronous composability lies the same fundamental challenge: **dual prestate control**.

For an L2 sequencer to synchronously compose with the L1, they need a stable view of *both* the L1 and L2 state at the time of execution. If either state changes underneath them, atomicity checks would reject the state transition, requiring a rollback with bad UX.

- **L2 prestate** is trivial to control — the sequencer already has a monopoly on L2 state transitions.
- **L1 prestate** is the hard part — the L1 proposer has the final say over what gets included in the L1 block. If someone modifies L1 state that the sequencer was composing against, the synchronous transaction breaks.

Every approach to L1-L2 synchronous composability must solve this dual prestate problem. Note that L2-to-L2 synchrony doesn't face this challenge since a shared sequencer already controls both prestates. They differ in *how* they give the sequencer confidence that the L1 state won't change underneath them.

# Four approaches

The design space for L1-L2 synchronous composability has evolved significantly. What was once thought to require based sequencing can now be achieved through several distinct mechanisms, each with different tradeoffs:

| Approach | How it solves dual prestate | Requires based sequencing? | Requires preconf infra? |
|----------|---------------------------|---------------------------|------------------------|
| [Fully based rollups](/website/education/composability/fully-based) | Proposer controls both L1 and L2 prestates naturally | Yes | Yes (for UX) |
| [SCOPE](/website/education/composability/scope) | L2 sequencer coordinates with L1 proposer on demand | No | Yes |
| [Slot-end handoff](/website/education/composability/slot-end-handoff) | Sequencer hands L2 control to proposer at end of slot | No (but proposer involved) | No |
| [State locks](/website/education/composability/state-locks) | Protected L1 state with delayed writes — no proposer coordination | No | No |

## Fully based rollups

The most natural path. Based rollups delegate sequencing to L1 proposers, who already have a write-lock on the L1. When the same proposer sequences both the L1 and based rollups, dual prestate control is satisfied by default. The tradeoff is UX — without a dedicated sequencer, users rely on preconfs for soft confirmations, and the rollup's liveness is tied to opted-in L1 validators.

<span class="fs-8">
[> Fully Based Rollups](/website/education/composability/fully-based){: .btn }
</span>

## SCOPE

[SCOPE](https://ethresear.ch/t/scope-synchronous-composability-protocol-for-ethereum/22978) is an accounting framework that allows an L2 sequencer to remain in control of their rollup while composing with L1 on demand. The sequencer coordinates with the L1 proposer — for example, by purchasing a top-of-block preconf — to get temporary guarantees about L1 state. When not composing, users get the familiar rollup UX. The tradeoff is that this coordination requires preconf infrastructure.

<span class="fs-8">
[> SCOPE](/website/education/composability/scope){: .btn }
</span>

## Slot-end handoff

[Proposed by Vitalik](https://ethresear.ch/t/combining-preconfirmations-with-based-rollups-for-synchronous-composability/23863), this approach takes the opposite direction: rather than giving the sequencer control over L1, the sequencer hands off temporary control of the L2 to the L1 proposer towards the end of the slot. By briefly restoring dual prestate control, the proposer can submit synchronously composable transactions during this window. The tradeoff is a smaller composability window and the proposer absorbing proving costs.

<span class="fs-8">
[> Slot-end Handoff](/website/education/composability/slot-end-handoff){: .btn }
</span>

## State locks

Rather than resolving dual prestate at execution time by involving the proposer, state locks solve the problem **ahead of time**. Protected L1 state is deployed via smart contracts where the L2 sequencer has privileged write access. Non-sequencer writes are delayed by at least one slot, guaranteeing the sequencer a stable L1 prestate without any proposer coordination. The tradeoff is bootstrapping — existing L1 contracts are not directly compatible and the protected state must be deployed from scratch.

<span class="fs-8">
[> State Locks](/website/education/composability/state-locks){: .btn }
</span>

# Series summary

Synchronous composability allows rollups to cross boundaries and feel like one chain. Universal synchronous composability is when everything — L1 and L2s — feel like one chain.

Throughout this series we covered the ingredients needed for USC:

- [*Atomic inclusion*](/website/education/composability/atomic-inclusion) is a necessary prerequisite that can be guaranteed by shared or based sequencers. Rollups should [take measures](/website/education/composability/atomic-inclusion#preventing-unbundling) to prevent unbundling and strive to share blobs. However, atomic inclusion is not enough to guarantee *safe* cross-chain interoperability on its own.
- The [*Open Intents Framework*](/website/education/composability/atomic-execution#an-aside----open-intents-framework-cryptoeconomic-safety) is a pragmatic way to achieve cross-chain interoperability that does not require shared sequencers or even rollup stacks to be aware of it (at the cost of capital efficiency). When [combined with shared or based sequencing](/website/education/composability/atomic-execution#oif--atomic-inclusion), it can improve efficiency.
- [*Atomic Execution*](/website/education/composability/atomic-execution) guarantees that cross-chain transactions either all succeed or do not execute at all. Protocols like [AggLayer](/website/education/composability/atomic-execution#the-agglayer-approach-cryptographic-safety) guarantee safety via cryptography. When combined with a shared or based sequencer, the execution can happen both synchronously and atomically.
- [*Composability*](/website/education/composability/atomic-composability#composability) allows contracts to access state on other rollups. For L2-to-L2 this requires atomicity enforcement. For L1-to-L2, achieving this synchronously requires solving the dual prestate problem — and the design space now includes [four distinct approaches](#four-approaches), each with different tradeoffs around proposer involvement, infrastructure requirements, and bootstrapping costs.

{: .important-title }
> Goal
>
> Fabric's goal is help shepherd the adoption of based rollups. We're using these learnings to motivate standards and public good infrastructure to help accelerate the based rollup ecosystem towards USC!
