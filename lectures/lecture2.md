# Sumcheck Protocol

Zero-knowledge proofs (ZKPs) have become a cornerstone of modern cryptography, enabling one party (the prover) to convince another party (the verifier) of the truth of a statement without revealing any information beyond the validity of the statement itself. A critical component in the construction of efficient and scalable ZKPs is the sumcheck protocol. This blog post aims to demystify the sumcheck protocol, outline its operation, and discuss its pivotal role in zero-knowledge proof systems.



## What is the Sumcheck Protocol?

The sumcheck protocol is a powerful interactive proof technique for asserting properties about polynomials. At its core, the protocol allows a prover to convince a verifier that the sum of evaluations of a polynomial over a finite set of points equals a certain value. This protocol is particularly effective in scenarios where directly computing the sum for the verifier would be prohibitively expensive.


Consider the task of verifying a claim about the sum of evaluations of a multilinear polynomial $P(x_1, x_2, ..., x_n)$ over a finite domain $\mathbb{H}$ with elements $\{0, 1\}^n$. A multilinear polynomial is one where each variable appears with at most degree one. The claim is that the sum of all evaluations of $\mathcal{P}$ over the  points in the domain $\mathbb{H}$ equals a specific value $\mathcal{S}$. The formal problem statement is as follows: For a multilinear polynomial $P(x_1, x_2, ..., x_n)$, verifying the claim that
$$\sum_{x_1=0}^{1}\sum_{x_2=0}^{1}...\sum_{x_n=0}^{1}P(x_1,x_2,...,x_n)=\mathcal{S}$$

In a straightforward approach to verifying the claim about the sum of evaluations of a multilinear polynomial $P(x_1,x_2,...,x_n)$, one might consider a naive method wherein the prover is tasked with sending all $2^n$ evaluations of the polynomial to the verifier. This process would entail the verifier meticulously checking each of these evaluations to ascertain the truth of the prover's claim regarding the aggregate sum $\mathcal{S}$. Such an approach, while theoretically sound, is highly impractical and inefficient.



Firstly, the sheer volume of evaluations that must be transmitted from the prover to the verifier becomes exponentially large as $n$ increases, leading to substantial communication overhead. This is not just burdensome in terms of the time and resources required to transmit this information, but it also poses significant challenges in terms of storage and computational resources for the verifier, who must process and verify each evaluation individually.



Furthermore, this method starkly contradicts the principles of zero-knowledge proofs, where the goal is to verify the validity of a claim without revealing any specific information about the underlying data. Transmitting every evaluation of $P(x_1, x_2, ..., x_n)$ directly exposes the function's behavior across its entire domain, thereby forfeiting the zero-knowledge property.



Therefore, while the naive approach provides a brute-force solution to the verification problem, it is neither practical nor desirable within the context of efficient cryptographic verification or the foundational principles of zero-knowledge proofs. It underscores the necessity for more sophisticated methods, such as the sumcheck protocol, which offers a more practical, efficient, and secure means of verification that aligns with the zero-knowledge paradigm.

## Application in Zero-Knowledge Proofs

The sumcheck protocol is instrumental in the design of zero-knowledge proofs for several reasons:

- Efficiency: It transforms the verification of a complex statement into a series of simpler checks, significantly reducing the computational load on the verifier.

- Scalability: It is particularly useful for proving statements about large datasets or complex computations, making it a good fit for blockchain applications and secure computation.

- Privacy: The sumcheck protocol can be adapted to prove the claimed sum without revealing the secret of the polynomials.



Within the rich landscape of cryptographic research, the sumcheck protocol manifests in various forms, each tailored to specific computational and security requirements. Notably, the literature distinguishes between different types of sumcheck protocols, such as the univariate sumcheck, exemplified in the Marlin protocol[1], and the multilinear sumcheck in the Libra protocol[2]. These variants underscore the protocol's versatility and its adaptability to diverse cryptographic scenarios.



The univariate sumcheck protocol, as employed in Marlin[1], is designed to efficiently handle sums over univariate polynomials. This variant is characterized by its quasilinear proving time, which, while slightly more demanding computationally, yields proofs of constant size. Such compact proofs are highly advantageous in scenarios where bandwidth or storage is limited, as they minimize the overhead associated with transmitting and storing proof data. Additionally, the verification process for univariate sumchecks remains remarkably efficient, requiring only a constant amount of computational resources, thereby facilitating rapid and scalable verification even in resource-constrained environments.



In contrast, the multilinear sumcheck protocol, integral to the Libra framework[2], embraces a different trade-off. It is designed to efficiently handle sums over multilinear polynomials, resulting in proofs and verifications that scale logarithmically with the number of variables. This logarithmic scaling is particularly beneficial for applications dealing with high-dimensional data or complex computations, as it ensures that the size of the proofs and the computational load of the verification process increase only modestly as the complexity of the underlying polynomial grows. Consequently, the multilinear sumcheck achieves linear proving time, making it highly efficient for provers and well-suited for applications where proving efficiency is paramount.



Given the focus on optimizing prover efficiency without sacrificing security or verifiability, our discussion in forthcoming blogs will predominantly center on the multilinear sumcheck protocol. This choice is motivated by the multilinear sumcheck's balanced trade-offs between proof size, proving time, and verification complexity, making it an exemplary candidate for cryptographic applications seeking to achieve high efficiency and scalability. By delving deeper into the mechanics and applications of the multilinear sumcheck, we aim to illuminate its role in advancing the state-of-the-art in zero-knowledge proofs and secure computation.



## How Does the Sumcheck Protocol Work for Multilinear Polynomials?

The sumcheck protocol operates in rounds, with each round focusing on a single variable of a multilinear polynomial. The protocol iteratively reduces the problem of verifying the sum over a multilinear polynomial to the problem of verifying the sum over a univariate polynomial and eventually to a constant. Here’s a simplified overview of the process:

- Initialization: The prover wants to demonstrate that the sum of evaluations of a polynomial $P(x_1, x_2, ..., x_n)$ over a certain domain $\mathbb{H}$ equals a specific value $\mathcal{S}$.

- Interactive Rounds:

    - In each round, the prover sends a claim about the sum of polynomial evaluations involving fewer variables.

    - The verifier challenges the prover by choosing a random value for one of the variables and asks the prover to reduce the problem to a polynomial with one fewer variable.

    - This process is repeated for each variable until only a univariate polynomial or a constant remains.

- Final Verification: In the final step, the verifier checks a single evaluation of the polynomial, which is computationally inexpensive. The verifier accepts the proof if the evaluation matches the prover's claim in the final round.



Let's dive deeper into a concrete instance of the sumcheck protocol for a multilinear polynomial $P(x, y)=x+y+xy$, focusing on a finite domain $\mathcal{H}$ with elements $\{0, 1\}^2$. The goal is to verify that the sum equals $4$.

- Initialization: The prover asserts that the total sum $\mathcal{S}=\sum_{x=0}^{1}\sum_{y=0}^{1}P(x, y)$ is equal to $4$, which corresponds to the evaluations at all four points in the domain $\{0, 1\}^2$.

- First Round (Variable $x$): 

    - The prover computes $g(0)=\sum_{y=0}^{1}P(0, y)$ and $g(1)=\sum_{y=0}^{1}P(1, y)$, which represent the sums of evaluations of $P$ for each possible value of $x$, and then sends these sums, $g(0)$ and $g(1)$, to the verifier. The degrees of $g$ correspond to the degrees of the variable $x$ within $P$, which, by the nature of multilinear polynomials, is one. This characteristic ensures that the two evaluations $g(0)$ and $g(1)$ sufficiently commit to the polynomial $g$, providing a comprehensive representation of its behavior with respect to $x$.

    - Upon receiving $g(0)$ and $g(1)$, the verifier then performs a critical check to ensure the integrity of the prover's claim: it verifies whether the sum $g(0)+g(1)$ aligns with the expected total sum of $P$'s evaluations for $x=0$ and $x=1$, which, in this context, is purported to be $4$.  Following this verification, the verifier selects a random value $r_x$ for the variable $x$, transitioning the original claim into a more focused assertion: that the sum $\sum_{y=0}^{1}P(r_x, y)=g(r_x)$. Remarkably, $g(r_x)$ can be independently computed by the verifier using the provided values of $g(0)$ and $g(1)$, leveraging the linear structure of $g$.  The result is a sub-sumcheck claim, now concentrated on verifying the sum of evaluations of $P$ at $r_x$ for the remaining variables.

- Second Round (Variable $y$): 

    - Following the successful reduction in the first round, the sumcheck protocol proceeds to the second round, focusing on the variable $y$. At this juncture, the verifier has already chosen a random value $r_x$ for $x$ and the prover's task is to now substantiate the sum of evaluations of $P(r_x, y)$ for $y=0$ and $y=1$. The prover calculates $h(0)=P(r_x, 0)$ and $h(1)=P(r_x,1)$, effectively summarizing the behavior of $P$ for the fixed value of $x$ across the two possible states of $y$. These values, $h(0)$ and $h(1)$, are then communicated to the verifier, serving as a commitment to the polynomial $h$, which is a derivative of $P$ with $x$ set to $r_x$.

    - The verifier, upon receiving $h(0)$ and $h(1)$, performs a verification step to ensure that the sum $h(0)+h(1)$ equals $g(r_x)$, the sum previously established in the first round. This step is crucial as it verifies the consistency of the prover's claims across the dimensions of the polynomial $P$. To further the protocol, the verifier selects another random value, this time $r_y$, for the variable $y$, refining the claim to $P(r_x,r_y)=h(r_y)$. This condition simplifies the verification process to checking a single evaluation of $P$ against the expected value $h(r_y)$, which can be computed directly from $h(0)$ and $h(1)$.

- Final Verification: In the final step, the verifier checks a single evaluation of the polynomial $P(r_x, r_y)$. The verifier accepts the proof if the evaluation matches the prover's claim $h(r_y)$ in the final round.



These interactive rounds exemplify the iterative nature of the sumcheck protocol, effectively reducing the multidimensional verification problem into a series of simpler, one-dimensional checks. Each round diminishes the number of variables under consideration, systematically narrowing the scope of verification and significantly reducing the computational complexity and communication overhead involved in the process. By the end of all rounds, the verifier is positioned to conclusively assess the veracity of the prover's original claim with high confidence, leveraging the random sampling strategy to mitigate the risk of deceit. More specifically, the soundness error would be $\frac{2(n-1)}{|\mathbb{F}|}$, where $|\mathbb{F}|$ is the size of the whole field. This is computed according to the fact that in each round, the verifier will choose a random point $r$ from the field $\mathbb{F}$, and the degree of each individual variable is one.



Note that in the final step, the verifier needs to check the evaluations of a multilinear polynomial $P(r_x,r_y)$. This can be achieved efficiently by using polynomial commitments, which we will discuss in future blogs. There are also advanced topics in sumcheck protocols, for example, how to prove the sumcheck for the product of multiple polynomials and how to achieve zero knowledge in sumcheck protocols. Please refer to the reference papers [1][2] for details, and we will also explore these topics in future blogs.

## Conclusion

The sumcheck protocol is a foundational tool in the construction of efficient and scalable zero-knowledge proofs. By allowing a prover to succinctly and interactively demonstrate the correctness of complex computations, it has facilitated the development of privacy-preserving cryptographic protocols across a wide range of applications. As cryptographic techniques continue to evolve, the sumcheck protocol remains a key building block in the ongoing quest for secure, private, and verifiable computing. 

## Reference

[1] Marlin: A Preprocessing zkSNARK for R1CS

[2] Libra: Succinct Zero-Knowledge Proofs with Optimal Prover Computation

























