+++
title = "one weird kernel trick"
author = ["Nilay Kumar"]
date = 2022-02-03T00:00:00-05:00
draft = false
+++

I recently stumbled upon a programming interview question and the first solution
that sprang to mind used a sprinkling of algebraic topology. I thought this
might be a fun little pedagogical way to introduce the idea of homology, so here
goes![^fn:1] I'll assume familiarity with matrices, the basics of
linear algebra, and a bit of python.

Say we're given a 2-dimensional matrix where each cell consists of either a
forward slash, a backward slash, or a space. We'll write a program to compute
the number of regions in the resulting space, treating the matrix edges as
boundaries. Given, for example, the 3-by-6 matrix,

```text
\    /
 \  /
  \/
```

our program will output 3.


## brainstorming {#brainstorming}

It's always good to start out with a handful of examples to think about. Let's
draw them out as strings, which we will then convert to 2-dimensional `numpy`
arrays.

```python
# we'll need numpy for working with our matrices
import numpy as np
# and some tools for solving linear systems of equations
from scipy import linalg

examples = [
    ' ',
    '/',
    (
    '\    /\n'
    ' \  / \n'
    '  \/  '
    ),
    (
    '\/\n'
    '/\\'
    ),
     (
    '\/  \/\n'
    ' \/\/ \n'
    '  \/  '
    )
]

# hopefully I counted these correctly
answers = [1, 2, 3, 4, 6]

# convert our string drawings to 2d numpy arrays
# recall that list converts a string to a list of characters
def string_to_np(s):
    return np.array(list(map(list, s.split('\n'))))
```

It might also help to draw in the edges of our matrix to help us visualize the
regions a little better.

```python
# add in the boundaries along the edges of the matrix
def viz_boundaries(matrix):
    vertical = np.full((matrix.shape[0], 1), '|')
    left_and_right = np.hstack((vertical, matrix, vertical))
    horizontal = np.full((1, matrix.shape[1] + 2), '-')
    boundarized = np.vstack((horizontal, left_and_right, horizontal))
    s = ''
    for i in range(matrix.shape[0] + 2):
        for j in range(matrix.shape[1] + 2):
            s += boundarized[i][j]
        s += '\n'
    return s

# visualize our examples
for i, example in enumerate(examples):
    print(f'Example {i}:')
    print(viz_boundaries(string_to_np(example)))
```

```text
Example 0:
---
| |
---

Example 1:
---
|/|
---

Example 2:
--------
|\    /|
| \  / |
|  \/  |
--------

Example 3:
----
|\/|
|/\|
----

Example 4:
--------
|\/  \/|
| \/\/ |
|  \/  |
--------
```

What makes a region a region? Well, let's see. Clearly it has to be
two-dimensional. That's an obvious statement, and one that we probably don't
really think explicitly about. Less trivially, our regions all have
1-dimensional boundaries: each is enclosed by a trail of line segments that
forms a loop. Aha, so we just need to figure out how to get all the loops in our
space!

Well, hold up a minute, sure every region gives us a
boundary loop, but not every loop corresponds to one of our regions. Take
Example 1 from before: the loop around the edge of the matrix corresponds to the
union of two triangular regions that we'd like to count. Even worse, we could
take pathological loops that go around a region 17 times! That corresponds to
the same region as the loop that just goes around once does... I guess?


## formalizing our intuition {#formalizing-our-intuition}

Let's think a bit more carefully about that first issue we're running into,
still in the context of Example 1. It'll help to number the corners --- the
vertices --- of our loops.

```python
'''
0 1
---
|/|
---
2 3
'''
```

If we choose to start from the top-left corner, we can write the loop that goes
around the edge of the matrix as the list of line segments `[01,13,32,20]`. The
regions we're really after, however, are bounded by `[01,12,20]` and
`[21,13,32]`. Since the quadrilateral region \\(Q\\) is a union, a sum of sorts,
of the two triangular regions \\(T\_1\\) and \\(T\_2\\), we should be able to somehow
write the boundary \\(\partial Q\\) of the quadrilateral as a sum of the boundaries \\(\partial T\_1\\)
and \\(\partial T\_2\\) of the triangles.

Here's the secret sauce: let's try to take the idea of sums more literally.
Okay, so \\(Q = T\_1 + T\_2\\) and

\begin{equation\*}
\partial Q = e\_{01} + e\_{13} + e\_{32} + e\_{20},
\end{equation\*}

where \\(e\_{ij}\\) represents the edge running from the \\(i\\)th vertex to the
\\(j\\)th vertex (it's kinda awkward to just write something like \\(01 + 13\\)). If
we cross our fingers and ask that this \\(\partial\\) operator, which seems to send
2-dimensional regions to 1-dimensional boundaries, distributes over sums, then

\begin{equation\*}
\partial Q = \partial (T\_1 + T\_2) = \partial T\_1 + \partial T\_2.
\end{equation\*}

We have an expression for the left (the four terms above) and on the right we have

\begin{equation\*}
e\_{01} + e\_{12} + e\_{20} + e\_{21} + e\_{13} + e\_{32}.
\end{equation\*}

Canceling out the common terms, we arrive at

\begin{equation\*}
e\_{12} + e\_{21} = 0.
\end{equation\*}

Okay, so this is just expressing that travelling along the edge
\\(e\_{12}\\) should be the same as travelling backwards along \\(e\_{21}\\).

Since we're now working implicitly with a directed graph, let's go ahead and
rewrite our paths from earlier, but this time respecting the ordering on the
numbering of the vertices:

\begin{align\*}
\partial Q &= e\_{01} + e\_{13} - e\_{23} + e\_{20}, \\\\
\partial T\_1 &= e\_{01} + e\_{12} - e\_{20}, \\\\
\partial T\_2 &= -e\_{12} + e\_{13} - e\_{23}.
\end{align\*}

This gives us a way to express our intuition that \\(Q\\) is formed by putting \\(T\_1\\)
and \\(T\_2\\) together. One interesting thing to note is that \\(\partial (17 T\_1) = 17 \partial
T\_1\\), so the pathological boundary looping around 17 times that we mentioned
above is, in these terms, the boundary of the sum of 17 copies of \\(T\_1\\). That's
kinda gnarly, but this doesn't help solve our initial problem. We can write down
a bunch of loops in the notation of above, but how do we get rid of those coming
from composite regions like \\(Q\\)? For that, we're going to have be a little less
wishy-washy handwavy about these symbols that we're pushing around.


## introducing vector spaces {#introducing-vector-spaces}

How do we start working more formally with these regions and their boundaries?
For that we're going to need a bit of linear algebra. The operations we've been
carrying out above are pretty simple: addition (\\(e\_{01} + e\_{12}\\)) and scalar
multiplication (\\(-1 \cdot e\_{12}\\)). These should remind you of vectors in a vector
space, I hope! As a quick review: let's think about the vector space \\(\R^2\\).
This is a vector space (over the real numbers) generated by two basis vectors.
That is to say, any two linearly independent vectors will span the whole space.
I'm personally fond of \\(e\_1 = (1, 0)^T\\) and \\(e\_2 = (0, 1)^T\\), but you could just
as well work with \\(\\{7e\_1, e\_1 + \pi e\_2\\}\\) if you enjoy suffering. Hence why we
say that \\(\R^2\\) is 2-dimensional.[^fn:2] Let's suppose, now, that we could get our
hands on a vector space of all loops in \\(X\\). Those pesky loops that correspond
to composite regions are sums of the 'irreducible' loops that we're after. All
we really care about are the _linearly independent_ loops. In other words, the
problem is just asking us to compute the dimension of this vector space of
loops! That's the key insight.

Let's make this precise. If we write out the space in Example 1 as \\(X\\) and the
set of edges as \\(X\_1\\), we can consider the real vector space \\(\R^{|X\_1|}\\) of
dimension \\(|X\_1|\\) --- that is, a vector space with as many directions as there
are edges in \\(X\_1\\). So, since there are 5 elements of \\(X\_1\\), we're working with
\\(\R^5\\), and we may as well choose a basis

\begin{align\*}
e\_{01} &= (1, 0, 0, 0, 0)^T \\\\
e\_{02} &= (0, 1, 0, 0, 0)^T \\\\
e\_{12} &= (0, 0, 1, 0, 0)^T \\\\
e\_{13} &= (0, 0, 0, 1, 0)^T \\\\
e\_{23} &= (0, 0, 0, 0, 1)^T
\end{align\*}

The way in which we've chosen these doesn't really matter, but these basis
vectors are easy on the eyes. An arbitrary vector in \\(\R^5\\) is going to be as
sum of these basis edges, which we can think of as a possibly-disconnected
path in \\(X\\). How do we figure out whether a vector is a loop or not?
Because we need a nice connected path, so if \\(v = e\_{01} + \cdots\\), there'd
better be a \\(e\_{1j}\\) (for some \\(j\\)) hiding somewhere in there too (or a
\\(e\_{j0}\\)). To be a loop is harder because we also need that the edges end up
where they started.

Let's think geometrically again. Remember our friend \\(\partial\\), who took 2-dimensional
regions to 1-dimensional loops. If we shift our dimensions down by one, we can
think about the boundary of a path. The boundary of a line segment is its two
endpoints: \\(\partial e\_{01} = e\_1 - e\_0\\). Two things to notice here. The first is that
we're implicitly introducing another vector space: a vector space \\(\R^{|X\_0|}\\)
with basis the vertices (here \\(X\_0\\) is the set of vertices of \\(X\\)). The second
is that funky minus sign. Intuitively, this sign captures the fact that \\(e\_{01}\\)
has a direction. We're moving _from_ the vertex \\(e\_0\\) to the vertex \\(e\_1\\).
Actually we already saw this when we computed, for example, \\(\partial T\_1 = e\_{01} +
e\_{12} - e\_{20}\\). Great, so this \\(\partial\\) operator can take regions to paths (loops,
in fact), and paths to points. Okay, so back to loops. Let's see what happens
when we take the boundary of the loop around \\(T\_1\\):

\begin{align\*}
\partial (\partial T\_1) &= \partial(e\_{01} + e\_{12} - e\_{20}) \\\\
&= e\_1 - e\_0 + e\_2 - e\_1 - (e\_2 - e\_0) \\\\
&= 0.
\end{align\*}

It's zero! In fact, loops are precisely the paths that have no boundary. This is
intuitive geometrically: there's no fixed start or endpoint of a loop because it
just sorta goes around in a circle.

We now have a linear algebraic criterion for determining whether a path in \\(X\\)
is a loop: check whether it's in the _kernel_ of \\(\partial\\)! To be a bit more precise,
we've constructed a linear transformation between two vector spaces:

\begin{align\*}
\partial : \R^{|X\_1|} &\to \R^{|X\_0|} \\\\
e\_{ij} &\mapsto e\_j - e\_i
\end{align\*}

and the loops in \\(X\\) are precisely the vectors in \\(\R^{|X\_1|}\\) that are sent to
zero in \\(\R^{|X\_0|}\\). This allows us to concretely compute the space of
loops --- we just need to compute the kernel of \\(\partial\\).


## computing the kernel: an example {#computing-the-kernel-an-example}

Let's focus on \\(X\\) being our Example 1 as usual. In this case we have 5 edges
and 4 vertices, so the boundary operator is a linear map \\(\partial: \R^5 \to \R^4\\)
determined by its action \\(\partial e\_{ij} = e\_j - e\_i\\) on the basis vectors. Remember
how we wrote the \\(e\_{ij}\\) as the standard basis vectors of \\(\R^5\\) earlier? Let's
do that for our space of vertices as well:

\begin{align\*}
e\_0 &= (1, 0, 0, 0)^T \\\\
e\_1 &= (0, 1, 0, 0)^T \\\\
e\_2 &= (0, 0, 1, 0)^T \\\\
e\_3 &= (0, 0, 0, 1)^T.
\end{align\*}

We can now write out what \\(\partial\\) looks like as a matrix! We know, for instance,
that,

\begin{equation\*}
\partial e\_{01} = \partial
\begin{pmatrix}
1 \\\ 0 \\\ 0 \\\ 0 \\\ 0
\end{pmatrix}
= e\_1 - e\_0 =
\begin{pmatrix}
-1 \\\ 1 \\\ 0 \\\ 0
\end{pmatrix}
\end{equation\*}

and so we can assemble the matrix:

\begin{equation\*}
\partial =
\begin{pmatrix}
-1 & -1 & 0 & 0 & 0 \\\\
1 & 0 & -1 & -1 &  0 \\\\
0 & 1 & 1 & 0 & -1 \\\\
0 & 0 & 0 & 1 & 1
\end{pmatrix}
\end{equation\*}

To find the kernel of \\(\partial\\), we want to solve the equation \\(\partial v = 0\\). If we write
\\(v = (x\_1, x\_2, x\_3, x\_4, x\_5)^T\\), a bit of algebra (or row reduction, if you
like) shows that the solutions are written

\begin{align\*}
\begin{pmatrix}
x\_3 - x\_5 \\\\
x\_5 - x\_3 \\\\
x\_3 \\\\
-x\_5 \\\\
x\_5
\end{pmatrix}
\end{align\*}

for \\(x\_3, x\_5 \in\R\\) free. We see immediately that the kernel is 2-dimensional,
which is awesome, because that's the answer we're after! Visually we have two
independent, irreducible loops \\(\partial T\_1\\) and \\(\partial T\_2\\). Congratulations! You've
computed your first so-called 'homology group'![^fn:3]

Rewriting in terms of our ordered basis for \\(\R^5\\), we find that the kernel of
\\(\partial\\) is spanned by

\begin{equation\*}
e\_{01} - e\_{02} + e\_{12} \text{ and } e\_{02} + e\_{23} - e\_{13} - e\_{01},
\end{equation\*}

coming from setting \\(x\_3 = 1, x\_5 = 0\\), and \\(x\_3 = 0, x\_5 = 1\\), respectively.
If we trace the vertices around Example 1, we can see that the first is exactly
\\(\partial T\_1\\), and the second is \\(-\partial Q\\). Why these two instead of \\(\partial T\_1\\) and \\(\partial T\_2\\),
the loops that we like to visualize? No fundamental reason really --- it's an
artifact of the bases we chose and the way we solved the system of equations.
We can always choose any \\(k\\) linearly independent vectors to span a
\\(k\\)-dimensional space. The coordinate-invariant object is the 2-dimensional
subspace \\(\ker\partial \subset \R^5\\), which doesn't care how we choose to map it out. Indeed,
we can always write \\(\partial T\_2\\) in terms of \\(\partial T\_1\\) and \\(-\partial Q\\):

\begin{equation\*}
\partial T\_1 - \partial Q = e\_{12} - e\_{13} + e\_{23}.
\end{equation\*}

Just to make sure we didn't make any algebra mistakes, let's ask `scipy` to
compute the dimension of the kernel for us.

```python
# the matrix constructed for the boundary operator in Example 1
example_d = np.array([[-1, -1, 0, 0, 0],
                      [1, 0, -1, -1, 0],
                      [0, 1, 1, 0, -1],
                      [0, 0, 0, 1, 1]])
# scipy can compute vectors spanning the kernel of a matrix
ker = linalg.null_space(example_d)
print(ker.round(4))
print(f'The kernel is {ker.shape[1]}-dimensional')
```

```text
[[-0.5    -0.3536]
 [ 0.5     0.3536]
 [ 0.     -0.7071]
 [-0.5     0.3536]
 [ 0.5    -0.3536]]
The kernel is 2-dimensional
```

You'll notice that the first column that `scipy` spits out is just \\(-\partial Q / 2\\),
but the second basis vector is who-knows-what. Definitely not the basis I'd have
chosen. Regardless, it's clear that the kernel is 2-dimensional. Sweet.


## the general solution {#the-general-solution}

To summarize, this little exercise in linear algebra shows us how to solve our
interview problem by computing the nullity of a certain matrix associated to our
shape. What we're doing when we do this is computing the number of
linearly independent loops in the shape. The loops are the vectors in
\\(\R^{|X\_1|}\\) sent to \\(0\in\R^{|x\_0|}\\) by \\(\partial\\), so the the dimension of the kernel
of \\(\partial\\) is what we're after. That's it!

What we've done above is a baby's first computation in algebraic topology, the
aptly named field of mathematics that uses algebraic techniques to attack
problems involving topology. We treated the given shape as what's called a
_simplicial complex_ (a one-dimensional simplicial complex, to be precise), and
computed its _first homology group_ (this was our real vector space \\(\text{ker
}\partial\\) of dimension 2). The adjective 'first' indicates the dimension of the
objects we were studying --- loops! I'll talk a bit more about homology below,
but first we had better get to coding up our solution.

We start by associating to our string a one-dimensional simplicial complex. To
create the matrix \\(\partial\\) above, all we needed was to number the vertices and the
edges, and then keep track of how things were connected. If our original given
matrix (whose entries held the slashes or spaces) was \\(m\times n\\) , we'll need an
\\((m+1) \times (n+1)\\) matrix to index the vertices. As a sanity check, recall our
Example 1 above had \\(m=1, n=1\\) and we needed 4 vertices \\(e\_0,\ldots, e\_3\\). We'll keep
a list called `complex` whose \\(i\\)th entry will contain the \\(j>i\\) for which we
have \\(e\_{ij}\in X\_1\\). So, in our example,

```python
'''
complex = [[1, 2], # e_{01} and e_{02}
           [2, 3], # e_{12} and e_{13}
           [3],    # e_{23}
           []]     # no edges starting at e_3
'''
```

To create `complex`, we just loop through the vertices from top left to bottom
right. If we're at a boundary vertex, we'll make sure to connect it to the
appropriate boundary vertices, but otherwise, if we're at a vertex on the
interior, we'll take a look below and add vertices according to the slashes we
find.

```python
def simplicial_complex(s):
    # get the numpy matrix
    matrix = string_to_np(s)
    mp1 = matrix.shape[0] + 1
    np1 = matrix.shape[1] + 1
    complex = []
    # loop through the vertices v at (i, j)
    for i in range(mp1):
        for j in range(np1):
            connections = []

            # special logic for vertices on the boundary
            # if v is in the top row, connect it to the vertex on its right
            if i == 0 and j != (np1 - 1):
                connections.append(i * np1 + j + 1)
            # if v is in the bottom row, connect it to the vertex on its right
            if i == (mp1 - 1) and j != (np1 - 1):
                connections.append(i * np1 + j + 1)
            # if v is in the left column, connect it to the vertex below it
            if j == 0 and i != (mp1 - 1):
                connections.append((i + 1) * np1 + j)
            # if v is in the right column, connect it to the vertex below it
            if j == (np1 - 1) and i != (mp1 - 1):
                connections.append((i + 1) * np1 + j)

            # check SW diagonal for /
            if i != (mp1 - 1) and j != 0 and matrix[i][j - 1] == '/':
                connections.append((i + 1) * np1 + j - 1)
            # check SE diagonal for \
            if i != (mp1 - 1) and j != (np1 - 1) and matrix[i][j] == '\\':
                connections.append((i + 1) * np1 + j + 1)

            # make sure they're sorted! This is important to faithfully
            # represent the linear algebra that we're doing
            connections.sort()
            complex.append(connections)
    return complex
```

Okay, so that's not too bad (ugh, indexing). Let's make sure we get what we
wanted.

```python
complex = simplicial_complex(examples[1])
for i, vertex in enumerate(complex):
    print(f'{i} | {vertex}')
```

```text
0 | [1, 2]
1 | [2, 3]
2 | [3]
3 | []
```

Looks good. Now we've gotta construct the matrix \\(\partial\\). This was actually pretty
simple. The number of rows was \\(|X\_0|\\) (the number of vertices) and the number
of columns was \\(|X\_1|\\) (the number of edges). For each edge \\(e\_{ij}\\), the map
\\(\partial\\) returns \\(e\_j - e\_i\\), so the column corresponding to \\(e\_{ij}\\) should have a
\\(-1\\) in the \\(i\\)th row and a \\(1\\) in the \\(j\\)th row. Zeroes everywhere else. Pretty
straightforward. We'll order our vertices in the obvious way, \\(e\_0, e\_1,\ldots\\), and
our edges by the starting vertex first and the ending vertex second (i.e.
reading left-to-right from the top-to-bottom in the output of our previous code
block ).

```python
def differential(complex):
    num_vertices = len(complex)
    num_edges = sum([len(row) for row in complex])
    # initialize our matrix with zeros
    d = np.zeros((num_vertices, num_edges), dtype = int)
    for start_vertex, v in enumerate(complex):
        for j, end_vertex in enumerate(v):
            # what's the column index of e_{start_vertex, end_vertex}?
            # count how many edges came above start_vertex in our ordering,
            # and then add the index of end_vertex in the list of vertices
            # that start_vertex is connected to
            column = sum([len(row) for i, row in enumerate(complex) if i < start_vertex]) + j
            d[start_vertex][column] = -1
            d[end_vertex][column] = 1

    return d
```

If we feed it Example 1...

```python
differential(simplicial_complex(examples[1]))
```

```text
array([[-1, -1,  0,  0,  0],
       [ 1,  0, -1, -1,  0],
       [ 0,  1,  1,  0, -1],
       [ 0,  0,  0,  1,  1]])
```

That's exactly the matrix that we had constructed earlier. You might notice that
I've called the function (by force of habit) `differential`. That's just a
synonym for 'boundary' in the context of homology theories.[^fn:4]

All that's left now is to compute its nullity. That's easy enough with `scipy`.

```python
def regions(s):
    # return the number of basis vectors in the basis computed by scipy
    # for the kernel of the boundary matrix d
    return linalg.null_space(differential(simplicial_complex(s))).shape[1]
```

If we run it on our examples, we find:

```python
print('Computed:')
print(list(map(regions, examples)))
print('Expected:')
print(answers)
```

```text
Computed:
[1, 2, 3, 4, 6]
Expected:
[1, 2, 3, 4, 6]
```

And there you have it! That's our solution to this interview problem![^fn:5] If
you take the time to write code to perform row-reduction and back-substitution,
you could even display a basis of loops that generate the kernel of \\(\partial\\) (in our
case earlier it was \\(\partial T\_1\\) and \\(-\partial Q\\)). Anyway, if you're interested in
learning a bit more about homology, stick around and I'll say a few words below.


## homology {#homology}

Okay, so now to get a bit more abstract.
We've been working with one-dimensional simplicial complexes, but in general we
can have vertices, edges, surfaces, volumes, etc. So if \\(X\\) is a simplicial
complex, \\(X\_k\\) represents the set of \\(k\\)-simplices. Given such an \\(X\\) in
general, we can form its _simplicial chain complex_:[^fn:6]

\begin{equation\*}
C\_\*(X) = \cdots \to C\_3(X) \xrightarrow{\partial\_3} C\_2(X) \xrightarrow{\partial\_2} C\_1(X) \xrightarrow{\partial\_1} C\_0(X) \to 0
\end{equation\*}

This is a sequence of vector spaces \\(C\_k(X) = \R^{|X\_k|}\\), where \\(C\_k\\) has
dimension the number of \\(k\\)-simplices in \\(X\\), together with a sequence of linear
transformations between them. The notation might be a bit intimidating, but it's
really quite concrete. For \\(X\\) the simplicial complex from Example 1 earlier,
the associated simplicial chain complex \\(C\_\*(X)\\) is just

\begin{equation\*}
C\_\*(X) = 0 \xrightarrow{0} \R^5 \xrightarrow{\partial} \R^4 \xrightarrow{0} 0.
\end{equation\*}

There's only two non-trivial vector spaces in the sequence, and therefore only
one non-trivial map: the map that takes an edge \\(e\_{ij}\\) to the difference
\\(e\_j - e\_i\\). But wait, you might protest, what about the regions \\(T\_1\\) and
\\(T\_2\\)? Those were 2-dimensional thingies so shouldn't we have a non-zero vector
space of 2-simplices \\(C\_2(X)\\)? Well we only worked with 0- and 1-dimensional
simplices in the calculations above: we treated \\(X\\) like a directed graph
instead of an object with surfaces. Think of empty space in those triangles that
you could poke your fingers into. As to why we made that choice, I'll explain in
a moment.

Returning to the setting of a general simplicial complex \\(X\\), the map \\(\partial\_k:
C\_k(X) \to C\_{k-1}(X)\\) sends a \\(k\\)-simplex (or linear combination of
\\(k\\)-simplices) to its boundary, which will be a sum of \\((k-1)\\)-simplices.
Again, think of a line segment sent to the difference of its endpoints, or a
triangle sent to a linear combination of its edges, or a triangular pyramid
(tetrahedron) sent to a linear combination of its faces.
At any point in the sequence --- say we're looking at \\(C\_k(X)\\) --- there's a
linear transformation \\(\partial\_{k+1}\\) coming _in_ to \\(C\_k(X)\\) and a linear
transformation _leaving_ \\(C\_k(X)\\). The crucial observation that underlies the
theory of homology theory is that \\(\partial\_k \circ \partial\_{k+1} = 0\\) for each \\(k\\). That is, if
you were to multiply the matrix for \\(\partial\_k\\) against the matrix for \\(\partial\_{k+1}\\), you'd
get the zero matrix. Why is that? Unfortunately, I can't give the precise proof
without writing down all the formulas for \\(\partial\_k\\) in general (and all of its gory
alternating signs), but we've actually already seen an example. Remember when we
were point out that the boundary of a loop is zero?

\begin{align\*}
\partial (\partial T\_1) &= \partial(e\_{01} + e\_{12} - e\_{20}) \\\\
&= e\_1 - e\_0 + e\_2 - e\_1 - (e\_2 - e\_0) \\\\
&= 0.
\end{align\*}

In our notation here, \\(\partial\_1(\partial\_2 T\_1)=0\\), or rather, \\((\partial\_1 \circ \partial\_2)(T\_1)=0\\). The
loop \\(\partial T\_1\\) is the boundary of the 2-simplex \\(T\_1\\) (which, again, we didn't
actually use in our calculations earlier) and the boundary of a boundary is
_always_ zero! You'll usually see this cornerstone of the theory of chain
complexes written as the beautifully simple[^fn:7]

\begin{equation\*}
\partial^2 = 0.
\end{equation\*}

From a linear algebraic standpoint, what happens when the composition of two
maps is zero? It means that the image \\(\text{im }\partial\_{k+1}\\) of the first map is
contained in the kernel \\(\text{ker }\partial\_k\\) of the second map. Make sure you parse
that out and it makes sense to you. The idea behind homology is really quite
simple: the \\(k\\)th homology group measures how much of \\(\text{ker }\partial\_k\\) is not hit
by \\(\partial\_{k+1}\\). More precisely, the \\(k\\)th simplicial homology vector space
of \\(X\\) is the quotient space

\begin{equation\*}
H\_k(X) = \frac{\text{ker }(\partial\_k:C\_k(X) \to C\_{k-1}(X))}{\text{im }(\partial\_{k+1}:C\_{k+1}(X) \to C\_k(X))}.
\end{equation\*}

If you're not too familiar with quotient spaces, you won't go too wrong in
thinking of the \\(k\\)th homology space as the orthogonal complement of \\(\text{im
}\partial\_{k+1}\\) inside \\(\text{ker }\partial\_k\\). If you were to just count dimensions, the
dimension of \\(H\_k(X)\\) is the nullity of \\(\partial\_k\\) minus the rank of \\(\partial\_{k+1}\\).

That's all a bit of a mouthful to parse, so let's back it up and think about
what that means geometrically for a moment. Let's go back to our handy
Example 1. We only have one non-zero differential, but let's compute the 0th and
1st homologies.

\begin{equation\*}
0 \xrightarrow{0} \R^5 \xrightarrow{\partial\_1} \R^4 \xrightarrow{0} 0.
\end{equation\*}

The 0th homology is the quotient of \\(\text{ker }\partial\_0\\) by \\(\text{im }\partial\_1\\). Well
\\(\partial\_0=0\\) so its kernel is all of its domain, \\(\R^4\\). That is to say, \\(\partial e\_i=0\\)
for all \\(i\\), which makes sense --- points don't have boundaries. There are no
\\((-1)\\)-simplices. What's the image of \\(\partial\_1\\)? Well we have that (I'll drop the
subscript on \\(\partial\_1\\) here)

\begin{align\*}
\partial e\_{01} &= e\_1 - e\_0 \\\\
\partial e\_{02} &= e\_2 - e\_0 \\\\
\partial e\_{12} &= e\_2 - e\_1 \\\\
\partial e\_{13} &= e\_3 - e\_1 \\\\
\partial e\_{23} &= e\_3 - e\_2
\end{align\*}

so how much of our space \\(\R^4\\) of vertices can we write as \\(\partial\\) of something?
First, notice that \\(\partial e\_{12} = \partial e\_{02} - \partial e\_{01}\\) and \\(\partial e\_{23} = \partial e\_{13} - \partial
e\_{12}\\) (do you see where we got these? hint: think \\(\partial^2=0\\)). The remaining \\(\\{\partial
e\_{01}, \partial e\_{02}, \partial e\_{13}\\}\\) are clearly linearly independent in \\(\R^4\\), and so
we see that the image of \\(\partial\_1\\) is 3-dimensional. Thus the homology \\(H\_0(X)\\),
which is the nullity of \\(\partial\_0=0\\) minus the rank of \\(\partial\_1\\), is
\\((4-3)\\)-dimensional:

\begin{equation\*}
H\_0(X) \cong \R^1.
\end{equation\*}

What about the 1st homology? Well that will have dimension equal to the nullity
of \\(\partial\_1\\) minus the rank of \\(\partial\_2\\). Since \\(\partial\_2=0\\), this is just the dimension of
the kernel of \\(\partial\_1\\). But we've already computed that --- that was the solution
to our interview problem! So

\begin{equation\*}
H\_1(X) \cong \R^2.
\end{equation\*}

In other words, our solution equated the number of regions to dimension of the
first homology space of the relevant one-dimensional simplicial complex. The
code was (after setting up the vertices and edges carefully) just a matter of
computing the kernel of a matrix.
Is there a 2nd homology group? No, because it would involve a quotient of the
kernel of \\(\partial\_2\\) but \\(C\_2(X)=0\\).

Notice that if we had instead worked with a 2-dimensional simplicial complex,
the dimension of the first homology would _not_ be what the problem was asking
for. The problem is looking for the nullity of \\(\partial\_1\\), but the dimension of the
homology would be smaller by the rank of \\(\partial\_2\\), due to the presence of a
non-trivial \\(C\_2(X)\\). Very roughly speaking, the \\(k\\)th homology counts the
number of distinct \\(k\\)-dimensional loops that can't be written as the boundary
of a \\(k-1\\)-dimensional simplex. And if we were to throw in the 2-simplices
into Example 1, all our loops would be boundaries, hence 0 in homology!

This post is too long already, so let me wrap it up with a few remarks to give
some perspective on the broader picture. The study of algebraic gadgets like
chain complexes is a part of the field of _homological algebra_, which at it
core studies the consequences of the equation \\(\partial^2=0\\). What we've been doing
here falls under the realm of algebraic topology, which uses such algebraic
techniques to better understand topological spaces. Simplicial homology, the
tool we used to solve the interview problem, is an example of such a technique.
There are many different types of homology theories for topological spaces ---
the simplicial one that we've been working with requires the least machinery to
get up and running. An important class of theorems determine the conditions
under which different homology theories spit out the same dimensions, and under
these conditions, it's often computation convenient to use one particular
homology theory over another.

Homology, in the general abstract, provides an _invariant_ of topological
spaces. That is to say, if you were to continuously deform a space \\(X\\) into a
space \\(Y\\), then \\(H\_k(X) \cong H\_k(Y)\\) for every \\(k\\). Hence the joke that topologists
cannot distinguish between tori and coffee mugs --- they're the same, as far as
homology can tell!

[^fn:1]: One weird subquotient trick somehow didn't have quite the
    same ring to it.
[^fn:2]: There's no real reason we've chosen to work with \\(\R\\) here. You
    could work instead with any field you like.
[^fn:3]: Technically, your first (first) Betti number, really.
[^fn:4]: There's a beautiful story about the relationship between homology,
    differential forms, and integration that justifies this terminology.
[^fn:5]: Interviewers everywhere hate him!
[^fn:6]: Yep, the word complex is used for two different things here. It can be a
    bit confusing at first.
[^fn:7]: If you know anything about differential \\(k\\)-forms (or maybe geometric
    algebra?), this should look awfully familiar.