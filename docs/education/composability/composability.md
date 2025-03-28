---
title: Composability
nav_order: 4
layout: katex
parent: Education
permalink: /education/composability
---
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

We often hear that rollups are presented as the solution to fragmentation. At the same time, various initiatives, such as the [Open Intents Framework](https://www.openintents.xyz/The-Open-Intents-Framework-Intents-As-A-Public-Good-1976d35200d680fb8215f28775e067ec), are actively addressing interoperability challenges.

The goal of this series is to clarify the concept of composability, enabling a deeper understanding of the differences, limitations, and trade-offs among these approaches. This understanding will help shape Fabric’s objectives as we move forward.

Note: Much of the terminology used is influenced by the writings of [Jon Charbonneau](https://dba.xyz/were-all-building-the-same-thing/#cross-chain-unbundling) and [James Prestwich](https://prestwich.substack.com/p/the-definitive-guide-to-sequencing).

### Terminology
- $$R_A$$: Rollup A
- $$R_B$$: Rollup B
- $$T_{L1}$$: Transaction 1 on $$L1$$
- $$T_{A1}$$: Transaction 1 on $$R_A$$
- $$T_{B1}$$: Transaction 1 on $$R_B$$
- $$B_{A1}$$: Block 1 on $$R_A$$
- $$B_{B1}$$: Block 1 on $$R_B$$
- Gateway: A third party with [Constraint](/website/development/l1-components/constraints-api) and [Commitment](/website/development/l1-components/commitments-api) submission authority granted by the L1 Proposer.

| Term | What it means (adapted from [we're all building the same thing](https://dba.xyz/were-all-building-the-same-thing/#definitions))
|----------|----------|
| Atomic         | All or nothing         | 
| Inclusion      | Transactions included when sequencing a rollup         |
| Execution      | Transactions execute as intended on a rollup         |
| Synchronous    | At the same time (slot height)         |
| Asynchronous   | At a different time (slot height)         |
| Composability  | Different contracts may access each other's state and take actions dependent upon it         |
| Universal  | Across rollups and the L1          |

Each of these terms can be thought of us building blocks that when combined provide different guarantees. The following are some examples of what is possible by combining these. In the later sections we'll examine them in greater detail:


| Combination | Example |
|----------|----------|
| Atomic Inclusion         | $$T_{A1}$$ will be sequenced on $$R_A$$ if and only if $$T_{B1}$$ is sequenced on $$R_B$$
| Atomic Execution         | $$T_{A1}$$ on $$R_A$$ will execute if and only if $$T_{B1}$$ on $$R_B$$ executes as intended successfully          |
| Asynchronous Composability         | $$T_{A1}$$ on $$R_A$$ executes using state from $$R_B$$ which whose state settles at a different slot          |
| Synchronous Composability         | $$T_{A1}$$ on $$R_A$$ executes using state from $$R_B$$ all within the same slot          |
| Universal Synchronous Composability         | $$T_{A1}$$ on $$R_A$$ executes and then $$T_{L1}$$ on the L1 executes using state from $$R_A$$ all within the same slot          |

### Setting our north star
Since Fabric is concerned with accelerating based rollup development, it is important to explicitly define the goals that are achievable by going down this path. Given L2 to L2 synchronous composability is achievable *without* based sequencing, the overarching goal and value prop of based rollups is to achieve synchronous composability between the **L1** and other L2s, which has been coined as Universal Synchronous Composability (USC). 

As Justin Drake puts it:

> In other words users can make arbitrary synchronous calls between [rollups], seamlessly interleaving execution across rollups. - [Reddit AMA](https://www.reddit.com/r/ethereum/comments/191kke6/comment/kh78s3m/?utm_source=share&utm_medium=web3x&utm_name=web3xcss&utm_term=1&utm_content=share_button)
> 

Achieving this is **only possible** with based sequencing because L1 proposers are the only entity with a write-lock on the L1, which is necessary to guarantee they can simultaneously sequence the L1 and based rollups.

### Synchrony requirements
If achieving USC is our goal, let's first start with what's required for synchrony.


> cross-chain synchronous composability definitionally requires some type of shared sequencer for that slot height. Chains without one can only ever have asynchronous composability - [Jon Charbonneau](https://dba.xyz/were-all-building-the-same-thing/#universal-synchronous-composability)
> 

The table below summarizes the requirements for synchrony. In the following sections we’ll understand that while based sequencing is a requirement for synchrony between the L1 and L2s, it isn’t sufficient for composability. For that we’ll need to introduce either cryptographic or cryptoeconomic solutions.

| Synchrony between | Requirement |
|----------|----------|
| L2<>L2 | Shared L2 sequencer |
| L1<>L2 | Based Sequencer |
| L2<>L3 + L3<>L3 | Shared L3 sequencer |

Note that a shared L2 sequencer includes your non-based interoperability solutions e.g., Espresso or the Superchain. Additionally, a shared L3 sequencer could be a shared L2 sequencer or more simply a centralized L2 sequencer (see Spire’s post).

<span class="fs-8">
[Atomic Inclusion](/website/education/composability/atomic-execution){: .btn }
</span>

