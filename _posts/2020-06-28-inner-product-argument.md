---
layout: project
title: "Understanding Inner Product Argument"
categories: project
tldr: "In this post, we will discuss the details of the inner-product argument described in the Bulletproofs paper. We will emphasize on the simplicity yet the powerful nature of the inner-product argument."
caption: ""
---

{% katexmm %}
The [Bulletproofs](https://eprint.iacr.org/2017/1066.pdf) paper introduces a log-sized inner-product argument and thereafter constructs the state-of-the-art range proof protocol in non-trusted setup setting.
We will discuss the inner-product argument and how it motivated the idea of recursive proof systems like [Halo](https://eprint.iacr.org/2019/1021.pdf).
We assume basic familiarity of the reader with elliptic curve group operations and Pedersen commitments.

### Introduction

Suppose you have some secret information (say, passkeys to two lockers) encoded in a couple of vectors $\textbf{a}, \textbf{b}$.
You (the *prover*) want to convince the government (the *verifier*) that you are the owner of those lockers, so naturally you will have to prove that you know the vectors $\textbf{a}, \textbf{b}$. But you cannot just reveal them for obvious reasons. Can you prove the knowledge of $\textbf{a}, \textbf{b}$ without revealing them? The inner-product argument lets you do so such that the government would be convinved about your ownership without knowing *much* about your secrets.
We assume that size of the vectors $\textbf{a}, \textbf{b}$ are a power of two, but it is fairly straghtforward to extend the analysis for cases otherwise.

### The Inner-Product Argument

The inner-product argument proves the knowledge of *witness* vectors $\textbf{a}, \textbf{b} \in \mathbb{F}_q^{n}$ given a Pedersen vector commitment $P \in \mathbb{G}$ to $\textbf{a}, \textbf{b}$ and the inner product $c = \langle \textbf{a}, \textbf{b} \rangle \in \mathbb{F}_q$.
The following relation proves describes the inner-product argument in a mathematical form.
$$
  \mathcal{L}_{\textsf{IP}}(\sigma) =
  \bigg\{ 
  \underbrace{\textbf{g}, \textbf{h} \in \mathbb{G}^n, P \in \mathbb{G}, c \in \ \mathbb{Z}_q}_{\textsf{statement}}; \  
  \underbrace{\textbf{a}, \textbf{b} \in \mathbb{Z}_q^n}_{\textsf{witness}} 
  \ \bigg| \ 
  \underbrace{P = \textbf{g}^{\textbf{a}} \cdot \textbf{h}^{\textbf{b}} \cdot u^{c} 
  \ \wedge \ c = \langle \textbf{a}, \textbf{b} \rangle}_{\textsf{relation}}
  \bigg\}
$$
where the *common reference string* $\sigma$ contains the public information $\textbf{g}, \textbf{h} \in \mathbb{G}^n, u \in \mathbb{G}$.
To construct the inner-product argument, we first split the secret vectors into half,
$$
\textbf{a} = (\textcolor{red}{\textbf{a}}_{1}, \textcolor{blue}{\textbf{a}}_{1}), 
\quad
\textbf{b} = (\textcolor{red}{\textbf{b}}_{1}, \textcolor{blue}{\textbf{b}}_{1})
$$
where the red vector denotes the left half while the blue denotes right half.
The $1$ in the subscript denotes the step number in recursion.
We now compute Pedersen vector commitments to $(\textcolor{red}{\textbf{a}}_{1}, \textcolor{blue}{\textbf{b}}_{1})$ and $(\textcolor{blue}{\textbf{a}}_{1}, \textcolor{red}{\textbf{b}}_{1})$. Note that we can split the base vectors too as $\textbf{g} = (\textcolor{red}{\textbf{g}}_{1}, \textcolor{blue}{\textbf{g}}_{1}), \textbf{h} = (\textcolor{red}{\textbf{h}}_{1}, \textcolor{blue}{\textbf{h}}_{1})$ and use them to compute:
$$
L_1 = \textcolor{blue}{\textbf{g}}_{1}^{\textcolor{red}{\textbf{a}}_{1}} \cdot \textcolor{red}{\textbf{h}}_{1}^{\textcolor{blue}{\textbf{b}}_{1}} \cdot u^{\langle \textcolor{red}{\textbf{a}}_{1} , \textcolor{blue}{\textbf{b}}_{1} \rangle},
\quad 
R_1 = \textcolor{red}{\textbf{g}}_{1}^{\textcolor{blue}{\textbf{a}}_{1}} \cdot \textcolor{blue}{\textbf{h}}_{1}^{\textcolor{red}{\textbf{b}}_{1}} \cdot u^{\langle \textcolor{blue}{\textbf{a}}_{1} , \textcolor{red}{\textbf{b}}_{1} \rangle}.
$$
We send $L_1, R_1 \in \mathbb{G}$ to the verifier. The verifier sends back a challenge scalar $x_1 \in \mathbb{F}_q$. 
For the next round, we (as well as the verifer) update the base vectors as the combined Pedersen commitment as
$$
\begin{aligned}
\textbf{g}_{1} &= \textcolor{red}{\textbf{g}}_{1}^{x_1^{-1}} \circ \textcolor{blue}{\textbf{g}}_{1}^{x_1} \in \mathbb{G}^{\frac{n}{2}}, \\
\textbf{h}_{1} &= \textcolor{red}{\textbf{h}}_{1}^{x_1} \circ \textcolor{blue}{\textbf{h}}_{1}^{x_1^{-1}} \in \mathbb{G}^{\frac{n}{2}}, \\
P_{1} &= L_1^{x^2} \cdot P \cdot R_1^{x^{-2}} \in \mathbb{G}.
\end{aligned}
$$
Notice that on substituting for $L_1, R_1$ and $P$, the commitment $P_1$ has the form
$$
\begin{aligned}
P_1 &= \left( \textcolor{blue}{\textbf{g}}_{1}^{\textcolor{red}{\textbf{a}}_{1}} \cdot \textcolor{red}{\textbf{h}}_{1}^{\textcolor{blue}{\textbf{b}}_{1}} \cdot u^{\langle \textcolor{red}{\textbf{a}}_{1} , \textcolor{blue}{\textbf{b}}_{1} \rangle} \right)^{x_1^2} 
\cdot
\left(  \textcolor{red}{\textbf{g}}_{1}^{\textcolor{red}{\textbf{a}}_{1}} \cdot 
        \textcolor{blue}{\textbf{g}}_{1}^{\textcolor{blue}{\textbf{a}}_{1}} \cdot 
        \textcolor{red}{\textbf{h}}_{1}^{\textcolor{red}{\textbf{b}}_{1}} \cdot 
        \textcolor{blue}{\textbf{h}}_{1}^{\textcolor{blue}{\textbf{b}}_{1}} \cdot 
        u^{\langle \textbf{a} , \textbf{b} \rangle} \right) 
\cdot
\left( \textcolor{red}{\textbf{g}}_{1}^{\textcolor{blue}{\textbf{a}}_{1}} \cdot \textcolor{blue}{\textbf{h}}_{1}^{\textcolor{red}{\textbf{b}}_{1}} \cdot u^{\langle \textcolor{blue}{\textbf{a}}_{1} , \textcolor{red}{\textbf{b}}_{1} \rangle} \right)^{x_1^{-2}}
\\ 
&= \textcolor{red}{\textbf{g}}_{1}^{\textcolor{red}{\textbf{a}}_{1} + x_1^{-2}\textcolor{blue}{\textbf{a}}_{1}} \cdot
\textcolor{blue}{\textbf{g}}_{1}^{x_1^2\textcolor{red}{\textbf{a}}_{1} + \textcolor{blue}{\textbf{a}}_{1}} \cdot
\textcolor{red}{\textbf{h}}_{1}^{\textcolor{red}{\textbf{b}}_{1} + x_1^{2}\textcolor{blue}{\textbf{b}}_{1}} \cdot
\textcolor{blue}{\textbf{h}}_{1}^{x_1^{-2}\textcolor{red}{\textbf{b}}_{1} + \textcolor{blue}{\textbf{b}}_{1}} \cdot
u^{ x_1^{2}\langle \textcolor{red}{\textbf{a}}_{1} , \textcolor{blue}{\textbf{b}}_{1} \rangle +
    \langle \textcolor{red}{\textbf{a}}_{1} , \textcolor{red}{\textbf{b}}_{1} \rangle +
    \langle \textcolor{blue}{\textbf{a}}_{1} , \textcolor{blue}{\textbf{b}}_{1} \rangle +
    x_1^{-2}\langle \textcolor{blue}{\textbf{a}}_{1} , \textcolor{red}{\textbf{b}}_{1} \rangle}
\\
&= \textcolor{red}{\textbf{g}}_{1}^{x_1^{-1} \cdot (x_1\textcolor{red}{\textbf{a}}_{1} + x_1^{-1}\textcolor{blue}{\textbf{a}}_{1})} \cdot
\textcolor{blue}{\textbf{g}}_{1}^{x_1 \cdot (x_1\textcolor{red}{\textbf{a}}_{1} + x_1^{-1}\textcolor{blue}{\textbf{a}}_{1})} \cdot
\textcolor{red}{\textbf{h}}_{1}^{x_1 \cdot (x_1^{-1}\textcolor{red}{\textbf{b}}_{1} + x_1\textcolor{blue}{\textbf{b}}_{1})} \cdot
\textcolor{blue}{\textbf{h}}_{1}^{x_1^{-1} \cdot (x_1^{-1}\textcolor{red}{\textbf{b}}_{1} + x_1\textcolor{blue}{\textbf{b}}_{1})} \cdot
u^{ \langle x_1 \textcolor{red}{\textbf{a}}_{1} , x_1 \textcolor{blue}{\textbf{b}}_{1} \rangle +
    \langle x_1 \textcolor{red}{\textbf{a}}_{1} , x_1^{-1} \textcolor{red}{\textbf{b}}_{1} \rangle +
    \langle x_1^{-1} \textcolor{blue}{\textbf{a}}_{1} , x_1 \textcolor{blue}{\textbf{b}}_{1} \rangle +
    \langle x_1^{-1} \textcolor{blue}{\textbf{a}}_{1} , x_1^{-1} \textcolor{red}{\textbf{b}}_{1} \rangle}
\\
&= \left( \textcolor{red}{\textbf{g}}_{1}^{x_1^{-1}} \circ \textcolor{blue}{\textbf{g}}_{1}^{x_1} \right)^{(x_1\textcolor{red}{\textbf{a}}_{1} + x_1^{-1}\textcolor{blue}{\textbf{a}}_{1})} \cdot
   \left( \textcolor{red}{\textbf{h}}_{1}^{x_1} \circ \textcolor{blue}{\textbf{h}}_{1}^{x_1^{-1}} \right)^{(x_1^{-1}\textcolor{red}{\textbf{b}}_{1} + x_1\textcolor{blue}{\textbf{b}}_{1})} \cdot
   u^{ \left\langle 
       (x_1\textcolor{red}{\textbf{a}}_{1} + x_1^{-1}\textcolor{blue}{\textbf{a}}_{1}), \ 
       (x_1^{-1}\textcolor{red}{\textbf{b}}_{1} + x_1\textcolor{blue}{\textbf{b}}_{1})
       \right\rangle }
\\
& = \textbf{g}_1^{(x_1\textcolor{red}{\textbf{a}}_{1} + x_1^{-1}\textcolor{blue}{\textbf{a}}_{1})} \cdot
    \textbf{h}_1^{(x_1^{-1}\textcolor{red}{\textbf{b}}_{1} + x_1\textcolor{blue}{\textbf{b}}_{1})} \cdot
    u^{ \left\langle 
       (x_1\textcolor{red}{\textbf{a}}_{1} + x_1^{-1}\textcolor{blue}{\textbf{a}}_{1}), \ 
       (x_1^{-1}\textcolor{red}{\textbf{b}}_{1} + x_1\textcolor{blue}{\textbf{b}}_{1})
       \right\rangle }
\end{aligned} 
$$
Therefore, $P_1$ is in fact the Pedersen commitment to the quantities
$$
\begin{aligned}
\textbf{a}_1 &= (x_1 \cdot \textcolor{red}{\textbf{a}}_{1} + x_1^{-1} \cdot \textcolor{blue}{\textbf{a}}_{1}) \in \mathbb{F}_q^{\frac{n}{2}} \\ 
\textbf{b}_1 &= (x_1^{-1} \cdot \textcolor{red}{\textbf{b}}_{1} + x_1 \cdot \textcolor{blue}{\textbf{b}}_{1}) \in \mathbb{F}_q^{\frac{n}{2}}.
\end{aligned}
$$
Note that we could send the vectors $(\textbf{a}_1, \textbf{b}_1)$ to the verifier and she could verify (and be convinced about our knowledge of vectors $(\textbf{a}, \textbf{b})$ without actually knowing them!) the last equality above, but this results in prover communication to be $n$ field elements. We wish to reduce it to logarithmic in $n$.
With that in mind, we can now repeat the above process with the new secrets $(\textbf{a}_1, \textbf{b}_1)$, bases $(\textbf{g}_1, \textbf{h}_1)$ and commitment $P_1$. We can keep doing so until the size of the new bases is $1$. In this process, the verifier would have sent $l = \text{log}_2(n)$ challenges, viz. $(x_1, x_2, \dots, x_{l})$ and we would have sent $(L_1, R_1, L_2, R_2, \dots, L_l, R_l) \in \mathbb{G}^{2l}$ elements.
For the last round, we send the elements $(a_{\text{last}}, b_{\text{last}}) \in \mathbb{F}_q^2$ and all the verifier has to check is
$$
\tag{1}
P_{\text{last}} \stackrel{?}{=} g_{\text{last}}^{a_{\text{last}}} \cdot h_{\text{last}}^{b_{\text{last}}} \cdot u^{a_{\text{last}} \cdot b_{\text{last}}}
$$
where $P_{\text{last}}, g_{\text{last}}, h_{\text{last}}$ can be computed by the verifier from all the information she has in the previous $l$ rounds!
Thus, the total prover communication would then be $2 \text{log}_2(n)$ group elements plus $2$ field elements.

### Prover and Verifier Costs

In the $i$-th round, the prover computes $L_i, R_i \in \mathbb{G}$ and base vectors $\textbf{g}_i, \textbf{h}_i \in \mathbb{G}^{\frac{n}{2^i}}$.
This amounts to 2 group exponentiations of size $2 \cdot \frac{n}{2^i} + 1$ and 2 group exponentiations of size $2 \cdot \frac{n}{2^i}$, so a total of $\frac{n}{2^{i-3}} + 2$ group exponentiations. In total, the prover would need $8(n-1)$ group exponentiations. Since field operations are typically orders of magnitude faster than group exponentiations, we consider the costs only due to group exponentiations.

On the other hand, the verifier only needs to validate equation $(1)$ in the last round. 
All she needs for validation is to compute $g_\text{last}, h_\text{last}$ and $P_\text{last}$.
Note that $g_\text{last}, h_\text{last}$ would depend only on the challenges $(x_1, x_2, \dots, x_l)$.
Let us try to compute the base vector $\textbf{g}_2$ for round 2 for $n=8$.
$$
\begin{aligned}
\textbf{g}_1 &= \textcolor{red}{\textbf{g}}_{1}^{x_1^{-1}} \circ \textcolor{blue}{\textbf{g}}_{1}^{x_1} \in \mathbb{G}^{\frac{n}{2}}
\\
             &= (g_1, g_2, g_3, g_4)^{x_1^{-1}} \circ (g_5, g_6, g_7, g_8)^{x_1}
\\
             &= (g_1^{x_1^{-1}} g_5^{x_1}, \  g_2^{x_1^{-1}} g_6^{x_1}, \  g_3^{x_1^{-1}} g_7^{x_1}, \  g_4^{x_1^{-1}} g_8^{x_1})
             \in \mathbb{G}^4
\\
\textbf{g}_2 &= \left( g_1^{x_1^{-1}} g_5^{x_1}, \  g_2^{x_1^{-1}} g_6^{x_1} \right)^{x_2^{-1}} \circ
                \left( g_3^{x_1^{-1}} g_7^{x_1}, \  g_4^{x_1^{-1}} g_8^{x_1} \right)^{x_2} 
\\
             &= \left( 
                 g_1^{x_1^{-1} x_2^{-1}} g_5^{x_1 x_2^{-1}} g_3^{x_1^{-1} x_2} g_7^{x_1 x_2}, \ 
                 g_2^{x_1^{-1} x_2^{-1}} g_6^{x_1 x_2^{-1}} g_4^{x_1^{-1} x_2} g_8^{x_1 x_2}
                \right)
                \in \mathbb{G}^2
\\
g_\text{last} = \textbf{g}_3 &= 
                \left( g_1^{x_1^{-1} x_2^{-1}} g_5^{x_1 x_2^{-1}} g_3^{x_1^{-1} x_2} g_7^{x_1 x_2} \right)^{x_3^{-1}} \circ
                \left( g_2^{x_1^{-1} x_2^{-1}} g_6^{x_1 x_2^{-1}} g_4^{x_1^{-1} x_2} g_8^{x_1 x_2} \right)^{x_3}
\\
                &= g_1^{x_1^{-1} x_2^{-1} x_3^{-1}} g_5^{x_1 x_2^{-1} x_3^{-1}} g_3^{x_1^{-1} x_2 x_3^{-1}} g_7^{x_1 x_2 x_3^{-1}}
                   g_2^{x_1^{-1} x_2^{-1} x_3} g_6^{x_1 x_2^{-1} x_3} g_4^{x_1^{-1} x_2 x_3} g_8^{x_1 x_2 x_3}
\\
                &= g_1^{x_1^{-1} x_2^{-1} x_3^{-1}} g_2^{x_1^{-1} x_2^{-1} x_3^{1}} g_3^{x_1^{-1} x_2^{1} x_3^{-1}} g_4^{x_1^{-1} x_2^{1} x_3^{1}}
                   g_5^{x_1^{1} x_2^{-1} x_3^{-1}} g_6^{x_1^{1} x_2^{-1} x_3^{1}} g_7^{x_1^{1} x_2^{1} x_3^{-1}} g_8^{x_1^{1} x_2^{1} x_3^{1}}
                   \in \mathbb{G}
\end{aligned}
$$
Note the following pattern in the powers of challenges $(x_1, x_2, x_3)$.

| generator | powers of $(x_1, x_2, x_3)$ | binary form | decimal |
|-----------|-----------------------------|-------------|---------|
| $g_1$     | $-1,-1,-1$                  | $000$       | 0       |
| $g_2$     | $-1,-1,1$                   | $001$       | 1       |
| $g_3$     | $-1,1,-1$                   | $010$       | 2       |
| $g_4$     | $-1,1,1$                    | $011$       | 3       |
| $g_5$     | $1,-1,-1$                   | $100$       | 4       |
| $g_6$     | $1,-1,1$                    | $101$       | 5       |
| $g_7$     | $1,1,-1$                    | $110$       | 6       |
| $g_8$     | $1,1,1$                     | $111$       | 7       |

Similarly, we can compute $h_{\text{last}}$ with a difference that the powers of the challenges in the exponent get inverted.
$$
h_{\text{last}} = \textbf{h}_3
= h_1^{x_1^{1} x_2^{1} x_3^{1}} h_2^{x_1^{1} x_2^{1} x_3^{-1}} h_3^{x_1^{1} x_2^{-1} x_3^{1}} h_4^{x_1^{1} x_2^{-1} x_3^{-1}}
                   h_5^{x_1^{-1} x_2^{1} x_3^{1}} h_6^{x_1^{-1} x_2^{1} x_3^{-1}} h_7^{x_1^{-1} x_2^{-1} x_3^{1}} h_8^{x_1^{-1} x_2^{-1} x_3^{-1}}
                   \in \mathbb{G}
$$
The ghastly looking expressions for $g_{\text{last}}$ and $h_{\text{last}}$ (for even $n=3$) can be written in a much simpler form as
$$
g_{\text{last}} = \prod_{i=1}^{n} g_i^{s_i}, \quad h_{\text{last}} = \prod_{i=1}^{n} h_i^{s_i^{-1}},
$$
such that 
$$
s_i = \prod_{j=1}^{l} x_j^{b(i,j)}, \quad \text{ where } \quad b(i,j) =
\begin{cases}
1 & \text{if the $j$-th bit of } (i-1) \text{ is 1} \\
-1 & \text{otherwise}.
\end{cases}
$$
On a similar note, we can compute $P_{\text{last}}$ as
$$
\begin{aligned}
P_2 &= L_2^{x_2^2} \cdot P_1 \cdot R_2^{x_2^{-2}} \\
    &= L_2^{x_2^2} \cdot \left( L_1^{x_1^2} \cdot P \cdot R_1^{x_1^{-2}} \right) \cdot R_2^{x_2^{-2}} \\
    &= L_2^{x_2^2} \cdot L_1^{x_1^2} \cdot P \cdot R_1^{x_2^2} \cdot R_2^{x_1^2} \\
\therefore \quad P_{\text{last}} &= \left(L_l^{x_l^2} \cdot \ldots \cdot L_2^{x_2^2} \cdot L_1^{x_1^2}\right)
                                    \cdot P \cdot 
                                    \left( R_1^{x_2^2} \cdot R_2^{x_1^2} \cdot \ldots \cdot R_l^{x_l^2} \right) \\
&= P \cdot \left(\prod_{j=1}^{l} L_j^{x_j^2} \cdot R_j^{x_j^{-2}}\right)
\end{aligned}
$$

Therefore from $(1)$, the verification of an inner-product argument boils to a single multi-exponentiation check of size $(2n + 2l + 1)$
$$
P \cdot \left(\prod_{j=1}^{l} L_j^{x_j^2} \cdot R_j^{x_j^{-2}}\right) 
\stackrel{?}{=}
\textbf{g}^{a \cdot \textbf{s}} \cdot \textbf{h}^{b \cdot \textbf{s}^{-1}} \cdot u^{a \cdot b} 
$$
where $\textbf{s} = (s_1, s_2, \dots, s_n)$.

### Closing Comments

The log-sized inner-product in a non-trusted setup was an important breakthrough in applied cryptography research.
For cryptocurrencies like Monero, the inner-product powered range proof protocol *bulletproofs* helped reduce the transaction sizes by a whopping 80%[^1]. 
The downside, however, of the inner-product argument is the linear verification times in spite of aggregated proof verification, causing practical bottlenecks for deployment for large arithmetic circuits.
Nevertheless, the inner-product argument opened up avenues for future research in the direction of recursive proofs.
Stay tuned to understand how the beautiful technique of inner-product argument helped in construction of [Halo](https://eprint.iacr.org/2019/1021.pdf).

[^1]: Bulletproofs in Moneropedia, Link: [https://web.getmonero.org/resources/moneropedia/bulletproofs.html](https://web.getmonero.org/resources/moneropedia/bulletproofs.html)
{% endkatexmm %}
