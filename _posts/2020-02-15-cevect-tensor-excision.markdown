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
\\[ C^*(\mathfrak{g}) = \left(\Sym_\R^*(\mathfrak{g}^\vee[-1]), d\right) \\]
with differential given on elements $\alpha\in\Sym^1(\mathfrak{g}^\vee[-1])$ by $d\alpha(X,Y)=-\alpha([X,Y])$ for $X,Y\in\mathfrak{g}$.

{% include def.html content="
Denote by $\mathsf{Mfld}_n$ the category of smooth $n$-manifolds equipped with embeddings.
For $M\in\mathsf{Mfld}_n$, denote by $\Vect M$ the Lie algebra of smooth vector fields on $M$.
Define a functor $F:\mathsf{Mfld}_n \to \mathsf{CAlg}$ by assigning to a manifold $M$ the commutative dg algebra $C^*(\Vect M)$.
" %}

Note that both $\mathsf{Mfld}_n$ and $\mathsf{CAlg}$ are symmetric monoidal categories under the operations of disjoint union $\sqcup$ and tensor product $\otimes$, respectively.
It is thus natural to ask whether the functor $F$ is symmetric monoidal.

<!-- REFERENCES -->

