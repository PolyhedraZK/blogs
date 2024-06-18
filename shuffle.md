Succinct proof of consensus
---

Within the Ethereum beacon chain, previously referred to as Ethereum 2.0, a Proof of Stake (PoS) mechanism is employed for consensus. Our overarching objective is to validate each block. This tool finds its applications in [zero-knowledge light clients](https://docs.zkbridge.com/zklightclient-overview/introduction). Specifically, a proof of consensus can be:
- Either a public randomness (upon which [a committee is selected](https://hackmd.io/@benjaminion/shuffling)), accompanied by a set of signatures from the committee members endorsing the block;
- Or a zero-knowledge proof that validates the aforementioned statement.

The second method is pivotal as it necessitates significantly less storage and communication ($O(1)$ vs $O(N)$ for a committee of $N$ validators), can be verified more efficiently, and, crucially, on-chain by a smart contract. This last feature is particularly beneficial as it further facilitates zero-knowledge, decentralized bridges with atomic swaps.

We've coined this solution as __Concise Proof of Consensus__.


# Task break down



Remember that to reach a consensus, a subset of validators ($n$) is selected from the entire validator set $N$. Each validator then casts their vote for a specific block. Once a quorum is achieved, the validators reach a consensus on the given block.

Let's tackle the simpler component first. Each validator signs their vote using the [BLS signature](https://eth2book.info/capella/part2/building_blocks/signatures/), an aggregatable signature scheme. For this scheme, a set of $k$ signatures (usually $k\geq n$ as there are duplicated signatures), all signing the same message, can be batch verified with approximately $o(k)$ hashes, $o(k)$ group operations, and two pairing operations. Standard methods exist for generating circuits for zero-knowledge proofs for these operations. For instance, refer to the [example code](https://github.com/PolyhedraZK/ExpanderCompilerCollection/tree/master/examples) in our Expander proof system.


To reach a consensus on a specific block, a subset of validators ($n$) is selected from the entire validator set ($N$). Each validator then casts a vote for this block. Once a quorum is achieved, the validators reach a consensus on the block.

Let's first address the simpler component. Each validator signs their vote using the [BLS signature](https://eth2book.info/capella/part2/building_blocks/signatures/), an aggregatable signature scheme. In this scheme, a set of $k$ signatures (where $k\geq n$ due to potential duplicate signatures), all endorsing the same message, can be batch verified with approximately $o(k)$ hashes, $o(k)$ group operations, and two pairing operations. Standard methods for generating circuits for zero-knowledge proofs for these operations exist. For instance, refer to the [tutorial code](https://github.com/PolyhedraZK/ExpanderCompilerCollection/tree/master/examples) in our Expander proof system.

The more complex component is to prove that the subset is derived correctly from the public randomness. This was done via a shuffling algorithm, depicted as below

$$
\forall i \in [0, N), \quad\quad\texttt{shuffled\_indices[i]} = \sigma( \texttt{seed}_{epoch},\quad indices[i])
$$