---
layout: project
title: "Comparing Bulletproofs+ and Bulletproofs - Part I"
categories: project
tldr: "Bulletproofs+ is a recently proposed log-sized range proof protocol building upon the current state-of-the-art Bulletproofs protocol. In this first part, we briefly discuss the key differences between the inner product argument and the weighted inner product argument on which the two protocols rely on respectively."
caption: "  "
---

{% katexmm %}
[Bulletproofs+](https://eprint.iacr.org/2020/735.pdf) is a recently proposed range proof which is similar in spirit to [Bulletproofs](https://eprint.iacr.org/2017/1066.pdf). Both of these range proof protocols enable proof sizes logarithmic in the number of bits of the range using a recursive inner product protocol. Bulletproofs relies on the improved inner product protocol [^1] while Bulletproofs+ uses a weighted inner product protocol [^2].

### Improved Inner Product Argument 

The recursive inner product argument introduced in [1] gives an argument of knowledge for the language given by

{% katex display %}
  \mathcal{L}_{\textsf{IP}}(\sigma) =
  \bigg\{ 
  \underbrace{\textbf{g}, \textbf{h} \in \mathbb{G}^n, P \in \mathbb{G}, c \in \ \mathbb{Z}_q}_{\textsf{statement}}; \  
  \underbrace{\textbf{a}, \textbf{b} \in \mathbb{Z}_q^n}_{\textsf{witness}} 
  \ \bigg| \ 
  \underbrace{P = \textbf{g}^{\textbf{a}} \cdot \textbf{h}^{\textbf{b}} \cdot u^{c} 
  \ \wedge \ c = \langle \textbf{a}, \textbf{b} \rangle}_{\textsf{relation}}
  \bigg\}
{% endkatex %}

where $\sigma = (\mathbb{G}, q = |\mathbb{G}|, u \in \mathbb{G})$ is the *common reference string* (crs). 
Note that $P$ is a Pedersen vector commitment to $\textbf{a}, \textbf{b} \in \mathbb{Z}_q^n$.
We can write the witness as $\textbf{a} = (\textbf{a}_L, \textbf{a}_R) \in \mathbb{Z}_q^{\frac{n}{2} \times 2}$ and $\textbf{b} = (\textbf{b}_L, \textbf{b}_R) \in \mathbb{Z}_q^{\frac{n}{2} \times 2}$.
Similarly, the base vectors can be written as $\textbf{g} = (\textbf{g}_L, \textbf{g}_R) \in \mathbb{G}^{\frac{n}{2} \times 2}$ and $\textbf{h} = (\textbf{h}_L, \textbf{h}_R) \in \mathbb{G}^{\frac{n}{2} \times 2}$. Now, we compute Pedersen vector commitments to the vector tuples $(\textbf{a}_L, \textbf{b}_R)$ and $\textbf{a}_R, \textbf{b}_L$ as 

{% katex display %}
L = \textbf{g}_R^{\textbf{a}_L} \cdot \textbf{h}_L^{\textbf{b}_R} \cdot u^{\langle \textbf{a}_L, \textbf{b}_R\rangle}, \quad 
R = \textbf{g}_L^{\textbf{a}_R} \cdot \textbf{h}_R^{\textbf{b}_L} \cdot u^{\langle \textbf{a}_R, \textbf{b}_L\rangle}. 
{% endkatex %}

Given a challenge scalar $x \in \mathbb{Z}_q$, we can blind the original witness to get

{% katex display %}
\hat{\textbf{a}} = x\textbf{a}_L + x^{-1}\textbf{a}_R, \quad  
\hat{\textbf{b}} = x^{-1}\textbf{b}_L + x\textbf{b}_R. 
{% endkatex %}

The 4-tuple $(L,R,\hat{\textbf{a}}, \hat{\textbf{b}})$ becomes a valid proof of knowledge of the witness.
This can be verified by checking the following

{% katex display %}
L^{x^2} \cdot P \cdot R^{x^{-2}} = 
(\textbf{g}_L^{x^{-1}} \circ \textbf{g}_R^x)^{\hat{\textbf{a}}}
\cdot 
(\textbf{h}_L^{x} \circ \textbf{h}_R^{x^{-1}})^{\hat{\textbf{b}}}
\cdot u^{\langle \hat{\textbf{a}}, \hat{\textbf{b}}\rangle}
{% endkatex %}

Instead of publishing vectors $(\hat{\textbf{a}}, \hat{\textbf{b}})$, we can further repeat the same process of dividing $(\hat{\textbf{a}}, \hat{\textbf{b}})$ in halves and compute $(L_1,R_1,\hat{\textbf{a}}_1, \hat{\textbf{b}}_1)$ and so on until we are left with scalars $(\hat{a},\hat{b}) \in \Z_q$.
This is the main idea of the log-sized inner product protocol and the proof is of the form 

{% katex display %}
(L, R),\ (L_1, R_1), \ \ldots, \ (L_{\lceil \text{log}_2n \rceil - 1}, R_{\lceil \text{log}_2n \rceil - 1}), \ (\hat{a},\hat{b}).
{% endkatex %}

Note that the above argument is perfectly complete and sound, but not zero-knowledge. 
This is because we cannot construct a PPT simulator who could generate a valid transcript given statement and the crs.

### Weighted Inner Product Argument

The weighted inner product argument uses a weighted inner product operation $\odot_y: \mathbb{Z}^{n}_q \times \mathbb{Z}^{n}_q \mapsto \mathbb{Z_q}$ defined as 
{% katex display %}
\textbf{a} \odot_{y} \textbf{b} = \langle \textbf{a}, \overrightarrow{y} \circ \textbf{b} \rangle
{% endkatex %}
where $\overrightarrow{y} = (y, y^2, \ldots, y^n) \in \mathbb{Z}_q^n$ and $n = |\textbf{a}| = |\textbf{b}|$. The weighted inner product is essentially a way to combine multiple equations into a single equation. For instance, 
$\textbf{a} \circ \textbf{b} = \textbf{c} \in \mathbb{Z}_q^n$ represent $n$ distinct equations. 
The weighted inner product combines these equations to give a single equation of the form
{% katex display %}
\sum_{i=1}^{n} y^{i} \cdot (a_ib_i) = \textbf{a} \odot_{y} \textbf{b} = \langle \textbf{c}, \overrightarrow{y} \rangle
{% endkatex %} 

The weighted inner product protocol is also bilinear and satisfies the following property [^3]
{% katex display %}
\textbf{a} \odot_{y} \textbf{b} = \textbf{a}_L \odot_{y} \textbf{b}_L + (y^{\frac{n}{2}} \cdot \textbf{a}_R) \odot_{y} \textbf{b}_R.
{% endkatex %} 

We now wish to design a proof system based on the weighted inner product. 
The language of the weighted inner product protocol becomes
{% katex display %}
  \mathcal{L}_{\textsf{WIP}}(\sigma) =
  \bigg\{ 
  \underbrace{\textbf{g}, \textbf{h} \in \mathbb{G}^n, P \in \mathbb{G}, c \in \ \mathbb{Z}_q}_{\textsf{statement}}; \  
  \underbrace{\textbf{a}, \textbf{b} \in \mathbb{Z}_q^n}_{\textsf{witness}} 
  \ \bigg| \ 
  \underbrace{P = \textbf{g}^{\textbf{a}} \cdot \textbf{h}^{\textbf{b}} \cdot u^{c} 
  \ \wedge \ c = \textbf{a} \odot_{y} \textbf{b}}_{\textsf{relation}}
  \bigg\}
{% endkatex %}

Similar to the inner product protocol, suppose the prover commits to vectors $(\textbf{a}_L, \textbf{b}_R)$ and $(\textbf{a}_R, \textbf{b}_L)$ and the scalars $c_L = \textbf{a}_L \odot_{y} \textbf{b}_R, \ c_R = (y^{\frac{n}{2}} \cdot \textbf{a}_R) \odot_{y} \textbf{b}_L$,
the prover must convince the verifier of the relation $c = \textbf{a} \odot_y \textbf{b}$. 
For this, we blind the original witness vectors using a challenge scalar $x \in \mathbb{Z}_q$.
{% katex display %}
\hat{\textbf{a}} = x\textbf{a}_L + x^{-1}(y^{\frac{n}{2}} \cdot \textbf{a}_R), \quad  
\hat{\textbf{b}} = x^{-1}\textbf{b}_L + x\textbf{b}_R. \\[4pt]
\implies \hat{\textbf{a}} \odot_y \hat{\textbf{b}} = x^2c_L + c + x^{-2}c_R.
{% endkatex %}

Now, similar to the inner product argument, the 4-tuple $(L,R,\hat{\textbf{a}}, \hat{\textbf{b}})$ becomes a valid proof of knowledge of the witness. We can shrink this argument too to logarithmic in the size of witness vectors.
The WIP based argument presented in Bulletproofs+ paper is zero-knowledge as against the inner product argument.
This is achieved by including randomly chosen blinding factors $d_L, d_R \in \mathbb{Z}_q$ in computing the Pedersen vector commitments $L, R$ as

{% katex display %}
L = \textbf{g}_R^{\textbf{a}_L} \cdot \textbf{h}_L^{\textbf{b}_R} \cdot u^{\langle \textbf{a}_L, \textbf{b}_R\rangle} \cdot h^{d_L}, \quad 
R = \textbf{g}_L^{\textbf{a}_R} \cdot \textbf{h}_R^{\textbf{b}_L} \cdot u^{\langle \textbf{a}_R, \textbf{b}_L\rangle} \cdot h^{d_R}. 
{% endkatex %}

where $h \in \mathbb{G}$. Further, in the last round of the protocol, instead of sending $\hat{a}, \hat{b} \in \mathbb{Z}$, we use a sigma-like protocol yeilding constant communication and computation.
This is the main difference in the inner product and the weighted inner product arguments.
We will see in the next blog how this helps building Bulletproofs+ - a range proof protocol with proof size shorter than that of Bulletproofs.

### References and Notes

[^1]: Benedikt Bünz et al, "Bulletproofs: Short Proofs for Confidential Transactions and More", in *Cryptology ePrint Archive, Report 2017/1066*, Available at [https://eprint.iacr.org/2017/1066.pdf](https://eprint.iacr.org/2017/1066.pdf)  

[^2]: Chung H. et al, "Bulletproofs+: Shorter Proofs for Privacy-Enhanced Distributed Ledger", in *Cryptology ePrint Archive, Report 2020/735*, Available at [https://eprint.iacr.org/2020/735.pdf](https://eprint.iacr.org/2020/735.pdf).

[^3]: Note that $\textbf{a}_L \odot_{y} \textbf{b}_L = \langle \textbf{a}_L, (y,y^2, \ldots, y^{\frac{n}{2}}) \circ \textbf{b}_L \rangle$ for $\textbf{a}_L, \textbf{b}_L \in \mathbb{Z}_q^{\frac{n}{2}}$.

{% endkatexmm %}