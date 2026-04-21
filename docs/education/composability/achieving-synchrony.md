---
title: Step 4 - Achieving Synchrony
nav_order: 4.4
layout: katex
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

In the previous sections we built up the ingredients for cross-chain composability: [atomic inclusion](/website/education/composability/atomic-inclusion), [atomic execution](/website/education/composability/atomic-execution), and [composability](/website/education/composability/atomic-composability) itself. Together, these allow contracts on different rollups to access each other's state and execute atomically — however, this can happen *asynchronously*, meaning the composing rollups do not need to settle in the same L1 slot.

For L2-to-L2, synchronous composability is achievable with a shared sequencer (or any L2-L2 coordinator). The remaining challenge is **L1-to-L2 synchrony**: composability between the L1 and L2s within a single slot, such that the unified system behaves as one chain. This is what we have been referring to as Universal Synchronous Composability (USC).

Achieving USC requires three ingredients. Each is a distinct problem with its own research agenda, and all three are required together.

# Ingredient 1: Reorg willingness

A synchronous composition binds state across chains. If one side reorgs and the other does not, atomicity is lost — the composition commits on one chain and disappears on the other. Since the L1 will occasionally reorg regardless of rollup design, the requirement is not to prevent reorgs but for the composing chains to **reorg together**.

- **L1-to-L2:** if the L1 reorgs out the slot that included the composition, the L2 must follow. An L2 that does not reorg with its L1 cannot safely offer L1-to-L2 synchronous composability.
- **L2-to-L2:** if two rollups compose synchronously and one of them reorgs, the other must reorg the matching portion of its history, otherwise the cross-chain invariant is violated.

This is a design choice at the rollup's fork-choice layer and has UX consequences — soft confirmations become conditional on the finality of all composing chains. Minimizing reorg risk via fast L1 finality, preconf-backed confirmation rules, and related mechanisms is what keeps this tradeoff acceptable.

→ See [L1 Reorgs](/website/education/l1-reorgs) for a deeper treatment of how L2s handle this in practice.

# Ingredient 2: Real-time proving

Covered in detail in [Step 3](/website/education/composability/atomic-composability#real-time-proving). In summary: for a cryptographic atomicity gadget to operate at L1 cadence, the L2's state transition function (along with any gadget-supplementary proofs) must be proven within a single L1 slot. The trust spectrum ranges from a single TEE to a committee of TEEs to a full ZK proof, with sub-slot ZK proving now viable as demonstrated in [recent work](https://ethresear.ch/t/synchronous-composability-between-rollups-via-realtime-proving/23998).

# Ingredient 3: Dual prestate control

For a synchronous composition to produce the result the user intended, the composing parties require a coordinated view of both chains' prestates at execution time. This requirement applies to both shapes of composability, for distinct reasons:

- **Optimistic case:** the sequencer simulates cross-chain values. If the real state diverges from the simulated state, the settlement-time gadget detects the mismatch and forces the composition to roll back. This is safe but expensive, and degrades UX if it occurs frequently.
- **Pessimistic case:** contracts read against committed state roots via storage proofs. The read is *authenticated* but potentially *stale* — if the source chain's state has advanced since the proven root, the composition executes against a historical view rather than the current one. A read-compute-writeback across chains A and B will not produce the user's intended result unless the two sequencers coordinate.

In both cases, the composing parties must guarantee stable, coordinated prestates across the participating chains.

- **L2 prestate** is straightforward to control — the sequencer holds a monopoly over L2 state transitions.
- **L1 prestate** is the challenging case — the L1 proposer has final authority over the contents of the L1 block. If the proposer modifies L1 state underneath the composition, the outcome diverges from the user's intent regardless of the composability shape.

L2-to-L2 synchrony is comparatively straightforward: a shared sequencer (or coordinated sequencer pair) already controls both prestates. L1-to-L2 is the harder case and is where most recent design work has focused. The optimistic path in particular ([SCOPE](/website/education/composability/scope), [slot-end handoff](/website/education/composability/slot-end-handoff), [state locks](/website/education/composability/state-locks), [fully based rollups](/website/education/composability/fully-based)) has received the most attention recently. Four distinct strategies have emerged for handling L1 prestate control:

| Strategy | How it controls L1 prestate | Requires preconf infra? | Requires proposer coordination? |
|----------|----------------------------|-------------------------|----------------------------------|
| [Fully based rollups](/website/education/composability/fully-based) | L1 proposer *is* the L2 sequencer | Yes (for UX) | N/A (same party) |
| [SCOPE](/website/education/composability/scope) | L2 sequencer acquires temporary L1 write-lock via preconf | Yes | Yes (on demand) |
| [Slot-end handoff](/website/education/composability/slot-end-handoff) | L2 sequencer transfers L2 control to proposer at slot end | No | Yes (every slot) |
| [State locks](/website/education/composability/state-locks) | L1 invariants guarantee stable prestate ahead of time | No | No |

The first three strategies resolve dual prestate at execution time by involving the L1 proposer. The fourth resolves it ahead of time via L1 smart contract invariants.

Each strategy is documented in its own page — these are parallel alternatives, not a sequence:

- [Fully Based Rollups](/website/education/composability/fully-based) — proposer holds both write-locks; strongest guarantees, highest UX cost.
- [SCOPE](/website/education/composability/scope) — sequencer acquires the L1 write-lock on demand; suited to designs where synchronous composability is an optional feature rather than the default.
- [Slot-end Handoff](/website/education/composability/slot-end-handoff) — sequencer transfers L2 control to the proposer at slot end; no preconf market required.
- [State Locks](/website/education/composability/state-locks) — invariants enforced at the contract level; decouples synchrony from proposer consent at the cost of bootstrapping.

# Series summary

Synchronous composability allows rollups to cross boundaries and behave as one chain. Universal synchronous composability extends this to the L1 and all participating L2s. To achieve USC, a system requires:

1. **Reorg willingness** — composing chains reorg together, preserving the atomicity binding across reorgs on either side.
2. **Real-time proving** — the L2's state transition function can be proven within a single L1 slot.
3. **Dual prestate control** — the composing chains maintain a coordinated view of each other's prestate, so the composition produces the user's intended result rather than one that is merely consistent with stale state.

Throughout this series we covered the building blocks that make all of this possible:

- [*Atomic inclusion*](/website/education/composability/atomic-inclusion) is the prerequisite — guaranteed by shared or based sequencers, with [measures](/website/education/composability/atomic-inclusion#preventing-unbundling) to prevent unbundling.
- [*Atomic execution*](/website/education/composability/atomic-execution) guarantees cross-chain transactions either all succeed or do not execute at all. Protocols such as [AggLayer](/website/education/composability/atomic-execution#the-agglayer-approach-cryptographic-safety) provide cryptographic safety; the [Open Intents Framework](/website/education/composability/atomic-execution#an-aside----open-intents-framework-cryptoeconomic-safety) provides cryptoeconomic safety.
- [*Composability*](/website/education/composability/atomic-composability#composability) allows contracts to access state across rollups. L2-to-L2 requires only atomicity enforcement; L1-to-L2 requires all three ingredients above.

## Going deeper

The strategy pages above ([Fully Based Rollups](/website/education/composability/fully-based), [SCOPE](/website/education/composability/scope), [Slot-end Handoff](/website/education/composability/slot-end-handoff), [State Locks](/website/education/composability/state-locks)) describe each approach at a conceptual level. For readers interested in more opinionated, implementation-oriented designs for L1-L2 synchronous composability, Fabric has published several research posts that dig into specific mechanisms:

- [SCOPE](/website/research/scope) — full protocol specification for the accounting framework outlined in the strategy page.
- [Signal-Boost](/website/research/signal-boost) — a plugin that lets rollups ingest generic L1 state synchronously, enabling L2 contracts to react to fresh L1 values within the same slot.
- [Tobasco](/website/research/tobasco) — a top-of-block inclusion primitive that supports synchronously composable operations by enforcing ToB execution against the latest L1 state.

These are opinionated designs rather than survey material — readers should expect concrete mechanism details, tradeoff analysis, and PoC code where applicable.

{: .important-title }
> Goal
>
> Fabric's goal is to help shepherd the adoption of based rollups and the broader infrastructure that makes Ethereum feel like a unified chain. The design space for dual prestate control has expanded — our role is to help the ecosystem select the appropriate tool for each context and build the standards that make them interoperable.
