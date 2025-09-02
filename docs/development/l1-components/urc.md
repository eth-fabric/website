---
title: Universal Registry Contract
nav_order: 7.2
layout: default
parent: L1 Components
permalink: /development/l1-components/urc
---
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---
# Universal Registry Contract
On the back of a proposal by Justin Drake and a [post](https://ethresear.ch/t/credibly-neutral-preconfirmation-collateral-the-preconfirmation-registry/19634) by mteam, teams started to work on a universal registry contract. The point of the contract is to have a standardized registration process for preconfs (or other commitments) from proposers and have a unified registry that L2s and others could leverage to understand who has opted in to provide sequencing / preconfs for based rollups. Below are some of the key design principles teams used: 

- The contract is super simple and short (ideally ~100 lines).
- Only ETH for deposits.
- All the constants are parametrised (so as to minimise bike shedding for now).
- Slashing (and delegation) logic is encapsulated by pieces of signed EVM bytecode shared offchain between relevant parties (eg users and gateways). This is for maximum credible neutrality, forward compatibility, simplicity, and gas efficiency.
- No dependence on any external code (especially restaking platforms).
- Zero governance, fully immutable.
- Open source from day 1, Apache 2.0 + MIT dual licensing.
- Nice to have: support for underwriters
- Nice to have: bootstrapping phase with freeze instead of burn
- Code is maintained in a neutral Github org.

Teams worked on an overview doc with details, this can be found [here](https://github.com/eth-fabric/urc/blob/main/docs/overview.md)

The team also provided some [reference implementations](https://github.com/eth-fabric/urc/tree/main/example) teams can use as starting points and to give more context around the URC. 
