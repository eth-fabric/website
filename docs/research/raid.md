---
title: Raid
nav_order: 5.2
layout: default
parent: Research
permalink: /research/raid
---
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

![Raid](/website/assets/images/raid.png)

{: .important-title }
> Important
>
> This is the latest iteration of research based on recent developments and conversations. For more context readers are encouraged to read original [Dido post](/website/research/Dido).

# TL;DR
- inbox contracts check the URC at blob submission time, ensuring preconditions are satisfied
- blob proposers need to submit a beacon state proof that the blob they're building on came from the right proposer 
- at least one slot after the blob submission, L2 nodes can verify if the proof via EIP-4788
- each subsequent proposer chains these beacon proofs so L2 nodes can determine the canonical chain
- this approach removes the need for an on-chain lookahead or to calculate it during L2 derivation
- inboxes are less "dumb" in this design Dido $$\rightarrow$$ Raid which stands for "Retroactive Attestation of Inbox Data"

# Context
During [Fabric Call #1](https://youtu.be/UngTQPjy4UA?si=Y8puLV91Bjg1Iko6&t=2214), [Lin Oshitani](https://x.com/linoscope) raised an important issue where the upcoming MaxEB ([EIP-7251](https://eips.ethereum.org/EIPS/eip-7251)) upgrade in Pectra exacerbates an existing problem that makes the beacon chain lookahead window non-deterministic between epochs. This issue motivated [new designs](https://youtu.be/UngTQPjy4UA?si=g_0Zy4fU-eC3kmhh&t=3094) as well as [EIP-7917](https://ethereum-magicians.org/t/eip-7917-deterministic-proposer-lookahead/23259) to make the lookahead deterministic, while also making it easier to prove on-chain. By combinining these with the Dido approach, we can arrive at a vastly simpler design which we call *Raid*, but before that some context.

## Lookahead non-determinism

### Aside - EIP-4788

For those unfamiliar, [EIP-4788](https://eips.ethereum.org/EIPS/eip-4788) exposes the beacon block root to the execution layer. This allows smart contracts to access up to the last 8091 beacon block roots and use them to prove things about the consensus layer (e.g., validator 10 was the proposer and had 31.99 ETH at this slot). 

![Beacon Block Header](/website/assets/images/beacon-block-header.png)
Pictured above is the `BeaconBlockHeader` schema and the `beacon_block_root` is the Merkle root obtained by hashing the fields of the header as shown. Since the root is accessible on-chain, you can make claims about the state of the beacon chain and verify via smart contract.

To prove who the proposer was at a specific slot you can supply a contract with the `slot`, `proposer_index`, `state_root`, and `body_root`. The contract's logic would Merklize everything and compare the result with the `beacon_block_root` fetched via EIP-4788. Note we don't need to supply the `parent_root` as it is simply the `beacon_block_root` from the previous slot and can also be fetched via EIP-4788. To make more complicated claims about the beacon chain's state, you can similarly prove against the `state_root` or `body_root`, and then connect it back to the `beacon_block_root` as just described.

A big limitation of EIP-4788 is that it only reports the `beacon_block_root` of the *last* slot. This means that while a contract is being executed, it has no way of knowing the current `beacon_block_root` and therefore cannot determine who the current proposer is. 

In the context of based sequencing, knowing who the current proposer is allows inboxes to filter for registered preconfers. This limitation of EIP-4788 is what motivated the need to use the beacon lookahead in the first place.

### How is the lookahead calculated?
Unfortunately, the lookahead is not stored in the beacon chain's state. If it was, then EIP-4788 could be used to prove the lookahead to a contract which inboxes could consume. Instead, consensus clients calculate the lookahead from the beacon state. 

The [`get_beacon_proposer_index`](https://github.com/ethereum/consensus-specs/blob/dev/specs/phase0/beacon-chain.md#get_beacon_proposer_index) function determines who the proposer should be at a slot given the beacon chain's state.

```python
def get_beacon_proposer_index(state: BeaconState) -> ValidatorIndex:
    """
    Return the beacon proposer index at the current slot.
    """
    epoch = get_current_epoch(state)
    seed = hash(get_seed(state, epoch, DOMAIN_BEACON_PROPOSER) + uint_to_bytes(state.slot))
    indices = get_active_validator_indices(state, epoch)
    return compute_proposer_index(state, indices, seed)
```

It calls [`compute_proposer_index`](https://github.com/ethereum/consensus-specs/blob/dev/specs/electra/beacon-chain.md#modified-compute_proposer_index) which depends on the validator's effective balance:

```python
def compute_proposer_index(state: BeaconState, indices: Sequence[ValidatorIndex], seed: Bytes32) -> ValidatorIndex:
    """
    Return from ``indices`` a random index sampled by effective balance.
    """
    assert len(indices) > 0
    MAX_RANDOM_VALUE = 2**16 - 1  # [Modified in Electra]
    i = uint64(0)
    total = uint64(len(indices))
    while True:
        candidate_index = indices[compute_shuffled_index(i % total, total, seed)]
        # [Modified in Electra]
        random_bytes = hash(seed + uint_to_bytes(i // 16))
        offset = i % 16 * 2
        random_value = bytes_to_uint64(random_bytes[offset:offset + 2])
        effective_balance = state.validators[candidate_index].effective_balance
        # [Modified in Electra:EIP7251]
        if effective_balance * MAX_RANDOM_VALUE >= MAX_EFFECTIVE_BALANCE_ELECTRA * random_value:
            return candidate_index
        i += 1
```

When  the consensus client calls [`process_block_header`](https://github.com/ethereum/consensus-specs/blob/dev/specs/phase0/beacon-chain.md#block-header) to verify a block, it verifies that the proposer was correct by calling `get_beacon_proposer_index`.


```python
def process_block_header(state: BeaconState, block: BeaconBlock) -> None:
    # Verify that the slots match
    assert block.slot == state.slot
    # Verify that the block is newer than latest block header
    assert block.slot > state.latest_block_header.slot
    # Verify that proposer index is the correct index
    assert block.proposer_index == get_beacon_proposer_index(state)
    # Verify that the parent matches
    assert block.parent_root == hash_tree_root(state.latest_block_header)
    # Cache current block as the new latest block
    state.latest_block_header = BeaconBlockHeader(
        slot=block.slot,
        proposer_index=block.proposer_index,
        parent_root=block.parent_root,
        state_root=Bytes32(),  # Overwritten in the next process_slot call
        body_root=hash_tree_root(block.body),
    )

    # Verify proposer is not slashed
    proposer = state.validators[block.proposer_index]
    assert not proposer.slashed
```

Importantly, while `get_beacon_proposer_index` is a deterministic function `compute_proposer_index` depends on the effective balances of the validators to select the proposer. At the end of each epoch, the [`process_epoch`](https://github.com/ethereum/consensus-specs/blob/dev/specs/electra/beacon-chain.md#epoch-processing) function is called which modifies the effective balances via [`process_effective_balance_updates`](https://github.com/ethereum/consensus-specs/blob/dev/specs/electra/beacon-chain.md#modified-process_effective_balance_updates).

```python
def process_epoch(state: BeaconState) -> None:
    process_justification_and_finalization(state)
    process_inactivity_updates(state)
    process_rewards_and_penalties(state)
    process_registry_updates(state)  # [Modified in Electra:EIP7251]
    process_slashings(state)  # [Modified in Electra:EIP7251]
    process_eth1_data_reset(state)
    process_pending_deposits(state)  # [New in Electra:EIP7251]
    process_pending_consolidations(state)  # [New in Electra:EIP7251]
    process_effective_balance_updates(state)  # [Modified in Electra:EIP7251]
    process_slashings_reset(state)
    process_randao_mixes_reset(state)
    process_historical_summaries_update(state)
    process_participation_flag_updates(state)
    process_sync_committee_updates(state)
```

A consensus client can sample the lookahead by repeatedly calling `get_beacon_proposer_index` with an increasing slot number. Since the effective balances do not change within an epoch, this is sufficient to guarantee a correct calculation of the *current* epoch's lookahead, even if you do not have the actual `BeaconState` for upcoming slots. This is in part because the [`get_seed`](https://github.com/ethereum/annotated-spec/blob/master/phase0/beacon-chain.md#get_seed) function returns a fixed value for the epoch.

```python
def get_seed(state: BeaconState, epoch: Epoch, domain_type: DomainType) -> Bytes32:
    """
    Return the seed at ``epoch``.
    """
    mix = get_randao_mix(state, Epoch(epoch + EPOCHS_PER_HISTORICAL_VECTOR - MIN_SEED_LOOKAHEAD - 1))  # Avoid underflow
    return hash(domain_type + uint_to_bytes(epoch) + mix)
```

### The epoch boundary
While the lookahead is fixed intra-epoch, non-determinism arrises when attempting to calculate the lookahead for the next epoch before the epoch boundary. Since `process_epoch` updates the effective balances at the epoch boundary, it is possible that a pre-computed lookahead ends up incorrect if a validator's effective balance changed. 

EIP-7251 introduces more opportunities for effective balances to change, i.e., if validators consolidate, which can increase the probability that the lookahead changes at the epoch boundary.

### Why this matters for preconf protocols
Based rollups use preconfs to improve their UX. L2 users can send transactions directly to preconfers instead of waiting for them to get picked up from the mempool by a proposer (see [here](/website/education/composability/composability-and-based#sequencer-selection) for more context).

Within an epoch it's possible to determistically calculate the lookahead so we're fine. Based rollups and wallets can use this to select preconfers and direct user transactions to them. However, between epochs, it's *impossible* to definitely know ahead-of-time what the lookahead will look like - there's always some possibility that the lookahead changes because of an effective balance change at the end of an epoch.

![Lookahead Instability](/website/assets/images/lookahead-instability.png)

Lin [summarizes the problem](https://hackmd.io/@linoscope/eip-7917-from-preconf-protocol) neatly in this image. Once slot 29 elapses, there are no more preconfers left in the epoch. The rollup is faced with a choice:
1. optimistically calculate the next epoch's lookahead and assign preconfers
2. disable preconfs until the next epoch starts

Option 1 is problematic if the lookahead changes, since the issued preconfs can no longer be guaranteed if the *assumed* proposer does not become the *actual* proposer. Option 2 is a bad UX - imagine if the last preconfer were much earlier in the epoch e.g., slot 8. The based rollups degrades back to [*total anarchy mode*](/website/education/composability/composability-and-based#total-anarchy) for 75% of the epoch!

## EIP-7917
Clearly, cross-epoch lookhead non-determinism is an issue for preconf protocols, so what can we do about it? Lin Oshitani and Justin Drake collaborated to define [EIP-7917](https://eips.ethereum.org/EIPS/eip-7917) which allows for the lookahead to be deterministically calculated across epoch-boundaries. 

The EIP requires extremely few changes to the spec (just ~20 lines of code). The solution introduces the lookahead as a field within the beacon state which conveniently addresses our previous problem - with this EIP, a contract can *prove the lookahead using EIP-4788*. Better yet, they can prove the lookahead for the current or next epoch given any beacon block root in the current epoch (it doesn't need to be just-in-time). 

The updated `process_epoch` function now calculates the lookahead for the upcoming *and subsequent* epoch meaning we always know the lookahead for the next epoch and our non-determinism is solved! 

The EIP is currently under review for inclusion in the Fulu upgrade, and if included will add stability to the network and significantly help the viability of preconf protocols.

## How can the lookahead be proven before EIP-7917?
Once EIP-7917 is in production, contracts will be able to easily prove the lookahead using EIP-4788. Until that time, alternative approaches need to be considered for rollup inbox contracts to be aware of who the proposer is.

### Pessimistic lookahead contract
Since the lookahead is *not* part of the `BeaconState`, we need to prove we executed the `get_beacon_proposer_index` function correctly. Since implementing the `get_beacon_proposer_index` logic in a contract is infeasible gas-wise, the alternative approach is to prove the execution via ZKP and supply a contract with the proof. The inbox contract can verify it and accept lookahead, then the contract can filter blobs for the correct proposer at submission time.

**Pros**

- Trustless 
- Straightforward

**Cons**

- Implementing consensus client the logic in a ZK circuit is non-trvial i.e., it's a smaller scope of what Beam chain is doing
- Proving increases latency and increases hardware requirements for proposers
- The cost to verify the proof on-chain will significantly increase the rollup's operating costs

---

### Optimistic lookahead contract 
The [Nethermind / Taiko’s PoC](https://github.com/NethermindEth/Taiko-Preconf-AVS/tree/master/SmartContracts/src/avs) pioneered this approach. Here, the first preconfer in the epoch will post the view of the entire lookahead to the inbox contract. The view is purely optimistic as no complex proving is done. If anyone detects fraud, they can prove the correct proposer after the slot has elapsed via EIP-4788 to slash them. 

**Pros**

- Optimistic is cheaper than proving
- Trustless because of fraud proof mechanism

**Cons**

- Need to incentivise the proposer to post the lookahead
- Need a reconciliation process should there be fraud

---

### L2 derivation
As proposed in [Dido](/website/research/Dido), we can move the logic of calculating the lookahead into the L2 node. During derivation, the L2 node can compute the lookahead by running `get_beacon_proposer_index` and use this to filter for whether a blob should be executed or skipped. 

**Pros**

- No on-chain logic or costs
- Trustless

**Cons**

- Adds complexity to derivation logic
- Requires the L2 node to know the full `BeaconState`

---

### Branching inbox contract
Originally suggested by [Swapnil](https://x.com/swp0x0) from the Nethermind team, the Branching Inbox Contract (BIC) approach stood out for its elegance and simplicity and is what we'll focus on for Raid. 

{: .important-title }
> important
>
> The Branching Inbox Contract approach removes the need for a lookahead!



BIC works similarly to Dido, except with even less assumptions. When a blob is submitted to the an inbox, the submitter omits any kind of proof pertaining to the lookahead and the inbox accepts the blob *without reverting*. Multiple proposers can queue up their blob transactions at this inbox but the canonical state is not immediately resolved, effectively creating branches of different possible L2 states (hence the name branching inbox contracts). Importantly, the inbox contract cannot determine who the proposer was while processing the blob transaction because the current `beacon_block_root` is still unknown at execution time.

From the POV of an L2 node, they can verify who the proposer was using EIP-4788 as soon as the blob transaction is published to the L1, because the EVM will now have access to the `beacon_block_root`. Therefore you can think of BIC as "anyone can post blobs but we'll figure out if they were allowed to later."

It should be noted that it is still possible for L2 nodes to determine who the proposer is ahead-of-time without waiting for the next slot by using the lookahead (i.e., by running a beacon node in parallel), BIC simply removes this logic from the L2's derivation pipeline.

**Pros**

- **Reduces on-chain complexity**: There’s no need for the inbox to have an on-chain view of the lookahead (optimistic / pessimistic lookahead approaches)
- **Reduces L2 derivation complexity**: There’s no need to execute the beacon node lookahead logic within L2 derivation

**Cons**

- **Incompatible with realtime settlement:** It isn’t possible for the rollup to settle in one block to compose with the L1 synchronously as the `beacon_block_root` is unknown at blob-posting time.

Clearly the pros of BIC are extremely compelling to simplify the ideas from Dido but being incompatible with realtime settlement negates the biggest promises of based sequencing which is universal synchronous composability.

# Proposed V2 design
Raid incorporates the ideas of BIC and Dido with a focus on pragmatism to achieve based sequencing today. Despite not being compatible with real-time settlement, we are yet to see real-time proving in production, so aiming for BIC in the short term is a way to significantly reduce based rollup complexity before the tradeoff becomes apparent. Once EIP-7917 is live, inboxes can switch to proving the lookahead on-chain via EIP-4788 to be compatible with real-time settlement. 

## High Level Overview
Raid assumes two L1 contracts, the `Inbox` and `PublicationFeed`. We refer readers to OpenZeppelin's implementation of their [`PublicationFeed`](https://github.com/OpenZeppelin/minimal-rollup/blob/main/src/protocol/PublicationFeed.sol) contract as we believe this approach is aligned with the design goals of minimizing the L1 footprint. 

When a `blobProposer` submits a blob transaction to build off of the *unverified* head of the chain, they must include all necessary proof data for L2 nodes to verify the validity of the *previous* `blobProposer`. They are incentivized to provide valid data, as L2 nodes will ignore their submission if the proof is invalid, wasting their slot.

![V2](/website/assets/images/branching-inbox.png)

- Anyone can permissionlessly submit $$TX_{blob}$$ to the `Inbox` contract.
- $$TX_{blob}$$ must include as input a reference to a previous publication $$pub_{prev}$$ and two proofs $$\pi_{beacon}$$ and $$\pi_{urc}$$.
    - $$\pi_{urc}$$ is used to prove statements against the URC such as "the proposer with BLS key `0xabcd...` was opted into preconf protocol `0x1234...` with at least 1 Ether collateral." These are simple Merkle proofs to prove that the validator key was registered before (since URC does not store them on-chain). The `Inbox` will save a commitment of the validator's public key.
    - $$\pi_{beacon}$$ contains an EIP-4788 Merkle proof pertaining to the previous `blobProposer's` blob that they're building off of, proving the proposer's public key for the slot (see [Tony's ethresearch post on how](https://ethresear.ch/t/slashing-proofoor-on-chain-slashed-validator-proofs/19421)). The `Inbox` contract does not process these proofs on-chain, rather they are used by L2 nodes later.
- `Inbox` publishes to the `PublicationFeed`, which stores a hash that is used in the future for proving (this becomes a future `blobProposer's` $$pub_{prev}$$).
- Upon receiving the publication event, an L2 node fetches the blob data and decodes $$\pi_{beacon}$$. 
- The L2 node uses EIP-4788 to fetch the `beacon_block_root` for the given slot and uses it to verify $$\pi_{beacon}$$.
- If the proposer's public key matches what was committed by the `Inbox`, then the L2 node is assured that:
    - $$TX_{blob}$$ was submitted during a slot where the proposer was opted into the URC
    - the `blobProposer` address was delegated to by the proposer (necessary since BLS keys cannot sign transactions!)
    - the proposer satisfied all preconditions, e.g., they had enough collateral in the URC 
    - the L2 node can continue by executing the transactions in the blob to derive the L2’s new state
- Else the blob is treated as a no-op and ignored.

### Assumptions
- If multiple valid $$TX_{blob}$$ transactions are submitted in a single slot, L2 nodes should only consider the first. This helps reduce branching paths and is reasonable given a based sequencer has a write-lock on the L1 and can ensure only their $$TX_{blob}$$ lands.
- Since EIP-4788 only stores the last 8091 beacon block roots, valid blob publications must happen at least daily to ensure consecutive proposals can access the required roots.
- `blobProposers` are incentivized to provide valid proof data, as L2 nodes will ignore their submission if the proof is invalid, wasting their slot.
- When proving L2 state, EIP-4788 is just a [smart contract](https://etherscan.io/address/0x000F3df6D732807Ef1319fB7B8bB8522d0Beac02), so the prover can be supplied with storage proofs.

### Design choices
- While it is possible to also check against the URC during L2 derivation, it adds complexity with little gains - hence the approach has abandoned the idea of completely "dumb inboxes" and changed them name from Dido to Raid.
- Requiring subsequent `blobProposers` to submit $$\pi_{beacon}$$ means that L2 follower nodes are *not* required to run a beacon node. Rather, only the `blobProposer` is which is already assumed for based sequencers. 

### Remaining considerations
Until we have EIP-7917 there is still the problem of lookahead instability between epoch boundaries due to changes in validators’ expected balances. At the cost of increased complexity, it is possible to add additional derivation logic that assigns a fallback “Epoch Bridge Proposer” [as described by Lin](https://youtu.be/UngTQPjy4UA?si=ODnqHoMS4qjAMh3v&t=2673) to ensure consistent preconf uptime between epoch boundaries.

