---
title: Dido
nav_order: 5.1
layout: default
parent: Research
permalink: /research/dido
---
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---
![DIDO](/website/assets/images/ditto.png)

The working name of this approach is  DIDO, which stands for “Dumb Inboxes, Derivation Only” and like the [pokemon](https://bulbapedia.bulbagarden.net/wiki/Ditto_(Pok%C3%A9mon)) can transform to work with any rollup.


## Context

### The Nethermind/Taiko PoC Design

In [Nethermind / Taiko’s PoC design](https://github.com/NethermindEth/Taiko-Preconf-AVS/tree/master/SmartContracts/src/avs), the first preconfer is required to (optimistically) post a view of the entire lookahead to their smart contracts. Given this on-chain view, their inbox contract can restrict sequencing rights to the registered preconfers. As we started to think about standardization in the context of this approach we started to ask why this approach was taken and its impact on standardization.

### Why take this approach?

This approach allows the right of exclusive write-lock over the L2s state to the next registered preconfer, allowing for L2 execution preconfs in the future.

If an L2 blob is posted to the L1 inbox contract and it doesn’t revert, it is implied that it could have only been posted by a registered preconfer and therefore should be considered canonical. Therefore, deriving the L2 state is a simple matter of replaying the blobs that were successfully posted to the inbox contract.

### How does this differ from other stacks?

In the OP stack, the L1 inbox (aka `batchInbox`) is an arbitrary address - there isn’t a contract to post to, meaning *anyone* can submit a blob there. The way the OP stack determines the canonical blob is during *derivation*. The rollup’s derivation function will process the blob only if the blob tx was signed by the canonical `batchSender` EOA address (e.g., belonging to OP mainnet or Base’s sequencer). 

A benefit of this approach is that it costs less gas to send a blob transaction to an EOA (the `batchInbox`) as opposed to a contract. However, the downside is that having a fixed `batchSender` address makes based sequencing challenging (i.e., to support it you need to leak the `batchSender` private key).

### What can we learn from OP stack’s implementation?

The problem we’re ultimately trying to solve is **blob provenance**: did this blob get submitted by the correct party?

The observation is that the Taiko PoC moves all complexity on-chain which consequently makes derivation simpler. More specifically, filtering for canonical blobs is done at the L1 rather than off-chain during derivation. Since doing anything on-chain introduces costs, it begs the question: what can we push into the derivation layer (where things are free) to minimize what’s done on-chain?

## Proposal

By determining blob provenance validity in the L2 derivation layer, we can achieve the same outcome as the Taiko Poc design - execution preconf compatibility - but more flexibly and cheaply.

### High Level Overview

- The L1 Inbox is set to a random `batchInbox` address that ***isn’t*** a contract (e.g., `0x000...42069)`
- There is no dedicated `batchSubmitter` address in the spec, rather anyone can submit blob txs and we’ll refer to the sender as the `blobProposer`
- The blob is expected to contain `blobProposerCredentials`, which is an opaque byte array
- We create a new L1 contract called `DerivationHelper`
- The derivation layer will use `blobProposerCredentials` in conjunction with historical L1 state and the `DerivationHelper` to determine whether or not the `blobProposer` was allowed to post during that slot (i.e., whether they had sequencing rights)
- If it was a valid `blobProposer`, then the blob’s data will be incorporated to derive the L2’s state, otherwise it is ignored.

### New Derivation Rules

We can define a contract called `DerivationHelper` to help the derivation function verify that the `blobProposer` address was authorized to sequence the rollup. The expectation is that the inputs `blobProposerCredentials` bytes supplied to `blobProposerIsAllowed()` originates from the blob / blob tx.

If the function returns true, then at the `blockNumber`, the proposer was part of the URC, opted in to the `rollupSlasher`, sufficiently collateralized, in the correct position of the lookahead, and had delegated committing rights to the `blobProposer`. The L2 node could then safely process the blob data to derive the L2 state.

Example Solidity contract:


```
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

interface IURC {
    function registrations(bytes32 registrationRoot) external view returns (Operator memory);
    function verifyMerkleProof(
        bytes32 registrationRoot,
        bytes32 leaf,
        bytes32[] calldata proof,
        uint256 leafIndex
    ) external view returns (uint256);
    function isOptedIntoSlasher(bytes32 registrationRoot, address slasher) external view returns (bool);
    function UNREGISTRATION_DELAY() external view returns (uint256);
    function OPT_IN_DELAY() external view returns (uint256);
    function getOptedInCommitter(
        bytes32 registrationRoot,
        IRegistry.Registration calldata reg,
        bytes32[] calldata merkleProof,
        uint256 leafIndex,
        address rollupSlasher
    ) external view returns (SlasherCommitment memory slasherCommitment, uint256 collateralGwei);
}

interface IRegistry {
    struct Registration {
        BLS.G1Point pubkey;
        BLS.G2Point signature;
    }
}

interface ILookahead {
    function getProposerAt(uint64 slot) external view returns (BLS.G1Point memory);
}

/// @notice An operator of BLS key[s]
struct Operator {
    /// The authorized address of the operator
    address owner;
    /// ETH collateral in GWEI
    uint56 collateralGwei;
    /// The number of keys registered per operator (capped at 255)
    uint8 numKeys;
    /// The block number when registration occurred
    uint32 registeredAt;
    /// The block number when deregistration occurred
    uint32 unregisteredAt;
    /// The block number when slashed from breaking a commitment
    uint32 slashedAt;
    /// Mapping to track opt-in and opt-out status for proposer commitment protocols
    mapping(address slasher => SlasherCommitment) slasherCommitments;
    /// Historical collateral records
    CollateralRecord[] collateralHistory;
}

struct SlasherCommitment {
    uint256 optedInAt;
    uint256 optedOutAt;
    address committer;
}

contract DerivationHelper {
    IURC public urc;
    ILookahead public lookahead;
    uint256 public constant REQUIRED_COLLATERAL_GWEI = 1 ether;

    constructor(address urcAddress, address lookaheadAddress) {
        urc = IURC(urcAddress);
        lookahead = ILookahead(lookaheadAddress);
    }

    function blobProposerIsAllowed(bytes calldata blobProposerCredentials) external view returns (bool) {
            (
            bytes32 registrationRoot, // merkle root from proposer's initial registration at URC
                    bytes32[] calldata merkleProof, // proof bls key is part of URC
                    uint256 leafIndex, // index of registration in merkle proof
                    BLS.G2Point regSignature, // proposer's bls signature from URC registration
                    uint64 blockNumber,    // block that blob was submitted at
                    address rollupSlasher, // the slasher contract for rollup
                    address blobProposer   // the sender address of the blob tx
        ) = abi.decode(blobProposerCredentials, (bytes32, bytes32[], uint256, BLS.G2Point, uint64, address, address));

    // Retrieve operator from URC
    Operator memory operator = urc.registrations(registrationRoot);
        
        // Fetch proposer BLS key from the on-chain Lookahead contract
        BLS.G1Point memory pubkey = lookahead.getProposerAt(_blockNumberToSlot(blockNumber));
        
        // Reconstruct the Registration struct
        Registration memory reg = Registration({pubkey: pubkey, signature: regSignature});
        
        // Retrieve slashingCommitment and proposer's collateral from URC
        // Fail if the `pubkey` from lookahead doesn't match what's in the URC
        (SlasherCommitment memory slasherCommitment, uint256 collateralGwei) = urc.getOptedInCommitter(registrationRoot, reg, merkleProof, leafIndex, rollupSlasher);
    
        // Verify collateral amount
        require(collateralGwei >= REQUIRED_COLLATERAL_GWEI, "Insufficient collateral");
    
        // Verify operator has not been slashed
        require(operator.slashedAt == 0, "Operator has been slashed");
    
        // Verify operator has not unregistered
        if (operator.unregisteredAt != type(uint256).max) {
            require(block.number < operator.unregisteredAt + urc.UNREGISTRATION_DELAY(), "Operator unregistered");
        }
    
        // Verify operator opted into rollup slasher
        require(slasherCommitment.optedOutAt < slasherCommitment.optedInAt, "Not opted into slasher");
        
        // Verify the Proposer opted into the Slasher sufficiently long ago
        require(blockNumber > slasherCommitment.optedInAt + urc.OPT_IN_DELAY(), "Too early to make commitments");
        
        // Verify the proposer's registered committer address was the blob sender
        require(slasherCommitment.committer == blobProposer, "Wrong blob submitter address");
        }
}
```

### Diagram
Note this is just a draft and subject to change after feedback and iteration.

![Fabric Overview with Dido](/website/assets/images/dido-overview.png)


### Standardizing derivation

Modifying how rollups derive their state is likely the most contentious thing about going based. If we can standardize this approach to derivation then we can potentially reduce the friction.

For starters, the `DerivationHelper` contract can be setup to enforce a dedicated `blobProposer` address by simply overloading the logic in `blobProposerIsAllowed()`, in which case the rollup would be backwards compatible with the OP stack today.

More generally this approach can be thought of as a hook, where the derivation function calls into a rollup-specified `DerivationHelper` to assert `blobProposerIsAllowed(bytes calldata blobProposerCredentials)` is true. This approach allows the rollup to apply *any* leader election mechanism they want (e.g., execution tickets / shared sequencing / etc). And if the, `DerivationHelper` is upgradeable, they can even modify derivation rules without modifying the core rollup stack logic (ofc this adds governance risk).

### Summary

- We can have extremely dumb inboxes!
    - OP stack can remain the same except derivation but derivation changes can be made backwards compatible with the current OP stack, it just requires modifying [this function](https://github.com/ethereum-optimism/optimism/blob/ed1e3416bddad4e9fce72352a8f41231f0d4a5b6/op-node/rollup/derive/data_source.go#L97) in their derivation pipeline
    - `DerivationHelper` can be adopted by any other stack
- We can reuse a single generic Lookahead contract across all based rollups
    - No more need to for each protocol to incentivize posting it as in Taiko PoC
    - No more tight coupling of lookahead + registry + inbox
- Gas costs are reduced because:
    1. we are sending blob transactions to an EOA address instead of a contract
    2. we do not need to pay for on-chain filtering logic
    3. we can easily support blob sharing
- Works with L2 execution preconfs
    - Preconfers monitor the URC + Lookahead to know if they have the write lock and can issue execution preconfs
    - `DerivationHelper` logic can be designed to prevent anyone from “cutting in line” to break a write lock
- Possibility of standardizing on how derivation works
    - `blobProposerIsAllowed(bytes calldata blobProposerCredentials)` can be overloaded to support other leader election mechanisms
- Compatible with real-time settlement
    - `DerivationHelper` logic can also be implemented in a different language, the logic just should be executed when proving STF

# Design Considerations

## Lookahead in derivation

Instead of having an on-chain view of the lookahead via L1 smart contract it’s possible to determine the L1 proposer’s public key during derivation as well via a Merkle inclusion proof against the beacon state root.

**Pros:**

- No need to post to the lookahead contract each epoch → reduce costs
- No need to develop complex on-chain code and introduce smart contract risk
- Proving is simpler and gasless in the L2 derivation

**Cons:**

- Requires access to beacon state to create the proof

It would be a dealbreaker if *all* L2 full nodes were required to run L1 beacon nodes as it increases hardware requirements. Fortunately, there is a workaround. Based sequencers are already required to run beacon nodes for their validator duties. We can simply require them to submit a Merkle proof that proves their slot in the lookahead as part of the encoded data in `blobProposerCredentials`. Then, any L2 full node can verify if the proposer was next in the lookahead *without* needing a beacon node, just the beacon state root. 

Putting this together, an example `blobProposerCredentials` could contain:

- proof they are part of the URC, collateralized, etc
- proof they were opted into the preconf protocol
- proof they were the proposer during that slot in the lookahead

---

## Proving requires blob hashes

After some discussion it is apparent that an EOA may not be sufficient as an inbox when it comes to proving. It was pointed out that the [blobhash](https://www.evm.codes/?fork=cancun#49) opcode is needed for a ZK rollup to prove against. It may be possible to prove a blob hash is valid against the beacon state (using the same proposed method for the lookahead); however, the blob hash is not known in advance so this would fail to meet the requirements for real-time settlement.

Therefore, to be compatible with ZK rollups and real-time settlement in the future, we should expect that instead of an EOA the inbox address points to a *contract* that at the very least stores the blobhash.

---

## Call for UBI

Directly following from the previous section, there is some notion of a minimal viable inbox contract, e.g., stores just the `blobhash` and `blockhash` when a blob is submitted. If this is it’s sole function, there really is no difference between inboxes which pushes us towards a “Universal Batch Inbox.” 

There is a bigger motivator than saving on contract deployment costs. To get atomic inclusion guarantees across based rollups, we need the based sequencer to produce a single global history for the involved rollups, which really points in the direction of blob sharing. With a UBI, all blobs can be posted to the same contract. The blob’s provenance and blob parsing can be determined during derivation and the blobhashes can be pulled from the UBI into the chain’s bridge contract during proving.

---

## Proving `DerivationHelper` execution

Having `DerivationHelper` exist as an L1 contract is convenient for two reasons:

1. it can easily call into the URC contract
2. it allows for a rollup to upgrade its sequencer selection rules without modifying their underlying node logic

Unfortunately, proving the execution of the `DerivationHelper` can be non-trivial depending on the proof system. For example, executing the `DerivationHelper` contract function as part of the OP stack’s derivation is sufficient for L2 full nodes to derive the canonical state. However, updating the fault prover to include this extra execution is non-trivial.

If we’re willing to sacrifice the benefits of 2) to make proving easier, then we can abandon the `DerivationHelper` as a smart contract and implement its logic in the native runtime of that proving system. In the OP stack, this could mean using storage proofs to fetch all the necessary data from the URC and then writing the logic from the `DerivationHelper` in Go. 

This approach would be sufficient to make the `DerivationHelper` logic provable, but makes standardization difficult as it essentially enshrines the sequencer selection rules into the node.