---
title: Based-Rollups-Componets
nav_order: 3.2
layout: default
parent: Based Rollup Series
permalink: /education/based-rollups/Based-Rollups-Componets
---


## Details Around Componenents and Sub-Componenets of Based Rollups from Fabric's POV

Building on Based Rollup 101, this provides context for the key components and sub-components of a based rollup stack. You'll find a visual representation of these elements, followed by a description of each and its role within the stack.

![Fabric Overview](/website/assets/images/fabric-overview.png)

---
## L1 Components
Below are sub-componenets that are focused on smart contract and API changes required around L1 infrastructure / to be built on top of Ethereum. 

### Commit-Boost

- **What is it**
    - Commit-Boost is a Ethereum validator sidecar focused on standardizing the communication between validators and third-party protocols (i.e., preconfs).
    - Enables validators to run a single sidecar while easily making multiple commitments without additional devops overhead.
    - Provides tools to streamline the development of off-chain proposer commitment protocols (e.g., standardizing validator signature requests).
    - Allows proposers to make a commitment to issue a preconf.
- **Resources:**
    - [repo](https://github.com/Commit-Boost/commit-boost-client)
    - [intro post](https://ethresear.ch/t/commit-boost-proposer-platform-to-safely-make-commitments/20107)
    - [docs](https://commit-boost.github.io/commit-boost-client/)

### Constraints API

- **What is it**
    - A minimal set of endpoints extending the existing PBS architecture for proposers to add constraints during block building. This unlocks proposer commitments, particularly for unsophisticated proposers who delegate block building responsibilities.
    - While the first use case is based preconfs, the Constraints API is designed to be generic and future-proof, accommodating a wide range of proposer commitments as the space evolves.
    - The spec defines how Proposers, Gateways, Builders, and Relays communicate to issue, prove, and verify constraints.
- **Resources:**
    - [OpenAPI](https://eth-fabric.github.io/constraints-specs/)
    - [annotated API](https://github.com/eth-fabric/constraints-specs/blob/main/specs/constraints-api.md)
    - [Proposer / Gateway / Builder specs](https://github.com/eth-fabric/constraints-specs/pull/22)

### Commitments API 

- **What is it**
    - A standardized API that defines how validators (or its delegate) receives commitment requests and issue signed commitments. While the first use case is preconfs, it includes flexibility in the schemas to support future commitment types, given the evolving design space (e.g., inclusion, execution, state lock).
    - In the context of preconfs, this enables consumers (Users, Wallets, Dapps) to request and verify preconfs from the party issuing the preconfs.
    - Helps converge on a standard messaging format, reducing implementation friction for parties issuing commitments.
    - Helps translate user “intents” into actionable requests within the block-building supply chain or in other terms, maps the commitment to a constraint that can be understood by the rest of the PBS pipeline.
    - Enables block explorers and wallets to agree on a way to report the statuses of preconfed transactions.
- **Considerations**
    - Nice to haves include:
        - A way for users to get commitment pricing information
        - A way for users to query gateway capabilities
- **Resources**
    - [repo](https://github.com/ethereum-commitments/commitments-specs)

### URC

- **What is it**
    - A minimal smart contract that enables registrations and slashing to enhance the credibility of generic proposer commitments.
    - Designed to be lightweight, avoiding enshrining any specific restaking protocol’s slashing logic.
- **Resources**
    - [repo](https://github.com/ethereum-commitments/urc)

---
## L2 Components
Below are sub-componenets that are focused on ...

### Lookahead Window 

- **What is it?**
    - A smart contract that provides an on-chain view of the beacon chain lookahead. This is essential for *most* based rollups to move beyond “total anarchy mode” by granting sequencing rights ahead-of-time—such as for enforcing state locks for L2 execution preconfs.
    - Proving the lookahead each epoch is gas-intensive, but costs can be amortized if shared across multiple based rollups.
    - No single entity has an incentive to bear these costs alone (tragedy of the commons), making it suitable for a subsidized entity like Fabric to coordinate and help getting funding / support as a public good.
- **Considerations**
    - A generic lookahead contract would report on-chain *all* upcoming BLS keys for the epoch. To grant state locks ahead-of-time, rollup inbox contracts still need to know which of the validators are preconfers for their rollups.
    - A lookahead is just one way to assign sequencing rights in a based rollup and ideally protocols do not have to tightly couple a lookahead with their inbox contracts, but can instead optionally tap into one.
- **Discussion**
    - Should we push for an EIP instead for medium to long-term and rollups will use their own on-chain implementations in the short-term?
    - An EIP for an on-chain view of the lookahead should be compatible with the beam chain if we use proposer indices instead of public keys
- **Resources**
    - [Nethermind/Taiko implementation](https://github.com/NethermindEth/Taiko-Preconf-AVS/blob/9cb5f467c0065cb84d152f9b217d819b294b8d5d/SmartContracts/src/avs/PreconfTaskManager.sol#L316)

### Inbox Contracts

- **What is it?**
    - The L1 smart contract where sequencers post DA. It is responsible for determining who can submit data and how they do it.
    - Since based sequencing requires rollups to delegate sequencing rights to validators, the inbox contract’s design significantly impacts the complexity of this process.
    - Different rollup stacks vary in how they implement this (e.g., EOA-submitted for OP Stack vs. contract-based for Taiko and others), so standardization an inbox contract interface will help current and future rollups maintain compatibility with based sequencing.
- **Considerations**
    - EOA-based submission is cheaper and a competitive advantage for OP stack chains but introduces complexity for based sequencing.
    - The OP stack allows for individual transactions to be posted, whereas Taiko allows delayed batches. Both designs have their tradeoffs and the interface will likely need to be opinionated.
    - The inbox contract should be decoupled from the leader-election process.
    - Ideally we can enshrine “total anarchy mode” and generic message passing into a design by default. Rollups can apply their leader election process on top (e.g., use the lookahead or elect a centralized sequencer).
- **Discussion**
    - Should alt-DA support be a priority now or deferred to avoid scope creep?
        - Alt-DA gives users cheaper options but introduces complexity. Excluding it can hurt adoption from incumbent rollups whose customers depend on it.
        - Alt-DA is more important before we have full DAS. If Fabric excludes alt-DA and frontruns the Ethereum roadmap, we could have a blob supply issue. Standardizing an interface that supports alt-DA can still be used for blob sharing + full DAS when it’s ready.
    - Should EXECUTE support be a priority now or deferred to avoid scope creep?
    - Is it worth pushing this now?
        - Non-based rollups also benefit from a standardized inbox because they can share the same messaging format.
        - But there will be headwinds convincing incumbent rollups to modify their inboxes as it **affects their derivation rules**, especially if they have to adopt a competitor’s inbox format.
        - But real-time proving could be a forcing function for change - if you want shared settlement each slot the shared bridging / blob sharing / inbox all start to collapse.
- **Resources**
    - ‣

### Blob Sharing

- **What is it**
    - Until blob supply is abundant, rollups need a way to share a single blob efficiently to reduce transaction costs.
    - Blob sharing helps to:
        - Increase competition – Lowers the barrier for smaller rollups that can’t efficiently fill a full blob.
        - Improve based rollup efficiency – Based rollups post blobs more frequently than classical rollups, preventing long amortization periods. Based rollups can improve efficiency by amortizing blob costs across multiple rollups (this also helps classical rollups)
        - Helps with shared real-time settlement -If you can guarantee that rollup A and rollup B share a blob then their data is either posted together or not at all.
- **Considerations**
    - Shared L1 transactions – Instead of multiple based L2s each paying 21,000 gas for separate L1 transactions, they can share a single transaction to amortize costs.
    - Streaming Compression – Compress → Pack to allow for parallelization.
    - Shared calldata packin**g** – Support multi-rollup calldata aggregation for when it’s economical to use.
    - Nethermind has received an EF grant to work on blob sharing 🔥
- **Discussion**
    - Should we prioritize transactions, state diffs, or both?
        - Transactions are more intuitive for based sequencing, but real-time proving could make state diffs more viable.
- **Resources**
    - https://hackmd.io/@linoscope/blob-sharing-for-based-rollups
    - https://x.com/Spire_Labs/status/1892030335212454308

### Sequencer Resolution

### Sequencer Shared Blob Parsing

