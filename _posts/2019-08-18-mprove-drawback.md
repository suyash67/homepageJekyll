---
layout: project
title: "Solving the Chicken & Hen Problem of MProve"
categories: project
image: "monero.jpg"
tldr: "MProve is a recent proof of reserves protocol for Monero. Although equipped with the guarantees of non-collusion, inflation resistance and address privacy, it has some drawbacks. I attempted to overcome those in this work."
caption: "Image courtesy: Bitcoin.com"
---

[MProve](https://eprint.iacr.org/2018/1210.pdf) is the first proof of reserves for Monero. Monero [^1] is a decentralized cryptocurrency, meaning it is secure digital cash operated by a network of users. It uses ring signatures, ring confidential transactions, and stealth addresses to hide the origins, amounts, and destinations of all transactions. Transactions on the Monero blockchain are untraceable. Read more about Monero [here](https://web.getmonero.org/).

{% katexmm %}
MProve leverages the ring signatures to generate a proof of reserves. In simple words, it considers an anonymity set $\mathcal{P}_{\text{anon}}$ which includes $\mathcal{P}_{\text{own}}$ the set of exchange-owned addresses. To generate MProve, an exchange selects $\mathcal{P}_{\text{anon}} = \{P_1, P_2, \dots, P_n\}$ from Monero blockchain. The commitment $C_i = g^{y_i} h^{a_i}$ for each $P_i \in \mathcal{P}_{\text{anon}}$. The exchange defines the quantity $C_i^{\prime}$ as follows 
{% katex display %}
C_i^{\prime} \coloneqq 
\begin{cases}
g^{z_i} & \text{if } P_i \in \mathcal{P}_{\text{own}} \\
g^{z_i}C_i & \text{if } P_i \notin \mathcal{P}_{\text{own}}
\end{cases}
{% endkatex %}
where $z_i$ is randomly chosen from $\mathbb{Z}_q$. Further, the exchange publishes ring signatures $\gamma_i \ \forall i \in \{1, \dots, n\}$ verifiable by the pair of public keys $(C_i^{\prime}, C_i^{\prime} - C_i)$ and linkable ring signatures $\sigma_i \ \forall i \in \{1, \dots, n\}$ verifiable by the pair of public keys $(P_i, C_i^{\prime} - C_i)$. Each $\sigma_i$ contains key-images $I_i$. These key-images help detect collusion and double spending. Checking if the published key-image doesn't match existing key-images on the blockchain ensures that the exchange is not using an already-spent address in its proof. Thus, a typical MProve is of the form 
{% katex display %}
\Pi_{\text{M}} = \{ P_i, C_i, C_i^{\prime}, \gamma_i, \sigma_i \}_{i=1}^{n}
{% endkatex %}

The problem is with the key-images $I_i$ for $i$ s.t $P_i \in \mathcal{P}_{\text{own}}$ which are a part of $\sigma_i$. Let's say an exchange publishes a MProve proof at time $t_1$. Now if that exchange tries to spend from address $P_i \in \mathcal{P}_{\text{own}}$ at some time $t_2 > t_1$, the transaction also includes the same key-image $I_i$ as the one present in the MProve proof.
So, any observer will know that address $P_i$ was owned by that exchange from the proof given at $t_1$. An obvious way out of this seems to change the way we define key-images in MProve. But then we won't be able to detect double spending in that case. Revealing key-images seem to be unavoidable.
{% endkatexmm %}

The way I thought I would solve this paradoxical situtation was to somehow prove that the key-images we generate do not use any of the private information of the existing spent addresses on the blockchain. Solving MProve drawback essentially means that we come up with an efficient proof of
reserves which does two things:
1. Checks for double spending using the already existing key images
2. Ensure non-collusion between exchanges

{% katexmm %}
Specifically, let $P_i$ be a one-time address owned by the exchange with $y_i$ as the secret key, and let $I_j$ be a key-image from the blockchain, indicating that the funds from address $P_j$ have already been spent, we have
{% katex display %}
P_i = g^{x_i}, \ I_j = H(P_j)^{x_j}
{% endkatex %}

If we prove, for each $P_i \in \mathcal{P}_{\text{own}}$, that the discrete-log of $I_j$ w.r.t $H(P_i)$ is not equal to the discrete-log of $P_i$ w.r.t $g \in \mathbb{G}$, for each spent address $P_j$ on the Monero blockchain, we are done. We can now have another definition of key-images in out proof of reserves. This fixes the paradoxical situation. Stay tuned to know how exactly we do it as we are in process of writing the manuscript of the protocol!
{% endkatexmm %}


[^1]: [Monero Project](https://web.getmonero.org/)