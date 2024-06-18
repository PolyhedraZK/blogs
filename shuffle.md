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

The more complex component involves proving that the subset is correctly derived from the public randomness. This is achieved via a shuffling algorithm, depicted as follows:

$$
\forall i \in [0, N), \quad\quad\texttt{shuffled\_indices[i]} = \sigma( \texttt{seed},\quad \texttt{indices}[i])
$$

Here, $\texttt{shuffled\_indices}$ collectively form the selected subset of the committee, while the entire $\texttt{indices}$ set represents all committee members. $\texttt{seed}$ is the randomness of the given epoch/slot. The exact value of the seed is computed from the [Randao](https://eth2book.info/capella/part2/building_blocks/randomness/#the-randao) value of the last slot of the epoch two epochs prior. In other words, the slots in the current epoch will have the same Randao value and the same seed. 

Moving forward, our primary focus will be on validating the correctness of the shuffle operation.

# Proof of shuffle operation

## Setup
The state seed $\texttt{seed}$ of a given epoch/slot on the beacon chain, the whole validator sets, and the committees are public inputs to the zero-knowledge circuit. 

## Shuffle algorithm

The algorithm was originally included in [the Spec](https://github.com/ethereum/consensus-specs/blob/v1.3.0/specs/phase0/beacon-chain.md#compute_shuffled_index). For completeness we recall the scheme here.

``` python
def compute_shuffled_index(index: uint64, index_count: uint64, seed: Bytes32) -> uint64:
    """
    Return the shuffled index corresponding to ``seed`` (and ``index_count``).
    """
    assert index < index_count

    # Swap or not (https://link.springer.com/content/pdf/10.1007%2F978-3-642-32009-5_1.pdf)
    # See the 'generalized domain' algorithm on page 3
    for current_round in range(SHUFFLE_ROUND_COUNT):
        pivot = bytes_to_uint64(hash(seed + uint_to_bytes(uint8(current_round)))[0:8]) % index_count
        flip = (pivot + index_count - index) % index_count
        position = max(index, flip)
        source = hash(
            seed
            + uint_to_bytes(uint8(current_round))
            + uint_to_bytes(uint32(position // 256))
        )
        byte = uint8(source[(position % 256) // 8])
        bit = (byte >> (position % 8)) % 2
        index = flip if bit else index

    return index
```

## Challenges
Computing the output of $\sigma( \texttt{seed},\quad \texttt{indices}[i])$ in circuit can be resource-intensive due to the following challenges:
- Hash operations in the circuit are costly. We are required to calculate $SR\times N + SR$ hashes in total, where $SR$ represents the number of shuffle rounds (for instance, $SR = 90$ as defined in Eth2).
- Modular arithmetic and comparison operations in the circuit are also costly. For each index and each shuffle round, we must execute a modular arithmetic operation to determine the `flip` value, then compare `flip` and `index` to select the larger one. This process results in $SR\times N$ modular arithmetic and comparison operations.

## Our solutions

### Improved hash circuits
To address the first challenge, we optimize the number of hash operations by implementing caching and lookup tables to share the cached results within a consecutive number of blocks.

We observe that the inputs to the hash function remain consistent within a consecutive of 256 blocks. Therefore, we compute the hashes in the circuit once when they first show up, and utilize lookup tables to replicate the witnesses when needed.

Remember that the shuffling algorithm uses `position//256` as part of the inputs to calculate a hash for a position, and the remaining part of the inputs stays the same within the same round. In other words, the second hash `source = hash(seed, current round, position//256)` remains the same for every `256` positions within a chunk `[256i, 256i+255]`, where `i` is in `[0, (N-1)//256]`. As a result, we only need to calculate a hash table of size $SR\times ((N+255)//256) + SR$.

The next challenge is sharing the hash output for positions within the same chunk. In a circuit, the positions are witnesses whose value is unknown to the verifier. Witness cannot be used as an index within a table because the circuit would no longer be static, and its shape would leak information about the witnesses. To overcome this, we implement lookup interfaces in the circuit. Typically, we initialize a lookup table with the initial elements (e.g., key-value pair). Whenever a function calls $lookup(key_i)$, a value ($value_i$) is returned and a query $Q_i(key_i , value_i)$ is recorded. Finally, the lookup circuit verifies that all queries are correct, i.e., all queried key-value pairs are in the table.

### Reduce modular arithmetic and comparison operations
Remember that in the shuffle algorithm, we must execute a modular arithmetic operation and compare a `flip` with an `index` for each `index` in every round. This results in `SR \times N` modular arithmetic and comparison operations.

We notice a correlation between a flip and an index:
$$
flip = N + P - index \bmod N
$$

In this context, $P$ represents a `pivot`, calculated as `hash(seed, current round) mod N`. Therefore, $P$ remains constant for all indices within the same round. We can circumvent the use of modular arithmetic by defining the following equations:

$$
flip =\left\{
\begin{array}{ll}
P - index  & index < P \\
N + P - index & index \geq P
\end{array}
\right.                                      
$$

In other words, if the `pivot` $P$ is larger than the index, we calculate `flip` using the first equation. Otherwise, we use the second equation. This approach effectively replaces a modular arithmetic operation with a comparison operation. However, a bit-by-bit comparison operation can still be costly. Our solution is to compute the difference between the two values and check if the difference is zero.

We introduce a range flag $F_0$. If this flag is $false$, the corresponding element will fall within the range $[0, P)$. If $true$, the element will fall within the range $[P, N-1]$. We initialize the flag as $0$. For each index, we perform $f_0 = \texttt{IsZero}(\texttt{Sub}(index, P))$. Once the output $f_0$ is $1$ (true), we flip the range flag from 0 to 1 (i.e., $F_0=\texttt{Xor}(F_0 , f_0)$), indicating the index now falls within the range $[P, N-1]$. This method allows us to avoid using modular arithmetic and comparison operations, at the cost of introducing the $\texttt{IsZero}$ operation, which is significantly cheaper in the circuit.

Extending this idea further,  we find that we can avoid comparison operations for calculating positions, which are the maximum between index and flip.

Since flip is either `P-index` or `N+P-index`, we know that position is either `index`, `P-index`, or `N+P-index`. 

Here, we define four ranges, $[0, P/2)$, $[P/2, P)$, $[P, (N+P)/2)$, $[(N+P)/2, N)$.

$$
P =\left\{
\begin{array}{ll}
index  & index \in [0, P/2)\\
flip  & index \in [P/2, P) \\
index &  index \in [P, (N+P)/2) \\
flip &  index \in [(N+P)/2, N)
\end{array}
\right.                                      
$$

In practice, we introduce another range flag, denoted by $F_1$, which stores a boolean value indicating whether the `pivot` is equal to the `index`. Specifically, for each index, we calculate:

- $f_0 = \texttt{IsZero}(\texttt{Sub}(index, P))$
- $f_1 = \texttt{IsZero}(\texttt{Sub}(index, P/2))$
- $f_2 = \texttt{IsZero}(\texttt{Sub}(index, (N+P)/2))$

Subsequently, we update $F_1 = \texttt{Xor}(F_1,\texttt{Or} (f_0, f_1,f_2))$. With $F_0$ and $F_1$, we can compute `pivot` and `flip` without the need for modular operations.