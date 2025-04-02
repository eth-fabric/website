---
title: Step 3 - Composability
nav_order: 4.3
layout: default
parent: Composability 101
permalink: /education/composability/atomic-composability
---
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---
So far by combining shared sequencing, shared bridging, and pessimistic proofs we’ve been able to achieve synchronous atomic execution. In the previous example, $$R_A$$ and $$R_B$$'s settlement was contingent on each other’s post-state root which provided the safety guarantees; however, the contracts involved with $$TX_{burn_{R_A}}$$ and $$TX_{mint_{R_B}}$$ didn’t necessarily know of each others’ existence. While satisfactory for cross-chain swaps, this is insufficient to cover all use cases (e.g., $$TX_{R_B}$$ depends on an oracle update in contract $$C_{R_A}$$). To really make the developer experience feel like one chain, we want *composability.* We’ll see now that we have all the ingredients to achieve this, but first as a recap:

> Synchronous composability requires the ability for state machine of each chain to access the other chain’s state (e.g., the bridge contract itself on $$R_B$$ needs to know the state of $$R_A$$) at the time of execution. [- Jon Charbonneau](https://dba.xyz/were-all-building-the-same-thing/#cross-chain-unbundling)
> 

Mathematically we can say that the new state of $$R_B$$ depends on the state of $$R_A$$: $$state_B = (STF_B(STF_A(X)))$$ - readers are encouraged to watch [Ellie’s talk](https://x.com/Agglayer/status/1870113677682389129) on this.

### Composability

The previous example shows how the AggLayer approach allows for cryptographically-safe atomic execution, allowing us to shed the trust and/or capital requirements of other approaches.

When moving from atomic execution to composability it helps to think of it as the optimistic versus pessimistic case:

- **optimistic case:** “my rollup’s state is correct because their rollup’s state is correct.” Even if contract $$C_{R_B}$$ doesn't know the state of $$R_A$$ during execution, it can safely settle if $$R_B$$ is contingent on the state of $$R_A$$. Essentially $$C_{R_B}$$ optimistically executes, unaware of the state of $$R_A$$ but cryptographic safety prevents anything bad from happening.
- **pessimistic case:** “my rollup’s state is correct because I used their rollup’s state when calculating it.” Here, $$C_{R_B}$$ can only execute where $$R_A$$'s state is made available to $$R_B$$ during runtime. 

In the first case, cross-chain *things* can happen but it's up to the sequencer to make sure the *right* things happen since the contracts are not aware of each other. In the second case, contracts are directly accessing the state of other rollups, so they have more control over *things* that happen. 

Graduating from atomic execution to composability only requires minor changes. In the atomic execution case, the contingent rollup's state is injected at settlement time (see [step four](/website/education/composability/atomic-execution#safe-asynchronous-atomic-execution)). For composability, the contingent rollup's state is injected earlier when computing the state.

Specifically for $$R_B$$ to compose with $$R_A$$, the validity proof $$\pi_B$$ will depend on the state root of $$R_A$$ as an input. When executing $$TX_{mint_{R_B}}$$, contract $$C_{R_B}$$ can first require that $$TX_{burn_{R_A}}$$ successfully burned tokens on $$C_{R_A}$$ by verifying a storage proof against the state root of $$R_A$$, which is made available to $$R_B$$'s runtime (i.e., via a precompile). The logic in $$C_{R_B}$$ enforces a safe mint, assuming the state of $$R_A$$ is valid, which the pessimistic proof guarantees.



### Sync vs async atomic composability

According to our definitions, synchrony would imply that the composing happens at the same slot height. So the real difference between synchronous and asynchronous atomic composability comes down to whether or not composing can happen in one slot (synchronous case) or over multiple (asynchronous case). 

The **asynchronous** case is more straightforward - it’s what’s achievable out-of-the box* with AggLayer. The state root of $$R_A$$ that $$R_B$$ consumes could be several slots old, but still allows $$C_{R_B}$$ to access the state of $$C_{R_A}$$ as described in the previous section:

- it doesn't require a shared sequencer
- the state roots could be proven over long time horizons

The **synchronous** case is more nuanced because of the timing constraint:

- it requires a shared sequencer to guarantee atomic inclusion (i.e., the rollups share a blob transaction)
- it requires [real-time proving](/website/education/composability/atomic-composability#real-time-proving) to ensure the rollups can settle together in one L1 slot

Let’s now revisit our example of a user submitting an atomic bundle with $$TX_{burn_{R_A}}$$ and $$TX_{mint_{R_B}}$$ to a shared sequencer that sequences both $$R_A$$ and $$R_B$$.

1. shared sequencer commits the shared rollup history to the L1 via shared blob transaction
2. $$R_A$$ and $$R_B$$'s derivation function guarantees the transactions cannot be unbundled (as described [before](/website/education/composability/atomic-inclusion#preventing-unbundling))
3. $$R_A$$ executes $$TX_{burn_{R_A}}$$ to burn the user’s 100 USDC, resulting in $$State_{R_A}$$
4. $$R_B$$ performs a cross-chain call using $$State_{R_A}$$ to verify the state of $$C_{R_A}$$. After verifying the burn was successful, $$TX_{mint_{R_B}}$$ mints the user 100 USDC, resulting in $$state_{R_B}$$
5. $$R_A$$ and $$R_B$$ commit block headers to AggLayer that commitments each other’s state roots ($$state_{R_A}$$ and $$state_{R_B}$$)
6. $$R_A$$ and $$R_B$$ submit $$\pi_A$$ and $$\pi_B$$  to AggLayer. Note that $$\pi_B$$ depends on $$State_{R_A}$$ as input
7. AggLayer generates and submits the pessimistic proof to the shared bridge

Step 4 demonstrates $$R_B$$ composing with $$R_A$$ to perform a cross-chain read. From the pov of the $$C_{R_B}$$ developer, this interaction functioned as if $$C_{R_A}$$ lived on the same chain! 

\* rollups still need a precompile to make cross-chain calls

### Real-time proving

The biggest blocker for synchronous composability is proving latency. Crucially, for the previous example to execute within a single slot, it requires steps 6 and 7 to be proven in less than 12 seconds. Compared with asynchronous composability where it could take on the order of minutes to hours for your cross-chain transaction to settle, achieving instant settlement is no easy feat. 

Interestingly, real-time proving allows for composability without AggLayer-like solutions; however, AggLayer is still extremely useful for safety (i.e., pessimistic proofs can protect you from soundness errors in individual chains' proof systems) and it abstracts away the complexities of composability from rollup developers.

**The state of real-time proving**

There are various efforts pushing the boundaries of ZK proving latency (can learn more [here](https://ethproofs.org/)).

Real-time proving is not yet production ready so various teams are exploring shorter term solutions such as using multiple TEEs to attest to the correctness of a rollup’s state transition function, allowing real-time synchronous composability today.

In the same line of thinking it is possible to create pessimistic proofs using TEEs as well to achieve the same safety guarantees as AggLayer, albeit relying on the trust assumptions of TEEs rather than ZKPs.

---

Now for what we've all been waiting for - how does this relate to based rollups?

<span class="fs-8">
[> Step 4 - Based-Rollups](/website/education/composability/composability-and-based){: .btn }
</span>