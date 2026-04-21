---
title: State Locks
nav_order: 4.8
layout: katex
parent: Composability 101
permalink: /education/composability/state-locks
---
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

# Solving dual prestate ahead of time

The first three approaches — [fully based](/website/education/composability/fully-based), [SCOPE](/website/education/composability/scope), and [slot-end handoff](/website/education/composability/slot-end-handoff) — all resolve the [dual prestate problem](/website/education/composability/achieving-synchrony#ingredient-3-dual-prestate-control) at execution time by involving the L1 proposer in some way. State locks take a different route: resolve it **ahead of time** at the contract level, so no proposer coordination is needed when the sync transaction actually runs.

The intuition: if we can guarantee that a specific region of L1 state will only be modified by the L2 sequencer (or with enough notice that the sequencer can react), then the sequencer has a stable L1 prestate *by construction*. No preconfs, no handoffs — just invariants enforced by L1 smart contracts.

# The two invariants

A state lock is a region of L1 state — a set of smart contracts — that enforces two properties for a chosen L2 sequencer:

1. **Delayed / sequencer-exclusive writes.** Writes from the designated sequencer land immediately. Writes from anyone else are queued with a delay of at least one L1 slot. This gives the sequencer sole authority over *fast-path* state transitions, with a liveness-preserving escape hatch for everyone else.
2. **Read isolation.** The state lock reads only from itself (and other known-stable sources). It cannot be read-dependent on arbitrary external L1 state that a random proposer could modify during the sync window.

Together these guarantee that from the sequencer's perspective, the L1 prestate of the locked region is a deterministic function of what the sequencer itself already knows.

# How it solves dual prestate

Recall the core problem: the L1 proposer can modify L1 state underneath the L2 sequencer, breaking atomicity checks on synchronous transactions.

State locks neutralize this by ensuring the only L1 state the sync transaction touches is state the proposer *cannot* modify in the current slot:

- The **sequencer-exclusive write** invariant means only the sequencer can change the locked state in this slot. The proposer can't front-run a sync composition.
- The **read isolation** invariant means the sync transaction doesn't depend on L1 state the proposer *can* freely change.

The proposer is still the one who includes the sequencer's settlement transaction in their L1 block — but by the time it executes, every piece of state it touches is already locked down. No opt-in from the proposer required.

# Comparison to the coordination approaches

| | Fully based | SCOPE | Slot-end handoff | State locks |
|---|---|---|---|---|
| Resolution timing | At execution | At execution (on demand) | At execution (slot end) | Ahead of time |
| Proposer involvement | Sequences both | Sells preconf | Temporarily sequences L2 | None (for sync) |
| Preconf infra required? | Yes | Yes | No | No |
| Works for arbitrary L1 state? | Yes | Yes | Yes | No — only state within the lock |

The state-lock approach trades *generality* for *decoupling*. It doesn't give you synchronous composability with arbitrary L1 contracts — only with contracts deployed inside the lock. In exchange, it requires zero real-time proposer cooperation.

# Tradeoffs

- **Bootstrapping:** existing L1 contracts (Aave, Uniswap, etc.) aren't inside the lock and can't be composed with synchronously unless they're redeployed or wrapped. This is the biggest cost compared to proposer-coordination approaches.
- **Liveness timeout tuning:** the delay applied to non-sequencer writes is a liveness vs. responsiveness knob. Too short and the sequencer can be front-run; too long and legitimate L1 users wait excessively during sequencer censorship.
- **Read-isolation discipline:** contracts inside the lock must be carefully audited to ensure they don't read from unlocked state, or the guarantee unravels.
- **Asset escrow model:** since the lock defines a distinct state region, user assets participating in synchronous composability typically need to live *inside* the lock. This is similar to a bridge but with stronger sync guarantees.

# When state locks fit

State locks are a natural fit for:
- Greenfield applications that can deploy their L1 side inside the lock from day one.
- Designs that want synchronous L1-L2 composability without introducing or depending on preconf markets.
- Situations where decoupling sequencing from L1 proposer consent is valuable — e.g., to preserve the rollup's dedicated-sequencer UX and lazy settlement economics.

They're less suited for applications that need to compose synchronously with pre-existing, external L1 state.

<span class="fs-8">
[< Back to Achieving Synchrony](/website/education/composability/achieving-synchrony){: .btn }
</span>
