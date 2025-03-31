---
title: Step 2 - Atomic Execution
nav_order: 4.2
layout: default
parent: Composability 101
permalink: /education/composability/atomic-execution
---
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

# Atomic Execution

## What is it
A stronger guarantee than atomic inclusion is atomic *execution*, which guarantees atomic inclusion *and* allows for credible commitments to be made about the resulting state.

In our [previous example](/website/education/composability/atomic-inclusion#example), what are ways to provide assurances that the bundle will execute atomically and successfully, i.e., $$TX_B$$ will only mint on $$R_B$$ if $$TX_A$$ successfully burned on $$R_A$$ (and vice-versa)?

> It would require some sort of proof on the [shared sequencer’s] end, or an economic guarantee that far outweighs the cost of having part of an atomic bundle revert. - [Elizabeth (Astria)](https://www.astria.org/blog/astria-the-shared-sequencer-network)

Here is the spectrum varying from most trusted to least trusted:
 
1. **Reputation** - shared sequencers want to retain customers so they can promise at the social layer that they will not include atomic bundles that revert (e.g., by inserting bundles at the top of the respective blocks).
2. **Cryptoeconomic Safety** - shared sequencers are slashed if they break their promises. A shared sequencer can credibly commit to an atomic bundle succeeding if they put down enough collateral to cover the worst-case of the bundle failing.
3. **Cryptographic Safety** - The cross-chain bundle will execute atomically without reverting because execution is contingent on verifying ZKPs that attest to the state of other rollups, allowing for untrusted shared sequencers. In other words, $$TX_B$$ will only execute successfully if the state of $$R_A$$ after executing $$TX_A$$ is first proven.

| $$TX_1$$ succeeds | $$TX_2$$ succeeds | Safety Guarantee | Result |
|-------------------|-------------------|------------------|---------|
| ✅ | ✅ | Reputation | Happy case |
|✅ | ✅| Cryptoeconomic | Happy case |
|✅ | ✅| Cryptographic | Happy case |
| ✅ | ❌ | Reputation | shared sequencer cancelled on twitter |
| ✅ | ❌ | Cryptoeconomic | shared sequencer slashed |
| ✅ | ❌ | Cryptographic | — not possible — |
| ❌ | ✅ | Reputation | shared sequencer cancelled on twitter |
| ❌ | ✅ | Cryptoeconomic | shared sequencer slashed |
| ❌ | ✅ | Cryptographic | — not possible — |
| ❌ | ❌ | Reputation | Happy case |
| ❌ | ❌ | Cryptoeconomic | Happy case |
| ❌ | ❌ | Cryptographic | Happy case |

## An aside -  Open Intents Framework (cryptoeconomic safety*)

The [Open Intents Framework](https://www.openintents.xyz/The-Open-Intents-Framework-Intents-As-A-Public-Good-1976d35200d680fb8215f28775e067ec) (OIF) is an open source framework leveraging [ERC-7683](https://www.erc7683.org/spec) designed to allow developers to easily deploy intents infrastructure by piecing together off-the-shelf modular components (i.e., off-chain filler** logic and on-chain intents contracts). A goal of the OIF is to make it easier for upcoming rollups to gain access to intents infrastructure and filler liquidity.

The OIF is an interoperability effort happening in *parallel* to based and shared sequencing. While on the surface they appear to be fundamentally different approaches, if you squint you can start to see similarities to the cryptoeconomic safety approach for execution. In fact, as we’ll see the OIF benefits from the atomic inclusion guarantees from shared sequencing, but first what are intents?

### Intents via ERC-7683

Intents are a technique that, when applied to rollups, enable fast cross-chain transfers. Following the language of ERC-7683, Users submit intent orders to an origin chain which specifies the conditions of their transfer*** (e.g., exchange 100 USDC on $$R_A$$ for 100 USDC on $$R_B$$). A filler will find the best trade route and supply their capital to fulfill the order at the destination chain. Once the intent protocol verifies the intent was fulfilled, the User’s escrowed assets are released to the filler, plus a fee for the incurred settlement risk and cost of capital.

\* Kinda, you’ll see

** aka “Solvers”

*** Note that intents extend beyond token transfers

### OIF through the cryptoeconomic safety lens

It is possible to view intents from the lens of cryptoeconomic safety. To give a credible commitment, the filler needs the full transfer amount (the 100 USDC) at the destination chain. Filling the intent (i.e., performing the transfer at the destination chain) is analogous to locking their collateral. If the filler incorrectly filled the order (i.e., broke their commitment) they wouldn’t be able to reclaim back the equivalent amount of capital at the origin chain in the future, which from their POV is the same as their collateral being slashed.

### Filler risk

While this is a helpful analogy, it isn’t the filler’s incentives that guarantee safety, rather it is how their collateral is used. Intents circumvent all of the problems that arise when doing cross-chain mints and burns because they use the filler’s existing tokens on the destination chain instead of minting new ones. If a leg of the intent fails, the consequences are less severe for users and token supplies.

| $$TX_{R_A}$$ succeeds | $$TX_{R_B}$$ succeeds | Result |
|-------------------|-------------------|---------|
| ✅ | ✅ | 1) Happy case - user receives 100 USDC on $$R_B$$ and filler reclaims 100 USDC + earns fee on $$R_A$$ |
| ✅ | ❌ | 2) Degraded UX, user must wait for another filler transaction |
| ❌ | ✅ | 3) Filler lost 100 USDC as there's no intent to fill to reclaim their capital |
| ❌ | ❌ | 4) Happy case - atomicity held |

The filler shields users and token supplies from risk but takes on risk of their own. As shown in the third case, if $$TX_{R_A}$$ reorgs out after $$TX_{R_B}$$ landed, then the filler cannot reclaim their capital or collect their fee on $$R_A$$. Therefore, filler fees must price in this inherent risk (plus the cost of capital).

### OIF 🤝 atomic inclusion

Filler risk arises from the fact that they cannot guarantee atomic execution. If fillers were always guaranteed that $$TX_{R_B}$$ only executes if $$TX_{R_A}$$ executes successfully, the result could be a more efficient system that drives down cross-chain transfer prices (users just pay for the cost of capital).

Now to the elephant in the room - if we could guarantee atomic execution, there really isn’t a need for fillers as the burn and mint model would be as safe and more capital efficient! But as we’ll see, cryptographically-safe atomic execution isn’t exactly easy, so what is a more pragmatic solution?

*Atomic inclusion* can guarantee both $$TX_{R_A}$$ and $$TX_{R_B}$$ land on-chain and reorg together. This solves the issue in the third case where $$TX_{R_A}$$ reorging out of $$R_A$$ causes the filler to lose their capital, but there is still one more issue. Atomic inclusion does not guarantee that transactions execute correctly, so $$TX_{R_A}$$ could still revert (e.g., the user transferring their 100 USDC before it can be locked in escrow by $$TX_{R_A}$$).

One simple and pragmatic way to address this is if the shared sequencer dually functions as a filler. How does this play out?

- The shared sequencer bundles user intent orders with their fill orders.
- The shared sequencer only includes a bundle if it will succeed (which can be simulated ahead-of-time). They are incentivized to capture filler fees and prevent the loss of their capital.
- The shared sequencer does not take on reorg risk (just smart contract risk from the intents contract).
- Users experience low-latency, synchronous and atomic cross-chain swaps (that can be further sped up via [preconfs](https://ethresear.ch/t/based-preconfirmations/17353)).

Combining the OIF with shared sequencing has the potential to drive down user costs but raises concern is around centralization of the shared sequencer who is required to be sufficiently collateralized and sophisticated enough to become a filler.

### OIF Summary

Intents achieve cross-chain transfers by leveraging well-capitalized fillers that are willing to take on risk so that users and smart contracts do not. This is an extremely pragmatic solution as it can address a user pain point right away while solutions like cryptographically-safe atomic execution develop. To further optimize intent-based interoperability, it can be combined with the atomic inclusion guarantees from a shared sequencer, reducing filler risk and therefore reducing costs for users.

## The AggLayer approach (cryptographic safety)

*Note that AggLayer is one of a few implementations for cryptographic safety but is what we’ll examine in this document.*

As we previously covered, intents address a big user pain point (cross-chain transfers) but do not give atomic execution guarantees, rather they approximate cryptoeconomic safety which is not capital efficient. Instead of focusing on incentives to guide honest behavior, cryptographic safety makes it impossible for safety faults to happen, eliminating trust issues. We’ll now explore how shared bridging and ZKPs can enable safe atomic execution.

### What is AggLayer?

[AggLayer](https://polygon.technology/agglayer) is a protocol that provides cryptographic safety for atomic execution between rollups. It does this by using ZKPs to enforce cross-rollup [contingencies](https://prestwich.substack.com/p/contingency), meaning that a rollup cannot advance its state unless a contingent rollup’s state advanced correctly. When combined with proofs of accounting, AggLayer allows for assets to be trustlessly bridged between rollups.

Classical rollups can plug into this protocol regardless of how they handle sequencing, but we’ll see how using a shared sequencer with AggLayer allows us to move from asynchronous to synchronous atomic execution.

### Shared bridging

A shared bridge is an L1 contract that acts as a ledger, maintaining records of token ownership and balances across the L2s that use it. AggLayer requires rollups to use a shared bridge as a shared settlement layer.

To achieve safety in our [previous example](/website/education/composability/atomic-inclusion#example), the shared bridge would enforce that a rollup cannot withdraw assets that it doesn’t own. But wait isn’t this is what bridges do today? Someone updates the bridge’s view of the L2 by posting the latest state root and a proof system is in place to enforce safety (e.g., prevent the sequencer from arbitrarily withdrawing user funds). If the proof system rejects the state root, then users need to wait to be able to withdraw to the L1. Importantly, the L2’s state is finalized once their blob transaction finalizes on the L1, so if a proof system rejects the state root it doesn’t “reorg” their transactions, rather a correct view needs to be resubmitted for the rollup to settle. 

A shared bridge differs in that *multiple* rollups settle to it and it’s possible to add an *additional proof system* on top to ensure that when rollups try to individually settle, they play nice with other rollups and ensure the bridge’s global accounting is correct. 

If done correctly, shared bridging has desirable properties:

- assets can flow freely between rollups without going through the L1 which is costly in both time and gas
- there is no need for [well-capitalized fillers](/website/education/composability/atomic-execution#intents-via-erc-7683) to provide liquidity
- assets can be fungible across rollups since they all share the same bridge risk

### Pessimistic proofs

How does AggLayer [achieve safety](https://mirror.xyz/0xfa892B19c72c2D2C6B10dFce8Ff8E7a955b58A61/TXMyZhhRFa-bjr7YHwmJpKBwt2-_ysirbh_VpNy3qZY)? It’s this additional proof system which they call [pessimistic proofs](https://polygon.technology/blog/introducing-the-pessimistic-proof-for-the-agglayer-zk-security-for-cross-chain-interoperability?utm_source=twitter&utm_medium=social&utm_campaign=AggLayer). 

Rollups will prove their own state normally (i.e., with validity proofs), then AggLayer aggregates all of the proofs and create a pessimistic proof. This involves recursively proving that the validity proofs were valid, their contingencies were satisfied, and the bridge's accounting is correct. The shared bridge contract will only accept the rollups’ state roots if it can verify the pessimistic proof.

> With the pessimistic proof, one chain’s issues definitionally cannot contaminate the rest of the chains on the unified bridge. - [Polygon Labs](https://polygon.technology/blog/introducing-the-pessimistic-proof-for-the-agglayer-zk-security-for-cross-chain-interoperability?utm_source=twitter&utm_medium=social&utm_campaign=AggLayer)
> 

In other words, if any of the rollups’ state transition proofs is invalid (i.e., a malicious rollup is trying to steal funds from the bridge) then the pessimistic proof will prevent them from settling and harming the other rollups. Specifically, a pessimistic proof will guarantee:

- that the shared bridge’s accounting is sound, i.e., no rollup withdrew tokens they didn’t already deposit
- that each rollup’s individual state transition proof was valid
- that no custom inter-rollup contingency rules were violated

What exactly is meant by *inter-rollup contingency rules*? AggLayer allows rollups to express dependencies when they are settling their state. By creating contingencies, rollups can condition their settlement on things like the settlement of a different rollup. This allows rollups to bind to each other, setting the stage for atomicity. For example, $$R_B$$ can define a rule that their state can only settle iff $$R_A$$ previously settled with a specific state root $$state_{R_A}$$. 

Now we’ll see how this allows us to add safety to our previous examples.

### Safe (asynchronous) atomic execution

Let’s return to our earlier example and assume we are *not* using a shared sequencer.

A user wants to transfer 100 USDC from $$R_A$$ to $$R_B$$ - both opted into AggLayer - so they submit a $$TX_{burn_{R_A}}$$ and $$TX_{mint_{R_B}}$$ to the respective rollups.

1. $$R_A$$ and $$R_B$$ include the transactions
2. $$R_A$$ optimistically burns the user’s 100 USDC, resulting in $$state_{R_A}$$
3. $$R_B$$ optimistically mints the user 100 USDC, resulting in $$state_{R_B}$$
4. $$R_A$$ and $$R_B$$ commit block headers to AggLayer (containing $$state_{R_A}$$ and $$state_{R_B}$$ respectively) - importantly they each include commitments to the other rollup’s state root as well ($$state_{R_B}$$ and $$state_{R_A}$$ respectively) to create a contingency for atomicity
5. $$R_A$$ and $$R_B$$ independently submit proofs $$\pi_A$$ and $$\pi_B$$ to AggLayer which prove the validity of their state roots
6. AggLayer generates and submits the pessimistic proof to the shared bridge

Step 4 makes $$R_A$$'s settlement contingent on $$R_B$$ settling with $$state_{R_B}$$ and vice-versa, which sets the stage for atomicity, then step 6 guarantees the contingency is held. If $$R_A$$'s burn transaction failed or was not included, then the $$state_{R_A}$$ that $$R_B$$ committed to their header would be incorrect. Similarly, if $$R_B$$'s mint transaction failed or was not included, then $$state_{R_B}$$ that $$R_A$$ committed to in their header would be incorrect. The pessimistic proof would catch either case and the shared bridge contract would reject state roots from $$R_A$$ and $$R_B$$. 

This is sufficient to guarantee atomic execution but raises some questions:

- How does $$R_B$$ know the correct $$state_{R_A}$$ to commit to and vice-versa? Do they trust the contingent rollup, verify a proof, or run full nodes for *every* contingent rollup?
- How can we avoid trust issues when inter-chain contingencies allow rollups to delay other rollups’ settlement?

### Safe (synchronous) atomic execution

Now let’s explore how a shared sequencer can address some of these questions while making this interaction *synchronous*.

This time the user submits an atomic bundle with $$TX_{burn_{R_A}}$$ and $$TX_{mint_{R_B}}$$ to a shared sequencer that sequences both  $$R_A$$ and $$R_B$$.

1. Shared sequencer commits the shared rollup history to the L1 via shared blob transaction
2. $$R_A$$ and $$R_B$$'s derivation function guarantees the transactions cannot be unbundled (as described [before](/website/education/composability/atomic-inclusion#preventing-unbundling))
3. $$R_A$$ optimistically burns the user’s 100 USDC, resulting in $$state_{R_A}$$
4. $$R_B$$ optimistically mints the user 100 USDC, resulting in $$state_{R_B}$$
4. $$R_A$$ and $$R_B$$ commit block headers to AggLayer (containing $$state_{R_A}$$ and $$state_{R_B}$$ respectively) - importantly they each include commitments to the other rollup’s state root as well ($$state_{R_B}$$ and $$state_{R_A}$$ respectively) to create a contingency for atomicity
5. $$R_A$$ and $$R_B$$ independently submit proofs $$\pi_A$$ and $$\pi_B$$ to AggLayer which prove the validity of their state roots
7. AggLayer generates and submits the pessimistic proof to the shared bridge

Like the previous example, atomicity is guaranteed from the contingencies and pessimistic proof. The difference is by sharing a sequencer, we can eliminate some cross-rollup trust issues. Rather than trusting a different rollup for their state root, the shared sequencer will know ahead-of-time about what $$state_{R_A}$$ and $$state_{R_B}$$ are since they are part of the block building process. 

Further, they can guarantee that $$TX_{burn_{R_A}}$$ and $$TX_{mint_{R_B}}$$ will be sequenced so that they execute without reverting because they can simulate ahead-of-time. It was possible to assure *themselves* of this with atomic inclusion, the difference now is that the pessimistic proof guarantees this to *others*.

You may notice there’s still an issue here - the pessimistic proof guarantees atomicity, but not necessarily the desired outcome. A malicious shared sequencer could sequence transactions so that $$TX_{burn_{R_A}}$$ fails leading to $$state_{R_A}'$$ and $$R_B$$ creates its contingency with this bad state. The result is saying $$R_B$$ can only settle if the burn transaction *failed*. Note that in this case the pessimistic proof should prevent $$R_A$$ and $$R_B$$ from settling due to failed global accounting but the issue is present for other cross-chain actions besides token transfers. It is rooted in the fact that $$R_A$$ and $$R_B$$ do not have a view on each other’s state during execution.

For this we want *composability* which we will cover in the next section.

<span class="fs-8">
[> Step 3 - Composability](/website/education/composability/atomic-composability){: .btn }
</span>
