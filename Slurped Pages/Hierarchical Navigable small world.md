---
link: https://centamori.com/index.php?slug=hierarchical-navigable-small-world-hnsw-php&lang=en
excerpt: Discover HNSW (Hierarchical Navigable Small World), the algorithm
  making vector search instant. A practical guide to PHP implementation using
  the Vektor library.
slurped: 2026-01-02T09:09
title: Ivan Centamori
---

## Hierarchical Navigable Small World (HNSW) in PHP

**Or: How to find a needle in a haystack without checking all the hay**

In the [previous article](https://centamori.com/index.php?slug=dungeons-dragons-vector-search) we saw how **vectors** can help find the right information simply by describing the concept we want to find.

We used **Cosine Similarity** to compare our request with _all_ available documents, scanning them one by one until we found those with the highest similarity. Does this approach work? _Yes._ Is it fast? _Sort of...._

Imagine if a librarian, to find your book, had to read the titles of **all 10 million volumes** in a National Library. Even if they spent one millisecond per book, it would take hours. This is the problem of linear search ($O(N)$).

To solve this problem, we introduce a concept that will transform that search from hours to milliseconds: **HNSW** (Hierarchical Navigable Small World).

## Small worlds and highways

Let's use an analogy to better understand the solution HNSW proposes: **How do we find a specific point in a huge city if we come from very far away and don't know the direction?**

Simple: in real life, we use a hierarchy of roads. That is, we start from a "broad" view of the map and subsequently "zoom in" towards the point of interest. Expressing this with an algorithm, we would:

1. Take the **highway** to get close to the region.
2. Exit onto the **state road** to reach the city.
3. Take the **main streets** for the neighborhood.
4. Finally, use the **side streets** to find the house number.

HNSW does exactly this with data. It builds a layered structure (Layers) where the higher levels are like highways (few points, long connections) and the lower levels are neighborhood streets (all points, short connections).

![](https://centamori.com/content/images/hnsw-display-1.png)

## The Code

Let's analyze how to implement this logic in PHP. The code we will examine comes from the open source project [Vektor](https://github.com/centamiv/vektor), a small service I built that allows for native vector search in PHP.

Technical note: In addition to `$ef`, there is a crucial parameter called `$M`. Imagine it as the maximum number of roads that can start from an intersection. The higher `$M` is, the more connected the city is, but the more memory our map will occupy.

### The descent to Level 0

The main logic is in the `search` method, which we see below. Imagine looking at a satellite map and progressively zooming in. We start from the global view (the nodes of the highest level).

```
public function search(array $queryVector, int $k, int $ef): array
{
    // Start from the highest level (e.g. Level 3)
    $entryPoint = $this->readHeader();

    // Calculate the initial distance, i.e., compare the Level 3 object with the search target
    $currObj = $entryPoint;
    $currDist = Math::cosineSimilarity($queryVector, $this->getVector($currObj));

    // Now progressively descend from high levels down to level 1
    // The for loop starts from the max level and repeats until we reach Level 1
    for ($lc = $maxLevel; $lc >= 1; $lc--) {

        // Navigate within the current level until we find the closest point
        while (true) {
            $changed = false;

            // Read the neighbors of the current node at this level
            $node = $this->readNode($currObj);
            $neighbors = $node['connections'][$lc] ?? [];

            // Search if there is a neighbor that is CLOSER to our target
            foreach ($neighbors as $neighborId) {
                $dist = Math::cosineSimilarity($queryVector, $this->getVector($neighborId));
                if ($dist > $currDist) {
                    $currDist = $dist;
                    $currObj = $neighborId;
                    $changed = true; // We found a better point!
                }
            }

            // If there were no changes in choosing the best node,
            // it means we are already at the closest possible point 
            // in this level, so we can "descend" to the lower level
            if (!$changed) break;
        }

        // Now $currObj is set with the starting point for the lower level ($lc - 1)
    }

    // At the end of the loop we will be at Level 0 and $currObj will contain the closest point!
}
```

This cycle is the equivalent of identifying the area of interest from above, moving towards the point closest to the destination, and then increasing the detail. Repeat until we are at the maximum detail level (Level 1).

### Precision Search (Level 0)

Once we arrive at Level 0, where _all_ the data resides, a single point is no longer enough. We want the **K** best results. This is where the `$ef` (Construction/Search Size) parameter comes into play.

`$ef` is the size of our "candidate list". The higher the number, the more nodes we evaluate, and the more precise (and slow) the search will be.

The `searchLayer` method adopts a **Greedy** approach: at each step, the algorithm makes the locally optimal choice, moving towards the neighbor that maximizes similarity with the query. Instead of exploring every possible path, the algorithm "follows the gradient" towards the target. The **Priority Queue** manages this process, allowing us to explore the most promising nodes first and stop the search as soon as new candidates stop improving our list of best results ($ef$).

```
public function searchLayer(int $entryPoint, array $queryVector, int $ef, int $level, ?int $k = null): array
{
    $visited = [$entryPoint => true];
    $candidates = new SplPriorityQueue(); // Promising nodes to explore

    $winners = [$entryPoint => $entrySim]; // Best results found so far (Winner list)

    while (!$candidates->isEmpty()) {
        $c = $candidates->extract(); // Take the best candidate

        // If the best candidate is worse than the worst of our current "winners"
        // and we have already filled the list ($ef), we can stop.
        // It's useless to explore further, we are moving away.
        if ($cSim < $worstSim && count($winners) >= $ef) {
            break;
        }

        // Explore the candidate's neighbors
        $node = $this->readNode($c);
        $neighbors = $node['connections'][$level] ?? [];

        foreach ($neighbors as $neighborId) {
            if (!isset($visited[$neighborId])) {
                $visited[$neighborId] = true;

                // For each neighbor, calculate its similarity with the target query
                $sim = Math::cosineSimilarity($queryVector, $this->getVector($neighborId));

                // If this neighbor is promising, add it to candidates
                if ($sim > $worstSim || count($winners) < $ef) {
                    $candidates->insert($neighborId, $sim);
                    $winners[$neighborId] = $sim;

                    // Keep the winner list W at a fixed size $ef
                    if (count($winners) > $ef) {
                        asort($winners);
                        $idToRemove = array_key_first($winners); // Remove the worst
                        unset($winners[$idToRemove]);

                        // Update the "worst acceptable" threshold
                        asort($winners);
                        $worstSim = $winners[array_key_first($winners)];
                    }
                }
            }
        }
    }

    // ... return the top K elements from the $winners list ...
}
```

## Why is it fast?

Let's go back to the library example.

- In **Linear Search**, the librarian checks every book. $O(N)$ operations.
- In **HNSW**, the librarian looks at the "Science" section (Level 2), then goes to the "Astronomy" shelf (Level 1), and finally checks only the 50 books on that shelf (Level 0). $O(\log N)$ operations.

On a dataset of 1 million objects, instead of making 1,000,000 comparisons, we might make only 500 or 1,000. It's a difference of orders of magnitude.

## Building the map

So far we have taken for granted that these "highways" and "neighborhood streets" already exist. But how are they built? The genius of HNSW lies in the fact that the map is drawn dynamically, one point at a time.

When we need to insert a new piece of data (a vector) into our index, we have to decide two things:

1. **Which zone is it in?** (Max level)
2. **Who are its neighbors?**

For the first question, HNSW relies on chance. It's like **flipping a coin** (or more technically, using an exponential probability distribution). Most points will end up at **Level 0** (the residential neighborhood). Some lucky ones will win a "ticket" to **Level 1** (the state roads). Very few "chosen ones" will reach the highest levels (the highways). This ensures that the upper levels remain uncluttered and fast to traverse.

For the second question, **we use the search seen previously to also perform the insertion.**

If our new point has been assigned to Level 1:

1. We start from the absolute highest level (e.g. Level 3).
2. We look for the closest point to our newcomer (using the logic of `searchLayer` seen before).
3. We go down to the level below and repeat, until we reach the assigned level (Level 1).
4. Here we stop and **connect** the new point to its nearest neighbors (based on the parameter `$M`, the maximum number of connections).

Basically, every new citizen who arrives in the city asks for directions to find their area, and once arrived, makes friends with the neighbors most similar to them. If a neighbor has too many friends, they cut ties with the least similar one to make room for the newcomer. It is this process of _natural selection_ of connections that keeps the graph efficient and navigable.

## Conclusion

Implementing HNSW from scratch is a fundamental exercise to understand how modern vector databases like Qdrant or Pinecone work. We are not just "searching for data", we are navigating a multidimensional space using probabilistic shortcuts.

These algorithms are the basis of modern recommendation systems ("Users who bought this also saw....") and RAG (Retrieval Augmented Generation) systems for artificial intelligence, allowing relevant context to be found in fractions of a second.

If you want to dive deeper into the complete implementation, check out the Vektor [https://github.com/centamiv/vektor](https://github.com/centamiv/vektor) repository on GitHub.