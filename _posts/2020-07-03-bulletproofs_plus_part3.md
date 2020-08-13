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

Coming soon...

[^1]: The challenge $x_u$ is used as to multiply the given inner product in the exponent of a generator $g \in \mathbb{G}$. BP paper uses $u \in \mathbb{G}$ in Protocol 1, 2 instead of $g$. 

{% endkatexmm %}
