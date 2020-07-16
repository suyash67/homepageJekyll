---
layout: project
title: "Comparing Bulletproofs+ and Bulletproofs - Part II"
categories: project
tldr: "In this second part on comparing Bulletproofs+ and Bulletproofs, we briefly discuss the Bulletproofs range proof protocol. We have implemented Bulletproofs+ in KZen-networks' Bulletproofs library for quantitative comparison between the two protocols."
caption: "  "
---

{% katexmm %}
[Bulletproofs+](https://eprint.iacr.org/2020/735.pdf) is a recently proposed range proof which is similar in spirit to [Bulletproofs](https://eprint.iacr.org/2017/1066.pdf). Both of these range proof protocols enable proof sizes logarithmic in the number of bits of the range using a recursive inner product protocol. We will try to highlight a small but crucial difference in the design of the two protocols and discuss its implications on their performance.

### Logarithmic Range Proof Protocol 

Suppose we wish to prove that a given quantity $a \in \mathbb{Z}_q$ is in the range $[0,2^n - 1]$.
Let the bitwise representation of $a$ be $\textbf{a}_L \in \{0,1\}^n$ and define vector $\textbf{a}_R \in \mathbb{Z}_q^n$ as
{% katex display %}
\textbf{a}_R \coloneqq \textbf{a}_L - \textbf{1}^n
{% endkatex %}

If we prove that $\textbf{a}_L$ and $\textbf{a}_R$ satisfy the following relations simultaneously,
{% katex display %}
\langle \textbf{a}_L, \textbf{2}^n \rangle = a, \qquad \textbf{a}_L \circ \textbf{a}_R = \textbf{0}^n, \qquad \textbf{a}_R = \textbf{a}_L - \textbf{1}^n,
{% endkatex %}
then it implies that $a$ indeed lies in the range $[0,2^n - 1]$. To use the inner product argument,
we need to embed the above constraints in an inner product form as follows.
{% katex display %}
\langle \textbf{a}_L, \textbf{2}^n \rangle = a, \qquad 
\langle \textbf{a}_L,  \textbf{a}_R \circ \textbf{y}^n \rangle = 0, \qquad 
\langle \textbf{a}_L - \textbf{a}_R - \textbf{1}^n, \textbf{y}^n \rangle = 0.
\hspace{2cm}
(1)
{% endkatex %}

Using a random scalar $z \in \mathbb{Z}_q$, we combine the above inner product relations to get a single inner product relation as follows.
{% katex display %}
z^2 \cdot \langle \textbf{a}_L, \textbf{2}^n \rangle + 
z \cdot \langle \textbf{a}_L - \textbf{a}_R - \textbf{1}^n, \textbf{y}^n \rangle + 
\langle \textbf{a}_L,  \textbf{a}_R \circ \textbf{y}^n \rangle = z^2 \cdot a, \\[6pt]
\implies 
\langle \textbf{a}_L, z^2 \cdot \textbf{2}^n \rangle +
\langle \textbf{a}_L - \textbf{a}_R - \textbf{1}^n, z \cdot \textbf{y}^n \rangle +
\langle \textbf{a}_L,  \textbf{a}_R \circ \textbf{y}^n \rangle = z^2 \cdot a,
{% endkatex %}
Combining first and third terms, we get
{% katex display %} 
\langle \textbf{a}_L, \textbf{a}_R \circ \textbf{y}^n + z^2 \cdot \textbf{2}^n \rangle +
\langle \textbf{a}_L - \textbf{a}_R - \textbf{1}^n, z \cdot \textbf{y}^n \rangle, 
= z^2 \cdot a
{% endkatex %}
Splitting the second inner product,
{% katex display %}
\langle \textbf{a}_L, \textbf{a}_R \circ \textbf{y}^n + z^2 \cdot \textbf{2}^n \rangle +
\langle \textbf{a}_L, z \cdot \textbf{y}^n \rangle +
\langle \textbf{a}_R, -z \cdot \textbf{y}^n \rangle
= z^2 \cdot a + z \cdot \langle \textbf{y}^n, \textbf{1}^n  \rangle,
{% endkatex %}
Combining first two terms and rewritting the third term, we get
{% katex display %} 
\langle \textbf{a}_L, \textbf{a}_R \circ \textbf{y}^n + z^2 \cdot \textbf{2}^n + z \cdot \textbf{y}^n \rangle +
\langle \textbf{a}_R \circ \textbf{y}^n, -z \cdot \textbf{1}^n \rangle
= z^2 \cdot a + z \cdot \langle \textbf{y}^n, \textbf{1}^n  \rangle,
{% endkatex %}
Adding $\langle z^2 \cdot \textbf{2}^n + z \cdot \textbf{y}^n, -z \cdot \textbf{1}^n \rangle = -z^3 \cdot \langle \textbf{2}^n,\textbf{1}^n \rangle - z^2 \cdot \langle \textbf{y}^n,\textbf{1}^n \rangle$ on both sides, 
{% katex display %} 
\langle \textbf{a}_L, \textbf{a}_R \circ \textbf{y}^n + z^2 \cdot \textbf{2}^n + z \cdot \textbf{y}^n \rangle +
\langle \textbf{a}_R \circ \textbf{y}^n, -z \cdot \textbf{1}^n \rangle +
\langle z^2 \cdot \textbf{2}^n + z \cdot \textbf{y}^n, -z \cdot \textbf{1}^n \rangle \\[4pt]
= 
z^2 \cdot a + (z-z^2) \cdot \langle \textbf{y}^n, \textbf{1}^n  \rangle -
z^3 \cdot \langle \textbf{2}^n,\textbf{1}^n \rangle, 
{% endkatex %}
Combining second and third terms and substituting $\delta(y,z) = z^2 \cdot a + (z-z^2) \cdot \langle \textbf{y}^n, \textbf{1}^n  \rangle - z^3 \cdot \langle \textbf{2}^n,\textbf{1}^n \rangle,$ we get
{% katex display %} 
\langle \textbf{a}_L, \textbf{a}_R \circ \textbf{y}^n + z^2 \cdot \textbf{2}^n + z \cdot \textbf{y}^n \rangle +
\langle \textbf{a}_R \circ \textbf{y}^n + z^2 \cdot \textbf{2}^n + z \cdot \textbf{y}^n, -z \cdot \textbf{1}^n \rangle
=
y^{n+1} \cdot a+ \delta(y,z),
{% endkatex %}
Combining the two terms on LHS, we get 
{% katex display %} 
\langle 
\textbf{a}_L -z \cdot \textbf{1}^n, 
\textbf{y}^n \circ (\textbf{a}_R + z \cdot \textbf{1}^n) + z^2 \cdot \textbf{2}^n  
\rangle
=
y^{n+1} \cdot a+ \delta(y,z).
{% endkatex %}

Hereafter, the two vectors in the above inner product are our new *embedded* secrets and we can use the inner product argument to prove the knowledge of these vectors and in turn prove that $a \in [0, 2^n-1]$.
Before doing so, we need to blind the embedded secret vectors since they contain our original secrets.
We do so by adding randomly chosen $\textbf{s}^L, \textbf{s}^R \in \mathbb{Z}_q^n$ to respective embedded vectors. Without going into the exact details of the protocol, such blinding makes the interaction between prover and verifier lengthy resulting in increased prover communication.

### Bulletproofs+

The weighted WIP protocol (as described in [previous]({{ site.baseurl }}/project/2020/07/03/bulletproofs_plus_part1.html) blog) w.r.t. $\overrightarrow{y}^n$ is a tailored protocol for proving a product relation between two hidden vectors $\textbf{a}_L$ and $\textbf{a}_R$ and the challenge $\overrightarrow{y}^n$ with zero-knowledge. Therefore, the prover doesn't need to blind the embedded secret vectors as in the case of Bulletproofs, saving significant prover communication.

Let us rewrite the inner product relation between embedded secret vectors in a weighted inner product form. Using scalars $(y^{n+1}, z \cdot \overrightarrow{y}^n, \overrightarrow{y}^n) \in \mathbb{Z}_q^{2n+1}$, we combine the inner product relations in $(1)$ to get a single inner product relation as follows.
{% katex display %}
y^{n+1} \cdot \langle \textbf{a}_L, \textbf{2}^n \rangle + 
z \cdot \langle \textbf{a}_L - \textbf{a}_R - \textbf{1}^n, \overrightarrow{y}^n \rangle + 
\langle \textbf{a}_L,  \textbf{a}_R \circ \overrightarrow{y}^n \rangle = y^{n+1} \cdot a,
{% endkatex %}
Simplifying the above equation into a single inner product relation, we get
{% katex display %} 
\langle 
\textbf{a}_L -z \cdot \textbf{1}^n, 
\overrightarrow{y}^n \circ (\textbf{a}_R + z \cdot \textbf{1}^n) + y^{n+1} \cdot \textbf{2}^n  
\rangle
=
y^{n+1} \cdot a - \zeta(y,z),
{% endkatex %}
where $\zeta(y,z)= zy^{n+1} \cdot \langle \textbf{2}^n,\textbf{1}^n \rangle + (z^2-z) \cdot \langle \textbf{1}^n,\overrightarrow{y}^n \rangle$. We can write the above expression in a weighted inner product form for a constant $y \in \mathbb{Z}_q$
{% katex display %} 
(\textbf{a}_L -z \cdot \textbf{1}^n)
\odot_y 
(\textbf{a}_R + z \cdot \textbf{1}^n + \textbf{2}^n \circ \overleftarrow{y}^n)
=
y^{n+1} \cdot a - \zeta(y,z).
{% endkatex %}
Now, the prover can compute a Pedersen commitment to vectors $\textbf{a}_L$ and $\textbf{a}_R$ as
$A = \textbf{g}^{\textbf{a}_L} \textbf{h}^{\textbf{a}_R} h^{\alpha}$ for a random $\alpha \in \mathbb{Z}_q$ and a Pedersen commitment to the amount $V = g^{a}h^{\gamma}$ for a random $\gamma \in \mathbb{Z}_q$. Hereafter, the prover as well as the verifier can compute a Pedersen vector commitment to the vectors in the weighted inner product relation. Concretely, the prover computes the secrets $\hat{\textbf{a}}_L = (\textbf{a}_L -z \cdot \textbf{1}^n)$, $\hat{\textbf{a}}_R = (\textbf{a}_R + z \cdot \textbf{1}^n + \textbf{2}^n \circ \overleftarrow{y}^n)$ and $\hat{\alpha} = \alpha + \gamma \cdot y^{n+1}$.
The verifier can compute Pedersen vector commitment to $\hat{\textbf{a}}_L, \hat{\textbf{a}}_R$ from publicly available information as
{% katex display %} 
\hat{A} = 
\underbrace{\textbf{g}^{\hat{\textbf{a}}_L} \cdot
\textbf{h}^{\hat{\textbf{a}}_R} \cdot
g^{\hat{\textbf{a}}_L \odot_y \hat{\textbf{a}}_R} \cdot
h^{y^{n+1} \cdot \gamma }}_{\textsf{Prover}}
=
\underbrace{A \cdot
\textbf{g}^{-z \cdot \textbf{1}^n} \cdot 
\textbf{h}^{z \cdot \textbf{1}^n + \textbf{2}^n \circ \overleftarrow{y}^n} \cdot
V^{y^{n+1}} \cdot g^{-\zeta(y,z)}}_{\textsf{Verifier}}
{% endkatex %}

Hereafter, the prover can run $zk\textrm{-}\textsf{WIP}(\textbf{g}, \textbf{h}, g, h, \hat{A}; \hat{\textbf{g}}, \hat{\textbf{g}}, \hat{\alpha})$.

### Performance Comparison

Although the construction of Bulletproofs and Bulletproofs+ is very similar, we expect some differences in their performances. We will analyse the differences in proof sizes and running times.  

#### Proof sizes

For secret vector size $n$, the proof size of the inner product (IP) argument used in Bulletproofs is $2\lceil \text{log}_2(n) \rceil$ elements in $\mathbb{G}$ and $2$ elements in $\mathbb{Z}_q$.
On the other hand, the proof size for the weighted inner product (WIP) argument is $2\lceil \text{log}_2(n) \rceil + 2$ elements in $\mathbb{G}$ and $3$ elements in $\mathbb{Z}_q$.
The additional elements in the WIP argument as against IP argument is because WIP argument is zero-knowledge and IP argument is not zero-knowledge. We summarise the proof sizes of aggregated Bulletproofs and Bulletproofs+ protocols in the following table. Note that if $m$ is the number of proofs aggregated, the proof size increases by only $2\lceil \text{log}_2(m) \rceil$ group elements.

<p align=center><EM>Table 1. Proof size comparison</EM></p>

| | # Elements in $\mathbb{G}$ | # Elements in $\mathbb{Z}_q$ |
Inner Product | $2\lceil \text{log}_2(n) \rceil$ | $2$ |
Bulletproofs (aggregated) | $2\lceil \text{log}_2(n) + \text{log}_2(m) \rceil + 4$ | $5$ |
Weighted Inner Product | $2\lceil \text{log}_2(n) \rceil + 2$ | $3$ |
Bulletproofs+ (aggregated) | $2\lceil \text{log}(n) + \text{log}_2(m) \rceil + 3$ | $3$ |

#### Running Times 

We have implemented Bulletproofs+ in the existing implementation of Bulletproofs in KZen-Networks' [bulletproofs](https://github.com/KZen-networks/bulletproofs) library.
The prover and verifier's computation is $\mathcal{O}(n)$ in both Bulletproofs and Bulletproofs+.
However, verification can significantly boosted by using a single multi-exponentiation check (read [this](https://github.com/KZen-networks/bulletproofs/pull/20#discussion_r436681850) to briefly understand how multi-exponentiation works). 
Batching proofs together further improves generation and verification times.
In most cryptocurrency systems, range proofs are required for proving that amounts hidden in Pederson commitments are 64-bit non-negative integers.
Thus, we compare proof sizes [^1] and running times for $n=64$ and different batching configurations as shown in the following table.
All simulations were performed on an Intel® Core™ i7-5500U CPU running at 2.40GHz.



### Applicability to Grin

[Grin](https://grin.mw/) is a cryptocurrency project built using the [MimbleWimble](https://github.com/mimblewimble/grin/blob/master/doc/intro.md) protocol.
It promises scalability, privacy and fungibility all at once.
The amounts in Grin are hidden in *outputs* which are Pedersen commitments.
Each output on the Grin blockchain is accompanied by a range proof proving that the amount hidden in it is in the range $[0,2^{64}-1]$. These range proofs constitute about 98% of the total blockchain size.
Thus, range proofs with smaller proof sizes are very crucial. 
Grin currently [employs](https://github.com/mimblewimble/grin/pull/711) Bulletproofs. 

We can further improve the range proofs used in Grin by employing Bulletproofs+.
For a March 2020 snapshot of the Grin blockchain, we have $N_b = 612,102$ valid blocks.
The total number of outputs on the Grin blockchain were $N_o = 3,185,556$ out of which $N_{utxo} = 124,034$ are UTXOs (unspent transaction outputs). The total number of transactions (equals to the number of kernels) in Grin were $N_{tx} = 1,765,941$. Thus, we have
- Number of outputs per transactions: $n_{optx} = \frac{N_o}{N_{tx}} = 1.80 \approx 2$
- Number of outputs per block: $n_{opbl} = \frac{N_o}{N_{b}} = 5.20 \approx 6$
- Number of trasactions per block: $n_{txpbl} = \frac{N_{tx}}{N_{b}} = 2.88 \approx 3$  

Therefore, using Bulletproofs+, 200 bytes per transaction can be saved.
Since we have 3 transactions per block on an average, 600 bytes per block can be saved.
The typical number of Grin blocks mined in a day is 1500 (~1 block per minute).
This implies that everyday, on an average, 1 MB of data can be saved from being added on to the blockchain, 
Further, if a transaction contains large number of outputs, the fees for inclusion of that transaction is higher because of its greater size. Thus, transaction fees also would be reduced slightly on employing Bulletproofs+.
Further, currently the UTXO set contains $\approx 165,000$ outputs. If we were to employ Bulletproofs+, the UTXO set size would reduce by about 16 MB. 
Moreover, proof generation and verification can be faster by 20% and 16% respectively using Bulletproofs+ in place of Bulletproofs.



### Applicability to Monero

[Monero](https://web.getmonero.org/) is one of the first privacy-centered cryptocurrency project built on the [CryptoNote](https://cryptonote.org/whitepaper.pdf) protocol. The amounts in Monero are also Pedersen commitments and the Bulletproofs is [used](https://github.com/monero-project/monero/tree/master/src/ringct) to prove that they are in the range $[0,2^{64}-1]$. Thus, Bulletproofs+ could be used instead of Bulletproofs to improve performance.

More specifically, each Monero transaction contains $2.5$ outputs on an average.
This means that each transaction is accompanied by $2.5$ range proofs (Bulletproofs as of now).
As Monero uses the $\texttt{Ed25519}$ curve, the size of a single Bulletproofs proof is $676$ bytes [^2].
This could be reduced by $96$ bytes by using Bulletproofs+.
In effect, about $240$ bytes per transaction could be saved. 
Average number of transactions on 16th July was $11,417$.
This implies that about $2.7$ MB of data can be saved everyday [^4]. 
From the start of 2020 till date, on an average $28$ MB of data is added daily to the blockchain [^5].
Therefore, Bulletproofs+ can save about $10\%$ of the data being added to the Monero blockchain!

The running times of Bulletproofs and Bulletproofs+ over $\texttt{Ed25519}$ curve are noted below in Table 3.
Note that the underlying Edwards curve's operations are used from [cryptoxide](https://lib.rs/crates/cryptoxide) library which is slower than the optimized operations in the more recent [curve25519-dalek](https://github.com/dalek-cryptography/curve25519-dalek)'s implementation.
As we are interested only in comparison as of now, slower operations in the underlying curve library won't matter [^6]. Bulletproofs+ shows a $21\%$ faster proof generation and $17\%$ faster proof verification than Bulletproofs. Although these numbers would slightly vary when the underlying library for group operations changes, they still look quite promising from a user (prover) as well as miner (verifier) point of view! 


[^1]: Grin uses $\texttt{secp256k1}$ curve in which private keys are $32$ bytes in size and public keys are $33$ bytes. Thus, the size of an element in $\mathbb{G}$ is $33$ bytes and that of an element in $\mathbb{Z}_q$ is $32$ bytes.

[^2]: Monero uses the Edwards curve $\texttt{Ed25519}$ in which public and private keys are $32$ bytes each.

[^3]: Monero Block Explorer, https://moneroblocks.info/stats/transaction-stats

[^4]: If we assume that the range proofs for outputs per block are aggregated (i.e. they are owned by a single entity), then Bulletproofs+ can save about $1.1$ MB of data. 

[^5]: Monero Blockchain Growth, https://moneroblocks.info/stats/blockchain-growth

[^6]: The differences in running times of BP and BP+ will of course vary for different underlying curve implementations. 

<!-- <p align=center><EM>Performance comparison of Bulletproofs and Bulletproofs+ over $\texttt{secp256k1}$ curve</EM></p> -->
<TABLE border="1">
<TR align=center><EM>Table 2. Performance comparison of Bulletproofs and Bulletproofs+ over $\texttt{secp256k1}$ curve</EM></TR>
<TR><TH rowspan="3">$m$
<TR><TH colspan="2">Proof size (B)<TH colspan="2">Generation time (ms)<TH colspan="2">Verification time (ms)
<TR><TH>BP<TH>BP+<TH>BP<TH>BP+<TH>BP<TH>BP+
<TR><TH>1
<TD>688<TD>591
<TD>75.0<TD>59.3 &nbsp;&nbsp;<small style="color:green"><i class="fas fa-long-arrow-alt-down" style="color:green"></i>20.9%</small>
<TD>30.8<TD>25.3 &nbsp;&nbsp;<small style="color:green"><i class="fas fa-long-arrow-alt-down" style="color:green"></i>17.8%</small>
<TR><TH>4
<TD>820<TD>723
<TD>295.1<TD>234.1 &nbsp;&nbsp;<small style="color:green"><i class="fas fa-long-arrow-alt-down" style="color:green"></i>20.7%</small>
<TD>116.1<TD>97.5 &nbsp;&nbsp;<small style="color:green"><i class="fas fa-long-arrow-alt-down" style="color:green"></i>16%</small>
<TR><TH>8
<TD>886<TD>789
<TD>590.9<TD>473.5 &nbsp;&nbsp;<small style="color:green"><i class="fas fa-long-arrow-alt-down" style="color:green"></i>19.8%</small>
<TD>229.1<TD>197.5 &nbsp;&nbsp;<small style="color:green"><i class="fas fa-long-arrow-alt-down" style="color:green"></i>13.8%</small>
<TR><TH>16
<TD>952<TD>855
<TD>1187.4<TD>978.0 &nbsp;&nbsp;<small style="color:green"><i class="fas fa-long-arrow-alt-down" style="color:green"></i>17.6%</small>
<TD>469.4<TD>406.3 &nbsp;&nbsp;<small style="color:green"><i class="fas fa-long-arrow-alt-down" style="color:green"></i>13.5%</small>
<TR><TH>32
<TD>1018<TD>921
<TD>2344.9<TD>2127.2 &nbsp;&nbsp;<small style="color:green"><i class="fas fa-long-arrow-alt-down" style="color:green"></i>9.3%</small>
<TD>927.7<TD>885.6 &nbsp;&nbsp;<small style="color:green"><i class="fas fa-long-arrow-alt-down" style="color:green"></i>4.5%</small></TD>


<!-- <p align=center><EM>Performance comparison of Bulletproofs and Bulletproofs+ over $\texttt{Ed25519}$ curve</EM></p> -->
<TABLE border="1">
<TR align=center><EM>Table 3. Performance comparison of Bulletproofs and Bulletproofs+ over $\texttt{Ed25519}$ curve</EM></TR>
<TR><TH rowspan="3">$m$
<TR><TH colspan="2">Proof size (B)<TH colspan="2">Generation time (ms)<TH colspan="2">Verification time (ms)
<TR><TH>BP<TH>BP+<TH>BP<TH>BP+<TH>BP<TH>BP+
<TR><TH>1
<TD>672<TD>576
<TD>140.5<TD>109.8 &nbsp;&nbsp;<small style="color:green"><i class="fas fa-long-arrow-alt-down" style="color:green"></i>21.8%</small>
<TD>56.5<TD>46.8 &nbsp;&nbsp;<small style="color:green"><i class="fas fa-long-arrow-alt-down" style="color:green"></i>17.2%</small>
<TR><TH>4
<TD>800<TD>704
<TD>550.3<TD>431.6 &nbsp;&nbsp;<small style="color:green"><i class="fas fa-long-arrow-alt-down" style="color:green"></i>21.5%</small>
<TD>214.3<TD>179.4 &nbsp;&nbsp;<small style="color:green"><i class="fas fa-long-arrow-alt-down" style="color:green"></i>16.3%</small>
<TR><TH>8
<TD>864<TD>768
<TD>1095.8<TD>864.8 &nbsp;&nbsp;<small style="color:green"><i class="fas fa-long-arrow-alt-down" style="color:green"></i>21.0%</small>
<TD>423.5<TD>359.0 &nbsp;&nbsp;<small style="color:green"><i class="fas fa-long-arrow-alt-down" style="color:green"></i>15.2%</small>
<TR><TH>16
<TD>928<TD>832
<TD>2190.3<TD>1772.0 &nbsp;&nbsp;<small style="color:green"><i class="fas fa-long-arrow-alt-down" style="color:green"></i>19.0%</small>
<TD>839.8<TD>730.0 &nbsp;&nbsp;<small style="color:green"><i class="fas fa-long-arrow-alt-down" style="color:green"></i>13.0%</small>
<TR><TH>32
<TD>992<TD>896
<TD>4392.0<TD>3687.4 &nbsp;&nbsp;<small style="color:green"><i class="fas fa-long-arrow-alt-down" style="color:green"></i>14.8%</small>
<TD>1702.4<TD>1528.3 &nbsp;&nbsp;<small style="color:green"><i class="fas fa-long-arrow-alt-down" style="color:green"></i>10.2%</small></TD>  
<!-- </TABLE>   -->





{% endkatexmm %}
