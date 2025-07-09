# Balancing Boundary Stability and Size Uniformity in Probabilistic B-Trees

Dialog DB is designed around local-first principles, aiming to operate without traditional servers by building on content-addressed storage. Like Git, local replicas can cooperate through remote content-addressed stores. To achieve efficient replication, we use probabilistic B-trees - data structures that maintain insertion-order-agnostic layouts and resilience to boundary shifts. However, we encountered an interesting problem with node size distribution that led us to explore different boundary detection strategies.

## A Brief Primer on Probabilistic B-Trees

The concept of probabilistic B-trees has evolved through several implementations, each contributing valuable insights. [Mikeal Rogers first introduced many of us to this concept](https://github.com/mikeal/prolly-trees), building on ideas from Noms. [Okra's excellent article](https://joelgustafson.com/posts/2023-05-04/merklizing-the-key-value-store-for-fun-and-profit) provides a clear explanation of how these data structures achieve their sync properties through content-based boundaries.

[G-trees simplified the algorithm further](https://g-trees.github.io/g_trees/) by deriving rank directly from node hashes - essentially "flipping bits with hashes" as described in [this article on generating geometric distributions](https://blog.textile.io/weeknotes-generating-distributed-random-variables-using-cryptographic-hashes,-solving-ethers-v6-hardhat-nonce-issues,-and-thinking-about-responsive-design). Our current Dialog DB implementation follows this approach: we derive rank from node hash and use that rank to determine boundaries. A node becomes a boundary if its rank is higher than the current level (with leaves at level 0).

[Dolt's work on chunk sizes](https://www.dolthub.com/blog/2022-06-27-prolly-chunker/) added another dimension to consider. While deterministic layout gets much attention, resilience to boundary shifts proves equally important for efficient sync.

## The Uneven Distribution Problem

Using geometric distribution based on node hashes works well on average - nodes end up with roughly the desired branching factor. However, variance is high. Some nodes accumulate far more children than desired while others remain sparse.

Here's how our boundary detection works, following the g-trees approach:

```typescript
function rank(node: Node): number {
  const hash = sha256(node.content);
  // Count leading zeros in hash to get rank
  return countLeadingZeros(hash);
}

function isBoundary(node: Node, currentLevel: number): boolean {
  return rank(node) > currentLevel;
}
```

This creates a geometric distribution where higher-ranked nodes become boundaries at higher tree levels. While mathematically elegant, it can produce highly uneven node sizes in practice.

The problem becomes acute when optimizing for network reads rather than local disk access. Traditional databases optimize chunk sizes for disk I/O, but we're reading over networks where latency dominates. Wasting a round trip on a tiny node hurts far more than reading a slightly oversized one. Conversely, massive nodes can blow through memory budgets and slow down operations.

## The Tradeoff: Boundary Stability vs Size Uniformity

Dolt's insight resonates with our experience: there's a fundamental tradeoff between boundary stability (resistance to boundary shifts) and size uniformity (consistent blob sizes). Pure content-based boundaries maximize stability but allow wide size variance. Size-based splitting creates uniform chunks but boundaries shift more easily.

This tradeoff matters differently for different node types:

**Index nodes** benefit from boundary stability. Shifting boundaries in the index cascade through the tree, potentially affecting many blobs. We want reasonably tight bounds on child count to maintain good fanout, but stability takes priority.

**Segment nodes** have different constraints. With columnar encoding, the correspondence between child count and blob size can be quite low - a few highly compressed columns might take less space than many sparse ones. Here, controlling blob size matters more than child count.

## A Constraint-Based Approach

Rather than choosing a fixed strategy, we're exploring a constraint-based API that lets different node types express their requirements:

```typescript
type NodeConstraints = 
  | { count: { min: number; max: number }; capacity: { min: number } }      // Index nodes
  | { count: { min: number }; capacity: { min: number; max: number } }      // Segment nodes
  | { count: { min: number; max: number }; capacity: { min: number; max: number } }; // Both

interface ConstrainedNode {
  shouldSplit(childToAdd: Node): boolean {
    // Hard boundaries always apply
    if (this.children.length >= this.constraints.count.max) return true;
    if (this.size + childToAdd.size >= this.constraints.capacity.max) return true;
    
    // Soft boundaries use adaptive probability
    const progress = calculateProgress(this);
    const adaptiveQ = deriveQ(progress, this.constraints);
    
    const hash = sha256(childToAdd.content);
    const threshold = 2**32 / adaptiveQ;
    return u32(hash) < threshold;
  }
}
```

This design acknowledges that different parts of the tree have different needs. Index nodes can prioritize boundary stability while keeping child count reasonable. Segment nodes can focus on blob size while maintaining minimum fanout for efficiency.

## Striking a Balance

Dialog DB aims to balance these competing concerns. Starting queries from an empty database should be efficient (requiring reasonable blob sizes), and syncing between replicas should minimize unnecessary blob transfers (requiring boundary stability).

By making the tradeoff explicit and tunable, we can experiment with different strategies for different workloads. Low-entropy datasets might need tighter size bounds to prevent degenerate cases. High-churn workloads might prefer more stability to reduce sync traffic.

The constraint-based approach also future-proofs the design. As we learn more about real-world usage patterns, we can refine the boundary detection strategies without changing the fundamental API.

## Next Steps

We haven't yet implemented or proven this approach will solve our distribution problems. But by learning from Okra, g-trees, Dolt, and others who've explored this space, we're developing a framework that acknowledges the fundamental tradeoffs rather than seeking a one-size-fits-all solution.

The key insight is recognizing that boundary stability and size uniformity exist in tension. Different use cases need different points on that spectrum. By making that choice explicit and tunable, we hope to build a system that works well across diverse workloads while maintaining the properties that make probabilistic B-trees valuable for local-first applications.