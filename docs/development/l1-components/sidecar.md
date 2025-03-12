---
title: Validator Sidecar
nav_order: 7.4
layout: default
parent: L1 Components
permalink: /development/l1-components/sidecar
---

{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---
# Validator Sidecars
The following will highlight sidecars that can be leveraged for Ethereum L1 proposers to issue preconfs for based rollups. A sidecar is an extra piece of software that validators run, in current instances, alongside their consensus client. This enables them to have additional functionality or features. The longest standing example of a sidecar is [MEV-boost]( https://github.com/flashbots/mev-boost).

# Commit-Boost
![Commit-Boost](/website/assets/images/Commit-Boost-Logo.png)
Due to the risks developing for Ethereum, core development, and its validators set, a group of teams / individuals are working on developing a public good called Commit-Boost. [Commit-Boost](https://x.com/Commit_Boost) is an open-source public good that is fully compatible with MEV-Boost, but acts as a light-weight validator platform to safely make commitments. Specifically, Commit-Boost is a new Ethereum validator sidecar focused on standardizing the last mile of communication between validators and third-party protocols. It's being developed in Rust from scratch, and has been designed with safety and modularity at its core, with the goal of not limiting the market downstream including stakeholders, flows, proposer commitments, enforcement mechanisms, etc. Some of its features include:

- Unification: Core devs will be able to interact and work with one standard during Ethereum forks / upgrades / when and if things go wrong
- Backward compatibility + more: Commit-Boost is not only backward compatible with MEV-Boost, but will improve the life of validators who only run MEV-Boost through increased reporting, telemetry / other off-the-shelf tools validators can employ
- Opt-in without running more sidecars: Commit-Boost will allow proposers who want to opt into other commitments do so without having to run multiple sidecars
- Robust support: Commit-Boost the software is supported by a not-for-profit entity. This team will also be focused on testing and adjustments needed during hard forks and have a team to interact with to help during adoption, improvements, and sustainment
- Not VC-backed public good: This team and effort will not be VC-backed. There is no monetization plan and no token. The entity will not be able to sell itself and will not start any monetizable side businesses. 

## How Validators Can Run Commit-Boost
More details can be found in the Commit-Boost [documentation](https://commit-boost.github.io/commit-boost-client/category/running).

## How Can I Build A Preconf Protoocl or Other Proposer Commitment
More details can be found in the Commit-Boost [documentation]([https://commit-boost.github.io/commit-boost-client/category/running](https://commit-boost.github.io/commit-boost-client/category/developing).

## Resources
Below are resources to learn about Commnit-Boost:
- [Repo](https://github.com/Commit-Boost/commit-boost-client)
- [Documentation](https://commit-boost.github.io/commit-boost-client/)
- [EILI5](https://x.com/Commit_Boost/status/1838943197172510956)
