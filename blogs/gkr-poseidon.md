GKRed Poseidon and beyond
------

Polyhedra is proud to introduce the GKRed Poseidon: a zero-knowledge proof system optimized for the Poseidon hash function. Leveraging our [Expander prover](https://github.com/PolyhedraZK/Expander-rs), we can now validate ~1 million~ Poseidon hashes within just 1 second, ~doubling~ the efficiency of the current state-of-the-art.

# Stateless Ethereum Clients

[Stateless Ethereum clients](https://consensys.io/blog/modelling-stateless-ethereum-a-journey-into-the-unknown) are designed to validate incoming blocks without needing to store the entire state database. These clients rely on proofs for state validity instead of maintaining a local copy of Ethereum's state. The traditional database, the [Merkle-Patricia tree](https://ethereum.org/en/developers/docs/data-structures-and-encoding/patricia-merkle-trie/), demands large proofs for each state, rendering it inefficient for such applications. The [Verkle tree](https://ethereum.org/en/roadmap/verkle-trees/) approach, outlined in the [Ethereum roadmap](https://x.com/VitalikButerin/status/1741190491578810445) as "the Verge", is anticipated to be a more suitable solution. Recently, Vitalik Buterin proposed the "Starked Poseidon" approach, which promises even better performance than the Verkle tree method.

The Starked Poseidon approach introduces significant innovations:
1. It replaces the Merkle-Patricia tree's hash functions with a circuit friendly hash function known as the Poseidon hash functions, operating over a small field, specifically, a 31-bit Mersenne prime (referred to M31).
2. It employs a [STARK protocol](https://eprint.iacr.org/2018/046) to produce a succinct proof, verifying the correctness of all Merkle-Patricia Tree (MPT) openings.

In [a recent announcement](https://starkware.co/blog/starkware-new-proving-record/), Starkware announced that their STWO system achieved a remarkable 600k hashes per second for the Poseidon hash over M31, using a MacBook Pro M3 Max laptop.

Embracing this methodology, we transitioned to our Expander proof system, a comprehensive proving stack for the GKR proof system, compatible with Gnark and Circom frontend. Utilizing the same hash function, we successfully demonstrated the capability to prove ~1 million~ Poseidon hashes on the same hardware platform.

| CPU           | Threads | STWO (kH/s) | Expander (kH/s) | Improvement (%) |
|---------------|---------|-------------|-----------------|-----------------|
| MBP M3 Max    | 1       |             |                 |                 |
| MBP M3 Max    | 16      |             |                 |                 |
| AMD 7950x3D   | 1       |             |                 |                 |
| AMD 7950x3D   | 16      |             |                 |                 |

# Our Technique
We presume a certain level of familiarity with the GKR proof system. At its core, GKR validates what is known as a layered circuit, where each layer's cells are computed from the preceding layer's cells. The integrity of such computations is ensured through a Sumcheck protocol. For an in-depth exploration of our GKR prover, refer to our [GKR By Hand](https://github.com/PolyhedraZK/blogs/blob/gkr-poseidon/blogs/gkr-by-hand.md) tutorial.

## Poseidon Hash Explained

We illustrate the process using the Poseidon hash over M31. The hash function follows these steps:
- **Initial State:** Start with 16 M31 field elements derived from the (padded) input. 
- **Full Rounds (x4):**
    - Add the round key to the state.
    - Apply the MDS matrix to the state.
    - Apply the S-box by raising each state element to the power of 5.
- **Partial Rounds (x14):**
    - Add the round key to the state.
    - Apply the MDS matrix to the state.
    - Apply the S-box by raising the first state element to the power of 5.
- **Full Rounds (x4):**
    - Repeat the steps in the full rounds section.
- **Output:** The first 8 elements of the state are the output of the hash function.

GKR proves to be highly efficient for validating Poseidon hashes. It operates in linear time relative to the circuit size, $O(N)$, outperforming mainstream provers like Plonk and Groth16, which run in at least $O(N\log N)$. This efficiency is particularly beneficial for functions like Poseidon, characterized by small input/output sizes but requiring substantial intermediate computations. Unlike Plonk and Groth16, where the prover must commit to all intermediate witnesses, GKR necessitates commitments only for the inputs and outputs, leveraging Sumcheck protocols to manage intermediate data. For applications where input/output privacy isn't a concern, such as GKRed Poseidon for Merkle Patricia Trees (MPT), commitments can be bypassed entirely, allowing direct transmission of inputs and outputs to the verifier.

## Linear GKR prover for Poseidon
In the standard GKR prover framework, the computation for two consecutive layers involves Add and Mul gates, defined as:

$$
\tilde{V}\_{i}(z) = \sum_{(\omega_1, \omega_2)\in \\{0,1\\}^{2n}} \left(
\tilde{Add}(z, \omega\_1, \omega\_2)(\tilde{V}\_{i+1}(\omega_1) + \tilde{V}\_{i+1}(\omega\_2)) + \tilde{Mul}(z, \omega\_1, \omega\_2)\tilde{V}_{i+1}(\omega\_1) \tilde{V}\_{i+1}(\omega\_2) \right)
$$

This operation spans over $\\{0,1\\}^{2n}$, leading to a quadratic computational cost of $O(2^{2n})$ for a prover with circuit size $2^n$. However, in our linear GKR construction, as detailed in [Libra](https://eprint.iacr.org/2019/317.pdf), we demonstrate a method to reduce this cost to linear. This is achieved by decomposing the gate function into two phases, each governed by a Sumcheck protocol over $n$ variables.


It's important to note that the round keys and the MDS matrix are constant inputs to the circuit. The additions and multiplications by constants are nearly cost-free in a layered circuit. The primary computational bottleneck is the S-box operation, which requires 3 consecutive layers. Consequently, the circuit for each Poseidon iteration comprises $(4 + 14 + 4) \times 3 + 2 = 68$ layers, where additional 2 layers come from the input and output layers. Under the vanilla linear GKR protocol, this equates to $68 \times 2 = 136$ Sumchecks, as each layer necessitates $2$ Sumchecks.

## New result: customized GKR for Poseidon

For the Poseidon hash circuit, which is the focus of our discussion, the requirement simplifies as it does not necessitate Add or Mul gates but solely a power 5 gate. This simplification is expressed as:

$$
\tilde{V}\_{i}(z) = \sum\_{\omega\in \\{0,1\\}^{n}} \left(
\tilde{Add}(z, \omega)\tilde{V}\_{i+1}(\omega) +
\tilde{Pow5}(z, \omega)\tilde{V}\_{i+1}(\omega) \right)
$$

This adjustment yields significant optimizations:
- Each partial or full round of the Poseidon hash requires only a single GKR layer.
- Each GKR layer mandates merely one Sumcheck protocol over $n$ variables.
- We have eliminated the need for claim aggregation in GKR, as we now utilize only one polynomial in each Sumcheck.

Consequently, this approach drastically reduces the number of required Sumcheck protocols from 136 to 24, albeit at the cost of employing a marginally more complex gate function. This optimization not only enhances computational efficiency but also streamlines the prover's operation for the Poseidon hash function within the GKR framework.

## Customized GKR beyond Poseidon

Our customized GKR framework extends its utility beyond the Poseidon hash function, incorporating a square gate at minimal cost. The equation below illustrates this capability:

$$
\tilde{V}\_{i}(z) = \sum\_{\omega\in \\{0,1\\}^{n}} \left(\tilde{Add}(z, \omega)\tilde{V}\_{i+1}(\omega) +
\tilde{Pow5}(z, \omega)\tilde{V}\_{i+1}(\omega)  + \tilde{Sqr}(z, \omega)\tilde{V}\_{i+1}(\omega)\right)
$$

This gate's completeness is notable, as it enables the derivation of multiplication gates. For instance, to calculate $a \times b$, one can compute $((a+b)^2 - a^2 - b^2) \times 2^{-1}$. The multiplication by the inverse of 2, being a constant multiplication, incurs no additional cost. Consequently, the product of two variables necessitates two layers, with each layer's computational expense halved compared to the Libra framework. This approach maintains the same cost efficiency as linear GKR for general circuits while offering enhanced efficiency for circuits like Poseidon.

## Implementation and open source

We implement the discussed optimization using our proving stack, which consists of a GKR prover named [Expander](https://github.com/PolyhedraZK/Expander-rs), and a compiler named [Expander Compiler Collection](https://github.com/PolyhedraZK/ExpanderCompilerCollection). This compiler is designed to convert Gnark and Circom's R1CS circuits into layered circuits, facilitating efficient GKR proof generation. Our implementation is fully open-sourced, promoting transparency and community collaboration.

For those interested in exploring our approach further, sample code for generating the Poseidon circuit and corresponding benchmarks are available:
- Poseidon circuit generation: [Expander Compiler Collection Example](https://github.com/PolyhedraZK/ExpanderCompilerCollection/blob/master/examples/poseidon_m31/main.go)
- Benchmarking: [Expander-rs Main](https://github.com/PolyhedraZK/Expander-rs/blob/main/src/main.rs)
