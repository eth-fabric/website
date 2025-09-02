---
title: Commitments API
nav_order: 7.3
layout: default
parent: L1 Components
permalink: /development/l1-components/commitments-api
---
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---
### Resources
- Annotated spec can be found [here](https://github.com/eth-fabric/commitments-specs).
- API can be found [here](https://eth-fabric.github.io/commitments-specs/)

# Commitments Spec
Neutral API specification for proposer commitments on Ethereum. This was developed via collaboration and contribution from teams across the Ethereum ecosystem who were interested in helping develop and enable proposer commitments on Ethereum. These efforts started at zuBerlin, continued through Edge City Sequencing Week, and have progressed through community calls (recordings can be found in the pm repo).

The Commitments API is a minimal set of endpoints for users (i.e., wallets) to request proposer commitments.

It aims to:
- Support both L1 and L2 commitments
- Support *generic* proposer commitments rather than just preconfs
- Support simple wallet integration
- Support for users to learn about capabilities / fees

### In Scope (Commitments API)
- Requesting a commitment to be made
- Requesting to see a previously signed commitment
- Requesting to see a the upcoming capabilities of a Gateway
- Requesting fee information for a specific commitment request

### Out of Scope
- The Constraints Specs and API.
