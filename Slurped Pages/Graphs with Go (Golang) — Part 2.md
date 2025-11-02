---
link: https://medium.com/@snassr/graphs-with-go-golang-part-2-4310c54ccac6
byline: snassr
site: Medium
date: 2022-10-28T01:39
excerpt: "Graphs with Go (Golang) — Part 2 Traversing Graphs using BFS & DFS. Graphs with Go Series Graphs with Go (Golang) — Part 1: Reviewing Graphs Graphs with Go (Golang) — Part 2: Traversing …"
twitter: https://twitter.com/@Medium
slurped: 2025-11-02T17:52
title: Graphs with Go (Golang) — Part 2
tags:
  - go
  - graph
---
Traversing Graphs using BFS & DFS.

Press enter or click to view image in full size

### Graphs with Go Series

- [Graphs with Go (Golang) — Part 1](https://medium.com/@snassr/graphs-with-go-golang-part-i-3e0f9392c294): Reviewing Graphs
- Graphs with Go (Golang) — Part 2: Traversing Graphs

## What & Why?

What are traversals, and why use them?

### Definition

A traversal or _search_ describes the process in which a program visits (possibly all) the vertices of a graph.

### Purpose

As discussed in [Part 1](https://medium.com/@snassr/graphs-with-go-golang-part-i-3e0f9392c294), graphs help us model complex relationships in the real world. Once we model and store these relationships in a graph, we need the ability to search and perform calculations on these models.

- What is the shortest path between A → D?
- What is the distance between B → C?
- Can we return the nodes in order?
- Can we list all possible permutations of 123?

This article will look at the two principal algorithms used to traverse graphs.

- Breadth-First Search
- Depth-First Search

We can implement BFS and DFS **_using iteration or recursion._**

The time and space complexity of both can be generalized as follows:

- **Time**: O(V+E), We visit every vertex (V) and edge (E) once.
- **Space**: O(V), We store every vertex in memory (queue or call stack).

## Algorithms

### Breadth-First Search (BFS)

BFS is a _level-order_ visitation algorithm. It visits neighboring vertices (via edges) at the same distance from the current node before exploring those nodes’ nodes.

BFS is a naturally _iterative_ algorithm but can be implemented recursively (by tracking the level in the recursive calls).

**Usage**

- Counting Levels: Returning the number of levels in an acyclical graph.
- Finding Shortest Path: Scanning level by level allows us to count levels as each move forward from a source node to a target node.  
    — Since BFS uses level-order visitation, it is always the shortest path! Assuming all edges in the graph have equal values (unweighted graph).

**Performance**

- BFS is excellent for graphs of an unknown/infinite size because we scan the graph in level order (level-by-level).
- Great for finding nodes close to the origin (shortest distance).
- Very slow for super-wide graphs because we check nodes level-by-level.

**Template (iterative)**

Iterative Breadth-First Search

Notice we can remove the `levelSize` loop if we only need the `ordered` node list!

_Note: If a (left to right) ordering is required, the above does not work since the order of values in a hashmap is not guaranteed (It’s why_ `_require.ElementsMatch_` _is used instead of_ `_require.Equal_`_)._

_Replacing the standard map with an ordered map would be the laziest way to get an order (at the cost of a dependency). Look here for_ [_more recommendations_](https://groups.google.com/g/golang-nuts/c/TDwGcRQe6mQ) _on map order._

### Depth-First Search (DFS)

DFS is a _furthest-first_ exploration algorithm. It starts at a node and explores the paths from that node as far as possible before backtracking to the starting node.

DFS is a naturally _recursive_ algorithm but can be implemented iteratively (similar to BFS but with a stack instead of a queue).

**Usage**

- Combinatorial problems (_Backtracking_)
- Cycle Detection
- Mutating a higher-scoped state (e.g., global)

**Performance**

- Faster on wide graphs since it operates on a deepest-first basis.
- Great for finding nodes far away from the root (e.g., exit in a maze).
- Memory-intensive for long, narrow graphs (b/c of the deep call stack).

**Template (recursive)**

Recursive Depth-First Search

Notice we `previsit` and `postvisit` in the function above! Usually, only one would be applied based on whether we want the operation before (pre) or after (post) visiting the subtrees.

## Examples

We can use BFS and DFS interchangeably to traverse graphs. However, as discussed above, sometimes a technique is better used against one problem than another.

My process for choosing one over the other:

1. Whichever is more intuitive first!
2. The more readable solution.
3. Production Performance — Consider the possible inputs and edge cases against each technique’s Usage/Performance listed earlier in the article. Generally, wide graph inputs should use DFS (smaller stack), while long and narrow graph inputs should use BFS (smaller queue).

### Finding the Shortest Path (e.g., Shortest Path In A Graph)

_Why_ **_BFS_**_?_ Finding the shortest path means moving level-by-level and counting those levels until we find the target node. We want to avoid DFS’s farthest node exploration.

BFS Shortest Path In A Graph

### Unknown or Infinite Size (e.g., Minimum Knight Moves)

_Why_ **_BFS_**_?_ Since we don’t know the size of the graph beforehand, It is prudent to go level by level to find the target node.

BFS Minimum Knight Moves

### Combinatorial Problem (e.g., Permutations)

Why **DFS**? We must build a path for each permutation and backtrack to get all the other combinations. The problem is a combinatorial + backtracking solution best suited for _recursive DFS_.

DFS Permutations

We haven’t discussed backtracking that much since it could (and maybe will) be a topic on its own. Briefly, backtracking allows us to retrace our steps as we go back up the recursive call stack.

In the function above, we want to build a path starting from `[]` but select numbers in a different order using the same loop `[1,2,3],[1,3,2]…`. To achieve this, we utilize the ability to “go back” or **backtrack** on the _path_ and _used_ variables.