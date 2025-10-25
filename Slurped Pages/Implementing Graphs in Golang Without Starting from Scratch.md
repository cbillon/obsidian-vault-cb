---
link: https://journal.hexmos.com/graph-implementation-in-go/
byline: Ganesh Kumar
site: Hexmos Journal
date: 2025-07-27T18:09
excerpt: |-
  Have you ever imagined a world without connections? Think about your social media full of random unknown people in your follow recommendations, maps not showing the shortest direction between two destinations, or file imports turning into a mess of unrelated modules after compiling. This would be hard even to imagine, right?



  This type of core data is managed by creating connections between it, thereby forming a network of relationships. In programming terms, we call this a graph data stru
twitter: https://twitter.com/@AmazingStardom
slurped: 2025-08-31T19:26
title: Implementing Graphs in Golang Without Starting from Scratch
tags:
  - go
  - graph
  - dag
---

Have you ever imagined a world without connections? Think about your social media full of random unknown people in your follow recommendations, maps not showing the shortest direction between two destinations, or file imports turning into a mess of unrelated modules after compiling. This would be hard even to imagine, right?

This type of core data is managed by creating connections between it, thereby forming a network of relationships. In programming terms, we call this a [graph data structure](https://en.wikipedia.org/wiki/Graph_\(abstract_data_type\)?ref=journal.hexmos.com).

In this article, you'll see how to use this structure to solve any problems related to relations.

## What is a Graph?

A graph is a nonlinear data structure that contains nodes and edges. A node (or vertex) is just a single unique value, while an edge represents a connection or relationship between two of these nodes.

![Graph Structure](https://journal-wa6509js.s3.ap-south-1.amazonaws.com/0257a2bee995a4edd6672035e058190224b49c9c47a875c289f1a9d3858bf5fc.png)

Think of something like Instagram: every user is a node, and every time you follow a user, you create a new edge connecting two nodes together.

This is known as a directed graph because the relationship flows one way—the follower follows the followed, and not vice‑versa.

![Directed Graph Structure](https://journal-wa6509js.s3.ap-south-1.amazonaws.com/c262a4419222045503a9db4262e018242c6a33b86ca05e76dc07dc13cc9a956e.png)

Now consider Facebook: your edges in the graph are friendships, and this relationship goes both ways, which is known as an undirected graph.

![Undirected Graph Structure](https://journal-wa6509js.s3.ap-south-1.amazonaws.com/a14a4b2a122805d217f7d3de57c0d5541f602935aa94d18c5ad0faca1a831ba7.png)

Graphs can also be **weighted**, which means the edge has some additional data about the relationship, like the distance between two airports.

A node might also point to itself, like an airplane that takes off and comes back to land at the same airport—this is known as a cycle.

When a directed graph has no cycles—that is, you can’t start at one node and eventually loop back—it’s called a **[Directed Acyclic Graph (DAG)](https://en.wikipedia.org/wiki/Directed_acyclic_graph?ref=journal.hexmos.com)**. DAGs are perfect for modeling hierarchical structures, where each link has a clear one‑way flow (for example, parent → child in a family tree).

One way to represent a graph in memory is with an **adjacency matrix** (a 2D array). You create one row and one column per node; if there’s an edge from node _i_ to node _j_, you place a `1` at cell _(i, j)_, otherwise a `0`. This makes lookups for a specific edge extremely fast, but costs **O(n²)** space and even **O(n²)** time to insert a new node.

**Example:** Imagine three people: Alice (A), Bob (B), and Carol (C).

- Alice follows Bob
    
- Bob follows Carol
    
- Carol doesn’t follow anyone
    

The adjacency matrix would look like this:

```

    A B C

A [ 0 1 0 ]

B [ 0 0 1 ]

C [ 0 0 0 ]

```

Here, A→B and B→C, but no connection from C to anyone.

An alternative approach is an adjacency list: in this representation, we start with a collection of nodes, then each node has its array of its neighbours. This makes it faster to iterate over a node's edges and is more efficient with memory, especially when you have many nodes and few edges.

**Example:**

- **Alice:** follows Bob
    
- **Bob:** follows Carol
    
- **Carol:** follows no one
    

This is like:

- A → B
    
- B → C
    
- C → —
    

This format is much more space-efficient when most people follow only a few others. It's like everyone keeping a short list of their connections instead of filling out a big full table.

## Applying Graph Concepts to a Family Tree

Let us now consider a **DAG** and apply it to something like **family relationships**:  
Imagine a family tree—parents pointing to children.

- Parent → Child
    
- Grandparent → Parent
    

No loops are possible because a child can't point back to an ancestor. This makes DAGs ideal for modeling things like **family trees**, where a person can have one or more children, but no backward links to parents or ancestors.

Let's work with a concrete example. Imagine you have a scattered list of family members:

```
// Sample Data: A scattered list of people.
familyData := []FamilyMember{
    {ID: "1", Name: "Jordon D", ParentName: ""},
    {ID: "2", Name: "Robert Smith", ParentName: ""},
    {ID: "3", Name: "Danis Jordan", ParentName: "Jordon D"},
    {ID: "4", Name: "John Smith", ParentName: "Robert Smith"},
    {ID: "5", Name: "Maria Smith", ParentName: "Robert Smith"},
    {ID: "6", Name: "Leo Smith", ParentName: "John Smith"},
}
```

This flat slice hides a clear family structure. We can model this with a DAG where each person is a **node** and each parent-child link is a directed **edge**.

First, we'll define a `struct` for our nodes using the [github.com/dominikbraun/graph](https://github.com/dominikbraun/graph?ref=journal.hexmos.com) library.

```
import (
    "fmt"
    "github.com/dominikbraun/graph"
)

// FamilyMember represents a person in our dataset.
type FamilyMember struct {
    ID         string
    Name       string
    ParentName string
}
```

## Building and Visualizing the Graph

The best way to understand our data is to build the graph and then visualize it. We'll use the **[Graphviz](https://graphviz.org/?ref=journal.hexmos.com)** tool to generate a PNG image of our family tree.

The function below handles building the graph from our data. Note that we now return both the library graph and a map of names to IDs, and we include error handling on every graph operation:

```
// buildFamilyGraph takes raw data and constructs a graph representation.
// Returns the graph and the name→ID map, or an error if any operation fails.
func buildFamilyGraph(
    members []FamilyMember,
) (graph.Graph[string, FamilyMember], map[string]string, error) {

    // Create a directed graph, using each member's ID as the unique key.
    g := graph.New(func(m FamilyMember) string { return m.ID }, graph.Directed())

    // Map for looking up IDs by name (warn on duplicates).
    nameToID := make(map[string]string)

    // Add vertices.
    for _, m := range members {
        if err := g.AddVertex(m); err != nil {
            return nil, nil, fmt.Errorf("adding vertex %q: %w", m.ID, err)
        }
        if prev, exists := nameToID[m.Name]; exists {
            fmt.Fprintf(os.Stderr,
                "warning: duplicate name %q for IDs %q and %q\n",
                m.Name, prev, m.ID,
            )
        }
        nameToID[m.Name] = m.ID
    }

    // Add edges for each parent→child link.
    for _, m := range members {
        if m.ParentName == "" {
            continue
        }
        parentID, ok := nameToID[m.ParentName]
        if !ok {
            fmt.Fprintf(os.Stderr,
                "warning: parent %q for %q not found\n",
                m.ParentName, m.ID,
            )
            continue
        }
        if err := g.AddEdge(parentID, m.ID); err != nil {
            return nil, nil, fmt.Errorf(
                "adding edge %q → %q: %w", parentID, m.ID, err,
            )
        }
    }

    return g, nameToID, nil
}
```

With the graph built, we can generate a `.dot` file and use Graphviz to render it as an image. This instantly clarifies the family structures.

![Graph Structure dot file Image](https://journal-wa6509js.s3.ap-south-1.amazonaws.com/d2907bfa62792f371c048a2de9177f56ef7f2a8ffa8ea481dd1a4bd86679574d.png)

## Graph Traversals & Queries

Now that we have a structured graph, we can perform operations to answer specific questions.

### Finding Descendants with Depth-First Search (DFS)

**Depth-First Search (DFS)** explores as far as possible down each branch before backtracking. It's perfect for tracing a single line of descent.

```
// Traces a line of descent from Robert Smith (ID: "2").
fmt.Println("\n[DFS] Tracing all descendants of Robert Smith:")
_ = graph.DFS(familyGraph, "2", func(value string) bool {
    member, _ := familyGraph.Vertex(value)
    fmt.Printf(" -> %s\n", member.Name)
    return false // Continue traversal
})
```

**Output:**

```
 -> Robert Smith
 -> John Smith
 -> Leo Smith
 -> Maria Smith
```

### Exploring by Generation with Breadth-First Search (BFS)

**Breadth-First Search (BFS)** explores level by level. It's ideal for seeing the family structure generation by generation.

```
// Explores the tree starting from Jordon D (ID: "1").
fmt.Println("\n[BFS] Exploring family by generation starting from Jordon D:")
_ = graph.BFS(familyGraph, "1", func(value string) bool {
    member, _ := familyGraph.Vertex(value)
    fmt.Printf(" - Found: %s\n", member.Name)
    return false // Continue traversal
})
```

**Output:**

```
 - Found: Jordon D
 - Found: Danis Jordan
```

### Identifying Original Ancestors (Roots)

**Root nodes** are those with no incoming edges—in our case, people with no parents in the dataset. We can find them by checking which nodes have no predecessors.

```
fmt.Println("\n[Roots] Finding the original ancestors:")
predecessors, _ := familyGraph.PredecessorMap()
for id, parents := range predecessors {
    if len(parents) == 0 {
        member, _ := familyGraph.Vertex(id)
        fmt.Printf(" - %s is a root ancestor.\n", member.Name)
    }
}
```

**Output:**

```
 - Jordon D is a root ancestor.
 - Robert Smith is a root ancestor.
```

### Finding the Youngest Generation (Leaves)

**Leaf nodes** are those with no outgoing edges—people who have no children in our dataset. We can find them by checking which nodes have no successors in the adjacency map.

```
fmt.Println("\n[Leaves] Finding members with no children:")
successors, _ := familyGraph.AdjacencyMap()
for id, children := range successors {
    if len(children) == 0 {
        member, _ := familyGraph.Vertex(id)
        fmt.Printf(" - %s has no children.\n", member.Name)
    }
}
```

**Output:**

```
 - Danis Jordan has no children.
 - Leo Smith has no children.
 - Maria Smith has no children.
```

By converting flat data into a graph, we unlock the ability to see and query complex relationships with ease. Go, combined with powerful libraries like `dominikbraun/graph`, provides a robust toolkit for turning chaotic data into clear, actionable insights.

## Conclusion

So, we've seen how turning a simple list into a graph can sustain a relationship of data. I hope you liked this article and learned a few new things.

If you're interested in exploring or building on this example, the complete source code is available here: **[github.com/Amazing-Stardom/go-graph-example](https://github.com/Amazing-Stardom/go-graph-example?ref=journal.hexmos.com)**

This is just one way to use a graph, but the possibilities are endless. What would you build with this? I'm genuinely curious to hear your thoughts, so feel free to leave a comment with any ideas or improvements for this approach.

By the way, if you find this kind of problem-solving interesting, you might want to check out what I'm building with [LiveAPI](https://hexmos.com/landing/liveapi?ref=journal.hexmos.com). It’s a tool that helps developers automatically discover and make sense of APIs in large, complex systems.

[![](https://journal-wa6509js.s3.ap-south-1.amazonaws.com/94beacb18af3fce3f795a16bf1a507fdda94245f04ce0ef3fd763cc5d1772cbf.png)](https://hexmos.com/liveapi?utm_source=https://journal.hexmos.com/graph-implementation-in-go&utm_medium=bannerad&utm_campaign=liveapi)

**[LiveAPI: Interactive API Docs that Convert](https://hexmos.com/liveapi?utm_source=https://journal.hexmos.com/graph-implementation-in-go&utm_medium=bannerad&utm_campaign=liveapi)**

Static API docs often lose the customer's attention before they try your APIs. With LiveAPI, developers can instantly try your APIs right from the browser, capturing their attention within the first 30 seconds.

 [![](https://journal-wa6509js.s3.ap-south-1.amazonaws.com/da11375afb47769cab987d89ac81a9a2ca8d863fa62267ebd86d64ff981a3981.gif)](https://www.linkedin.com/company/hexmos?ref=journal.hexmos.com)[![](https://journal-wa6509js.s3.ap-south-1.amazonaws.com/4f3ad2e999c95ddfb3e4e1a7c683e9bbd4866f3e7b4255e68c4fc553da5ea82e.gif)](https://twitter.com/HexmosTech?ref=journal.hexmos.com)