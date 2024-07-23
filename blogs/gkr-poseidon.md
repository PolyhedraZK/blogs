GKRed Poseidon
------

Polyhedra is proud to introduce the GKRed Poseidon: a zero-knowledge proof system optimized for the Poseidon hash function. Leveraging our Expander prover, we can now validate 1 million Poseidon hashes within just 1 second, doubling the efficiency of the current state-of-the-art.

# Stateless Ethereum Clients

Stateless Ethereum clients are designed to validate incoming blocks without needing to store the entire state database. These clients rely on proofs for state validity instead of maintaining a local copy of Ethereum's state. The traditional database, the Merkle-Patricia tree, demands large proofs for each state, rendering it inefficient for such applications. The Verkle tree approach, outlined in the Ethereum roadmap, is anticipated to be a more suitable solution. Recently, Vitalik Buterin proposed the Starked Poseidon approach, which promises even better performance than the Verkle tree method.

The Starked Poseidon approach introduces significant innovations:
1. It replaces the Merkle-Patricia tree's hash functions with Poseidon hash functions, operating over a small field, specifically, a 31-bit Mersenne prime (referred to M31).
2. It employs a Stark protocol to produce a succinct proof, verifying the correctness of all Merkle-Patricia Tree (MPT) openings.

In a recent announcement, Starkware announced that their STWO system achieved a remarkable 620k hashes per second for the Poseidon hash over M31, using a MacBook Pro M3 Max laptop.

Embracing this methodology, we transitioned to our Expander proof system, a comprehensive proving toolchain for the GKR proof system, compatible with Gnark and Circom frontend. Utilizing the same hash function, we successfully demonstrated the capability to prove 1 million Poseidon hashes on the same hardware platform.

| CPU           | Threads | STWO (kH/s) | Expander (kH/s) | Improvement (%) |
|---------------|---------|-------------|-----------------|-----------------|
|MBP M3 Max     | 1       |             |                 |                 |
|MBP M3 Max     | 16      | 620         |                 |                 |
|AMD 7950x3D    | 1       |             |                 |                 |
|AMD 7950x3D    | 16      |             |                 |                 |

# Our Technique
We presume a certain level of familiarity with the GKR proof system. At its core, GKR validates what is known as a layered circuit, where each layer's cells are computed from the preceding layer's cells. The integrity of such computations is ensured through a sumcheck protocol. For an in-depth exploration of our GKR prover, refer to [GKR By Hand](https://github.com/PolyhedraZK/blogs/blob/gkr-poseidon/blogs/gkr-by-hand.md).

## Vanilla Construction for Poseidon Hash

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

## Tailored linear GKR prover for Poseidon
In the standard GKR prover framework, the computation for two consecutive layers involves Add and Mul gates, defined as:

$$
\tilde{V}\_{i}(z) = \sum_{(\omega_1, \omega_2)\in \\{0,1\\}^{2n}} \left(
\tilde{Add}(z, \omega\_1, \omega\_2)(\tilde{V}\_{i+1}(\omega_1) + \tilde{V}\_{i+1}(\omega\_2)) + \tilde{Mul}(z, \omega\_1, \omega\_2)\tilde{V}_{i+1}(\omega\_1) \tilde{V}\_{i+1}(\omega\_2) \right)
$$

This operation spans over $\\{0,1\\}^{2n}$, leading to a quadratic computational cost of $O(2^{2n})$ for a prover with circuit size $2^n$. However, in our linear GKR construction, as detailed in [Libra](https://eprint.iacr.org/2019/317.pdf), we demonstrate a method to reduce this cost to linear. This is achieved by decomposing the gate function into two phases, each governed by a sumcheck protocol over $n$ variables.


It's important to note that the round keys and the MDS matrix are constant inputs to the circuit. The additions and multiplications by constants are nearly cost-free in a layered circuit. The primary computational bottleneck is the S-box operation, which requires 3 consecutive layers. Consequently, the circuit for each Poseidon iteration comprises $(4 + 14 + 4) \times 3 + 2 = 68$ layers, where additional 2 layers come from the input and output layers. Under the vanilla linear GKR protocol, this equates to $68 \times 2 = 136$ sumchecks, as each layer necessitates $2$ sumchecks.

For the Poseidon hash circuit, which is the focus of our discussion, the requirement simplifies as it does not necessitate Add or Mul gates but solely a power 5 gate. This simplification is expressed as:

$$
\tilde{V}\_{i}(z) = \sum\_{\omega\in \{0,1\}^{n}} \left(
\tilde{Pow5}(z, \omega)\tilde{V}\_{i+1}(\omega) \right)
$$

This adjustment yields significant optimizations:
- Each partial or full round of the Poseidon hash requires only a single GKR layer.
- Each GKR layer mandates merely one sumcheck protocol over $n$ variables.

Consequently, this approach drastically reduces the number of required sumcheck protocols from 136 to 24, albeit at the cost of employing a marginally more complex gate function. This optimization not only enhances computational efficiency but also streamlines the prover's operation for the Poseidon hash function within the GKR framework.


## Implementation

We implement the discussed optimization using our proving stack, which consists of a GKR prover named [Expander](https://github.com/PolyhedraZK/Expander-rs), and a compiler named [Expander Compiler Collection](https://github.com/PolyhedraZK/ExpanderCompilerCollection). This compiler is designed to convert Gnark and Circom's R1CS circuits into layered circuits, facilitating efficient GKR proof generation. Our implementation is fully open-sourced, promoting transparency and community collaboration.

For those interested in exploring our approach further, sample code for generating the Poseidon circuit and corresponding benchmarks are available:
- Poseidon circuit generation: [Expander Compiler Collection Example](https://github.com/PolyhedraZK/ExpanderCompilerCollection/blob/master/examples/poseidon_m31/main.go)
- Benchmarking: [Expander-rs Main](https://github.com/PolyhedraZK/Expander-rs/blob/main/src/main.rs)
