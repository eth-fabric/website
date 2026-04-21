---
title: Slot-end Handoff
nav_order: 4.7
layout: katex
parent: Composability 101
permalink: /education/composability/slot-end-handoff
---
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

# The opposite direction

SCOPE gives the L2 sequencer temporary control over L1 state by coordinating with the proposer. Slot-end handoff, [proposed by Vitalik](https://ethresear.ch/t/combining-preconfirmations-with-based-rollups-for-synchronous-composability/23863), flips the direction: instead of the sequencer reaching up to L1, the L2 sequencer hands off temporary control of the L2 back to the L1 proposer near the end of the slot.

During that window — usually the last portion of the 12-second L1 slot — the L1 proposer holds both the L1 and L2 write-locks and can submit synchronously composable transactions. Once the window closes, control returns to the dedicated sequencer for the next slot.

# How it solves dual prestate

The [dual prestate problem](/website/education/composability/achieving-synchrony#ingredient-3-dual-prestate-control) goes away by briefly restoring *single-party* control over both chains.

1. For most of the slot, the L2 operates with a dedicated sequencer — classical rollup UX.
2. Near slot end, the sequencer stops accepting new L2 transactions and hands off the L2's execution to the upcoming L1 proposer.
3. The proposer now controls both L1 prestate (via their L1 block) and L2 prestate (via the handoff). They can bundle synchronous L1-L2 transactions during this window.
4. The slot closes; the sequencer resumes normal operation for the next slot.

The key insight: you don't need to solve dual prestate for the whole slot — only for the moments that need sync composability. Slot-end handoff gives you a predictable, recurring window for that without any standing preconf market.

# What it looks like in practice

- **Default path (most of the slot):** users send transactions to the L2 sequencer and get soft confirmations immediately.
- **Handoff window (slot end):** the sequencer closes its mempool. The L1 proposer gathers sync-composable bundles submitted by users or searchers, sequences them against a frozen L2 prestate, and includes the result in their L1 block.
- **Transition:** control returns to the sequencer for the next slot; L2 state includes anything the proposer sequenced during the handoff.

# Comparison to SCOPE

| | SCOPE | Slot-end handoff |
|---|---|---|
| Who holds the write-lock during sync? | L2 sequencer (via preconf from proposer) | L1 proposer |
| Preconf infrastructure required? | Yes | No |
| When can sync happen? | Anytime the sequencer buys a preconf | Fixed window at slot end |
| Proving cost burden | L2 sequencer | L1 proposer |

SCOPE and slot-end handoff are both *coordination-based* solutions to dual prestate — they differ in direction and in who pays the coordination cost.

# Tradeoffs

- **Narrower composability window:** users wanting sync composability must submit during the handoff window or wait for the next slot. This is predictable but less flexible than SCOPE's on-demand model.
- **Proposer absorbs proving costs:** because the proposer is building the sync bundle, they need access to whatever proving infrastructure the rollup requires. For real-time-proven rollups this can be significant.
- **Dead zone after handoff:** the L2 sequencer cannot resume L2 sequencing until after the L1 block has been confirmed, otherwise they could be building off the wrong L2 state. This introduces a "dead zone" while waiting, hurting UX. The new [fast confirmation rule](https://fastconfirm.it/) could potentially be used to shorten this.

# When slot-end handoff fits

Slot-end handoff works best for rollups that:
- Want to avoid standing preconf markets and the infrastructure overhead they imply.
- Can tolerate synchronous composability being available in a predictable window rather than on demand.
- Have L1 proposers (or their delegated builders) willing to absorb the proving and sequencing burden during handoff.

<span class="fs-8">
[< Back to Achieving Synchrony](/website/education/composability/achieving-synchrony){: .btn }
</span>
