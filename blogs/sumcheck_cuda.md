# Accelerating Ethereum's Verge: Harnessing GPU Power for Zero-Knowledge Proofs

## Introduction

Ethereum, the world's leading smart contract platform, is on the cusp of a revolutionary upgrade known as [The Verge](https://ethroadmap.com/verge.html). This upgrade aims to optimize storage and reduce node sizes through the implementation of [verkle trees](https://ethereum.org/en/roadmap/verkle-trees/), a more efficient data structure than the currently used Merkle trees. As of early 2024, The Verge remains in development, forming a crucial part of Ethereum's ambitious roadmap alongside other upgrades like The Surge and The Purge.

In this article, we delve into groundbreaking research on GPU acceleration of zero-knowledge proofs, a development that could significantly advance two critical areas of Ethereum's Verge: enhancing [light client](https://ethereum.org/en/developers/docs/nodes-and-clients/light-clients/) security and enabling efficient [stateless clients](https://ethereum.org/en/roadmap/statelessness/).

Our code is open source and readily available for the community to explore and contribute. We invite you to check out our [Expander repository](https://github.com/PolyhedraZK/Expander) for the core proving system and our [CUDA repository](https://github.com/PolyhedraZK/Sumcheck-cuda) for GPU-accelerated Sumcheck implementations. We welcome collaboration and feedback from the community. For more ways to get involved or to reach out to our team, please refer to the contact information provided at the bottom of this blog post.

## The Verge: A Primer

Before we dive into our research, let's briefly explore what The Verge means for Ethereum:

- **Purpose**: To enhance Ethereum's scalability by making it easier to run nodes.
- **Key Feature**: Implementation of verkle trees.
- **Expected Outcomes**: 
  - Faster network synchronization
  - Reduced storage requirements
  - Improved overall network efficiency

## Critical Areas of Ethereum Research

Our work focuses on two key areas that are crucial for Ethereum's evolution:

### 1. More Secure Light Clients

- **Current Challenge**: Light clients face security trade-offs due to their reliance on a small sync committee. Currently, the sync committee, responsible for securing consensus for Ethereum's entire $290 billion market cap, only requires 16,384 ETH (approximately $40 million) in staked assets.

- **Our Approach**: Harness the power of zero-knowledge proofs to efficiently verify the entire Ethereum consensus, which primarily consists of 90 million SHA2-256 hashes and 2048 aggregated BLS signatures. This innovative method could enable light clients to validate consensus from a substantially larger set of validators, eliminating the need to rely solely on the limited sync committee.

- **Potential Impact**: Significantly enhanced security for light clients without compromising resource efficiency.

### 2. Efficient Stateless Clients

- **Current Challenge**: Light clients struggle to validate state transitions independently, as they lack a complete local copy of the Ethereum state. This forces them to rely on RPC nodes or full nodes for blockchain interaction, introducing unwanted trust dependencies and potential security vulnerabilities.

- **Our Approach**: Develop a system where each state is accompanied by a succinct cryptographic proof, verifying its validity without requiring the full state history. This innovative method necessitates proving a substantial number of hash computations—approximately 500,000 Keccak hashes per block—but offers a breakthrough in efficient state verification.

- **Potential Impact**: Progress towards truly stateless clients, reducing storage requirements while maintaining or improving Ethereum's existing cryptographic assumptions.

### Status Quo

Zero-knowledge proof generation acceleration is crucial for our challenges. Our [Expander proving system](https://medium.com/polyhedra-network/introducing-expander-the-fastest-gkr-proof-system-to-date-bdd07d05c23e), based on the GKR protocol, has made significant strides, achieving [2 million Poseidon hashes per second](https://medium.com/polyhedra-network/expander-still-the-worlds-fastest-zk-prover-6aa5fd428609) on a $500 consumer CPU. However, this speed is insufficient for the scale of Pairing, SHA2-256, and Keccak operations we're tackling within our time constraints. Thus, we're turning to GPU acceleration. 

The GKR protocol's core component, the Sumcheck protocol, is inherently GPU-friendly and parallelizable. Our work will focus on GPU-accelerating Sumcheck protocols, leveraging massive parallelism to potentially revolutionize zero-knowledge systems' efficiency across blockchain applications. This approach aims to achieve the speed and scalability necessary for our ambitious zero-knowledge proof goals.

## Harnessing GPU Power: Our Breakthrough

Our research harnesses the parallel processing capabilities of GPUs to optimize key components of zero-knowledge proof systems, focusing on accelerating the Sumcheck protocol.

### Promising Results

We've achieved significant improvements in performance across various GPU platforms and fields:

| Num Variables | Field | CPU | NVIDIA 4090 | NVIDIA H100 | Improvement |
|:-------------:|:-----:|:-----------:|:-----------:|:-----------:|:-----------:|
| 23 | Mersenne Ext3 |  0.95 s   | 2.9497 ms  | 2.6267 ms | 362 x |
| 23 | Bn254         | 18.75 s   | 10.7427 ms | 9.3293 ms | 2009 x|
| 27 | Mersenne Ext3 | 15.08 s   | 41.04 ms   | 16.40 ms  | 919 x |
| 27 | Bn254         | 300.6 s   | 75.9ms     | 114.63 ms | 2622 x|
| 29 | Mersenne Ext3 | 60.66 s   | OOM        | 59.52 ms  | 1019 x|
| 29 | Bn254         | 1277.4 s  | OOM        | 451.9 ms  | 2826 x|

*Note: OOM means out of memory; CPU is single-thread implementation running on Intel(R) Xeon(R) Platinum 8460Y+*

*Note: The absolute time here is two-round sumcheck for both X and Y described in Libra. A fair comparison to single-round Sumcheck should be careful and at least divided by two.*

### Technical Deep Dive

Our GPU acceleration techniques focus on optimizing the Sumcheck protocol, a critical component of many zero-knowledge proof systems. Key aspects of our implementation include:

1. **Field-Specific Optimizations**: Tailored algorithms for both Mersenne Ext3 and Bn254 fields.
2. **GPU Kernel Optimization**: Custom kernels for efficient field operations, including pairing and hash computations.
3. **Memory Management**: Careful data layout and transfer strategies to maximize GPU memory bandwidth.
4. **Parallel Processing**: Leveraging GPU architecture for massive parallelism in proof generation.
5. **Platform-Specific Tuning**: Tailored optimizations for different GPU models (AMD, NVIDIA consumer, and NVIDIA data center GPUs).

### Detailed Insights from Our Experiment

We experimented with implementing M31ext3 Sumcheck on Triton and later extended it with a CUDA-based implementation for a full Sumcheck, including Fiat-Shamir:

- **Takeaway**: Sumcheck is memory-bound. 
- **64-round, 2^28-size Sumcheck with Triton**: 46 ms per round, 3 seconds total.
- **Complete CUDA-based Sumcheck**: Including Fiat-Shamir, 30 ms per round.

In accelerating Sumcheck using CUDA for Linear GKR (a crucial operation in proving systems), our experiment showed that in the CPU implementation, Sumcheck occupied 50% of the time:

```
-------------------------------------------
		* CPU Prover *
-------------------------------------------
    - phase 1: two eq evals 	4.0300	s
    - phase 1: vec addition 	0.2990	s
    - phase 1: build gx(mult) 	2.8000	s
    - phase 1: build gx(add) 	1.4930	s
    - phase 2: build hy(mult) 	6.2640	s
Total LinearGKR Prepare:	15.3920	s
-------------------------------------------
Total CPU <> CUDA (PCIe):	0.0000	ms
-------------------------------------------
    - PolyEval:  		8445.4870	ms
    - Fiat-Shamir:  		0.0627	ms
    - Challenge:  		6638.2030	ms
Total Sum-check:  		15083.7529	ms
-------------------------------------------
Input size: 2 ^ 27, Output size: 2 ^ 27
Field type: M31 extension-3
Prove time: 30.4760 seconds
Proof size: 1968 bytes

-------------------------------------------
		* Verifier *
-------------------------------------------
Verify pass = 1
```

By leveraging GPU, the time spent on Sumcheck decreased to about 1%, and this is the corresponding GPU performance:

```
-------------------------------------------
		* GPU Prover *
-------------------------------------------
    - phase 1: two eq evals 	2.3330	s
    - phase 1: vec addition 	0.4960	s
    - phase 1: build gx(mult) 	1.0710	s
    - phase 1: build gx(add) 	0.5210	s
    - phase 2: build hy(mult) 	2.1840	s
Total LinearGKR Prepare:	7.1190	s
-------------------------------------------
Total CPU <> CUDA (PCIe):	737.1550	ms
-------------------------------------------
    - PolyEval:  		9.4830	ms
    - Fiat-Shamir:  		0.0214	ms
    - Challenge:  		6.8960	ms
Total Sum-check:  		16.4004 ms
-------------------------------------------
Input size: 2 ^ 27, Output size: 2 ^ 27
Field type: M31 extension-3
Prove time: 7.8790 seconds
Proof size: 1968 bytes

-------------------------------------------
		* Verifier *
-------------------------------------------
Verify pass = 1
```

## Impact of Machine Combinations on Sumcheck Acceleration:

- **3090 vs 5950X**: M31 max speedup 104x, M31ext3 max speedup 292x, BN254 max speedup 951x
- **4090 vs W7-3465X**: M31 max speedup 105x, M31ext3 max speedup 292x, BN254 max speedup 1565x
- **H100 vs 8460Y+**: M31 max speedup 329x, M31ext3 max speedup 1045x, BN254 max speedup 2826x

## Field Type Comparison:

On the H100 GPU, performance with input size 2^29:

- M31: 23 ms
- M31ext3: 59 ms
- BN254: 451 ms

These results indicate that __Sumcheck’s performance bottleneck is memory bandwidth__, which can be confirmed through NVIDIA's ncu profiler.

## Deep Dive of Technical Insights

The experiments we've conducted highlight several important factors that influence the effectiveness of GPU acceleration in the Sumcheck protocol. Let's dive deeper into some critical aspects:

### 1. **Memory Bandwidth as the Bottleneck**

Through extensive profiling, it became clear that **memory bandwidth** is the primary bottleneck in Sumcheck computations, especially when working with larger field types like BN254. 

- For smaller field types such as **M31**, the GPU's core processing power plays a relatively bigger role, and the memory bandwidth doesn't saturate as quickly. This is evident from the fact that for input sizes of 2^27, the performance of M31 on different GPUs (such as 3090 and 4090) remains almost the same, despite the difference in core frequency.
  
- In contrast, for **BN254**, which requires more memory to store intermediate field values and state, the difference in performance between GPUs with varying memory bandwidth is much more pronounced. For example, moving from 3090 (936 GB/s) to 4090 (1008 GB/s) gives a 70% speedup in BN254 computation, as 4090’s slightly higher bandwidth helps it handle the memory load more efficiently.

Thus, **bandwidth saturation** explains why field types with higher memory requirements (like BN254) benefit significantly more from GPUs with higher memory bandwidth. This was confirmed using NVIDIA's ncu profiler, which showed that the memory throughput approached the theoretical limit of the device during Sumcheck operations.

### 2. **Field Size and Sumcheck Complexity**

Another key factor is the **size of the field** being used in the computations. Larger field sizes increase the complexity of arithmetic operations, leading to more intensive computation and memory access patterns. This results in a steeper performance drop for larger fields as the input size grows.

For example:

- **M31** (a smaller field) is relatively lightweight and demonstrates good scaling as the input size increases. This is why we observe much faster execution times (23 ms for input size 2^29) compared to BN254.
  
- **BN254** is a larger field, and its arithmetic operations are significantly more expensive, resulting in longer execution times (451 ms for input size 2^29). However, despite the heavier computational burden, BN254 benefits the most from GPUs with higher memory bandwidth and core frequency, making it an ideal candidate for GPU acceleration.

### 3. **When Does GPU Acceleration Become Effective?**

As mentioned previously, GPU acceleration becomes more effective as the input size increases, but it is also heavily influenced by the field size. Our data shows that:

- For **M31**, the GPU starts to outperform the CPU at around 2^15 inputs.
- For **M31ext3**, the crossover point is at about 2^12 inputs.
- For **BN254**, the GPU begins to shine as early as 2^7 inputs.

This analysis indicates that larger fields not only benefit more from GPU acceleration but also reach the point where GPU acceleration is advantageous much earlier, making them ideal candidates for zero-knowledge proof systems that rely on large field operations.

### 4. **Impact of Core Frequency**

Although **memory bandwidth** is the dominant factor, **GPU core frequency** still has an impact, particularly in larger field operations like BN254. For instance, in our tests:

- For **M31** and **M31ext3**, the performance of Sumcheck between 3090 and 4090 is nearly identical because the memory bandwidth is the limiting factor.
- However, for **BN254**, the higher core frequency of the 4090 (2.52 GHz compared to the 3090’s 1.70 GHz) provides a noticeable improvement, as the arithmetic operations in BN254 are more complex and benefit from faster core processing.

This is why, despite the main bottleneck being memory bandwidth, **core frequency** can still play a critical role in improving performance, especially for fields with more computational overhead.

### 5. **PCIe Transfer Overhead**

A small, but important, aspect of performance optimization is the **PCIe transfer time** between the CPU and GPU. In our experiments:

- For the **CPU-only prover**, there is no PCIe overhead.
- For the **GPU-accelerated prover**, we measured a PCIe transfer overhead of about 737 ms when moving data back and forth between the CPU and GPU.

This overhead is relatively small compared to the time saved by accelerating the core Sumcheck computation on the GPU, but it is still a factor that needs to be considered in end-to-end optimization, especially as we integrate more components of the Linear GKR protocol into the GPU. After we move all components of Linear GKR to GPU, the PCIe transfer time will be minimum and only need to be done once.

## Analysis and Key Takeaways

From our experiments and technical profiling, we can draw several key conclusions about the impact of GPU acceleration on Sumcheck and, by extension, zero-knowledge proof systems:

1. **Memory Bandwidth Dominates**: For most field types, particularly larger ones like BN254, memory bandwidth is the primary factor determining performance. The faster the memory, the better the GPU performs, especially for larger input sizes.

2. **Larger Fields Benefit More**: Field types that require more computational resources (like BN254) gain more from GPU acceleration than smaller fields. This suggests that zero-knowledge proof systems operating in these fields can achieve significant speedups by leveraging high-bandwidth GPUs.

3. **Core Frequency Matters for Complex Fields**: While memory bandwidth is critical, higher GPU core frequencies provide additional benefits for complex field operations, especially in BN254, where arithmetic operations are more intensive.

4. **Early Crossover for Large Fields**: Large field types, such as BN254, begin to show significant performance improvements with GPU acceleration at much smaller input sizes compared to smaller fields like M31. This means that for many practical applications involving large field operations, GPU acceleration will be beneficial even for relatively small problem sizes.

5. **End-to-End Optimization Required**: While Sumcheck has shown substantial performance gains on the GPU, achieving true end-to-end acceleration will require further optimization of other components of the proof system that may not be as GPU-friendly.

## Future Work

While we've made significant progress in accelerating the Sumcheck protocol, which forms the majority of computations in the GKR prover, there's still work to be done:

1. **End-to-End Optimization**: Accelerating remaining operations that may be less hardware-friendly.
2. **Cross-Platform Performance**: Further tuning for a wider range of GPU architectures.
3. **Integration with Ethereum Clients**: Collaborating with client teams to implement these optimizations.

## Get Involved

We invite the community to engage with our research:

1. **Open Source Contributions**: Check out our [Expander repository](https://github.com/PolyhedraZK/Expander) and [CUDA repository](https://github.com/PolyhedraZK/Sumcheck-cuda) for implementation details and contribute to the project.
2. **Research Collaboration**: We are organizing research seminars to promote Sumcheck protocol and related prove systems.
3. **Stay Updated**: Follow our blog and social media channels for the latest developments and breakthroughs.

Together, we can accelerate the future of Ethereum and decentralized technologies!
