---
link: https://medium.com/@snassr/graphs-with-go-golang-part-i-3e0f9392c294
byline: snassr
site: Medium
date: 2022-10-24T03:48
excerpt: Graphs with Go (Golang) — Part 1 Reviewing Graphs and how we can work with them in Go. What and Why? First, let’s briefly review graphs and why they are used. Definition A graph G(V, E) is a set …
twitter: https://twitter.com/@Medium
slurped: 2025-11-02T17:47
title: Graphs with Go (Golang) — Part 1
tags:
  - go
  - graph
---
Reviewing Graphs and how we can work with them in Go.

Press enter or click to view image in full size

Graph With Vertices (A, B, C, D) & Weighted (1, 2, 3) Edges

## What and Why?

First, let’s briefly review graphs and why they are used.

### Definition

A graph `G(V, E)` is a set of vertices (nodes) and edges (connections). Graphs are traversed using paths. Paths are also subgraphs since they explore a series of vertices and edges within a graph.

- **Vertices** are nodes that store data about themselves (e.g., name, price, lat/long…)
- **Edges** are connections that store data about the relationship between two nodes. The existence of an edge indicates a connection, and the edge itself can store data about that connection.  
    — E.g., we can store the weight of a connection between A → B, which may be at a different weight than the connection between B → A.

### Properties

Graphs have the following core properties:

1. **Connected**/Disconnected: In a **connected** graph, any two vertices are connected by the same path (All subgraphs are reachable from any vertex in the graph).
2. **Undirected**/Directed: In an **undirected** graph, the edges between nodes can be traversed in either direction; for example, if we have any two connected nodes, A and B, then we can traverse between A and B in either direction: A → B or B → A.
3. **Acyclic**/Cyclic: In an **acyclic** graph, no vertex has a path to itself directly or via waypoint(s), also known as a _cycle_.

### Use-Cases

Graphs are a robust data structure because they capture relationships between data entities. Below are some real-world products that use graphs.

- Maps: A path on a map is a collection of vertices and edges.
- Social Circles: A connection between two more people can be modeled as a collection of vertices (people) and edges (relationship).
- Networks: Connections between nodes in a network are modeled as a graph.

Let’s take a look at some examples using the properties above:

- If a road is bidirectional, it must have edges from start point A to endpoint B and vice versa.
- Can we suggest a friend to users on our social network by looking at their friends?
- Let’s send alerts when a part of our network is disconnected.
- We need to track the prices and routes of flights between cities.

## Representations

Graph data can be represented in Go code using 3 forms:

### **Matrix**

First, a matrix of values. Each position in the graph is a vertex, and the neighboring vertices are connected using _contextual_ edges. The problem’s context must provide the matrix's directionality (edges)!

A Matrix as a Graph

Let’s say our simple rules say that this is a board game, and each position on the board is connected in 4-ways **← ↑ → ↓**.

As an example, vertex **1** `{0, 0}` is connected by 2 edges only! `1 → 2` (right) and `1 → 4` (down) since the left and up edges are out of bounds.

### **Adjacency List**

An adjacency list is another way to represent a graph. The list uses every vertex as a key and maintains a list of vertices connected to that vertex [vertex → neighbor(s)].

An Adjacency List as a Graph

Vertex **1** is connected to 2 edges: `1 → 2` & `1 → 4` . Compared to the Matrix Graph, we do not have a clear view of the bounds; instead, we are given the specific vertices (keys) and edges (list) that form the graph. It needs no context since the edges, and possible paths are clear.

A possible path to vertex **9** from vertex **1**: `1 → 4 → 5 → 6 → 9`

- Note the above is not necessarily the “best” path, just a random one. We take a look at traversal and pathfinding in Part II.

### **Adjacency Matrix**

An adjacency matrix like the adjacency list requires no context since it defines the vertices and edges.

An adjacency matrix is a 2-D table where each pair of rows and columns represent the vertex. The intersection of a row and column represents if there is an edge between the two vertices.

Vertex **1** is connected to 2 vertices: `1 → 2` & `1 → 4`

vertex1, vertex2, vertex3 := 1-1, 2-1, 3-1 AdjacencyMatrix[vertex1][vertex2] // 1  
AdjacencyMatrix[vertex1][vertex3] // 0  
AdjacencyMatrix[vertex1][vertex4] // 1

Of course, the format for the adjacency matrix is up to the designer. The important thing is that each node is found on an equidistant row and column, and the relationship between each node and column is marked at that location on the matrix.

Take note of the bi-directionality in this example; since the example is based on the Matrix Graph, it is bi-directional! both `1 → 2` and `2 → 1` can be found.

## Abstract Data Structure

The core of a graph data structure is very similar to an adjacency list.

### **Types**

We need three types for the data structure. A Graph, a Vertex, and an Edge.

Graph Types

- Graph: The graph structure contains a map of vertex keys and objects. We use a map to optimize the search `O(1)`for a vertex. These are essentially the keys in the adjacency list.
- Vertex: The vertex is a structure that holds the data for a vertex. In our structure, we store an integer value and a map of edges. These are essentially the list part of the adjacency list.
- Edge: The edge is a structure that holds the data bout the connection between two vertices. In this case, the weight on the edge and the destination vertex.

### Methods

Next, we need to create our methods.

Graph Methods

- AddVertex: Adds a vertex to the graph without edges.
- AddEdge: Adds an edge between two vertices.
- Neighbors: returns (outgoing) connections for a vertex.

### Constructor

As a bonus, we create a constructor with a [functional option](https://dave.cheney.net/2014/10/17/functional-options-for-friendly-apis) to initialize a graph with an adjacency list!

The entire code file _with a test_ can be found 
[Code](https://gist.github.com/snassr/e79f4953eeb8d813be82eda00adeef57)
