---
title: Constraints API
nav_order: 7.1
layout: default
parent: L1 Components
permalink: /development/l1-components/constraints-api
---
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---
# Constraints Spec
Neutral API specification for proposer commitments on Ethereum. This was developed via collaboration and contribution from teams across the Ethereum ecosystem who were interested in helping develop and enable proposer commitments on Ethereum. These efforts started at zuBerlin, continued through Edge City Sequencing Week, and have progressed through community calls (recordings can be found in the [pm](https://github.com/eth-fabric/pm) repo).

In summary the Constraints API is:
- A minimal set of endpoints extending the existing PBS architecture for proposers to add constraints during block building. This unlocks proposer commitments, particularly for unsophisticated proposers who delegate block building responsibilities.
- While the first use case is based preconfs, the Constraints API is designed to be generic and future-proof, accommodating a wide range of proposer commitments as the space evolves.
- The spec defines how Proposers, Gateways, Builders, and Relays communicate to issue, prove, and verify constraints.

### In Scope (Constraints API)
- the delegation of block-building responsibilities from proposers to gateways
- the submission of constraints to the relay
- the retrieval of constraints from the relay
- the submission of blocks and proofs of constraint validity to the relay
- the retrieval of block headers and proofs of constraint validity from the relay

### Out of Scope (Commitments API)
- any interaction between third parties (Users, Wallets, etc) and the Gateway.
- commitments from Gateways to third parties

### Flow

![Flow of Constraints-API](/website/assets/images/Constraints-API.png)

More details on the API definitions and specs can be found [here](https://github.com/eth-fabric/constraints-specs/blob/main/specs/constraints-api.md).
