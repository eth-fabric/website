---
title: Validator Sidecar
nav_order: 7.4
layout: default
parent: L1 Components
permalink: /development/l1-components/sidecar
---

# Validator Sidecars
The following will highlight sidecars that Ethereum L1 proposers can leverage to issue preconfs for based rollups. A sidecar is an additional piece of software that validators run—typically alongside their consensus client—to enable additional functionality or features. The longest-standing example of a sidecar is [MEV-boost]( https://github.com/flashbots/mev-boost).

# Commit-Boost

![Commit-Boost](/website/assets/images/Commit-Boost-Logo.png)

Due to the risks developing for Ethereum, its core development, and its validator set, a group of teams and individuals are working on a public good called Commit-Boost. [Commit-Boost](https://x.com/Commit_Boost) is an open-source public good that is fully compatible with MEV-Boost but acts as a lightweight validator platform to safely make commitments. Specifically, Commit-Boost is a new Ethereum validator sidecar focused on standardizing the last mile of communication between validators and third-party protocols. It is being developed in Rust from scratch and has been designed with safety and modularity at its core, ensuring it does not limit the downstream market, including stakeholders, flows, proposer commitments, enforcement mechanisms, and more. Some of its features include:

- Unification: Core developers will be able to interact with and work under one standard during Ethereum forks, upgrades, or when unexpected issues arise.
- Backward compatibility and more: Commit-Boost is not only backward compatible with MEV-Boost but also improves the experience for validators running MEV-Boost through enhanced reporting, telemetry, and other off-the-shelf tools.
- Opt-in without running additional sidecars: Commit-Boost allows proposers who want to opt into other commitments to do so without the need to run multiple sidecars.
- Robust support: Commit-Boost is supported by a not-for-profit entity. This team will focus on testing and necessary adjustments during hard forks and provide assistance with adoption, improvements, and long-term sustainability.
- A non-VC-backed public good: This initiative is not backed by venture capital. There is no monetization plan and no token. The entity cannot be sold and will not launch any monetizable side businesses.

## How Validators Can Run Commit-Boost
More details can be found in the Commit-Boost [documentation](https://commit-boost.github.io/commit-boost-client/category/running).

## How Can I Build A Preconf Protoocl or Other Proposer Commitment
More details can be found in the Commit-Boost documentation [here]([https://commit-boost.github.io/commit-boost-client/category/running) and [here](https://commit-boost.github.io/commit-boost-client/category/developing).

## Resources
Below are resources to learn about Commnit-Boost:
- [Repo](https://github.com/Commit-Boost/commit-boost-client)
- [Documentation](https://commit-boost.github.io/commit-boost-client/)
- [EILI5](https://x.com/Commit_Boost/status/1838943197172510956)
