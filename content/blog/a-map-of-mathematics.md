+++
title = "a map of mathematics"
author = ["Nilay Kumar"]
date = 2023-09-26T00:00:00-04:00
draft = true
+++

> Something has disappeared: the sovereign difference, between one and the other,
> that constituted the charm of abstraction. Because it is difference that
> constitutes the poetry of the map and the charm of the territory, the magic of
> the concept and the charm of the real.
>
> <div class="attribution">
>
> Baudrillard, _Simulacra and Simulation_
>
> </div>

A few years ago, it must have been, I came across a fantasy-style illustrated
map of physics by field and subfield that I thought was a great overview of the
landscape of physics research. I can't quite remember where I saw it or who had
made it, but here's a few that I've found that I quite like.

The first is a beautiful 1939 map made by [Bern Porter](https://en.wikipedia.org/wiki/Bern_Porter), drawn only slightly
stylized, with important concepts of physics labelled in large print and
notable physicists' names written in as villages along a coast ("daring souls
who venture there"):

{{< figure src="/ox-hugo/bernard-porter-map-of-physics.jpeg" alt="alt-text here" caption="<span class=\"figure-number\">Figure 1: </span>caption here" >}}

To a certain extent, similar ideas (and physicists who worked on those ideas)
are placed close together, with a river marked "energy" -- in its various forms
-- flowing through the landscape and cutting out islands.

Here's a more recent rendition by science educator and YouTuber [Dominic Walliman](https://dominicwalliman.com/)
(who has a number of such maps for different subjects) which is much more
stylized and conceptual:

{{< figure src="/ox-hugo/walliman-physics-jpg.jpg" alt="alt-text here" caption="<span class=\"figure-number\">Figure 2: </span>caption here" >}}

In this rendition, the spatial proximity of subfields is not as closely linked
to conceptual proximity (whatever that means), although the "chasm of ignorance"
that separates general relativity from quantum field theory and the standard
model seems to me pretty reasonable. I think this map is definitely more
educational, though I do enjoy Porter's rather traditional cartographic
yellow-on-blue style.


## maps of mathematics {#maps-of-mathematics}

So I got to thinking -- what would a map of mathematics look like? Can we
organize various subfields of pure mathematics into a 2-dimensional cartographic
representation in which the areas that interact with each other more frequently
are closer? And how exactly would we quantify "conceptual proximity"? Before we
get to answering these questions, let's take a look at some already-existing
maps of mathematics.

Here's Walliman's map on mathematics, which is styled very similarly to his map
of physics:

{{< figure src="/ox-hugo/walliman-mathematics-jpg.jpg" alt="alt-text here" caption="<span class=\"figure-number\">Figure 3: </span>caption here" >}}

There's also this map (and [variations](https://math.meta.stackexchange.com/questions/6479/a-graph-map-of-math-se)) by Piotr Migdal, which is more accurately
a map of question tags on Mathematics Stack Exchange:

{{< figure src="/ox-hugo/migdal-map.jpg" alt="alt-text here" caption="<span class=\"figure-number\">Figure 4: </span>caption here" >}}

This map is definitely heavily influenced by the nature of Math Stack Exchange
being a question and answer site comprised of mostly non-research oriented
questions. I'd expect linear algebra, for instance (as commenters on the Math SE
post noted) to be much more central, but questions with the `linear-algebra` tag
generally don't tend to be tagged with `abstract-algebra` or `functional-analysis`
often enough for this algorithm to pick up those connections.

The last example I've got on hand is that of [Paperscape](https://paperscape.org/), which comes closest
among these to what I've got in mind. Paperscape is an interactive map that
visualizes the arXiv  with papers "clustered together according to how they
reference each other... as particles in a physical system, with references
acting as attractive forces between papers".

{{< figure src="/ox-hugo/paperscape-map.jpg" alt="alt-text here" caption="<span class=\"figure-number\">Figure 5: </span>caption here" >}}

I strongly recommend zooming in and panning around to see what famous papers you
recognize, and where they're positioned relative to others. I especially enjoy
the intricate coastlines and the sparse ocean of papers to the west of the main
math islands. The [code and data](https://github.com/paperscape/paperscape-mapclient) for the Paperscape project is available online.
While the map uses a large amount of data (papers between 1991 and 2017), the
authors note that references to non-arXiv articles cannot be reliably captured
in general -- indeed, on average only about 8% of references in math papers
could be linked to other arXiv papers. This might explain the large set of
sparsely distributed math papers. The ratio is much better in high energy
physics, sitting at around 70-80%. I guess this is an inevitable limitation of
any dataset focusing on online publications.

Now, when it comes to a map of mathematics, I think Paperscape is a bit _too_
granular. For such a map, we might want the finest resolution to be subfields
(or maybe sub-subfields) of mathematics. There are a few different ways to
classify papers/documents into mathematical subfields. If we were to restrict to
arXiv preprints, we could use the `math.XX` tags that are used to categorize
submissions, found [here](https://arxiv.org/archive/math), which I've scraped below:

```sh
curl -s https://arxiv.org/archive/math | \
    sed -En 's/.*<b>(math.[A-Z]{2}.*)<\/b>/\1/p'
```

```text
math.AG - Algebraic Geometry
math.AT - Algebraic Topology
math.AP - Analysis of PDEs
math.CT - Category Theory
math.CA - Classical Analysis and ODEs
math.CO - Combinatorics
math.AC - Commutative Algebra
math.CV - Complex Variables
math.DG - Differential Geometry
math.DS - Dynamical Systems
math.FA - Functional Analysis
math.GM - General Mathematics
math.GN - General Topology
math.GT - Geometric Topology
math.GR - Group Theory
math.HO - History and Overview
math.IT - Information Theory
math.KT - K-Theory and Homology
math.LO - Logic
math.MP - Mathematical Physics
math.MG - Metric Geometry
math.NT - Number Theory
math.NA - Numerical Analysis
math.OA - Operator Algebras
math.OC - Optimization and Control
math.PR - Probability
math.QA - Quantum Algebra
math.RT - Representation Theory
math.RA - Rings and Algebras
math.SP - Spectral Theory
math.ST - Statistics Theory
math.SG - Symplectic Geometry
```


## the mathematics subject classification {#the-mathematics-subject-classification}

There is a more sophisticate classification than the arXiv's maintained by
[zbMATH Open](https://zbmath.org/about/) and the American Mathematical Society's [Mathematical Reviews](https://mathscinet.ams.org/mathscinet/publications-search), called
the **Mathematics Subject Classification (MSC)**. The most recent revision is called
the [MSC2020](https://msc2020.org/), and can be [browsed](https://zbmath.org/classification/) at zbMATH Open. The classification scheme is
fairly straightforward, consisting of 5-character MSC codes of the form `##X##`,
where the `#`'s are digits and `X` is either a letter `A-Z` or a hyphen `-`. The first
two digits indicate the top-level mathematical subfield: documents classified as
algebraic topology, for instance, fall under `55X##`, while those classified under
partial differential equations are marked as `35X##`. The hyphen indicates a
document that does not directly contain research mathematics, while letters
indicate sub-subfield. The final two digits indicate with mathematical concepts
are treated. So, for example:

> `35` / partial differential equations
>
> -   `35-04` / software, source code, etc. for problems pertaining to partial differential equations
>
> `55` / algebraic topology
>
> -   `55-04` / software, source code, etc. for problems pertaining to algebraic topology
> -   `55Q` / homotopy groups
>     -   `55Q10` / stable homotopy groups
>     -   `55Q40` / homotopy groups of spheres

Now most research mathematics does not fit neatly into one of these full
five-digit boxes -- typically mathematicians use a wide variety of tools,
techniques, and tricks from various subfields to attack difficult problems.
Because of this, it's common to see quite a few codes attached to any given
paper. Take for instance, [this paper](https://zbmath.org/1328.14027), which sits roughly in the region
where algebraic geometry meets symplectic geometry and mathematical physics
(with a sprinkling of higher categories). It's tagged with:

> `53D05` / symplectic manifolds (general theory)
>
> `53D12` / lagrangian submanifolds; maslov index
>
> `14A15` / schemes and morphisms
>
> `18F20` / presheaves and sheaves, stacks, descent conditions (category-theoretic aspects)

along with some older, outdated MSC2010 codes. Returning to the idea of building
a map of mathematics, we can think of this paper as a witness to the conceptual
proximity between these fields or ideas of mathematics. In the rest of this
article, we'll use the MSC and the data available from zbMATH Open to build a
map of mathematics based on MSC codes, with distances between two codes
determined by the presence of literature jointly tagged with those codes.

One benefit of using zbMATH Open is that their index is the most comprehensive
and long-running index available (as well as freely available to the public via
well-designed APIs and dataset releases). This does not, however, guarantee that
every reference is captured and/or classified. Take, for instance, the [series](https://zbmath.org/0050.39304) of
[landmark](https://zbmath.org/0055.41704) [papers](https://zbmath.org/0057.15302) by Eilenberg and Mac Lane that defined and studied what are now
known as [Eilenberg-MacLane spaces](https://en.wikipedia.org/wiki/Eilenberg%E2%80%93MacLane_space) \\(K(G, n)\\). There are no MSC codes assigned to
these papers,[^fn:1] perhaps due to how old they are, having been published in the
1950's. In many fields of research, concern about the lack of much older
references might be justifiably brushed aside as newer papers often replace
older ones. Mathematics is a bit different, however, due in part to the
sociological nature of definitions and theorems as relatively immutable. We
might therefore expect research papers to hold more lasting influence when
measured in terms of citations. Indeed, analysis of citation datasets shows
evidence to support this hypothesis. In fact, mathematicians since the end of
World War 2 have been citing papers from relatively further and further back
in time [<a href="#citeproc_bib_item_1">1</a>].

{{< figure src="/ox-hugo/bannister-teschke.jpg" alt="alt-text here" caption="<span class=\"figure-number\">Figure 6: </span>caption here" >}}

This is apparently _opposite_ the trend seen in most other research areas, which
marks mathematics as a bit of an oddball (and also indicates the irrelevance of
"impact factors" for mathematics, as they tend to focus only on recent
time-scales).

All that is to say: while the MSC data from zbMATH Open may be more
comprehensive than data from arXiv, there are still significant gaps and caveats
to keep in mind as we proceed.


## zbmath open data {#zbmath-open-data}

zbMATH Open has a long and interesting [history](https://zbmath.org/about/), but only very recently -- as of
early 2021 -- has it become open access (it was formerly known as Zentralblatt
MATH), with the support of the German government [<a href="#citeproc_bib_item_2">2</a>]. It is an
incredible source for tracking down mathematical research documents and even
includes links to the [OEIS](https://oeis.org/) and backrefs from [MathOverflow](https://mathoverflow.net/), among other things.
There is an [API](https://oai.zbmath.org/) for programmatic access[^fn:2], but for our purposes it will be
easier to just use a bulk dataset provided on [Zenodo](https://zenodo.org/record/6448360). The data is provided as a
`csv` that weighs in at around `1.6G`. I've repackaged the the data for myself as a
`parquet` which roughly halves the size (this is easy to do using `pandas`). The
columns in this dataset are labeled as follows:

> `de` / eight digits internal zbMATH identifier
>
> `doi` / digital object identifier
>
> `msc` / MSC of the article
>
> `keyword` / keywords of the article
>
> `title` / title of the article
>
> `refs` / MSCs occurring in the references

There's actually an extra column called `text` that seems to contain the abstract,
but it's typically empty, and we won't be using it anyway.

Before we get to working with the data, let's make sure we've got a virtual
environment with all our packages ready to go.

```emacs-lisp
(pyvenv-activate "../../.venv")
```

We can now load the data into memory, only keeping columns we'll use.

```python
import numpy as np
import pandas as pd
data_file = '~/data/zb.parquet.gzip'
cols = ['doi', 'msc', 'keyword', 'refs']
df = pd.read_parquet(data_file)[cols]
print(f"{df['msc'].isna().mean() * 100:0.1f}% of records missing MSC code, dropping...")
df = df[df['msc'].notna()]
print(f"{len(df)} records remaining.")
print(f"{df['refs'].isna().mean() * 100:0.1f}% of remaining records missing reference MSC codes, dropping...")
df = df[df['refs'].notna()]
print(f"{len(df)} records remaining.")
```

As you can see, there's a large amount of missing data. We've restricted to only
those records that have non-trivial MSC data as well as non-trivial MSC data for
the documents they reference. This leaves us still with about a million rows. If
you look through the data, you'll find that the `msc` column needs to be cleaned a
bit, as there are some entries stored as lists (`"['20M99', '20M18', '08A30']"`)
while others are stored as single-codes not in lists (`"70F10"`).[^fn:3] This is easy
to fix if we list-ify everything (similarly for the `refs` column):

```python
mask_msc_not_lists = ~df['msc'].str.startswith('[')
df.loc[mask_msc_not_lists, 'msc'] = '[\'' + df[mask_msc_not_lists]['msc'] + '\']'
df['msc'] = df['msc'].str.replace('\[|\]|\'|,', '', regex=True).str.split()
mask_refs_not_lists = ~df['refs'].str.startswith('[')
df.loc[mask_refs_not_lists, 'refs'] = '[\'' + df[mask_refs_not_lists]['refs'] + '\']'
df['refs'] = df['refs'].str.replace('\[|\]|\'|,', '', regex=True).str.split()
```

We've also done a bit of cleanup to make the `msc` and `refs` columns consist of
lists of codes (as opposed to strings representing lists of codes).


## a weighted graph {#a-weighted-graph}

At it's simplest, a map gives us a sense of important locations and how they are
arranged relative to each other. The data that we have so far is readily
assembled into an undirected graph, with vertices \\(v\_i\\) consisting of MSC codes,
and an edge \\(e\_{ij}=e\_{ji}\\) between codes \\(v\_i\\) and \\(v\_j\\) if there exists a paper in
the dataset tagged as \\(v\_i\\) referencing a paper tagged as \\(v\_j\\). We will assume
that codes appearing together more often indicates closer conceptual proximity
between the concepts classified by those codes. This suggests that we work with
a weighted graph: let \\(n\_{ij}\\) denote the number of papers witnessing an edge
between codes \\(v\_i\\) and \\(v\_j\\). The edge \\(e\_{ij}\\) should be weighted by \\(n\_{ij}\\) in
some way, probably by some monotonically increasing function \\(f\\). Since we're
more interested in visualization than getting all the numbers right, we'll play
with the choice of \\(f\\) later.

To summarize, we define our graph \\(G\\) by:

\begin{align\*}
V\_{G\_3} &= \\{v\_i \mid v\_i \in \text{MSC}\_3\\} \\\\
E\_{G\_3} &= \\{e\_{ij} \text{ with weight } f(n\_{ij})\\}
\end{align\*}

where \\(\text{MSC}\_3\\) is the set of MSC codes truncated to the first 3
characters. We'll truncate to 3 characters for two main reasons. The first is
that there are documents in the dataset tagged with codes like `57Rxx`, which
carry no more information than `57R` does. The second, more important reason, is
that I don't really want to get as granular as the full, five-character
classification codes. This does mean, unfortunately, that we'll be putting
all codes of the form `##-##` into one group, thus treating general reference
works, historical expositions, conference proceedings, source code, etc. as one
big category.

```python
msc_codes = []
for row in df.itertuples():
    msc_codes += row.msc + row.refs
msc_codes = pd.Series(np.unique(msc_codes))
# we need to get rid of any remaining 2-character codes
# and restrict to the 3-character level
msc3 = msc_codes[msc_codes.str.len() > 2].str[0:3].unique()
print(f"Found {len(msc3)} unique 3-character MSC codes")
```

```text
Found 641 unique 3-character MSC codes
```

At the 3-character level, then, there aren't too many codes. We can feasibly
work with our graph \\(G\_3\\) in code via its adjacency matrix \\(M\\), defined by

\begin{align\*}
M\_{ij} = \begin{cases} n\_{ij} & e\_{ij} \in E\_{G\_3} \\\ 0 & e\_{ij} \notin E\_{G\_3} \end{cases}
\end{align\*}

for any \\(v\_i,v\_j\in\text{MSC}\_3\\). We can apply our function \\(f\\) element-wise
afterwards. The code to construct this adjacency matrix is pretty
straightforward, though it took a minute or two to run on my machine.

```python
N = len(msc3)
# the matrix indexing is associated to the MSC codes using the indexing in msc3
idx_dict = {code: i for i, code in enumerate(msc3)}
M = np.zeros((N, N), dtype='i')
for row in df.itertuples():
    for document_code in row.msc:
        v1 = document_code[0:3]
        if len(v1) != 3:
            continue
        for ref_code in row.refs:
            v2 = ref_code[0:3]
            if len(v2) != 3:
                continue
            # the adjacency matrix of an undirected graph is symmetric
            M[idx_dict[v1], idx_dict[v2]] += 1
            M[idx_dict[v2], idx_dict[v1]] += 1
print(M)
```

```text
[[   14   573    23 ...     0     1     0]
 [  573 12402   782 ...     1    43     0]
 [   23   782   722 ...     0     4     0]
 ...
 [    0     1     0 ...     0     0     0]
 [    1    43     4 ...     0    28     0]
 [    0     0     0 ...     0     0     0]]
```

We can visualize this adjacency matrix using a heatmap:

```python
import matplotlib.pyplot as plt
plt.imshow(np.log1p(M))
labels = [code if i % 40 == 0 else '' for i, code in enumerate(msc3)]
plt.tick_params(left=False, bottom=False)
plt.xticks(range(len(labels)), labels, rotation=90, fontsize=8)
plt.yticks(range(len(labels)), labels, rotation=0, fontsize=8)
plt.title('MSC code citation heatmap')
plt.savefig('a-map-of-mathematics/heatmap.jpg');
```

{{< figure src="/ox-hugo/heatmap.jpg" >}}

Note that for this heatmap we've applied \\(f(x)=\log(1+x)\\) element-wise to `M`, as
the matrix elements vary too widely in magnitude to be easily visualized. As
expected, the diagonal is heavily populated -- papers in a subfield tend to
primarily cite other papers from that subfield. There are a couple of other
interesting patterns that emerge as an artifact of the lexicographical ordering
we've taken here. There's an overall block diagonal structure clearly visible
with a smaller block in the top left and a larger block in the bottom right. The
smaller block seems to consist of more algebraic codes while the larger block,
which starts to kicks off around the differential equations codes, consists of
more analytic codes. We can also see a sizable interaction between these two
rough groupings when we get to the geometric tags, which is not surprising.
Amusingly, this heatmap looks a bit like the zbMATH Open [logo](https://zbmath.org/static/zbMATH@2x.gif).


## visualizing the graph {#visualizing-the-graph}


### graph embeddings {#graph-embeddings}


### scikit-learn's laplacian eigenmaps {#scikit-learn-s-laplacian-eigenmaps}


## laplacian eigenmaps {#laplacian-eigenmaps}


### how the algorithm works {#how-the-algorithm-works}


## acknowledgements {#acknowledgements}


### thanks to fabian and moritz {#thanks-to-fabian-and-moritz}


## <span class="org-todo todo TODO">TODO</span>  {#d41d8c}

-   downsize images
-   image attribution + permissions
-   alt text
-   clicking on image links to full size image
-   check whether eilenberg-maclane spaces paper appear in the dataset
-   convert some of these links to references, appropriately
-   consider vectorizing the slow construction of M

<hr>

<style>.csl-left-margin{float: left; padding-right: 0em;}
 .csl-right-inline{margin: 0 0 0 1em;}</style><div class="csl-bib-body">
  <div class="csl-entry"><a id="citeproc_bib_item_1"></a>
    <div class="csl-left-margin">[1]</div><div class="csl-right-inline"> <span style="font-variant:small-caps;">Bannister</span>, A. and <span style="font-variant:small-caps;">Teschke</span>, O. (2017). <a href="https://doi.org/10.4171/NEWS/106/15">An update on time lag in mathematical references, preprint relevance, and subject specifics</a>. <i>Eur. Math. Soc. Newsl.</i> <b>106</b> 41–3.</div>
  </div>
  <div class="csl-entry"><a id="citeproc_bib_item_2"></a>
    <div class="csl-left-margin">[2]</div><div class="csl-right-inline"> <span style="font-variant:small-caps;">Hulek</span>, K. and <span style="font-variant:small-caps;">Teschke</span>, O. (2020). <a href="https://doi.org/10.4171/NEWS/116/12">The transition of zbMATH towards an open information platform for mathematics</a>. <i>Eur. Math. Soc. Newsl.</i> <b>116</b> 44–7.</div>
  </div>
</div>

[^fn:1]: It might be interesting to carry out an analysis of the distribution of MSC
    codes in the literature. Not only in terms of the proportion of tagged documents
    over time, but also to understand the subfield-by-subfield development and
    evolution of research mathematics. This is, of course, a bit tricky because
    while the MSC is quite old, it has changed substantially since the 1940s (see
    [here](https://web.archive.org/web/20230305101919/https://mathscinet.ams.org/mathscinet/help/field_help.html), for instance).
[^fn:2]: As zbMATH Open compiles data from various sources that may enforce a number
    of different licenses, the API provides less data than is available on the web
    interface, with data often replaced by the string `zbMATH Open Web Interface
    contents unavailable due to conflicting licenses.` This is described in the API
    [documentation](https://oai.zbmath.org/). Examples of API queries can be found on zbMATH Open's Github [here](https://github.com/zbMATHOpen/mscHarvester).
[^fn:3]: While `pandas` is convenient for quickly exploring data if you're working in a
    python environment (especially with varied file formats such as `parquet`), I've
    recently started using `xsv` (see [here](https://github.com/BurntSushi/xsv)), which is a speedy little command line
    utililty for working with `csv` files. To explore what some of the values in the
    `msc` column look like, for instance, you could run:

    ```sh
    xsv select msc ~/Downloads/out.csv | xsv sample 10
    ```

    ```text
    msc
    34C10
    "['76M10', '76D05', '65M60', '76M20']"
    "['81Pxx', '94Axx', '68Mxx']"
    "['26A33', '34K37', '34K45']"
    74K99
    "['82-02', '82D25', '82C26']"
    "['46L05', '46J10']"
    "['92B05', '62P10', '62F15']"
    "['11R33', '11R04', '11R29', '11R37']"
    "['82B43', '60K35', '82B20', '05C40']"
    ```

    This runs in a couple of seconds on my Macbook Air.