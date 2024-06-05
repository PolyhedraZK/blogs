## Bivariate Kate-Zaverucha-Goldberg (KZG) Constant-Sized Polynomial Commitments

KZG commitment scheme, published in 2010 by Kate, Zaverucha, and Goldberg, is one of the most widely used polynomial commitment schemes.

In this article, we will introduce a variant of KZG commitment, bivariate KZG commitment, which supports us to commit to polynomials with two variables.

### Trust Setup

In univariate KZG commitment, we need to commit to degree $\le \ell$ polynomials, so we need to setup $\ell$-SDH public parameters: $(g_1, g_1^{\tau}, g_1^{\tau^2}, ..., g_1^{\tau^\ell}) = (g_1^{\tau^i})_{i\in [0,\ell]}$, where $\tau$ is called the trapdoor. After generating $g_1^{\tau^i}$, $\tau$ should be forgotten. Similarly, in bivariate KZG commitment, we need to commit to degree $\le N$ and degree $\le M$, therefore, we need to setup $N \times M$-SDH public parameters in a cyclic group $g_1$:

$$
(g_1^{\tau_0^i\tau_1^j})_{i\in [0,N], j\in [0, M]} = \\
(g_1, g_1^{\tau_0}, g_1^{\tau_0^2}, ..., g_1^{\tau_0^N}, g_1^{\tau_1}, g_1^{\tau_0\tau_1}, g_1^{\tau_0^2\tau_1}, ..., g_1^{\tau_0^N\tau_1}, ..., g_1^{\tau_0^N\tau_1^M})
$$

Here, $\tau_0$ and $\tau_1$ are the trapdoors. Moreover, since we need to commit to a bivariable polynomial, we also setup two public parameters in a cyclic group $g_2$ as verifying keys:

$$
g_2^{\tau_0}, g_2^{\tau_1}
$$

### Commitments

Given a bivariate polynomial:

$$f(x, y) = \sum\_{i=0}^{N−1}coeff\_{i,j}x^iy^j$$

we compute the commitment:

$$
c = \prod_{i\in [0,N], j\in [0, M]}(g_1^{\tau_0^i\tau_1^j})^{coeff_{i,j}}
$$

Here, we need to get the coefficients of the polynomial to compute the commitment. In practice, we may be given a lagrange polynomial, i.e., a list of the lagrange points. Hence, we need to slightly change the setup and commitment phases.

### Lagrange Polynomial

Assume we are given a list of points: $(\omega^0, a_0), (\omega^0, a_0), ..., (\omega^{N-1}, a_0)$, where $\omega$ is the $N$-th root of unity. Let lagrange polynomial be $L_i(x)$, it satisfies following properties:

$$
\displaylines{
L_i(\omega^i) = 1 \\
\forall j\neq i, L_j(\omega^j) = 0
}
$$

Therefore, we can construct $L_i(x)$ and $f(x)$ as follows

$$
\displaylines{
L_i(x) = \frac{\prod_{j\neq i}(\omega^j-x)}{\prod_{j\neq i}(\omega^j-\omega^i)}\\
f(x) = \sum\limits_{i=0}^{N-1}a_iL_i = \sum\limits_{i=0}^{N-1}a_i\frac{\prod_{j\neq i}(\omega^j-x)}{\prod_{j\neq i}(\omega^j-\omega^i)}
}
$$

#### Bivariable Lagrange Polynomial

Given a list of points


$$
\displaylines{
((\omega_0^0, \omega_1^0), a_{0,0}), ((\omega_0^1, \omega_1^0), a_{1,0}), ((\omega_0^2, \omega_1^0), a_{2,0}),  ..., ((\omega_0^{N-1}, \omega_1^0), a_{N-1,0}) \\
((\omega_0^0, \omega_1^1), a_{0,1}), ((\omega_0^1, \omega_1^1), a_{1,1}), ((\omega_0^2, \omega_1^1), a_{2,1}),  ..., ((\omega_0^{N-1}, \omega_1^1), a_{N-1,1}) \\
... \\
((\omega_0^0, \omega_1^{M-1}), a_{0,M-1}), ((\omega_0^1, \omega_1^{M-1}), a_{1,M-1}), ((\omega_0^2, \omega_1^{M-1}), a_{2,M-1}),  ..., ((\omega_0^{N-1}, \omega_1^{M-1}), a_{N-1,M-1})
}
$$

Here $\omega_0$ is the $N$-th root of unity and $\omega_1$ is the $M$-th root of unity. We assume $N, M$ are power of 2.
With the list of points, we have the bivariable lagrange polynomial:

$$
f(x,y) = \sum\limits_{i=0}^{N-1}\sum\limits_{j=0}^{M-1}a_{i,j}L_i^N(x)L_j^M(y)\\
= \sum\limits_{i=0}^{N-1}\sum\limits_{j=0}^{M-1}a_{i,j}\frac{\prod_{k\neq i}(\omega_0^k-x)}{\prod_{k\neq i}(\omega_0^k-\omega_0^i)}\frac{\prod_{k\neq j}(\omega_1^k-y)}{\prod_{k\neq j}(\omega_1^k-\omega_1^j)}
$$

#### Bi-KZG Setup

Since we are given a lagrange polynomial $f(x,y)$ (with two variable: $x$ and $y$), we should update the setup. Typically, we need to setup the following parameters:

$$
\displaylines{
(g_1^{L_i^N(\tau_0)L_j^M(\tau_1)})\_{i\in [0,N], j\in [0, M]} = \\
(g_1, g_1^{L_1^N(\tau_0)}, g_1^{L_2^N(\tau_0^2)}, ..., g_1^{L_{N-1}^N(\tau_0^{N-1})},\\
g_1^{L_1^M(\tau_1)}, g_1^{L_1^N(\tau_0)L_1^M(\tau_1)}, g_1^{L_2^N(\tau_0^2)L_1^M(\tau_1)}, ..., g_1^{L_{N-1}^N(\tau_1^{N-1})L_1^M(\tau_1)},\\
...,\\
g_1^{L_{M-1}^M(\tau_1)}, g_1^{L_1^N(\tau_0)L_{M-1}^M(\tau_1)}, g_1^{L_2^N(\tau_0^2)L_{M-1}^M(\tau_1)}, ..., g_1^{L_{N-1}^N(\tau_1^{N-1})L_{M-1}^M(\tau_1^{M-1})})
}
$$

Besides, we need to setup the public parameters in the cyclic group $g_2$: $(g_2^{\tau_0}, g_2^{\tau_1})$ as verifying keys.

#### Bi-KZG Commitment

Now, with the updated setup parameters and the given lagrange points, we can compute the commitment as follows:

$$
c = g_1^{f(\tau_0, \tau_1)}
= g_1^{\sum\limits_{i=0}^{N-1}\sum\limits_{j=0}^{M-1}a_{i,j}L_i^N(\tau_0)L_j^M(\tau_1)} 
= \prod_{i=0}^{N-1}\prod_{j=0}^{M-1}(g_1^{L_i^N(\tau_0)L_j^M(\tau_1)})^{a_{i,j}}
$$

Here, $a_{i, j}$ are given during commitment phase, and $g_1^{L_i^N(\tau_0)L_j^M(\tau_1)} (i \in [0, N], j \in [0, M])$ are generated during setup phase.

### Bi-KZG Prove

Assume a prover is given a pair of two values $(a, b)$ and she returns an evaluation $u$, where $u$ is expected to be equal to $f(a, b)$.

To prove that, the prover needs to compute two quotients:

$$
\displaylines{
Q_0(x, b) = \frac{f(x,b) - u}{x-a}\\
Q_1(a,y) = \frac{f(a,y) - u'}{y-b}
}
$$

Here, $u = f(a,b)$ and $u' = f(\tau_0, b)$. Since we have evaluations on $\omega_0^i$ ($i \in [0, N]$) and $\omega_1^j$ ($j \in [0, M]$), we can compute $f(x,b)$ and $f(a,y)$. Furthermore, we can compute $Q_0(x,b)$ and $Q_1(a,y)$ with their evaluation on $\omega_0^i$ ($i \in [0, N]$) and $\omega_1^j$ ($j \in [0, M]$).

Then, the prover can compute two proofs as follows:

$$
\displaylines{
\pi_0 = g_1^{Q_0(\tau_0, b)}\\
\pi_1 = g_1^{Q_1(a, \tau_1)}
}
$$

Then, the prover sends the evaluation $u$ and proofs $\pi_0, \pi_1$ to the verifier.

#### Verifying

Given the verifying keys $(g_2^{\tau_0}, g_2^{\tau_1})$, the commitment $c = g_1^{f(\tau_0,\tau_1)}$, the evaluation $u = f(a, b)$ and the proofs $\pi_0 = g_1^{Q_0(\tau_0, b)}$ and $\pi_1 = g_1^{Q_1(a, \tau_1)}$, the verifier can verify the evaluation using following pairings:

$$
\displaylines{
e(\pi_0,g_2^{\tau_0−a} )=e(g_1^{u'−u},g_2) \\
e(\pi_1,g_2^{\tau_1−b} )=e(g_1^{c−u'}, g_2)
}
$$

Let us slightly adjust them, so we can compute:

$$
\displaylines{
e(\pi_0\cdot \pi_1, g_2^{\tau_0-a}\cdot g_2^{\tau_1-b}) = e(g_1^{u'-u}\cdot g_1^{c-u'}, g_2)\\
e(\pi_0\cdot \pi_1, \frac{g_2^{\tau_0}\cdot g_2^{\tau_1}}{g_2^a\cdot g_2^b}) = e(\frac{g_1^c}{g_1^u}, g_2)
}
$$

That is, the verifier needs to compute $g_2^a$ and $g_2^b$, and the pairings.

### Distributed Computing

In practice, we may use distributed computing to fasten the phases on the prover side (i.e., setup, commitment, and prove phases). Typically, we can use MPI (Message Passing Interface) to distribute multiple tasks to multiple machines. Below, we describe the change of each phase under the distributed achitecture.

#### MPI Setup

Since we split the original task into multiple sub-tasks, each machine can setup a part of setup parameters. Assume we have $n$ machines, then machine $i$ can setup the following paramters:

$$
(g_1^{L_i^N(\tau_0)L_j^M(\tau_1)})_{i\in [start_i,end_i), j\in [0, M]}
$$

Here, $start_i = i * chunk$ and $end_i = min((i + 1) * chunk, N)$, and $chunk = \lfloor \frac{alpha + n - 1}{n} \rfloor$

#### MPI Commitment

Each machine can compute the partial commitment, then the root machine can collect them and compute the final commitment. Typically, the i-th machine needs to compute:

$$
\displaylines{
c_i = g_1^{\sum\limits_{i=start_i}^{end_i-1}\sum\limits_{j=0}^{M-1}a_{i,j}L_i^N(\tau_0)L_j^M(\tau_1)} \\
    = \prod_{i=start_i}^{end_i-1}\prod_{j=0}^{M-1}(g_1^{L_i^N(\tau_0)L_j^M(\tau_1)})^{a_{i,j}}
}
$$

After collecting all partial commitments, the root machine can compute the final commitment $c = \sum\limits_{i=0}^{n-1}c_i$ and send to the verifier.

#### MPI Prove

Since each machine keeps a part of lagrange points, a machine will first calculate the partial evaluation $u_i$, then the root machine collects all partial evaluation and compute $u = \sum\limits_{i=0}^{n-1}u_i$.

Similarly, all machines can compute the partial proofs, $\pi_{0,i}$ and $\pi_1$ based on their lagrange points. Finally, the root machine can compute $\pi_0 = \sum\limits_{i=0}^{n-1}\pi_{0,i}$.
