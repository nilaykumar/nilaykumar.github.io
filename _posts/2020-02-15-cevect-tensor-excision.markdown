---
layout: post
mathjax: true
draft: true
date: 2020-02-15
title:  "Lie algebra cohomology of vector fields is tensor-excisive"
---

<span hidden>
$ 
    \newcommand{\Vect}{\mathop{\rm Vect}\nolimits}
    \newcommand{\Sym}{\mathop{\rm Sym}\nolimits}
$
</span>

Recall that the Lie algebra cohmology of a real Lie algebra $\mathfrak{g}$ is the cochain complex
`$$
C^*(\mathfrak{g}) = \left(\Sym_\R^*(\mathfrak{g}^\vee[-1]), d\right)
$$`
with differential given on elements $\alpha\in\Sym^1(\mathfrak{g}^\vee[-1])$ by $d\alpha(X,Y)=-\alpha([X,Y])$ for $X,Y\in\mathfrak{g}$.

{% include thm type="definition" content="
Denote by $\mathsf{Mfld}_n$ the category of smooth $n$-manifolds with embeddings as morphisms.
For $M\in\mathsf{Mfld}_n$, denote by $\mathfrak{g}_M=\Vect M$ the Lie algebra of smooth vector fields on $M$.
Define a functor $F:\mathsf{Mfld}_n \to \mathsf{CAlg}$ by assigning to a manifold $M$ the commutative dg algebra $C^*(\mathfrak{g}_M)$.
" %}

Note that both $\mathsf{Mfld}_n$ and $\mathsf{CAlg}$ are symmetric monoidal categories under the operations of disjoint union $\sqcup$ and tensor product $\otimes$, respectively.
It is thus natural to check whether the functor $F$ is symmetric monoidal. Take $U,V\in\mathsf{Mfld}_n$. Then[^1]
`$$
\begin{align*}
    F(U\sqcup V) &= C^*(\mathfrak{g}_{U\sqcup V}) = C^*(\mathfrak{g}_U \times \mathfrak{g}_V) \\
    &= \Sym^*\left( (\mathfrak{g}_U\times\mathfrak{g}_V)^\vee[-1] \right) \\
    &??? \\
    &\cong \Sym^*(\mathfrak{g}_U^\vee[-1]) \otimes \Sym^*(\mathfrak{g}_V^\vee[-1]) \\
    &=C^*\mathfrak{g}_U \otimes C^*\mathfrak{g}_V.
\end{align*}
$$`
The goal of this post is to prove the following result.
{% include thm type="theorem" content="
The functor $F$ defined above is $\otimes$-excisive, i.e. given a collar-gluing of manifolds $M_1\cup_{M_0\times\R}M_2\cong M$, the map
$$ F(M_1) \otimes_{F(M_0\times\R)} F(M_2) \to F(M) $$
is an equivalence.
" %}

## References
<!-- REFERENCES -->


## Footnotes
[^1]: Our Lie algebras here are infinite-dimensional... how does this work? Perhaps I need to be more careful about topologies on vector spaces from the get-go.


