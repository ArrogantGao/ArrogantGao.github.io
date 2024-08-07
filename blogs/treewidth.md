+++
title = "TreeWidthSolver.jl"
hascode = true
date = Date(2024, 8, 3)
rss = "An introduction to TreeWidthSolver.jl, which can be used to solve tree decomposition with minimal treewidth of a given simple graph, which can be used to find the optimal contraction order of tensor networks."

tags = ["syntax", "code"]
+++

# Tensor Network Contraction Order Optimization with Tree Decomposition

This blog is a technical note for the [Google Summer of Code 2024](https://summerofcode.withgoogle.com) project ["Tensor network contraction order optimization and visualization"](https://summerofcode.withgoogle.com/programs/2024/projects/B8qSy9dO) released by **The Julia Language**, where I developed a package [TreeWidthSolver.jl](https://github.com/ArrogantGao/TreeWidthSolver.jl) for calculating the tree decomposition with minimal treewidth of a given simple graph and made it a backend of [OMEinsumContracionOrders.jl](https://github.com/TensorBFS/OMEinsumContractionOrders.jl).

This blog covers the following contents:
1. Contraction order of tensor network
2. Tree decomposition and its relation to contraction order
3. Finding tree decomposition with minimal treewidth

## Tensor Network Contraction Order

In this blog, we will not introduce the basic concept of tensor network, since it has been well introduced in many other places.
Instead, we recommend the following references:
* For readers with physics background: [https://tensornetwork.org/diagrams/](https://tensornetwork.org/diagrams/)
* For readers want to get a formal definition: Chapter 2 of [https://epubs.siam.org/doi/abs/10.1137/22M1501787](https://epubs.siam.org/doi/abs/10.1137/22M1501787)

For simplicity, in the following sections, we will use the Einstein summation formula to represent the tensor network as high dimensional arrays' multiplication:
$$R_{i,j,k,...} = \sum_{a, b, c, ...} A_{a,...}B_{b,...}...$$
It is easy to see that the most direct way to calculate the result is to loop over all the indices naively, which requires $O(d^N)$ operations, where $d$ is the dimension of the index and $N$ is the number of indices.
However, such a direct calculation is not efficient.
Considering the following simple tensor network:

![](/assets/treewidth_figs/D10.png)

where the tensors are represented by the circles and the indices are represented by the lines, $A, B, C, D$ are all rank-4 tensors, all indices are of dimension $D$.
Naively loop over all the indices requires $O(D^{12})$ operations and producing no intermediate tensors.
On the other hand, consider the following contraction order:


![](/assets/treewidth_figs/ABCD_contraction.png)

where we first contract $A$ and $B$ to get $AB$, then contract $C$ and $D$ to get $CD$, and finally contract $AB$ and $CD$ to get the result.
In this way, the total number of operations is $O(D^{10})$, which is much smaller than the naive calculation, while the trade-off is that we need to store the intermediate tensors $AB$ and $CD$ with size of $O(D^{8})$.
We say such a contraction is with **time complexity** of $O(D^{10})$ and **space complexity** of $O(D^{8})$, which represents the number of operations and the largest size of the intermediate tensors, respectively.

In actual calculation, we prefer such binary contractions, i.e., contracting two tensors at a time, and then contract the result with another tensor, and so on, so that we can make use of BLAS libraries to speed up the calculation.
A given contraction order can be represented as a tree, and since we only apply binary contractions, the tree is a binary tree. 
The contraction tree can be represented as a rooted tree, where the leaves are the tensors to be contracted and the internal nodes are the intermediate tensors.

### Tools for Tensor Network Contraction Order Optimization

In practice, there are many tools for tensor network contraction order optimization, including:
* [OMEinsumContractionOrder.jl](https://github.com/TensorBFS/OMEinsumContractionOrders.jl): an open-source Julia package for finding the optimal contraction order of tensor networks, is used as backend of [OMEinsum.jl](https://github.com/under-Peter/OMEinsum.jl) and [ITensorNetworks.jl](https://github.com/mtfishman/ITensorNetworks.jl).
* [Cotengra](https://cotengra.readthedocs.io/en/latest/)[^Gray] : a python library for contracting tensor networks or einsum expressions involving large numbers of tensors.

Various methods have been proposed for optimizing the contraction order, here we introduce some of them:

#### Greedy Algorithm

The Greedy method is one of the simplest and fastest method for optimizing the contraction order. The idea is to greedily select the pair of tensors with the smallest cost to contract at each step. In each step, for all possible pairs of tensors, the cost of the contraction is evaluated, and the pair with the smallest cost is selected and then contracted, which forms a new tensor.  This process is repeated until all tensors are contracted. This method is fast, however it is easy to be trapped in local minima.

An enhanced method is called the hyper-greedy method, where in each step one does not directly select the pair with the smallest cost, but instead samples according to the Boltzmann distribution given by $\mathcal{P}(v_i, v_j) = e^{-{L(v_i, v_j)} / {T}}$, where $L(v_i, v_j)$ is the cost function. In this case, it is possible for the process to escape from local minima. Then we run this process multiple times and select the best result.
The method has already been used in both OMEinsumContractionOrders.jl and Cotengra. 

#### Binary Partition

A given tensor network can be regarded as a hypergraph, where the tensors are the vertices and the shared indices are the hyperedges, and the cost of contracting a hyper edge can be encoded as its weight. The binary partition method is to partition the hypergraph into two parts, and then recursively partition each part. Cost of each partition can be evaluated by the sum of the weights of the hyperedges cut by the partition, while we prefer to make the partition as balanced as possible (balance means size of the subgraph should be similar). Thus, the problem is reduced to a balanced min cut problem on a hypergraph. In the past few decades, the graph community has developed many algorithms for the balanced min cut problem and provided the corresponding software packages, such as [KaHyPar](https://kahypar.org) [^kahypar], which has already been used in both OMEinsumContractionOrders.jl and Cotengra. 

#### Tree Simulated Annealing

Tree simulating annealing (TreeSA) [^Kalachev] is another type of the optimization method based on local search and simulating annealing. TreeSA is based on the following rules:

* Associativity: $T \times (S \times R) = (T \times S) \times R$,
* Commutativity: $T \times S = S \times T$.

These rules lead to the four possible transforms of the contraction tree as shown in the following figure.

![](/assets/treewidth_figs/treesa.png)

The TreeSA method starts from a random contraction tree and then applies the above rules to transform the tree. The cost of the contraction tree is evaluated and the tree is updated according to the Metropolis criterion. During the process, the temperature is gradually decreased, and the process stop when the temperature is low enough.
The method has already been used in OMEinsumContractionOrders.jl.


The method listed above are powerful and efficient, which can be used to find great contraction orders of the extremely large tensor networks.
However, they are heuristic methods and may not guarantee to find the optimal contraction order. In the following sections, we will introduce a theoretical method to find the optimal contraction order based on the tree decomposition.


## Finding the Optimal Contraction Order

In the previous section, we introduce the concept of tensor network and its contraction order, so that now you should understand why the contraction order so important. 
Then the next question is how to find the optimal contraction order.

In this section, we will introduce one way to find the theoretical optimal contraction order based on the exact tree width solver, according to the following well known theorem[^Markov] :

**Theorem 1**. Let $C$ be a quantum circuit with $T$ gates and whose underlying circuit graph is $G_c$. Then $C$ can be simulated deterministically in time $T^{O(1)} e^{O(tw(G_C))}$, where $tw(G_C)$ is the treewidth of $G_C$.

Using the language of tensor network, we can rewrite the above theorem as follows: the bottleneck of time complexity of the contraction of a tensor network is $O(e^{O(tw(L(G)))})$, where $L(G)$ is the line graph of the hypergraph of the tensor network. Therefore, if we can find the tree decomposition of the tensor network with minimal treewidth, we can find the optimal contraction order of the tensor network.

To further make use of the theorem, we need to introduce the concepts of line graph, tree width and tree decomposition.

### Line Graph

A formal definition of the [**line graph**](https://en.wikipedia.org/wiki/Line_graph) is as follows:

**Definition 1**. Given a graph G, its line graph $L(G)$ is a graph such that: 
   1. each vertex of $L(G)$ represents an edge of $G$; 
   2. two vertices of $L(G)$ are adjacent if and only if their corresponding edges share a common endpoint ("are incident") in $G$. 

That is, it is the intersection graph of the edges of G, representing each edge by the set of its two endpoints.

For a tensor network, we can construct a hypergraph $G$ whose vertices are the tensors and whose hyperedges are the indices. Then the line graph $L(G)$ of the hypergraph $G$ is the graph whose vertices are the indices and whose edges are the tensors, which is a simple graph, as shown in the following figure:

![Fig.1](/assets/treewidth_figs/linegraph.png)

Since we are considering a tensor network, dimension of the indices have to be considered. 
Therefore, for each vertex of the line graph $L(G)$, we define its weight as $\log_2(d)$, where $d$ is the dimension of the index. 

In this way, any possible tensor can be represented as a subgraph induced by a set of vertices in $L(G)$, and its log2 size is given by the sum of weights of the boundary vertices of the subgraph. For example, the tensor $C$ in the figure above corresponding to the subgraph induced by the vertices $\{b, d, f\}$, and its log2 size is 
$${dims(b) \times dims(d) \times dims(f)} = 2^{w(b) + w(d) + w(f)}$$
where $w(*)$ for weights of vertices.

### Tree Decomposition and Tree Width

Intuitively, a [**tree decomposition**](https://en.wikipedia.org/wiki/Tree_decomposition) represents the vertices of a given graph $G$ as subtrees of a tree, in such a way that vertices in $G$ are adjacent only when the corresponding subtrees intersect.

The tree decomposition of a graph is a tree whose nodes are subsets of the vertices of the graph, and the following conditions are satisfied:

1. Each vertex of the graph is in at least one node of the tree.
2. For each edge of the graph, there is a node of the tree containing both vertices of the edge.
3. Bags containing the same vertex have to be connected in the tree.

An example of tree decomposition is shown in the following figure:

![](/assets/treewidth_figs/treedecomposition.png)

where the left graph is the original graph and the right one is the tree decomposition of the graph.

All the nodes of the tree are called **tree bags**, and the intersection of two bags is called a **separator**.
The width of a tree decomposition is the size of the largest bag minus one, thus the width of the example above is $2$.
Clearly, one graph can have multiple tree decompositions, and the **tree width** of a graph is the minimum width of all possible tree decompositions.
For graphs with weights on vertices, we define the width as sum of weights of the vertices in the bags minus one.

### From Tree Decomposition to Contraction Order

Then how tree decomposition and tree width are related to the contraction order of the tensor network?
In this sub-section, we will answer the following questions: how to get a contraction order from a tree decomposition? And why the contraction order is optimal if the tree width is minimal?

For the first question, according to the definition of tree decomposition, we can obtain vertices elimination order by traversing the tree decomposition in a bottom-up manner. 
For each node of the tree decomposition, a vertex can be eliminated if it is in the bag of the node and not in the bags of its parent node.
The elimination order can then be used to determine the contraction order of the tensor network, where two tensors are contracted if they share a common index and the index is eliminated.
An example of contraction order is shown in the following figure:

![](/assets/treewidth_figs/elimination.png)

where the tree decomposition gives the elimination order $a, b, d, f, c, h, e, g, i$, and this lead to a contraction order $(((((A, B), C), F), D), E)$.
Of course, the contraction order is not unique by selecting different node as the root of the decomposition tree, but the space complexity of the contraction is the same for all possible contraction orders.

For the second question, we have the following two reasons:

1. Width of the tree decomposition is an upper bound of the space complexity of the corresponding contraction.
2. Total dimension of the indices in the same tree bag is the time complexity of the corresponding step of the contraction.

For example, in the first step of the construction above, the index $a$ is eliminated, and the corresponding operations is
$$A'_{bcde} = \sum_{a} A_{abc} B_{cde}$$
where the space complexity is bounded by width and the time complexity is given by the dimension of the indices in bag.

Thus, finding the tree decomposition with minimal width is equivalent to finding the optimal contraction order of the tensor network, with a bounded space complexity and minimal time complexity.

## The Bouchitté–Todinca Algorithm for Tree Width

In the previous sections, we introduced the connection between tree decomposition and contraction order, and the importance of finding the tree decomposition with minimal width. 
Thus, next question is: **How to find the tree decomposition with minimal width?**

In previous few decades, methods have been proposed to find the tree decomposition with minimal width, including exact algorithms and heuristic algorithms.
In this section, we will introduce the Bouchitté–Todinca (BT) algorithm[^Bouchitté] for finding the tree decomposition with minimal width.

Generally speaking, the BT algorithm is a dynamic programming algorithm. 
Given a set of possible tree bags, the algorithm can search a set of tree bags forming a tree decomposition with minimal width.
It has been proved that for a decomposition with minimal width, all bags are **potential maximal cliques** (pmc).
Thus the BT algorithm contains two phases, it first search for all pmcs, and then search the minimal width decomposition with the pmcs.

### Minimum Separator and Potential Maximal Cliques

To introduce the algorithm, we first need to introduce the concept of **minimum separator**[^Berry] and **potential maximal cliques**[^BouchittéListing], which are the intersection of two bags and possible tree bags, respectively.

**Definition 2**. A subset $S \subseteq V$ is an $a, b$-separator for two nonadjacent vertices $a, b \in V$ if the removal of $S$ from the graph separates $a$ and $b$ in different connected components. $S$ is a minimal $a, b$-separator if no proper subset of $S$ separates $a$ and $b$.

For all pairs of nonadjacent vertices $a, b \in V$, the set of all minimal $a, b$-separators are called the minimal separators of the graph, represented by $\Delta(G)$.
In practice, we can examine a set of vertices $S$ by checking properties of the connected components of the graph after removing $S$, represented as $\mathcal{C}(G / S)$:
* If there are two or more connected components $C \in \mathcal{C}(G / S)$ whose neighbor is exactly the set $S$, then $S$ is a separator, and the components are called **full components**.

**Definition 3**. A vertex set $\Omega$ of a graph $G$ is called a potential maximal clique if there is a minimal triangulation $H$ of $G$ such that $\Omega$ is a maximal clique of $H$.

Minimal triangulation is a graph obtained by adding edges to the graph such that the resulting graph is chordal.
The set of all possible pmcs of a graph is represented by $\Pi(G)$.
In practice, we can use the following conditions to check whether a set of vertices $\Omega$ is a pmc:
* $\Omega$ has no full component.
* For any pair of distinct vertices $u, v \in Ω$, either $(u, v) \in E(G)$ or there is a component $C \in \mathcal{C}(G / \Omega)$ with $(u, v) \in N(C)$.

In the first phase, the BT algorithm enumerate the set of all pmcs, $\Pi(G)$.

#### Enumerating Minimal Separators

We first introduce the method introduced by Berry[^Berry] to enumerate all minimal separators of a graph, which is part of the method to search for all pmcs.
Briefly, the algorithm makes use of the fact that given a separator, neighbors of its components are minimal separators.
Then the algorithm starts from the simplest minimal separator, i.e., neighbors of a single vertex, and then iteratively add vertices to the separator to find all minimal separators.
The algorithm is shown below:

![alt text](/assets/treewidth_figs/min_seps.png)

An example to enumerate all minimal separators is shown below:
```julia
julia> using TreeWidthSolver, Graphs

julia> g = smallgraph(:cubical)
{8, 12} undirected simple Int64 graph

# covert the graph to a labeled simple graph, which is a structure of graph defined in TreeWidthSolver.jl
julia> all_min_sep(LabeledSimpleGraph(g))
Set{Set{Int64}} with 14 elements:
  Set([4, 7, 8, 1])
  Set([4, 7, 2])
  Set([5, 7, 2])
  Set([4, 6, 2, 8])
  Set([5, 7, 3, 1])
  ...
```


#### Enumerating Potential Maximal Cliques

Then we introduce the method to enumerate all pmcs of a graph[^BouchittéListing], where the set of pmcs is constructed iteratively.

The algorithm first constructs a set of induced subgraphs $G_1, G_2, ..., G_N$, starting from only one vertex and then iteratively add vertices to the subgraphs.
For the $i$-th subgraph $G_i$, the algorithm first find all minimal separators $\Delta_i$ of the graph, and then use $\Delta_i$ and $\Pi_{i - 1}$ to find $\Pi_i$, where $\Pi_{i - 1}$ is the set of pmcs of the previous subgraph, since all pmcs of $G_i$ satisfies one of these conditions:
* $\Omega = \Omega' \cup \{v_i\}$, $\Omega' \in \Pi_{i - 1}$, $v_i = V(G_i) \setminus V(G_{i - 1})$
* $\Omega = \Omega'$, $\Omega' \in \Pi_{i - 1}$
* $\Omega = S \cup \{a\}$, $S \in \Delta_i$
* $\Omega = S_1 \cup (C \cap S_2)$, $S_* \in \Delta_i$, $C \in \mathcal{C}(G_i \setminus S)$

The algorithm is shown below:
![alt text](/assets/treewidth_figs/pmcs.png)

![alt text](/assets/treewidth_figs/one_more_vertex.png)

Here is an example to enumerate all pmcs of a graph:
```julia
julia> using TreeWidthSolver, Graphs

julia> g = smallgraph(:cubical)
{8, 12} undirected simple Int64 graph

julia> all_pmc(LabeledSimpleGraph(g))
Set{Set{Int64}} with 34 elements:
  Set([5, 4, 8, 3, 1])
  Set([4, 6, 7, 2, 1])
  Set([5, 4, 2, 8, 3])
  Set([5, 6, 7, 2, 1])
  Set([4, 6, 3, 1])
  ...
```

### Constructing the Tree Decomposition

In the second phase, with all pmcs, the algorithm constructs the tree decomposition with minimal width.
First, we use the pmcs to construct all possible $(\Omega, S, C)$ in $G$, where $S \subsetneq \Omega \subseteq (S \cup C)$.
For example, $(\Omega_1, S_1, C_1)$ in the following graph is a possible triple.
Then we can sort all these triples according to the size of $S \cup C$, and the largest ones are simply $(\Omega, \emptyset, G)$, $\Omega \in \Pi(G)$.

![alt text](/assets/treewidth_figs/SC_pairs.png)

It has been proved that the following property holds: for each pairs of $(S, C)$, its width is given by the size of $\Omega$ minus one or the largest width of all pair of $(S', C')$ as its subset.
For example, width of $(S_1, C_1)$ written as $w(S_1, C_1)$ is given by
$$w(S_1, C_1) = \max(|\Omega_1| - 1, w(S_2, C_2))$$
while
$$w(S_2, C_2) = \max(|\Omega_2| - 1, w(S_3, C_3), w(S_4, C_4)).$$
If the pair $(S, C)$ has no subset, then its width is given by the size of $\Omega$ minus one, corresponding to the leaf of the tree decomposition.
In the end, $tw(G) = w(\emptyset, G)$.

This leads to a dynamic programming algorithm to find minimal width, where we can calculate the width of the leaves first, and then we will be able to iteratively calculate the width of the larger pairs.
The algorithm[^Tuukka] is shown below, we used the version rewritten by Tuukka Korhonen, which is clearer and more readable.

![alt text](/assets/treewidth_figs/btdp.png)

The algorithm gives both minimal width and also the corresponding tree decomposition, which can be used to find the optimal contraction order of the tensor network.
For details about the implementation of the BT algorithm, please refer to the [TreeWidthSolver.jl](https://github.com/ArrogantGao/TreeWidthSolver.jl) package.

Here is an example:
```julia
julia> using TreeWidthSolver, Graphs

julia> g = smallgraph(:cubical)
{8, 12} undirected simple Int64 graph

julia> exact_treewidth(LabeledSimpleGraph(g))
tree width: 3
tree decomposition:
Set([4, 6, 3, 1])
└─ Set([6, 8, 3, 1])
   ├─ Set([5, 6, 8, 1])
   ├─ Set([6, 7, 8, 3])
   └─ Set([2, 8, 3, 1])
```
where both tree width and the corresponding tree decomposition are given.

### Tensor Network with Open Edges

In the previous sections, we introduced how to construct an optimal contraction order for a tensor network with no open edges by finding the optimal tree decomposition of its line graph, where all indices are contracted.
However, in practice, we often encounter tensor networks with open edges, where some indices are not contracted.
In this case, method based on bi-partition or tree decomposition can not be directly applied, since the complexity contributed by the open edges will not be reduced by any contraction, and neither balance min cut nor tree decomposition can correctly handle that.

To solve this problem, we develop a method by add a dummy tensor to the tensor network, where the dummy tensor has the same dimension as the open indices, and the dummy tensor is connected to all the tensors with open indices.
Then we can obtain the contraction order of the new network, which can be easy since now there is no open edges.
Finally, the contraction tree is rotated without changing the contraction complexity, which makes the dummy tensor the last tensor to be contracted so that can be removed, and the contraction order of the original tensor network is obtained.

For example, consider the following contraction:

![alt text](/assets/treewidth_figs/dummy.png)

where the leaf one is the original tensor network with three open edges, and a dummy tensor $D$ is added as shown in the right figure.
Then we may get a contraction order shown below, where the dummy tensor can be anywhere in the contraction tree, and the red line represents the path to it.
Along the path to the dummy tensor, we rotate the nodes locally.
For example, from the left one to the middle one, we consider the contraction $B, ijmn \to jkm$, and we rotate the $ijmn$, which is an intermediate tensor on the path, as root and make $B$ and $jkm$ ($C$) its children.
Similarly, in next step we rotate the contraction $A, D \to ikmn$, and make $D$ the last tensor to be contracted, so that by removing $D$ we get the contraction order of the original tensor network.

![alt text](/assets/treewidth_figs/pivot_tree.png)

It should be noted that in such process we only rotate the nodes locally without inducing any additional intermediate tensors, thus the complexity of the contraction is not changed.
This strategy can change the construction order optimization problem of tensor network with open edges to that of network without open edges, so that various graph based methods can be applied.

The method have been implemented in the [OMEinsumContractionOrders.jl](https://github.com/TensorBFS/OMEinsumContractionOrders.jl), please see the function `pivot_tree` for detailed implementation.
Here is an simple example:
```julia
julia> using OMEinsumContractionOrders

# the original tensor network
julia> eincode_origin = OMEinsumContractionOrders.EinCode([['a', 'b'], ['a', 'c', 'd'], ['b', 'c', 'e', 'f'], ['e']], ['d', 'f'])
ab, acd, bcef, e -> df

# with dummy tensor
julia> eincode = OMEinsumContractionOrders.EinCode([['a', 'b'], ['a', 'c', 'd'], ['b', 'c', 'e', 'f'], ['e'], ['d', 'f']], Vector{Char}())
ab, acd, bcef, e, df ->

julia> nested_code = optimize_code(eincode, uniformsize(eincode, 2), ExactTreewidth())
ab, ab ->
├─ ab
└─ acf, bcf -> ab
   ├─ acd, df -> acf
   │  ├─ acd
   │  └─ df
   └─ bcef, e -> bcf
      ├─ bcef
      └─ e

# remove the dummy tensor
julia> OMEinsumContractionOrders.pivot_tree(nested_code, 5)
acf, acd -> df
├─ ab, bcf -> acf
│  ├─ ab
│  └─ bcef, e -> bcf
│     ├─ bcef
│     └─ e
└─ acd

# it can also be used directly, the process above will be done automatically
julia> nested_code_direct = optimize_code(eincode_origin, uniformsize(eincode, 2), ExactTreewidth())
acf, acd -> df
├─ ab, bcf -> acf
│  ├─ ab
│  └─ bcef, e -> bcf
│     ├─ bcef
│     └─ e
└─ acd
```

<!-- reference -->
## Reference

[^Gray]: Gray, Johnnie, and Stefanos Kourtis. “Hyper-Optimized Tensor Network Contraction.” Quantum 5 (March 15, 2021): 410. [https://doi.org/10.22331/q-2021-03-15-410](https://doi.org/10.22331/q-2021-03-15-410).
[^kahypar]: Schlag, Sebastian, Tobias Heuer, Lars Gottesbüren, Yaroslav Akhremtsev, Christian Schulz, and Peter Sanders. “High-Quality Hypergraph Partitioning.” ACM Journal of Experimental Algorithmics 27 (December 31, 2022): 1–39. [https://doi.org/10.1145/3529090](https://doi.org/10.1145/3529090).
[^Kalachev]: Kalachev, Gleb, Pavel Panteleev, and Man-Hong Yung. “Multi-Tensor Contraction for XEB Verification of Quantum Circuits.” arXiv, May 18, 2022. [https://doi.org/10.48550/arXiv.2108.05665](https://doi.org/10.48550/arXiv.2108.05665).
[^Markov]: Markov, Igor L., and Yaoyun Shi. “Simulating Quantum Computation by Contracting Tensor Networks.” SIAM Journal on Computing 38, no. 3 (January 2008): 963–81. [https://doi.org/10.1137/050644756](https://doi.org/10.1137/050644756).
[^Bouchitté]: Bouchitté, Vincent, and Ioan Todinca. “Treewidth and Minimum Fill-in: Grouping the Minimal Separators.” SIAM Journal on Computing 31, no. 1 (January 2001): 212–32. [https://doi.org/10.1137/S0097539799359683](https://doi.org/10.1137/S0097539799359683).
[^Berry]: Berry, Anne, Jean-Paul Bordat, and Olivier Cogis. “Generating All the Minimal Separators of a Graph.” In Graph-Theoretic Concepts in Computer Science, edited by Peter Widmayer, Gabriele Neyer, and Stephan Eidenbenz, 1665:167–72. Lecture Notes in Computer Science. Berlin, Heidelberg: Springer Berlin Heidelberg, 1999. [https://doi.org/10.1007/3-540-46784-X_17](https://doi.org/10.1007/3-540-46784-X_17).
[^BouchittéListing]: Bouchitté, Vincent, and Ioan Todinca. “Listing All Potential Maximal Cliques of a Graph.” Theoretical Computer Science, 2002. [https://doi.org/10.1016/S0304-3975(01)00007-X](https://doi.org/10.1016/S0304-3975(01)00007-X).
[^Tuukka]: Tuukka Korhonen, "Finding Optimal Tree Decompositions", 2020. [https://tuukkakorhonen.com/papers/msc-thesis.pdf](https://tuukkakorhonen.com/papers/msc-thesis.pdf)