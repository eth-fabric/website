---
title: Roadmap
nav_order: 2
layout: default
permalink: /roadmap
---
# Fabric Roadmap

This page provides an overview of how Fabric views the components required for a based rollup. We begin by providing context for this perspective, then reorder the components based on our current priorities in collaboration with the community. This approach has been shaped by countless conversations with teams and community members. We welcome feedback and input and will continue to refine our approach as we research, engage with others, and fully develop the specifications.

![Fabric Overview](/website/assets/images/fabric-overview.png)

{: .no_toc }

## Table of contents

{: .no_toc .text-delta }

1. TOC
{:toc}

---
# Priorities

Below are two visuals around the current priortization of efforts around Fabirc after speaking with teams and gathering input weighing what should be a public good, areas where things should and can potentially be standardized, what can be achieved in the near-term, and where there may already be teams focused. We look forward to more feedback on this, but believe we can accelerate some items and starting moving on a few that need some TLC! We look forward to continuing this effort with the community! 

### Visual By Summary Description
![Fabric Priorities](/website/assets/images/fabric-priorities.png)

### Timeline Fabric Priorities
![Timeline Fabric Priorities](/website/assets/images/Timeline-Fabric-Priorities.png)

___
# Details On Sub-Components
### Education

- **What is it**
    - Spreading awareness of based rollups is essential for adoption. Users should understand their choices and clearly see the benefits and potential tradeoffs.
    - There is no single definitive source of information, making “base pilling” a challenge.
    - Many projects are focused on shipping and lack the time to educate users, so Fabric can contribute by providing accessible educational resources.
- **Considerations**
    - Create an awesome-based-rollups repo to aggregate everything based
    - Create a one-pager ELI5 explainer website for based sequencing (can be static page hosted via github pages with the [basedsequencing.org](http://basedsequencing.org) domain that the community can iterate on)
    - ELI5 Diagrams
- **Discussion**
    - Should Fabric maintain a dedicated knowledge base or collaborate with existing educational platforms?
    - What are the most effective ways to reach different audiences (developers, users, projects)?
    - How can we ensure neutrality while promoting based rollups?
- **Similar Resources**
    - https://docs.spire.dev/spire-overview/based-resources
    - https://github.com/NethermindEth/awesome-preconfirmations

### Commit-Boost

- **What is it**
    - Commit-Boost is a Ethereum validator sidecar focused on standardizing the communication between validators and third-party protocols (i.e., preconfs).
    - Enables validators to run a single sidecar while easily making multiple commitments without additional devops overhead.
    - Provides tools to streamline the development of off-chain proposer commitment protocols (e.g., standardizing validator signature requests).
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
    - A standardized API that defines how gateways receive commitment requests and issue signed commitments. While the first use case is preconfs, it includes flexibility in the schemas to support future commitment types, given the evolving design space (e.g., inclusion, execution, state lock).
    - Enables consumers (Users, Wallets, Dapps) to request and verify preconfs from Gateways.
    - Helps preconf protocols converge on a standard messaging format, reducing implementation friction for Gateways.
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

### Lookahead Window 

- **What is it?**
    - A smart contract that provides an on-chain view of the beacon chain lookahead, essential for *most* based rollups to move beyond “total anarchy mode” by granting sequencing rights ahead-of-time—such as for enforcing state locks for L2 execution preconfs.
    - Proving the lookahead each epoch is gas-intensive, but costs can be amortized if shared across multiple based rollups.
    - No single entity has an incentive to bear these costs alone (tragedy of the commons), making it suitable for a subsidized entity like Fabric to operate as a public good.
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
    

### Shared Bridging

- **What is it**
    - Shared bridging is where multiple rollups leverage a common L1 bridge to transfer assets and messages between each other, improving interoperability, while reducing security by not relying on third parties.
- **Considerations**
    - Also referred to as the **“**Universal Deposit Contract”
    - Could be implemented as a “Universal Embedded Bridge” via an embedded rollup.
    - Essential for non-based rollup interop.
    - 🚧 There are several excellent teams working on this already 🚧
- Discussions
    - 
- **Resources**
    - https://hackmd.io/@EspressoSystems/SharedSequencing
    - https://ethresear.ch/t/embedded-rollups-part-2-shared-bridging/21461
    

### Transaction Fee Mechanisms

- **What is it**
    - Taiko’s mainnet implementation does not price in L1 base fees when charging for L2 transactions.
    - As a result, when L1 base fees spike, there is no incentive to propose Taiko blocks.
    - For better margins, a proposer will wait until the L1 prices fall, harming UX for Taiko users.
    - Preconfs solve the UX problem but commit the proposer to taking on the base fee volatility during their slot.
    - Without a TFM to fix this, it could be hard to onboard preconfers, especially for low-activity based rollups.
- **Considerations**
    - All based rollups will face this issue as they do not have the flexibility to wait for lower L1 base fees to post data without sacrificing UX (due to a lack of soft-confirmations).
- **Discussions**
    - Is this something worth standardizing or is it a vector for based rollups to compete?
        - It may be more appropriate to collaborate on research than to push a standard.
- **Resources**
    - Todo



