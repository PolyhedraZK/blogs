
## Welcome to Proof Arena

In recent years, there's been a lot of work on new zero-knowledge (ZK) technologies. We're excited to introduce Proof Arena, a platform for testing ZK algorithms that values fairness and detailed data. We invite everyone to help out by adding new tasks and algorithms. 

With the success of projects like ZCash and zkRollups/zkEVM, many new open-source ZK projects have appeared. However, benchmarking these projects has been a challenge. Some projects share impressive numbers without enough details for others to reproduce the results.

Let's use performance on CPU, GPU, and EVM to find the best algorithms, not just claims on social media with misleading numbers only.

To create a fair benchmarking platform, we need these features:
- **Equity:** All algorithms run on the same type of machine for each task.
- **Accessibility:** Users should easily access and analyze all benchmark data.
- **Expandability:** New algorithms and benchmarks can be added over time.
- **Reliability:** We maintain older machines for long-term data consistency.
- **Modernization:** We provide the latest hardware for benchmarking.

Here's a comparison of different benchmarking methods:

| Method             | Equity | Accessibility | Expandability | Reliability | Modernization |
| ------------------ | ------ | ------------- | ------------- | ----------- | ------------- |
| Social Media       | No     | Yes           | No            | No          | No            |
| GitHub Source Code | Yes    | No            | Yes           | No          | No            |
| Pantheon (Celer)   | Yes    | Yes           | No            | No          | No            |
| Proof Arena        | Yes    | Yes           | Yes           | Yes         | Yes           |

### How It Works

1. **Visit Proof Arena**: Start by visiting our platform at [Proof Arena](https://arena.proof.cloud). Here, you can explore detailed information about our benchmarking process, current challenges, and the community's contributions.

2. **Understand the Benchmarking Process**: Familiarize yourself with our benchmarking process. Our list of challenges is maintained on [GitHub](https://github.com/PolyhedraZK/ProofArena), where you can view existing tasks and see how benchmarks are structured.

3. **Submit Your Prover**: If you have a prover algorithm to contribute, follow these steps:
   - **Prepare Your Prover**: We prefer binary executables that meet our API standards. You can find the API specifications on our [GitHub repository](https://github.com/PolyhedraZK/ProofArena). While submitting source code is optional, it is encouraged. However, submitting your verifier code is mandatory, as all verifier code will be audited.
   - **Submit Your Prover**: Submit your prover via our GitHub repository by creating a pull request. Include all necessary files and documentation as specified in our submission guidelines.

4. **Run Benchmarks**: Once your prover is submitted and accepted, you can run it on the requested tasks. You will have access to our standard Google Cloud machines to ensure consistency and fairness in benchmarking. The detailed specifications of these machines are available for verification.

5. **View Results**: After running your prover, time and memory measurements will be automatically displayed on the [Proof Arena](https://arena.proof.cloud) website. These results are public, allowing the community to see and compare different provers' performance.

6. **Contribute New Challenges**: If you have ideas for new benchmarking challenges, you can submit them through our GitHub repository. Create a pull request with your proposed challenge, including detailed instructions and requirements.

7. **Engage with the Community**: Engage with other contributors and stay updated on the latest developments by joining discussions on our GitHub repository. Share your insights, provide feedback, and collaborate with others to advance zero-knowledge technology.

By following these steps, you can actively participate in the Proof Arena community, contributing to the development and benchmarking of cutting-edge zero-knowledge algorithms.

### Join Us

We invite everyone to contribute their algorithms and new challenge ideas. Visit our platform at [Proof Arena](https://arena.proof.cloud) to learn more about how you can participate. There, you'll find detailed information on our benchmarking process, current challenges, and how to submit your own work. Whether you are a researcher, developer, or enthusiast, your contributions are valuable to us. Together, we can push the boundaries of zero-knowledge technology.

### Thanks

We thank Google Cloud for their support in developing this platform and Lianmin Zheng for inspiring us and giving us valuable suggestions.

### How to Cite Us

If you find our work useful, please cite:
```
@misc{ProofArena,
  title = {Proof Arena},
  year = {2024},
  url = {https://arena.proof.cloud}
}
```
