---
title: Step 3 - Composability
nav_order: 4.3
layout: katex
parent: Composability 101
permalink: /education/composability/atomic-composability
---
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---
So far by combining shared sequencing, shared bridging, and pessimistic proofs we've been able to achieve synchronous atomic execution. In the previous example, $$R_A$$ and $$R_B$$'s settlement was contingent on each other's post-state root which provided the safety guarantees; however, the contracts involved with $$TX_{burn_{R_A}}$$ and $$TX_{mint_{R_B}}$$ didn't necessarily know of each others' existence. While satisfactory for cross-chain swaps, this is insufficient to cover all use cases (e.g., $$TX_{R_B}$$ depends on an oracle update in contract $$C_{R_A}$$). To really make the developer experience feel like one chain, we want *composability.*

> Synchronous composability requires the ability for state machine of each chain to access the other chain's state (e.g., the bridge contract itself on $$R_B$$ needs to know the state of $$R_A$$) at the time of execution. [- Jon Charbonneau](https://dba.xyz/were-all-building-the-same-thing/#cross-chain-unbundling)
>

Mathematically we can say that the new state of $$R_B$$ depends on the state of $$R_A$$: $$state_B = (STF_B(STF_A(X)))$$ — readers are encouraged to watch [Ellie's talk](https://x.com/Agglayer/status/1870113677682389129) on this.

## Two ingredients: access and atomicity

Composability takes two ingredients. Contracts need some way to **access** cross-chain state during execution, and the composition needs an **atomicity binding** that guarantees the whole cross-chain interaction either commits together or unwinds together.

Focusing on the synchronous case — where the composition must happen in a single slot — the interesting question is *when* atomicity is enforced. Two shapes emerge:

- **Optimistic composability:** the sequencer doesn't have cross-chain state available at execution time, so it *simulates* the values it needs. Atomicity is enforced later, at **settlement**, via an explicit gadget. If the simulation was wrong, settlement rejects the whole thing.
- **Pessimistic composability:** contracts read cross-chain state *directly* at execution time (typically via storage proofs against another rollup's state root). Atomicity is enforced immediately — a bad state or a failing proof reverts the transaction right there, at **execution time**.

The two shapes aren't mutually exclusive and some systems combine them, but the *when* is the cleanest way to tell them apart.

### Optimistic composability

In the optimistic shape, the sequencer builds the cross-chain bundle assuming values it can't directly observe — for example, a cross-chain call's return value. It emits commitments describing what it assumed, and a settlement-time gadget checks those commitments against what actually happened on the source chain. If they match, the composition commits; if not, the whole thing reverts.

This gives you composability without requiring the smart contract itself to access the other chain's state at execution time — the sequencer does the work of simulating cross-chain values off to the side, and the contract simply trusts them subject to the settlement-time check. The tradeoff is that the atomicity gadget becomes the load-bearing piece — get that wrong and the safety property breaks.

Concrete instantiations of optimistic composability are covered in the Step 4 strategy pages, since which gadget you use depends on which synchrony strategy the rollup adopts.

### Pessimistic composability

In the pessimistic shape, $$R_B$$'s runtime is given direct access to $$R_A$$'s state root (e.g., via a precompile), and contracts on $$R_B$$ verify storage proofs against that root as part of their execution logic.

Specifically for $$R_B$$ to compose with $$R_A$$, the validity proof $$\pi_B$$ will depend on the state root of $$R_A$$ as an input. When executing $$TX_{mint_{R_B}}$$, contract $$C_{R_B}$$ can first require that $$TX_{burn_{R_A}}$$ successfully burned tokens on $$C_{R_A}$$ by verifying a storage proof against the state root of $$R_A$$. The logic in $$C_{R_B}$$ enforces a safe mint, and any inconsistency causes the transaction to revert right there — no separate gadget needed.

Atomicity is still enforced, just at a different layer of the stack: the proof verification layer rather than a separate settlement-time check.

## The AggLayer approach (pessimistic, worked through)

AggLayer is a natural fit for the pessimistic shape in the synchronous setting. Let's revisit our example of a user submitting an atomic bundle with $$TX_{burn_{R_A}}$$ and $$TX_{mint_{R_B}}$$ to a shared sequencer that sequences both $$R_A$$ and $$R_B$$.

1. shared sequencer commits the shared rollup history to the L1 via shared blob transaction
2. $$R_A$$ and $$R_B$$'s derivation function guarantees the transactions cannot be unbundled (as described [before](/website/education/composability/atomic-inclusion#preventing-unbundling))
3. $$R_A$$ executes $$TX_{burn_{R_A}}$$ to burn the user's 100 USDC, resulting in $$State_{R_A}$$
4. $$R_B$$ performs a cross-chain call using $$State_{R_A}$$ to verify the state of $$C_{R_A}$$. After verifying the burn was successful, $$TX_{mint_{R_B}}$$ mints the user 100 USDC, resulting in $$state_{R_B}$$
5. $$R_A$$ and $$R_B$$ commit block headers to AggLayer that commit to each other's state roots ($$state_{R_A}$$ and $$state_{R_B}$$)
6. $$R_A$$ and $$R_B$$ submit $$\pi_A$$ and $$\pi_B$$ to AggLayer. Note that $$\pi_B$$ depends on $$State_{R_A}$$ as input
7. AggLayer generates and submits the pessimistic proof to the shared bridge

Step 4 demonstrates $$R_B$$ composing with $$R_A$$ to perform a cross-chain read. From the pov of the $$C_{R_B}$$ developer, this interaction functioned as if $$C_{R_A}$$ lived on the same chain! Note that AggLayer's pessimistic *proof* at settlement (step 7) is a safety net at the system level — orthogonal to the pessimistic *composability* shape described here, which is about when atomicity is enforced during the cross-chain call itself.

\* rollups still need a precompile to make cross-chain calls

## Settlement cadence: where the real-time proving requirement comes from

The defining difference between the two shapes is *when* the atomicity check happens — at execution, or at settlement. That matters because settlement is what gives an atomicity gadget a deadline.

**L2-to-L2 composability:** rollups choose when they settle. If two rollups want to compose, their shared atomicity gadget can run on their own cadence. Settlement can happen several L1 slots later and the atomicity guarantee still holds. No real-time proving required — atomicity enforcement is sufficient.

**L1-to-L2 composability:** the L1 settles every single slot. If the composition touches L1 state, the atomicity gadget has no choice — it must operate at the L1's cadence, inside one L1 slot. There's no "we'll just settle next slot" option on the L1 side.

This is where real-time proving enters. For a *cryptographic* atomicity gadget to operate at L1 cadence, the L2's state transition has to be provable inside a single L1 slot. Without that, the gadget either waits (breaking synchrony) or relies on non-cryptographic safety (TEEs, cryptoeconomics, optimism windows).

### Sync vs async recap

With that settlement-cadence lens, the sync vs async distinction is cleaner:

- **Asynchronous** — what AggLayer does out of the box. State roots can be several slots old; the gadget settles on whatever cadence works. Doesn't require a shared sequencer.
- **Synchronous (L2-to-L2)** — requires atomic inclusion coordination (shared sequencer or equivalent coordination), but does **not** require real-time proving.
- **Synchronous (L1-to-L2)** — the tight constraint. Atomicity gadget operates at L1 cadence, which forces the real-time proving requirement for cryptographic shapes. Covered in [Achieving Synchrony](/website/education/composability/achieving-synchrony).

## Real-time proving

Cryptographic atomicity at settlement time requires two things to prove: the **rollup's state transition function** (to establish the new L2 state is valid) and any **supplementary proofs for the atomicity gadget itself** (e.g., the cross-chain commitments reconcile). Both must land within the cadence of the fastest settling chain in the composition — for L1-to-L2, that means within a single L1 slot (~12s).

### The state of real-time proving

There's a spectrum of trust for how you produce these proofs:

- **Single TEE** — a TEE attests to the correctness of the rollup's state transition. Fast, but you're trusting the TEE vendor and the enclave implementation.
- **Committee of TEEs** — multiple TEEs from different vendors attest; you're only compromised if a threshold collude or are broken simultaneously. Stronger than one TEE, still not trustless.
- **ZK proof** — fully cryptographic, no external trust assumption. The hardest to achieve because the whole L2 state transition function has to prove in under one L1 slot.

Pessimistic proofs (as used by AggLayer) can be constructed over any of these — they protect against soundness errors in the underlying proof system, so a TEE-backed pessimistic proof gives you AggLayer-style safety with TEE trust assumptions, and a ZK-backed pessimistic proof gives you the same with no external trust.

The ZK path was considered out of reach for synchronous composability until recently, but [synchronous composability via realtime proving](https://ethresear.ch/t/synchronous-composability-between-rollups-via-realtime-proving/23998) demonstrates it's viable today — sub-slot ZK proving of full rollup state transitions is now a shipping capability, not a research aspiration. You can learn more about the broader proving landscape at [ethproofs.org](https://ethproofs.org/).

---

Now for what we've all been waiting for — how do we achieve *synchronous* composability with the L1?

<span class="fs-8">
[> Step 4 - Achieving Synchrony](/website/education/composability/achieving-synchrony){: .btn }
</span>
