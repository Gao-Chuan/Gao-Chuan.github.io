---
title: 'zkSync L3 exploration'
date: 2022-12-12
permalink: /posts/2022/12/zksync_l3_exploration/
tags:
  - zkSync
  - Layer 3
  - topics
---

# Summary

zkSync's Layer 3, which they refer to as 'HyperChains,' is envisioned as an ecosystem of customizable and trustlessly linked blockchains powered by zkEVM (zkSync's EVM-compatible system). They believe that Layer 3 will enable limitless scaling with customization options, leading to significant advancements in several key areas.

The key motivations behind designing zkSync's Layer 3 are as follows:

1. Security: zkSync aims to address the inherent weaknesses in non-native bridges that often contribute to hacks. In Layer 3, all interactions between fractal HyperChains occur via native bridges, resulting in a 10X improvement in security.
2. Performance: While Layer 2 solutions already provide significant performance increases (10–100X), zkSync believes that Layer 3 can achieve limitless performance improvements.
3. Cost: zkSync expects exponential reductions in data costs in Layer 3 due to the availability of various data solutions. Developers can choose data availability options that best suit their project's needs, balancing price, performance, and security.
4. Ease of Use: zkSync anticipates significant improvements in software development kits (SDKs), command-line interfaces (CLIs), and low-code/no-code solutions for different use cases. These improvements will make application creation 10X easier, attracting more developers to the ecosystem.
5. Composability: zkSync's LLVM compiler not only supports the Solidity programming language but also modern languages like Rust, C++, and Swift. This expanded language support offers a 10X increase in accessibility for developers with expertise in these languages, fostering greater innovation and development.

Additionally, zkSync highlights three different data availability options in Layer 3:

- ZK rollup: Provides full Ethereum-level security, suitable for decentralized finance (DeFi) applications.
- zkPorter: Offers a mixture of on-chain and off-chain data to optimize for affordability, speed, and security. This option is particularly suitable for gaming applications.
- Validium: Emphasizes ultimate performance but with slightly less security than the Ethereum network, opening up a wide range of use cases beyond DeFi.

Moreover, zkSync aims to allow developers to customize their fractal HyperChains further. They envision features such as customizable privacy settings, the ability for ecosystem partners to have their own tokens to secure various components, and the replacement of non-native bridges with native bridges (HyperBridges) for enhanced security and trust.

In Q1 of 2023, zkSync plans to release a prototype of their Layer 3 called 'Opportunity' on a public testnet. Opportunity will serve as a foundation for public experimentation, research, and development of Layer 3 solutions. The team is excited about the possibilities and welcomes researchers and developers interested in partnering or joining their project.

Overall, zkSync's motivation behind designing Layer 3 is to create an ecosystem of highly scalable, customizable, and interconnected blockchains that can push the boundaries of Ethereum's capabilities and facilitate widespread adoption of blockchain technology.

# How it works

1. Users sign transactions and submit them to validators.
2. Validators roll up thousands of transactions together in a single block and submit a cryptographic commitment (the root hash) of the new state to the smart contract on mainnet along with a cryptographic proof (a SNARK) that this new state is indeed the result of the application of some correct transactions to the old state.
3. Additionally to the proof, the state ∆ (a small amount of data for every transaction) is published over the mainchain network as cheap `calldata`. This enables anyone to reconstruct the state at any moment.
4. The proof and the state ∆ are verified by the smart contract, thus verifying both the validity of all the transactions included in the block and the block data availability.

## Transaction finality

Validators elected to participate in the **zkSync** block production will have to post a significant security bond to the **zkSync** smart contract on the mainnet. A consensus run by the validators provides a subsecond confirmation to the user that their transaction will be included in the next **zkSync** block, signed by a supermajority of (more than) ⅔ of the consensus participants (weighted by stake).

If a new **zkSync** block is produced and submitted to the mainchain, it cannot be reverted. However, if it doesn’t contain the promised transactions, the security bond of the intersection of the signers of the original receipt and the signers of the new block will be slashed. This intersection is guaranteed to have more than ⅓ of the stake. This guarantees that at least ⅓ of the security bond can be slashed and that only malicious validators will be punished.

A portion of the slashed funds will be used to compensate the tx recipient. The rest will be burned.

## Congest

The validator's node is configured to automatically increase the gas price to over-the-average level to get **zkSync** blocks mined with high priority. The cost per transaction are ~1/100th of the cost of corresponding plain transaction on the L1.

## Hyperchains & Hyperbridges

![Untitled](images/zksync_l3_exploration/p1.png)

1. Rollups have validating bridges that are trustless.
2. Native bridges can easily burn and mint the native tokens for transfers between members of the ecosystem.
3. The L1 serves as a single source of truth, so the rollups cannot hard fork.
4. The ecosystem can coordinate the hard fork together in case a vulnerability is found using a governance framework on L1, similar to how the L1 would react to a vulnerability.

L1 settlement of ZK rollups is fast (down to minutes), and we can securely connect chains due to the cryptographic nature of the validity proofs.

![Untitled](images/zksync_l3_exploration/p2.png)

### **Scalability**

#1 

If every Hyperchain wanted to settle their proofs to L1 independently, the total load on the L1 would still be proportional to the total number of Hyperchains. The shared prover aggregates the proof of different Hyperchains, settling all of them on L1 in a single proof.

The shared prover will be optional, Hyperchains can choose to not participate. In this case, they can settle their proofs directly to Ethereum for a much larger fee. It will also be decentralized and widely accessible, meaning the hardware requirements to run a prover will be as low as possible.

#2 

Another alternative for Hyperchains is fractal scaling, i.e. L3s and above. These will also be part of the ecosystem and will have interoperability with other Hyperchains. L3s settling on the same L2 will have faster messaging between each other and will have cheap atomicity via transactions forced through the L2.

### Sovereignty

All Hyperchains will be sovereign in the ecosystem. This means two things.

First the L3s and above will be able to exit their L2, and either be L2s in their own right or become L3s on another L2.

Second, Hyperchains will be able to permissionlessly join and exit the ecosystem, adding or removing all their assets to the common pool. 

Joining is self-explanatory, everyone will have the right to boot up new Hyperchains and join the ecosystem in a Chain Factory contract. Exiting will usually not be a similarly wise decision, as interoperability will be lost with other Hyperchains. However, the ecosystem could sometimes upgrade due to governance, and in this case, it is imperative that each Hyperchain have the right to rage quit. In this case, there will be a mandatory upgrade period during which the Hyperchain that disagrees with the upgrade can exit in a coordinated fashion.

![Untitled](images/zksync_l3_exploration/p3.png)

## **Modularity: Hyperchain Customization**

### **Sequencing transactions**

1. Centralized sequencer: A single trusted operator accepts transactions for low-latency confirmation (<100ms).
2. Decentralized sequencer: A consensus algorithm coordinated by a Hyperchain is used for transaction inclusion.
    
    We can take advantage of the fact that finality checkpoints are guaranteed by the underlying L1, and implement an algorithm that is simpler and boasts higher performance.
    
3. Priority queue: This simply means absence of any sequencer. Transactions can be submitted via a queue from an underlying L2 or even L1 chain, providing stronger censorship resistance.

### Privacy

1. **Validium**. For a Hyperchain running in the validium mode, privacy to the outer world is achieved out of the box as long as the operator keeps the block data secret. This might be an interesting option for enterprise users.
2. **Privacy protocols**. To implement user-level privacy, a specialized L3 protocol is required. Projects such as Aztec or Tornado can be implemented either directly on the one or more hyperchains (taking advantage of account abstraction and cheap recursive ZKP verification on zkSync), or they can opt into standalone special-purpose Hyperchains for more flexibility.
3. **Self-hosted rollups,** based on user-maintained data availability and self-proved off-chain state transitions, will offer ultimate privacy and unlimited scalability in the long term.

### **Data availability**

1. **zkRollup**
2. **zkPorter**
    
    The data availability of zkPorter accounts will be secured by zkSync token holders, termed Guardians. They will keep track of state on the zkPorter side by signing blocks to confirm data availability of zkPorter accounts. Guardians participate in proof of stake (PoS) with the zkSync token, so any failure of data availability will cause them to get slashed. This gives cryptoeconomic guarantees of the data availability.
    
3. **Validium**
4. **zkRollup (inputs only)**
5. **zkRollup (self-hosted)**

In zkSync 2.0, the L2 state will be divided into 2 sides: zkRollup with on-chain data availability and zkPorter with off-chain data availability.

Both parts will be composable and interoperable: contracts and accounts on the zkRollup side will be able to seamlessly interact with accounts on the zkPorter side, and vice versa. That’s right! From a user’s perspective, the only perceptible difference will be a 100x reduction in fees for zkPorter accounts.

It is important to note that PoS in zkSync is significantly more secure than PoS in other systems such as sidechains. This is because zkSync guardians are essentially powerless: guardians cannot steal funds. They can only freeze the zkPorter state (freezing their own stake).

![Untitled](images/zksync_l3_exploration/p4.png)

## Ref

[Security | zkSync Documentation](https://docs.zksync.io/userdocs/security/#security-overview)

[zksync/docs/protocol.md at f59e154865374bdc0f5ded2e2604dac599cb75ee · matter-labs/zksync](https://github.com/matter-labs/zksync/blob/f59e154865374bdc0f5ded2e2604dac599cb75ee/docs/protocol.md)

[](https://blog.matter-labs.io/zkporter-a-breakthrough-in-l2-scaling-ed5e48842fbf)