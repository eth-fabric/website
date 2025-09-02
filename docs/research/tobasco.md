---
title: Tobasco
nav_order: 5.4
layout: default
parent: Research
permalink: /research/scope
---
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

![Tobasco](/website/assets/images/tobasco.png)

# Tobasco: Top of Block for Atomic Synchronously Composable Operations

## What it Does
[Tobasco](https://github.com/eth-fabric/tobasco) is a proof-of-concept implementation of a preconf protocol that enforces top-of-block (ToB) L1 inclusions [based on a proposal from Brecht](https://x.com/Brechtpd/status/1854192593804177410). 

## How it Works
At the heart of Tobasco is the `Tobasco.onlyTopOfBlock()` modifier that reverts if the transaction isn't executed at ToB by checking gas consumption. If it succeeds, the modifier records a flag that indicates that ToB execution succeeded. This flag serves as evidence that can be used to punish proposers in proposer commitment protocols.

The repo contains an example Slasher contract that uses the [URC](../development/l1-components/urc.md) and [Constraints API](../development/l1-components/constraints-api.md).

## Why it Matters
Tobasco is a simple but powerful primitive that can be used to ensure L1 super transactions as described in [Ultra TX](../education/composability/ultra-txs.md), [Signal-Boost](signal-boost.md), and [SCOPE](scope.md) execute with the latest L1 state for L1<>L2 synchronous composability. Beyond that it is a simple primitive for proposers to sell the ToB position to arbitragers.