---
layout: project
title: "Comparing Bulletproofs+ and Bulletproofs - Part III"
categories: project
tldr: "In the third part on comparing Bulletproofs+ and Bulletproofs, we discuss the single multi-exponentiation verification in Bulletproofs and Bulletproofs range proof protocols. This greatly speeds up the verification speeds. Dive in to see the math behind it!"
caption: ""
---

{% katexmm %}
In this third part on comparing [Bulletproofs+](https://eprint.iacr.org/2020/735.pdf) and [Bulletproofs](https://eprint.iacr.org/2017/1066.pdf), we will delve into the math of aggregate verification of the range proofs. We compare the verification speeds of both of the protocols qualitatively as well as quantitatively. Please read blogs [1](https://suyash67.github.io/homepage/project/2020/07/03/bulletproofs_plus_part1.html), [2](https://suyash67.github.io/homepage/project/2020/07/03/bulletproofs_plus_part2.html) for a primer on Bulletproofs and Bulletproofs+.


### Aggregated Range Proof Protocols 

An aggregated range proof implies proving that each of the quantity $a_j \in \mathbb{Z}_q$ for $j=1,2,\dots,m$ is in the range $[0,2^n - 1]$ using a single proof. An aggregated Bulletproofs and Bulletproofs+ proofs have the following structure.
{% katex display %}
\begin{aligned}
\texttt{crs}_{bp} &= \{ \textbf{g}, \textbf{h} \in \mathbb{G}^{n\cdot m}, \ g,h \in \mathbb{G}, \ \textbf{V}=(V_1,V_2, \dots, V_m) \in \mathbb{G}^m \} \\[6pt]
\texttt{wit}_{bp} &= \{ \textbf{a}, \mathbf{f} \in \mathbb{Z}_q^m \} \\[6pt]
\texttt{stmt}_{bp} &= \{ V_j = g^{a_j} h^{f_j} \text{ and } a_j \in [0,2^{n}-1] \ \forall j \in [m] \} \\[6pt]
\Pi_{bp} &= \left\{ A,S,T_1,T_2, (\textbf{L,\ R}) = \big( L_i, R_i \big)_{j=1}^{\text{log}_2(nm)} \in \mathbb{G}^{2 \times \text{log}_2(nm)}, \tau, \hat{t}, \mu, a,b \in \mathbb{Z}_q \right\}
\end{aligned}
{% endkatex %}

{% katex display %}
\Pi_{bp+} = \left\{ A,(\textbf{L,\ R}) = \big( L_i, R_i \big)_{j=1}^{\text{log}_2(nm)} \in \mathbb{G}^{2 \times \text{log}_2(nm)}, r',s',\delta' \in \mathbb{Z}_q,  A',B' \in \mathbb{G} \right\}
{% endkatex %}

Note that the $\texttt{crs, wit, stmt}$ remain unchanged for Bulletproofs+.

### Bulletproofs Verification

Given an aggregated BP proof, a verifier needs to check the following:
{% katex display %}
\tag{1} 
g^{\hat{t}} \cdot h^{\tau} \stackrel{?}{=} g^{\delta(y,z)} \cdot \textbf{V}^{z^2 \cdot \textbf{z}^m} \cdot T_1^x \cdot T_2^{x^2} 
{% endkatex %}

{% katex display %}
\tag{2}
\text{verify-ipp}(\textbf{g}, \textbf{h}', g^{x_u}, Ph^{-\mu}g^{x_u \cdot \hat{t}}, \hat{t})
{% endkatex %}

where $\textbf{h}' = (h_1, h_2^{y^{-1}}, \dots, h_{mn}^{y^{1-mn}})$.
Note that the verifier can compute $P$ as 

$$\tag{3} P = A \cdot S^x \cdot \textbf{g}^{-z \cdot \textbf{1}^{mn}} \cdot (\textbf{h}')^{z \cdot \textbf{y}^{mn} + \textbf{d}}$$ 

where $\textbf{d} = (z^2 \cdot \textbf{2}^n, z^3 \cdot \textbf{2}^n, \dots, z^{m+1} \cdot \textbf{2}^n) \in \mathbb{Z}^{mn}_q$ from publicly available information.
The inner product proof in $(2)$ can be verified in a single multi-exponentiation equation [^1] as

$$
\tag{4}
\textbf{g}^{a \cdot \textbf{s}} \cdot \textbf{h}^{b \cdot \textbf{s}'} \cdot g^{x_u \cdot ab} \stackrel{?}{=} (Ph^{-\mu}) \cdot \prod_{j=1}^{\text{log}_2(nm)} L_j^{x_j^2} \cdot R_j^{x_j^{-2}}
$$

where $\textbf{s} = (s_1, s_2, \dots, s_{mn}), \textbf{s}' = (s_1^{-1}, s_2^{-1}, \dots, s_{mn}^{-1}) \in \mathbb{Z}_q^{mn}$ such that for all $i \in [nm]$

$$
s_i = \prod_{j=1}^{\text{log}_2(nm)} x_j^{b(i,j)}
\quad
\text{ where }
\quad
b(i,j) = 
\begin{cases}
   1 &\text{if $j$-th bit of } (i-1) \text{ is 1} \\
   -1 &\text{otherwise}
\end{cases}
$$

and $(x_1, x_2, \dots, x_{\text{log}_2(nm)})$ are the challenges using in $\text{log}_2(nm)$ rounds of inner product protocol. Substituting $(3)$ in $(4)$ to get a single inner product proof check, we get

$$
\textbf{g}^{a \cdot \textbf{s}} \cdot \textbf{h}^{b \cdot \textbf{s}'} \cdot g^{x_u \cdot ab} \stackrel{?}{=} 
\left(
    A \cdot S^x \cdot \textbf{g}^{-z \cdot \textbf{1}^{mn}} \cdot (\textbf{h}')^{z \cdot \textbf{y}^{mn} + \textbf{d}} \cdot h^{-\mu} \cdot g^{x_u \cdot \hat{t}}
\right) 
\cdot \prod_{j=1}^{\text{log}_2(nm)} L_j^{x_j^2} \cdot R_j^{x_j^{-2}}
$$

Bringing everything to the left, we have
$$
\textbf{g}^{a \cdot \textbf{s} + z \cdot \textbf{1}^{mn}} \cdot 
\textbf{h}^{b \cdot \textbf{s}' - z \cdot \textbf{1}^{mn} - (\textbf{y}')^{mn} \circ \textbf{d}} \cdot 
g^{x_u (ab - \hat{t})} \cdot
h^{\mu} \cdot 
A^{-1} \cdot 
S^{-x} \cdot
\prod_{j=1}^{\text{log}_2(nm)} L_j^{-x_j^2} \cdot R_j^{-x_j^{-2}}
\stackrel{?}{=}
1
$$

where $\textbf{y}' = (1, y^{-1}, y^{-2}, \dots, y^{-mn+1})$. Substituting 
$$
\begin{aligned}
\textbf{x}_L &= (-x_1^2, -x_2^2, \dots, -x_{\text{log}_2(nm)}^2) \\
\textbf{x}_R &= (-x_1^{-2}, -x_2^{-2}, \dots, -x_{\text{log}_2(nm)}^{-2}),
\end{aligned}
$$
we get 

$$
\tag{5}
\textbf{g}^{a \cdot \textbf{s} + z \cdot \textbf{1}^{mn}} \cdot 
\textbf{h}^{b \cdot \textbf{s}' - z \cdot \textbf{1}^{mn} - (\textbf{y}')^{mn} \circ \textbf{d}} \cdot 
g^{x_u (ab - \hat{t})} \cdot
h^{\mu} \cdot 
A^{-1} \cdot 
S^{-x} \cdot
\textbf{L}^{\textbf{x}_L} \cdot
\textbf{R}^{\textbf{x}_R}
\stackrel{?}{=}
1
$$

A verifier, thus, has to verify equations $(1), (5)$ to verify a BP range proof. We can re-write $(1)$ as
$$
\tag{6}
g^{\hat{t} - \delta(y,z)} \cdot 
h^{\tau} \cdot
\textbf{V}^{-z^2 \cdot \textbf{z}^m} \cdot
\cdot T_1^{-x} \cdot 
T_2^{-x^2}
\stackrel{?}{=} 
1
$$

Lastly, she can combine equations $(5), (6)$ into a single multi-exponentiation check using a random scalar $c \in \mathbb{Z}_q$ as follows:

$$
\left(
    g^{\hat{t} - \delta(y,z)} \cdot 
    h^{\tau} \cdot
    \textbf{V}^{-z^2 \cdot \textbf{z}^m} \cdot
    \cdot T_1^{-x} \cdot 
    T_2^{-x^2}
\right)^c 
\cdot
\textbf{g}^{a \cdot \textbf{s} + z \cdot \textbf{1}^{mn}} \cdot 
\textbf{h}^{b \cdot \textbf{s}' - z \cdot \textbf{1}^{mn} - (\textbf{y}')^{mn} \circ \textbf{d}} \cdot 
\\
g^{x_u (ab - \hat{t})} \cdot
h^{\mu} \cdot 
A^{-1} \cdot 
S^{-x} \cdot
\textbf{L}^{\textbf{x}_L} \cdot
\textbf{R}^{\textbf{x}_R}
\stackrel{?}{=}
1
\\
$$

$$
\tag{7}
\implies
\textbf{g}^{a \cdot \textbf{s} + z \cdot \textbf{1}^{mn}} \cdot 
\textbf{h}^{b \cdot \textbf{s}' - z \cdot \textbf{1}^{mn} - (\textbf{y}')^{mn} \circ \textbf{d}} \cdot 
g^{x_u (ab - \hat{t}) + c(\hat{t} - \delta(y,z))} \cdot
h^{\mu + c\tau} \cdot 
\\
A^{-1} \cdot 
S^{-x} \cdot
\textbf{L}^{\textbf{x}_L} \cdot
\textbf{R}^{\textbf{x}_R} \cdot
\textbf{V}^{-c z^2 \cdot \textbf{z}^m} \cdot
\cdot T_1^{-c x} \cdot 
T_2^{-c x^2}
\stackrel{?}{=}
1
$$

Thus, an aggregated BP range proof can be verified by a single multi-exponentiation check of size $2mn + 2\text{log}_2(mn) + m + 6$.

### Bulletproofs+ Verification

To verify an aggregated Bulletproofs+ range proof $\Pi_{bp+}$, a verifier needs to compute $\hat{A}$ and run the $\text{verify-}zk\text{-}\textsf{WIP}$.

$$
\tag{8}
\hat{A} = A \cdot
\textbf{g}^{-z \cdot \textbf{1}^{mn}} \cdot 
\textbf{h}^{z \cdot \textbf{1}^{mn} + \textbf{d} \circ \overleftarrow{y}^{mn}} \cdot
\textbf{V}^{y^{mn+1} \cdot z^2 \cdot \textbf{z}^m} \cdot g^{\zeta(y,z)} 
\\[6pt]
\text{verify-}zk\text{-}\textsf{WIP}_{y}(\textbf{g}, \textbf{h}, g,h, \hat{A})
$$

where $\zeta(y,z) = (z - z^2) y \cdot \langle \textbf{1}^{mn},\textbf{y}^{nm} \rangle - zy^{mn+1} \cdot \langle \textbf{1}^{mn}, \textbf{d} \rangle$ and $\overleftarrow{y}^{nm} = (y^{mn}, y^{mn-1}, \dots, y)$.
Similar to the inner product argument, the verification in weighted inner product argument too can be expressed as a single equation by unrolling recursion.

$$
\tag{9}
\textbf{g}^{e \cdot r' \cdot \textbf{s}} \cdot 
\textbf{h}^{e \cdot s' \cdot \textbf{s}'} \cdot 
g^{r' \odot s'} \cdot
h^{\delta'} 
\stackrel{?}{=} 
(\hat{A})^{e^2} \cdot 
\left(
    \prod_{j=1}^{\text{log}_2(nm)} L_j^{e^2 \cdot x_j^2} \cdot R_j^{e^2 \cdot x_j^{-2}}
\right) \cdot
(A')^{e} \cdot B 
$$

By substituting $\hat{A}$, the verification boils downs to a single multi-exponentiation check.

$$
\textbf{g}^{e \cdot r' \cdot \textbf{s}} \cdot 
\textbf{h}^{e \cdot s' \cdot \textbf{s}'} \cdot 
g^{r' \odot s'} \cdot
h^{\delta'} 
\stackrel{?}{=} 
\left(
    A \cdot
    \textbf{g}^{-z \cdot \textbf{1}^{mn}} \cdot 
    \textbf{h}^{z \cdot \textbf{1}^{mn} + \textbf{d} \circ \overleftarrow{y}^{mn}} \cdot
    \textbf{V}^{y^{mn+1} \cdot z^2 \cdot \textbf{z}^m} \cdot g^{\zeta(y,z)} 
\right)^{e^2} \cdot 
\\
\left(
    \prod_{j=1}^{\text{log}_2(nm)} L_j^{e^2 \cdot x_j^2} \cdot R_j^{e^2 \cdot x_j^{-2}}
\right) \cdot
(A')^{e} \cdot B 
$$

$$
\implies 
\textbf{g}^{e \cdot r' \cdot \textbf{s} + ze^2 \textbf{1}^{mn}} \cdot 
\textbf{h}^{e \cdot s' \cdot \textbf{s}' - ze^2 \cdot \textbf{1}^{mn} - e^2 \cdot \textbf{d} \circ \overleftarrow{y}^{mn}} \cdot 
g^{r' \odot s' - e^2\zeta(y,z)} \cdot
h^{\delta'} \cdot
A^{-e^2} \cdot
\\
\textbf{V}^{-e^2 y^{mn+1} \cdot z^2 \cdot \textbf{z}^m} \cdot 
\textbf{L}^{\textbf{x}_L} \cdot
\textbf{R}^{\textbf{x}_R} \cdot
(A')^{-e} \cdot B^{-1} 
$$

Note here, we have 
$$
\begin{aligned}
\textbf{x}_L &= e^2 \cdot (-x_1^2, -x_2^2, \dots, -x_{\text{log}_2(nm)}^2) \\
\textbf{x}_R &= e^2 \cdot (-x_1^{-2}, -x_2^{-2}, \dots, -x_{\text{log}_2(nm)}^{-2}),
\end{aligned}
$$

Thus, an aggregated BP+ range proof can be verified by a single multi-exponentiation check of size $2mn + 2\text{log}_2(mn) + m + 5$.

### Comparison of Verification Times

The following plot shows verification times for BP and BP+ for $64$-bit range proofs over $\texttt{secp256k1}$ curve in Rust.
The code can be found [here](https://github.com/KZen-networks/bulletproofs).
Note that the verification speed can be further improved using [Pipenger's exponentiation algorithm](https://cr.yp.to/papers/pippenger.pdf) for computing a multi-exponentiation.
All simulations were performed on an Intel® Core™ i7-5500U CPU running at 2.40GHz.

<!-- 
|  $m$ 	| BP Verification (ms) 	| BP+ Verification (ms) 	|
|:--:	|:--------------------:	|:---------------------:	|
| 1  	|         14.15        	|         14.36         	|
| 2  	|         27.26        	|         26.91         	|
| 4  	|         62.81        	|         53.51         	|
| 8  	|        121.19        	|         109.23        	|
| 16 	|        279.70        	|         235.26        	|
| 32 	|        718.34        	|         556.90        	|
-->

<center>
{% include image.html name="bp_bp_plus_ver_plot-1.png" caption="Plot of verification times of $64$-bit BP and BP+ range proofs for different aggregation sizes $m$." %}
</center>

[^1]: The challenge $x_u$ is used to multiply the given inner product $\langle \textbf{a},\textbf{b} \rangle$ in the exponent of a generator $g \in \mathbb{G}$. BP paper uses $u \in \mathbb{G}$ in Protocol 1, 2 instead of $g$. 

{% endkatexmm %}
