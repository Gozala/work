# Tuning the Tradeoff Between Boundary Stability and Size Uniformity

Dialog DB is designed with local-first principles, aiming to operate without traditional servers by building on content-addressed storage. Like Git, local replicas can cooperate through remote content-addressed stores. To achieve efficient replication, we use probabilistic B-trees - data structures that maintain insertion-order-agnostic layouts and resilience to boundary shifts. However, we encountered a problem with node size distribution that led us to explore different boundary detection strategies.

## Network IO vs Disk IO Considerations

Traditional databases optimize chunk sizes for disk I/O, but Dialog DB reads blobs (serialized nodes) over networks where latency dominates. Wasting a round trip on a tiny node hurts far more than reading a slightly oversized one. Conversely, massive nodes can blow through memory budgets and slow down operations. This fundamentally different access pattern shapes our approach to boundary detection.

## A Brief Primer on Probabilistic B-Trees

The concept of probabilistic B-trees was first introduced by [Noms](https://github.com/attic-labs/noms). [Mikeal Rogers introduced me to this concept](https://github.com/mikeal/prolly-trees) through his implementation. [Joel Gustafson's article about Okra's design](https://joelgustafson.com/posts/2023-05-04/merklizing-the-key-value-store-for-fun-and-profit) provides excellent insights into the sync properties of these data structures.

[G-trees](https://g-trees.github.io/g_trees/) present a similar data structure that shares many properties with probabilistic B-trees but with an even simpler algorithm. In Okra, you derive tree layout by building it up from leaves level by level, checking for boundary nodes. In G-trees, you derive ranks for all leaves which determine their position on the vertical axis. Our current Dialog DB implementation follows this G-tree approach, as described in [this article on flipping bits with hashes](https://blog.textile.io/weeknotes-generating-distributed-random-variables-using-cryptographic-hashes,-solving-ethers-v6-hardhat-nonce-issues,-and-thinking-about-responsive-design).

[Dolt's work on chunk sizes](https://www.dolthub.com/blog/2022-06-27-prolly-chunker/) added important insights about the relationship between boundary detection and blob sizes.

## The Uneven Distribution Problem

Using geometric distribution based on node hashes works well on average - nodes end up with roughly the desired branching factor. However, variance is high. Some nodes accumulate far more children than desired while others remain sparse.

Our boundary detection follows the G-tree approach where we derive rank from node hash. A node becomes a boundary if its rank is higher than the current level (with leaves at level 0). While mathematically elegant, it can produce highly uneven node sizes in practice.

## Understanding Boundary Shifts

In pure content-based systems like Okra, inserting a node never shifts boundaries - it may split nodes but boundaries depend only on node content. However, when we start accounting for additional context like node sizes, this property weakens. 

Consider what happens when capacity constraints influence boundaries: a node might become a boundary only because its parent reached capacity. Then the next insertion at the same position with a lower key could make the previously boundary node into a non-boundary node. This reduces boundary stability and could potentially cause cascading effects if adjacent nodes also became boundaries due to capacity constraints.

That said, cases where we'd see cascading effects would also be cases where Okra would have giant nodes spanning the range of the cascade. It's a tradeoff between two problems.

## The Tradeoff: Boundary Stability vs Size Uniformity

Dolt's insight resonates with our experience: there's a tradeoff between boundary stability (resistance to boundary shifts) and size uniformity (consistent blob sizes). Pure content-based boundaries maximize stability but allow wide size variance. Size-based splitting creates uniform chunks but boundaries shift more easily.

My intuition is that this tradeoff matters differently for different node types:

**Index nodes** benefit from boundary stability. We want tighter bounds on child count while still having some boundaries on chunk sizes to ensure low-entropy data doesn't go completely out of bounds.

**Segment nodes** are different. Like [Datomic's segments](https://tonsky.me/blog/unofficial-guide-to-datomic-internals), our segment nodes don't conceptually have children - they hold facts. The number of facts can vary wildly depending on content. With columnar encoding applied, the correspondence between fact count and blob size can be fairly low. Size matters more here, though letting content also play a role can provide more stability.

## A Constraint-Based Approach

Rather than choosing a fixed strategy, we're exploring a constraint-based API that lets different node types express their requirements:

```typescript
type NodeConstraints = 
  | { count: { min: number; max: number }; capacity: { min: number } }      // Index nodes
  | { count: { min: number }; capacity: { min: number; max: number } }      // Segment nodes
  | { count: { min: number; max: number }; capacity: { min: number; max: number } }; // Both
```

This design acknowledges that different parts of the tree have different needs. Index nodes can prioritize boundary stability while keeping child count reasonable. Segment nodes can focus on blob size while maintaining minimum fanout for efficiency.

## Striking a Balance

Dialog DB aims to balance these competing concerns so that starting queries from an empty database can be efficient and syncing between replicas minimizes unnecessary blob transfers.

By making the tradeoff explicit and tunable, we can experiment with different strategies for different workloads. Low-entropy datasets might need tighter size bounds to prevent degenerate cases. High-churn workloads might prefer more stability to reduce sync traffic.

The constraint-based approach also future-proofs the design. As we learn more about real-world usage patterns, we can refine the boundary detection strategies without changing the fundamental API.

## Next Steps

We haven't yet implemented or proven this approach will solve our distribution problems. But by learning from Okra, G-trees, Dolt, and others who've explored this space, we're developing a framework that acknowledges the tradeoffs rather than seeking a one-size-fits-all solution.

The key insight is recognizing that boundary stability and size uniformity exist in tension. Different use cases need different points on that spectrum. By making that choice explicit and tunable, we hope to build a system that works well across diverse workloads while maintaining the properties that make probabilistic B-trees valuable for local-first applications.