---
title: Tuning the Tradeoff Between Boundary Stability and Size Uniformity
date: 2025-07-09
---

#  Tuning the Tradeoff Between Boundary Stability and Size Uniformity

Dialog DB is designed with [local-first][] principles, aiming to operate without traditional servers by building on content-addressed storage. Like Git, local replicas can cooperate through remote content-addressed stores. To achieve efficient replication in such settings, dialog uses search tree algorithm that is [history independent][] and resilient to [boundary shifts][]. Narrowing [size variance][] is currently under exploration and subject of this article.

## A Brief Primer on Search Trees

### [B-Tree][]

> In [computer science](https://en.wikipedia.org/wiki/Computer_science), a **B-tree** is a self-balancing [tree data structure](https://en.wikipedia.org/wiki/Tree_data_structure) that maintains sorted data and allows searches, sequential access, insertions, and deletions in [logarithmic time](https://en.wikipedia.org/wiki/Logarithmic_time). 

As so many good things were invented in 70s and is a classic data structure for databases. 

They are not [history-independent][] though as the tree layout depends significantly on the insertion order. Typical implementations perform rebalancing which also causes  [boundary shifts][]. This makes them impractical for Dialog DB.

### Merkle Search Trees (MST)

> *A. Auvolat, F. Taïani. (2019) “*[Merkle Search Trees: Efficient State-Based CRDTs in Open Networks](https://hal.inria.fr/hal-02303490/document)*“*

To my knowledge first known algorithm  defining a B-tree that is [history-independent][] and resilient to [boundary shifts][]. Ideas is elegant - hash entries and place them in a tree at a level proportional to the number of 0’s in the prefix of their hash. I do however find  insertion / removal algorithm a lot more complicated than algorithms that follow. 

Notably they MST has being [incorporated into @ATProtocol spec](https://atproto.com/specs/repository#repo-data-structure-v3)

### Probabilistic B-Tree

#### Prolly Tree

Originally introduced in [Noms](https://github.com/attic-labs/noms/blob/master/doc/intro.md#prolly-trees-probabilistic-b-trees) and invented by [Aaron Boodman](https://aaronboodman.com/). Original idea was to apply content-defined chunking (CDC) via [rolling hash function](https://en.wikipedia.org/wiki/Rolling_hash) to delimit nodes into sibling groups, create nodes from those groups forming a parent layer and repeat until you have tree. Updates are also simple, add / remove leaf and perform prior logic starting from the target sibling group. Since tree layout is derived from applying CDC on leaves it is [history-independent][]. Algorithm also inherits resilience to  [boundary shifts][] from CDC.

#### Chunky Tree

I was first introduced to the novel search trees by [Mikeal Rogers](https://github.com/Gozala/mikeals-cancer-diaries/blob/832f1952b5a271e04acbf1c3a40995abc7172f2f/best-memories/one-who-cares.md) who [independently came up](https://github.com/attic-labs/noms/issues/3878) with [Chunky Tree](https://github.com/mikeal/prolly-trees) algorithm that was remarkably similar to Prolly Tree. Main difference was that entries were delimited into sibling groups by identifying **boundary nodes**, which were determined from node hash itself based on probability distribution.

> I suspect difference in idea may have stemmed from the fact that Mikeal worked on IPLD where content to store was already identified by hash.

After Mikeal discovered Prolly Trees he adopted and continued implementation under that name.

#### Okra

[Joel Gustafson](https://joelgustafson.com) recognized an interesting symmetry between [Prolly Tree][Prolly Tree] and [Skip List][] if you add anchoring backbone it, which also happens to eliminate (leftmost) [boundary shift][boundary shifts] that is possible otherwise. **Okra** is an implementation of this design and a manifestation of another insight - you can map _Prolly Tree_ to a key value store *(that usually would be B-Tree)*.

> I highly recommend [Merklizing the key/value store for fun and profit](https://docs.canvas.xyz/blog/2023-05-04-merklizing-the-key-value-store.html#key-value-stores-all-the-way-down) as the best introduction to Prolly Trees and inherent properties that make them a great synchronization primitive in distributed settings.

While idea of mapping prolly tree onto key value store is brilliant, it [comes with a tradeoffs](https://docs.canvas.xyz/blog/2023-05-04-merklizing-the-key-value-store.html#concurrency-constraints) like prevent git style branching workflows. My attempts to come up with a different mapping strategy that would address this have failed.

#### Dolt

Dolt evolved from idea of original [Prolly Tree][] optimizing it for SQL database with Git-like version control. They have identified problem with **[size variance][]** and addressed it with an elegant solution: dynamically adjust probability that node needs to be determined a **boundary node**. They also took inspiration from Mikeal's idea and [switched from rolling hashes to key hashes](https://www.dolthub.com/blog/2022-06-27-prolly-chunker/#rolling-hash-stability). 

### Geometric Search Tree (G-tree)

[G-tree](https://g-trees.github.io/g_trees/) is the newest data structure that offers similar properties, but with even simpler algorithm - Instead of delimiting nodes into siblings groups at every level of the tree you assign geometrically chosen [ranks](https://g-trees.github.io/g_trees/#rank_informal) and treat them as nodes vertical position (same ranked adjacent nodes form sibling groups making). Geometric distribution can be [simulated from entry hashes](https://textile.notion.site/Flipping-bits-and-coins-with-hashes-205770b56418498fba4fef8cb037412d).

#### Search Tree in Dialog

Dialog's search tree implementation is based on the [G-tree][] algorithm. It slightly diverges from paper by pushing entries into leaf layer and in that regard is more similar to [Okra][] in terms of layout, however it or other [prolly tree][] we don't need to detect boundaries in the upper levels of the tree by node hashes we can do it by comparing rank of leftmost leaf (of that node) to tree height is not same it is a boundary.

### Characteristics of Search Trees

It appears that there are following key characteristics that impact query, update and replication performance and are in tension with each other.

- strong [history Independence][]
- resilience to [boundary shifts][]
- narrow [size variance][]

It seems that you can guarantee two out of three, or compromise some to a degree to make some guarantees on third. Current hypothesis is that optimal balance can be achieved for Dialog by compromising some on first two and improve third.

### History Independence

> In a history independent data structure, nothing can be learned from the memory representation of the data structure except for what is available from the abstract data structure. 

More simply if the structure of the tree is independent of insertion order it is [history independent](https://dl.acm.org/doi/10.1007/s00453-004-1140-z). This property is very important if you want to synchronize uncoordinated changes to the tree. That is because it allows finding differences between two replicas without having to explore all paths. If insertion order could load to different tree structures we would not even be able to tell if two replicas are the same scanning the whole thing. 

For a while I was thinking about _history Independence_ as a binary property but that seems little close minded. Better way to think about is in terms of how much drift from perfect can be tolerated - how strongly history Independent structure is. It is important to remember why we care about this property and I would argue that if we can weaken 

 so we can identify differences efficiently, however there are scenarios where we can retain ability to check differences  

 and in certain cases you may actually improve on this by allowing bit of drift. 

### Boundary Shifts

Stronger the resilience to boundary shifts the smaller the impact of a random edit. To make it more concrete it would a lot simpler that mentioned algorithms to delimit nodes into sibling groups by a fixed sizes and build tree that way this would give us perfectly [history independent][] tree and perfectly narrow [size variance][] but absolutely terrible resilience to boundary shifts because inserting an entry would update everything on the left of it. On the other hand [Okra][] has perfect [history independence][] a high resilient to [boundary shifts][] allowing to make very little changes

> Making a random change in our key/value store with 16 million entries only requires changing a little less than 7 merkle tree nodes on average.

It's properly obvious why high resilience is relevant generally, but it is even more important in distributed settings where shifts would lead to redundant network roundtrips and wasted storage.

### Size Variance

I'm deliberately vague about what size means it could be [branching factor](https://en.wikipedia.org/wiki/Branching_factor) which corresponds to the size of sibling groups or it could be byte size of the serialized node and often there is high correspondence between the two. High variance size makes performance characteristics unpredictable as some nodes may end up huge and others tiny. 

All the mentioned search tree algorithms other than [B-Tree][] compromise this property in order to gain first two. In theory because boundaries are detected from cryptographic hashes by basing boundaries on probability it is expected to get a balanced distribution, in practice there are no guarantees and results can vary.

> In Dialog while nodes tend to be **on average** of desired density (child count), variance is very high and median and average are far apart.

### Calibrating Compromises

Authors of Dolt came up with an interesting solution of increasing probability with the increase of size. I'm going to describe idea conceptually which is not what Dolt does but I find it simpler to explain and understand - We decide if node is  **boundary** of the sibling group by counting leading `0`s in the node hash in binary encoding. For desired branching factor we need `n`, but as we get closer to desired branching factor we decrease `n` to increase probability of finding a **boundary**. 

While this is guaranteed to provide a narrower [size variance][] it must either weaken [history independence][] or resilience to [boundary shifts][].

> I do not know which one dolt compromises here, but it must be one or the other, because insertion may split a sibling group at which point split out segment either joins the next group causing potentially cascading [boundary shifts][] or stay independent compromising [history independence][].

It is worth observing that probability distribution adds some resilience to boundary shifts although I don't believe it can't be guaranteed. Same could be said about weakened [history independence][] it gets slightly compromised but over time it in normal circumstances it should chart towards convergence as opposed to divergence although it seems possible to simulate contrary scenario.



#### Weakening History Independence

It is important to remember why we care about [history independence][] - It allows us to quickly identify differences, which in turn we need to be able to reconcile changes across replicas. If we are able to make a compromise in way that does not weaken our ability to identify difference we'll have a bargain deal as we'll be able to strengthen some of the other two properties.

Lets consider [Datomic][] inspired design where our search tree in addition to (index) tree also has novelty buffer, all inserts will accumulate in the buffer until it's reaches capacity and when it does all entries from novelty gets flushed into the (index) tree

```ts
interface Tree<Entry> {
  novelty: Set<Entry>
  index: IndexNode<Entry>
}

interface IndexNode<Entry> {
  children: Node<Entry>[]
}

interface SegmentNode<Entry> {
  children: Entry[]
}
```

In such a design we weaken history independence, because if have an entry the novelty another replica would have had different one if insertion occurred there in reverse order. However this design does not weaken our ability to identify differences in practice what we could do instead of comparing index 

🚧 UNFINISHED FROM HERE ON 🚧

### Adjusting boundary detection

It appears that if boundary nodes are determined purely by node content you get high resilience to boundary shifts because single edit can at most affect two adjacent nodes and path leading to them. However it is also possible to end up with a very densely populated nodes. if boundary detection takes into account additional context like sibling group density it becomes possible to make compromise and induce boundary at which point we have to weaken one of the other two guarantees

#### Compromising History Independence

If we compromise [history independence][] we may end can end up with trees in which differences can't be easily detected however it's not  



but also additional context like sibling group sizes or blob size, this property weakens

## Considerations

### Network vs Disk IO

Traditional databases optimize for disk I/O, Dialog DB optimizes for network IO where latency dominates is more relevant. Wasting a round trip on a tiny node hurts far more in this context.

### 

However it does not seem possible to impose low bound  small 



---

In Okra, inserting a node never shifts boundaries - it may split sibling group if inserted node is determined to be a boundary node, but that has no effect on other boundary nodes because it is determined by node content alone. 

> Deleting node can shift boundaries by joining sibling groups, but that is less relevant in Dialog context so we won't worry about it.

However, if boundary nodes are determined not purely by node content but also additional context like sibling group sizes or blob size, this property weakens. 

Consider what happens when capacity constraints influence boundary. A node may be determined as boundary only because sibling group got too large. Later inserting node in the same position can flip boundary status of the previously inserted node

>  Previous pressure of large sibling group no longer applies so in new context node is no longer a boundary node.

In this scenario boundary shifts from previously inserted node to a new node. This reduces stability of the tree shape and in pathological cases could lead to cascading effects where boundaries shift across several sibling groups because each group started with a boundary determined by previous group size and not a node content.

> That said, it is worth recognizing that cases where we'd experience cascading effects would also be pathological in implementation where context is not used because reason cascading effect happens when no boundary node exists in the affected sibling groups otherwise they'd be broken up





## The Tradeoff: Boundary Stability vs Size Uniformity

There's a tradeoff between boundary stability (resistance to boundary shifts) and size uniformity (consistent blob sizes). Pure content-based boundaries maximize stability but allow wide size variance. Size-based splitting creates uniform chunks but boundaries shift more easily.

My intuition is that this tradeoff matters differently for different node types:

**Index nodes** benefit from boundary stability - We want tighter bounds on child count while still having some boundaries on blob sizes to ensure low-entropy data doesn't go completely out of bounds.

**Segment nodes** pack actual content as their children and number of children can vary widely based on the content implying very low correspondence between child count and blob size. It seems logical that size here would be more important than boundary shifts, although we can't fully ignore boundary shifts or will be prone to cascading shifts.

## A Constraint-Based Approach

Rather than choosing a fixed strategy, we're exploring a constraint-based API that lets different node types express their requirements:

```typescript
type NodeConfiguration
  | LimitDensity
  | LimitCapacity
  | Limit

type Constraint = { min: UInt, max: UInt, bias?: Uint }

type DensityLimit = {
  count: Constraint
  capacity: {min: UInt }
}

type CapacityLimit = {
  count: {min: UInt}
  capacity: Constraint
}

type Limit = {
  count: Constraint
  capacity: Constraint
}

```

This design acknowledges that different parts of the tree have different needs. Index nodes can prioritize boundary stability while keeping child count reasonable. Segment nodes can focus on blob size while maintaining minimum fanout for efficiency.

## Striking a Balance

Dialog DB aims to balance these competing concerns so that starting queries from an empty database can be efficient and syncing between replicas minimizes unnecessary blob transfers.

By making the tradeoff explicit and tunable, we can experiment with different strategies for different workloads. Low-entropy datasets might need tighter size bounds to prevent degenerate cases. High-churn workloads might prefer more stability to reduce sync traffic.

The constraint-based approach also future-proofs the design. As we learn more about real-world usage patterns, we can refine the boundary detection strategies without changing the fundamental API.

## Next Steps

We haven't yet implemented or proven this approach will solve our distribution problems. But by learning from Okra, G-trees, Dolt, and others who've explored this space, we're developing a framework that acknowledges the tradeoffs rather than seeking a one-size-fits-all solution.

The key insight is recognizing that boundary stability and size uniformity exist in tension. Different use cases need different points on that spectrum. By making that choice explicit and tunable, we hope to build a system that works well across diverse workloads while maintaining the properties that make probabilistic B-trees valuable for local-first applications.





[history independent]:#History_Independence
[history independence]:#History_Independence
[boundary shifts]:#Understanding_Boundary_Shifts
[B-Tree]:https://en.wikipedia.org/wiki/B-tree
[local-first]: https://www.inkandswitch.com/essay/local-first/
[Prolly Tree]: #Probabilistic_B-Tree_a_k_a_Prolly_Tree
[Skip List]: https://en.wikipedia.org/wiki/Skip_list
[Okra]:#Okra
[size variance]:#size_variance