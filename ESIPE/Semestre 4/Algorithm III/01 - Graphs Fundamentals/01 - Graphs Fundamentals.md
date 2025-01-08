[[Algorithm III]]
***
**Table of Contents**
```table-of-contents
```

****
## Graph ?

### Concept

Here is what a graph looks like:
![[graph.png]]
> [!info]
> In this situation, we can see that our graph is **oriented** (directed graph), but this is not mandatory for a graph (same as trees).

A graph `G` is a pair `(V, E)` where:
- `V` is the set of vertices
- `E ⊆ V × V` is set of edges, we write x → y instead of (x, y)
> [!example]
> For the graph above, we have:
> - `V = {A, B, C, D, E, F}`
> - `E = { A → B, A → D, B → D, B → E, C → B, C → E, C → F, D → A, E → D, F → F }`

Given an edge `x → y ∈ E`, we say that:
- `x` is the beginning of `x → y`
- `y` is the end of `x → y`
- `y` is a **successor** of `x`
- `x` is a **predecessor** of `y`
> [!info]
> An edge `x -> x` is called a loop.


### Path

A path is a sequence of edges, where each end vertex of an edge is the starting edge of the next one.
If there is a path from a vertex s to a vertex `t`, we say that `t` is **accessible** from `s`.
A **circuit** is a path that begins and ends at the same vertex.

> [!example]
> Considering the previous graph:
> - A path of length 5 (number of edges) from vertex `C` to `A`:
> `C → E → D → A → D → A`
> - A circuit at `E` of length 6 (number of edges):
> - `E → D → A → D → A → B → E`
> 
> *Note that several solutions exists for both.*


### Undirected Graph

Graph where there is an edge `x → y` whenever there is an edge `y → x`.
	*In other words, if one can go from `x` to `y`, we can go from `y` to `x`

Edges are represented by simple links:
![[undirected.png]]

An undirected graph is **connected** when there is **a path from every vertex to any other vertex**:
![[connected_graph.png]]

> [!info]
> This notion of connection also exists for directed graphs, but we call it a **strongly connected graph** in this context.


### Weighted Graph

A graph, either directed or undirected, is weighted when there is **a value (a real number) attached to every edge**, its weight.
	*Note that weight can also be negative. However, we generally avoid using a weight of "0". 
	(In our [[01 - Graphs Fundamentals#Encoding|internal representation]], a weight of "0" is reserved to signify the absence of an edge)*

> [!example]
> ![[graph_weight.png]]
> The shortest path from `D` to `B` is `D -> A -> B`. 
> Even though we have to traverse two edges, the total weight is lower than `D -> B`.


### Models

Graphs are mostly used to organise data and as models for many practical (and theoretical) frameworks:
- Subway: Vertices are stations and edges are trackages
- WWW: Vertices are webpages and edges are hyperlinks
- Social network: Vertices are people accounts or groups and edges are “friends” or “subscribed” relation
- Finite state automata: Vertices are states and edges are transitions labeled by letters


***
## Encoding

Good so far, but how do we represent a graph in our favourite programming language?

Well, before choosing our internal representation, we must define the **settings** of our graphs.
We will go with the most common characteristics:
1. We consider that the graph is not altered during the execution of the algorithm.
	*We say that the graphs are static (as opposed to dynamic).*
2. The graphs can be either:
	- **Dense**: with a lot of edges (`|E| ∈ Θ(|V|²)`)
	- **Sparse**: with few edges (`|E| ≪ |V|²`)

> [!important]
> To emphasise that an algorithm is efficient (or not) for sparse graphs, we use two parameters for the complexity:
> - The number of edges `|E|`
> - The number of vertices `|V|`


### Adjacency Matrix

The adjacency matrix of a graph is a matrix `M` of size `|V| × |V|`, where each value represents the weight of an edge linking two vertices.
	*If our graph is not weighted, our matrix can simply contain boolean values instead of numbers*

![[adjacency_matrix.png]]

If our graph is not oriented, edges must be set with the same weight on both directions!


### Adjacency Lists

The adjacency lists representation of a graph consists of a size-`|V|` array `A`, indexed by the vertices, of linked lists.
The list `A[i]` contains all the successors of vertex `i` (not necessarily ordered).

![[adjacency_list.png]]

If it is a weighted graph, the elements of the lists contain both the successor and the weight of the edge!


### Comparing both solutions

Using adjacency matrices requires `O(|V|²)` memory space, while using adjacency lists requires `O(|V| + |E|)` memory space
The later is also more compact unless we work on dense graphs.

Complexities of some operations:

| **Operation**                                   | **Adjacency Matrix** | **Adjacency List** |
| ----------------------------------------------- | -------------------- | ------------------ |
| Checking if there is an edge \(s \to t\)        | O(1)                 | O(\|V\|)           |
| Checking if \(s\) is a sink (no outgoing edges) | O(\|V\|)             | O(1)               |
| Listing all edges                               | O(\|V\|²)            | O(\|V\| + \|E\|)   |
