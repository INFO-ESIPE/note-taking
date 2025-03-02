[[Algorithm III]]
***
**Table of Contents**
```table-of-contents
```

****
## Refresher: Trees

We have seen how to do a depth traversal on trees. What is important is the order of traversal, and the properties of this order.
![[dft_property.png|550]]


***
## DFS

We want to do the same thing, but on a graph.

> [!question]
> 1. Unlike trees, there is no root, where do we start?
> 2. Circuits will induce infinite loops, how do we handle this?

Here is how we could do it:
1. We start from an arbitrary vertex, and call the recursive DFS on all unvisited vertex until everything is visited.
2. Mark visited vertices and do not call on a marked vertex


### Basic DFS

Here is a basic DFS we can apply on a graph:
```java
void dfs(Graph graph) {
    var visited = new boolean[graph.vertexCount()]; // "false" as default value, avoids filling the array 
    for (int s = 0; s < graph.vertexCount(); s++) {
        if (!visited[s]) {
            recDFS(graph, s, visited);
        }
    }
}

void recDFS(Graph graph, int s, boolean[] visited) {
    visited[s] = true;
    for (int t : graph.getSuccessors(s)) {
        if (!visited[t]) {
            recDFS(graph, t, visited);
        }
    }
}
```
> [!info]
> Complexity:
> `O(∣V∣²)` for adjacency matrix representation.
> `O(∣V∣+∣E∣)` for adjacency list representation.

Here is what our DFS looks like:
![[dfs_bool.gif|350]]


### Coloured DFS

The **Forest** of a Graph DFS is the **set of trees obtained by keeping the edges corresponding to the recursive calls only**.

We also classify some of the edges for the DFS:
- **Tree edges** are the one used for the recursive calls
- **Backward edges** are the one linking a vertex to one of its ancestor (or itself) in the forest
![[backward.png|450]]
> [!note]
> Tree edges in blue, backward edges in pink

Now, we would like a way to **identify backward edges** in a DFS. The coloured DFS is the first approach at this problem.
We alter the DFS algorithm by using three colours (instead of true/false) for marking the vertices:
- green when not visited
- blue during visit
- red when the visit is finished

This way, we know that there is a backward edge `s → t` whenever `t` is blue during the recursive call on `s`.

![[dfs_colour.gif]]
> [!note]
> We will do the code in laboratory class.


### Timed DFS

A variant of the coloured DFS: We use a timer counter and store its value when **we start the call on a vertex and when we end the call**.
![[dfs_timed.gif]]

This is very useful for [[02 - DFS#Topological Sorting|topological sorting]] and [[02 - DFS#Transposal with Kosaraju|Kosaraju's algorithm]].


***
## Constraints

Say we have some **tasks** `V` to be done, and there are some **order constraints** `C` on the tasks:  
	some must be performed before others, which we write `s < t` if `s` must be done before `t`.

Let's take the following situation:
`V` = `{underpants, shoes, socks, belt, shirt, trousers, vest, tie}`
`C` = `underpants < trousers; shirt < belt; shirt < tie < vest; socks < shoes; belt < vest; trousers < shoes;`

> [!question]
> How can we model this problem as a graph?

We can build the **constraints graph** whose vertices are `V` and with an edge `x → y` whenever `x < y` in `C`:
![[constraint_graph.png]]

We ask two questions:
1. Is it possible to order the task so that the constraints are fulfilled?
2. Can we compute such an order efficiently?

A graph is called **acyclic** if it contains no cycle of length `≥ 1`. There exists a compatible order if and only if the
constraints graph is acyclic

> [!important]
> We know a graph is acyclic if it doesn't contain any backward edge.
> (By contraposition, if there is a backward edge x → y, it forms a cycle because by definition, there is a path from y to x.).
> 
> If there is a backward edge in a DFS on our graph, there is a backward edge in any DFS 


### Topological Sorting

What if we want to find an order of tasks that satisfies given constraints (e.g., prerequisites).
The logic is the following: We **order vertices by their reverse finish time in a timed DFS**.
	*In other words, we perform a timed DFS (and keep track of record end times), and then sort vertices by decreasing end time*

```java
void topoSortRec(Graph graph, int s, boolean[] visited, List<Integer> result) {
    visited[s] = true;
    for (int t : graph.getSuccessors(s)) {
        if (!visited[t]) {
            topoSortRec(graph, t, visited, result);
        }
    }
    result.add(0, s); // Add to the beginning
}

List<Integer> topoSort(Graph graph) {
    boolean[] visited = new boolean[graph.vertexCount()];
    List<Integer> result = new ArrayList<>();
    for (int s = 0; s < graph.vertexCount(); s++) {
        if (!visited[s]) {
            topoSortRec(graph, s, visited, result);
        }
    }
    return result;
}
```
> [!info]
> `O(∣V∣²)` for matrix, `O(∣V∣+∣E∣)` for lists.


***
## Strongly connected components

An SCC is a maximal subset of vertices such that any two vertices are reachable from each other.

> [!example]
> ![[scc.png]]
> CA = {A, B, D}
> CE = {E}
> CF = {C, F}
> CB = {A, B, D}

The decomposition of a graph into its SCCs is a fundamental construction on graphs, as it allows us to have a strategic and simplified vision of
the paths in the graph:
![[scc_decomp.png]]

We can even merge all the nodes into the same SCC, which results in the SCC-graph:
![[scc_graph.png]]


### Transposal with Kosaraju

The transposal Gt of a graph G is the graph obtained after flipping all the edges.
	*In other words, an edge `s -> t` becomes `t -> s`. We reverse directions...*

![[transposal.png]]

We can do this via Kosaraju's Algorithm:
1. Perform a timed DFS and order vertices by decreasing finish time.
![[kosaraju_1.gif]]

2. Compute the transposed graph Gt by reversing all edges.


3. Perform DFS on Gt in the order obtained from step 1. Each DFS tree corresponds to an SCC.
