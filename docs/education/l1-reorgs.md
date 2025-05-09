---
title: L1 Reorgs
nav_order: 5
layout: default
parent: Education
permalink: /education/l1-reorgs
---
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

# Spectrum of L1 Reorg Tolerance

### What are Reorgs

A reorg (short for *reorganization*) is when the current view of a blockchain’s state is replaced by an alternative version. On Ethereum, reorgs typically affect just 1–2 blocks and are most often caused by block propagation delays or proposer timing games. Readers are referred [here](https://www.paradigm.xyz/2021/07/ethereum-reorgs-after-the-merge) for more info.

For example:

1. `Block 100` is proposed by Proposer `A` and includes your transaction.
2. Some validators receive and attest to it, but because it propagates slowly, it doesn’t gather enough attestations.
3. Meanwhile, Proposer `B` publishes `Block 101` that receives enough attestations.
4. The network eventually drops `Block 100` resulting in `Block 99` → `Block 101`
5. Execution clients reprocess the chain starting from `Block 99`, omitting the transactions from `Block 100`.

### How Do Reorgs Affect Rollups?

Rollups definitionally derive their state from the L1, particularly from two sources:

- **L2 transaction data**: user-submitted transactions sequenced into batches or blobs and posted to the L1
- **L1 inputs**: deposits, bridge messages, forced inclusions, etc posted to L1

Because of this dependency, L1 reorgs can force rollups to re-execute affected L2 blocks. If the L1 data changes, e.g., a deposit is removed due to a reorg, then any L2 state that relied on it must be rolled back or reprocessed. This can lead to:

- Dropped or reverted transactions
- Entire L2 blocks producing different outcomes upon re-execution

**Example**: A user deposits ETH into the rollup. The rollup observes the deposit in L1 `Block 100` and mints the equivalent ETH on L2. If `Block 100` is reorged out, the rollup must handle the L1 deposit no longer existing.

Since this harms UX, rollups may choose to wait for L1 data to be finalized (waiting ~12 mins) before consuming it to guarantee reorgs won’t effect them.

### Case Study – OP Stack

The OP Stack takes a pragmatic middle-ground approach to L1 reorg tolerance. The sequencer doesn’t require full finality before ingesting L1 data, but it also doesn’t follow the L1 head in real time. Instead, it introduces a `SequencerConfDepth` buffer set to `4` blocks.

This means the sequencer lags `4` blocks behind the L1 head when choosing which L1 block to use as the origin for building L2 blocks. By doing so, it reduces the chance that a referenced L1 block will be reorged out.

In the OP Stack, “**unsafe**” transactions are those confirmed by the Sequencer but not yet published to the DA layer. Once published, they are promoted to “safe”, and the rollup’s state can be derived solely from these safe transactions. 

In the event of an L1 reorg, to quote the OP Stack:

> If Ethereum experiences a reorg, OP Stack chains will attempt a graceful recovery. OP Stack nodes will downgrade "safe" transactions to "unsafe" if needed, while the Sequencer republishes transaction data to maintain chain continuity. [src](https://docs.optimism.io/stack/transactions/transaction-finality#ethereum-reorgs-cause-op-stack-chain-reorgs)
> 

To break this down:

- Assume the Sequencer publishes their blob `B` during `Block 100`
- Due to the `SequencerConfDepth` of `4`, blob `B` only processes L1 inputs up to `Block 96`
- If `Block 100` reorgs out, the Sequencer can repost blob `B` in a later block without issue
- However, if the reorg extends back to `Block 96`, then blob `B` references L1 inputs that no longer exist
- Regardless of when `B` is posted, the resulting rollup state will diverge from the original execution

Because reorgs deeper than 4 blocks are rare, this approach significantly reduces the chance of L1 reorgs affecting rollup state. But when it does happen, the OP Stack must recover by making a hard choice:

- Accept the new state
- Rollback the rollup’s state

### L1 Synchronous Composability

L1 Synchronous composability refers to the ability for a rollup to *immediately* read and act on the latest L1 state and vice-versa within the same atomic operation. This is in contrast to asynchronous models, where L1-originated data (like deposits or messages) is delayed by several blocks or minutes before being reflected on L2.

The goal of L1 synchronous composability is to:

- Enable real-time cross-domain interactions (e.g., L1<>L2 swaps or contract calls)
- Improve DevEx by simplifying cross-chain interactions
- Heal liquidity and user fragmentation by making the L1 and rollups feel like a single, unified chain

To enable synchronous access to L1 data, a rollup must track and reorg with the L1 head *in real time*. That involves:

- Reading L1 state before it is finalized or even confirmed
- Accepting that this L1 state may be reorged, forcing the rollup to adapt

In effect, L1 synchronous composability requires the rollup to disable or bypass mechanisms like `SequencerConfDepth`, which exist specifically to delay L1 ingestion until the risk of reorgs is low. Without that buffer, the rollup assumes the risk of L1 reorgs, potentially invalidating previously “safe” L2 state.

**Example:**

Suppose a user wants to atomically swap L1 ETH for L2 USDC in a single L1 slot. With synchronous composability:

1. The user submits an L1 ETH deposit transaction and an L2 swap transaction
2. The L1 deposit is processed by the L1 builder
3. The L2 sequencer *optimistically* injects the corresponding L2 deposit transaction
4. The L2 sequencer proceeds with executing the L2 swap transaction

However, if the L1 block is reorged out:

- The L1 deposit is gone.
- The L2 deposit no longer exists.
- The L2 swap reverts because the L2 user lacks the ETH.
- Even if they republish the atomic transaction, the L1 deposit could now be invalid so there is no guarantee to arrive back at the expected state.
- Therefore, the rollup must either rollback to earlier or accept the new state that includes the revert.

### Based Rollups

Based rollups are sequenced by L1 proposers. This creates a tighter coupling with Ethereum than classical rollups, but it does **not** mean all based rollups must reorg with the L1 - there’s a spectrum of design choices depending on how sequencing and reorg handling are implemented.

**Vanilla Based Rollups**

The original [based rollup proposal](https://ethresear.ch/t/based-rollups-superpowers-from-l1-sequencing/15016), as implemented by projects like Taiko, follows a “total anarchy” model: anyone can sequence the rollup by publishing blobs to L1. In this design, the rollup must reorg with the L1. If a blob is removed due to an L1 reorg, nodes must revert to the last valid state reflected on L1.

Because there is no designated sequencer, there is no guarantee that a reorged-out blob will be re-published. This contrasts with classical rollups, where a dedicated sequencer can choose between:

- Rolling back the state, or
- Republishing the blob with the same data

In vanilla based rollups, rollback is the only viable path.

**Based Rollups with Preconfs**

[Based preconfirmations](https://ethresear.ch/t/based-preconfirmations/17353) allow scheduled L1 proposers to commit to sequencing transactions ahead of L1 publication, enabling UX similar to classical rollups. Conceptually, a based rollup with preconfs is like a classical rollup, just with a frequently rotating sequencer chosen by the L1.

With this mental model, you can apply the same reasoning regarding L1 reorgs. 

**Option 1 - Reorg in Real-Time**

The rollup ingests L1 inputs immediately, unlocking synchronous composability. If the L1 reorgs, the rollup has two recovery paths:

- **Rollback:** Revert to the previous L2 safe head. This ensures safety but weakens the value of preconfs and harms UX.
- **Republish:** Require the next preconfer to republish the previous blob. This is a best-effort attempt at preserving preconf continuity; however, as L1 inputs have likely changed or are stale, re-execution may yield different L2 state.

**Option 2: Maintain a** `SequencerConfDepth` **Buffer**

The rollup delays ingesting L1 inputs until they reach a configurable depth (e.g., 4 blocks), similar to OP Stack. This approach minimizes L1 reorg risk but sacrifices one of the core benefits of based rollups: **L1 synchronous composability**.

### Spectrum of Reorg Tolerance

Rollups exist on a spectrum of how closely they track and respond to Ethereum’s L1 chain. At one extreme, rollups maximally decouple from the L1 to maximize UX stability; at the other, they tightly couple to enable synchronous L1↔L2 composability, accepting L1 reorg risk as a tradeoff.

This spectrum reflects a core tension:

> More L1 coupling = better composability, higher reorg risk
> 

> More L1 decoupling = stronger L2 UX guarantees, reduced interop
> 

| **Rollup Type** | **L1 Coupling** | **Reorg Tolerance** | **Composability** | **Example** |
| --- | --- | --- | --- | --- |
| **Classical** | Low | Avoids L1 reorgs | Async only | Waits for L1 finality before processing deposits |
| **OP Stack** | Medium | Buffered (e.g., 4 blocks) | Limited sync | Uses `SequencerConfDepth` to reduce L1 reorg risk |
| **Based w/ Preconfs** | Medium–High | Optional | Sync possible | Reorgs if needed, but can try to preserve UX |
| **Vanilla Based** | High | Required | Sync default | Fully reorgs with L1, no fallback logic |

The middle of the spectrum is under-explored but offers flexibility. For instance, an OP Stack rollup could **dynamically lower `SequencerConfDepth` to `0`** when synchronous access to L1 is needed, and raise it back to `4` for normal operation. If the reorg risk is clearly communicated, this model can strike a practical balance between safety and interoperability.

The key takeaway is **reorg tolerance is not binary.** Rollups can dynamically choose how tightly to track the L1 based on application needs and risk appetite. 

### Mitigations to Reorgs

While reorg risk is a real consideration for any rollup thinking about interoperability with the L1, there are ongoing developments that help to **reduce the frequency** reorgs in the future. 

Most L1 reorgs occur when proposers maximally delay publishing blocks to play MEV timing games. In a world where L1 proposers also act as preconfers, this behavior becomes less attractive if:

- Preconf payments exceed the marginal MEV gain from timing games
- The opportunity cost of a block being reorged out and the slashing penalty of failing to deliver preconfs are incentives for timely block inclusion
- When the preconfs are for slot futures, incentives are especially aligned. If a proposer sells a future block via preconf, the buyer (e.g., a rollup sequencer) and preconfer are incentivized to make sure the block lands on time.

Proposals like **Attester Proposer Separation (APS)** allow for highly specialized, high-bandwidth L1 proposers without compromising the decentralization of Ethereum’s attester set.

By design, these proposers operate sophisticated infrastructure with faster networking and better availability, which reduces reorgs caused by propagation delays and bandwidth bottlenecks. As APS matures, the Ethereum network may become more stable, reducing the risk for rollups to couple with it.