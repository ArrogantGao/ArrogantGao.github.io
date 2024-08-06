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
1. What is tensor network contraction order and why it is important?
2. What is tree width and tree decomposition?
3. What is the relationship between tree decomposition and contraction order?
4. How to find the optimal tree decomposition?

## Tensor Network Contraction Order

In this blog, we will not introduce the basic concept of tensor network.
Instead, we recommend the following references:
* For readers with physics background: [https://tensornetwork.org](https://tensornetwork.org)
* For readers want to get a formal definition: Chapter 2 of [https://epubs.siam.org/doi/abs/10.1137/22M1501787](https://epubs.siam.org/doi/abs/10.1137/22M1501787)


## Finding the Optimal Contraction Order

In the previous section, we introduce the concept of tensor network and its contraction order, so that now you should understand why the contraction order so important. Then the next question is how to find the optimal contraction order. Then in this section, we will introduce one way to find the theoretical optimal contraction order based on the exact tree width solver, according to the following well known theorem[^Markov] :

**Theorem 1**. Let $C$ be a quantum circuit with $T$ gates and whose underlying circuit graph is $G_c$. Then $C$ can be simulated deterministically in time $T^{O(1)} e^{O(tw(G_C))}$, where $tw(G_C)$ is the treewidth of $G_C$.

Using the language of tensor network, we can rewrite the above theorem as follows: the bottleneck of time complexity of the contraction of a tensor network is $O(e^{O(tw(L(G)))})$, where $L(G)$ is the line graph of the hypergraph of the tensor network. Therefore, if we can find the tree decomposition of the tensor network with minimal treewidth, we can find the optimal contraction order of the tensor network.

To further make use of the theorem, we need to introduce the concepts of line graph, tree width and tree decomposition.

### Line Graph

A formal definition of the **line graph**[^linegraph] is as follows:

**Definition 1**. Given a graph G, its line graph $L(G)$ is a graph such that: 
   1. each vertex of $L(G)$ represents an edge of $G$; 
   2. two vertices of $L(G)$ are adjacent if and only if their corresponding edges share a common endpoint ("are incident") in $G$. 

That is, it is the intersection graph of the edges of G, representing each edge by the set of its two endpoints.

For a tensor network, we can construct a hypergraph $G$ whose vertices are the tensors and whose hyperedges are the indices. Then the line graph $L(G)$ of the hypergraph $G$ is the graph whose vertices are the indices and whose edges are the tensors, which is a simple graph, as shown in the following figure:

![Fig.1](/assets/treewidth_figs/linegraph.png)

Since we are considering a tensor network, dimension of the indices have to be considered. Therefore, we define the line graph $L(G)$ as a weighted graph, and the weight of its vertices is given by the $\log_2(d)$, where $d$ is the dimension of the index. 

In this way, any possible tensor can be represented as a subgraph induced by a set of vertices in $L(G)$, and its log2 size is given by the sum of weights of the boundary vertices of the subgraph. For example, the tensor $C$ in the figure above corresponding to the subgraph induced by the vertices $\{b, d, f\}$, and its log2 size is 
$${dims(b) \times dims(d) \times dims(f)} = 2^{w(b) + w(d) + w(f)}$$
where $w(*)$ for weights of vertices.

### Tree Decomposition and Tree Width

Intuitively, a **tree decomposition** represents the vertices of a given graph $G$ as subtrees of a tree, in such a way that vertices in $G$ are adjacent only when the corresponding subtrees intersect[^treedecomp].

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
For weighted graphs, we define the width as sum of weights of the vertices in the bags minus one.

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

For the section question, we have the following two reasons:

1. Width of the tree decomposition is an upper bound of the space complexity of the corresponding contraction.
2. Total dimension of the indices in the same tree bag is the time complexity of the corresponding step of the contraction.

For example, consider the example contraction above, in the first step $a$ is eliminated, and the corresponding operations is
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

To introduce the algorithm, we first need to introduce the concept of **potential maximal cliques** and  **minimum separator**[^BouchittéListing], which are possible tree bags and the intersection of two bags, respectively.

**Definition 2**. A subset $S \subseteq V$ is an $a, b$-separator for two nonadjacent vertices $a, b \in V$ if the removal of $S$ from the graph separates $a$ and $b$ in different connected components. $S$ is a minimal $a; b$-separator if no proper subset of $S$ separates $a$ and $b$.



### Constructing the Tree Decomposition


## Appendix A: Tensor Network with Open Edges

In this section, we will introduce how to handle the tensor networks which are not closed, i.e., there are open edges in these tensor networks, which is common in practice.

<!-- reference -->

[^Markov]: Markov, Igor L., and Yaoyun Shi. “Simulating Quantum Computation by Contracting Tensor Networks.” SIAM Journal on Computing 38, no. 3 (January 2008): 963–81. https://doi.org/10.1137/050644756.
[^linegraph]: [https://en.wikipedia.org/wiki/Line_graph](https://en.wikipedia.org/wiki/Line_graph)
[^treedecomp]: [https://en.wikipedia.org/wiki/Tree_decomposition](https://en.wikipedia.org/wiki/Tree_decomposition)
[^Bouchitté]: Bouchitté, Vincent, and Ioan Todinca. “Treewidth and Minimum Fill-in: Grouping the Minimal Separators.” SIAM Journal on Computing 31, no. 1 (January 2001): 212–32. https://doi.org/10.1137/S0097539799359683.
[^BouchittéListing]: Bouchitté, Vincent, and Ioan Todinca. "Listing All Potential Maximal Cliques of a Graph." Theoretical Computer Science, 2002.
