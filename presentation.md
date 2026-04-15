# Heaps and Priority Queues

**Computer Science Fundamentals Series**

Binary heaps · Heap sort · Fibonacci heaps · Build-heap · Decrease-key · Priority queue ADT

*Mid-level software engineer track -- 20 slides*

---

## Table of Contents

1. [Priority Queue ADT](#slide-02--priority-queue-adt)
2. [Binary Heap -- Structure Property](#slide-03--binary-heap--structure-property)
3. [Binary Heap -- Heap Property](#slide-04--binary-heap--heap-property)
4. [Array Representation](#slide-05--array-representation)
5. [Sift-Up (Insert)](#slide-06--sift-up-insert)
6. [Sift-Down (Extract)](#slide-07--sift-down-extract)
7. [Build-Heap in O(n)](#slide-08--build-heap-in-on)
8. [Build-Heap -- Proof of O(n)](#slide-09--build-heap--proof-of-on)
9. [Heap Sort](#slide-10--heap-sort)
10. [Min-Heap vs Max-Heap](#slide-11--min-heap-vs-max-heap)
11. [D-ary Heaps](#slide-12--d-ary-heaps)
12. [Binomial Heaps](#slide-13--binomial-heaps)
13. [Fibonacci Heaps -- Structure](#slide-14--fibonacci-heaps--structure)
14. [Fibonacci Heaps -- Amortised Analysis](#slide-15--fibonacci-heaps--amortised-analysis)
15. [Pairing Heaps](#slide-16--pairing-heaps)
16. [Indexed Priority Queues](#slide-17--indexed-priority-queues)
17. [Application -- Dijkstra's Algorithm](#slide-18--application--dijkstras-algorithm)
18. [Applications -- Huffman, Median, Scheduling](#slide-19--applications--huffman-median-scheduling)
19. [Summary & Further Reading](#slide-20--summary--further-reading)

---

## Slide 02 -- Priority Queue ADT

### Abstract interface

A priority queue is a container where each element has an associated priority. It supports:

- **insert(key, priority)** -- add an element with the given priority
- **extract-min / extract-max** -- remove and return the element with the highest priority
- **peek-min / peek-max** -- return the highest-priority element without removing it
- **decrease-key(handle, new_priority)** -- lower the priority of an existing element (critical for graph algorithms)
- **merge(pq1, pq2)** -- combine two priority queues into one

### Why not just sort?

Sorting gives you all elements in order, but costs `O(n log n)` up front and requires all elements to be known. A priority queue supports dynamic insertion and extraction -- elements arrive over time.

### Implementations compared

| Implementation | insert | extract-min | decrease-key | merge |
|---------------|--------|-------------|-------------|-------|
| Unsorted array | O(1) | O(n) | O(1) | O(1) |
| Sorted array | O(n) | O(1) | O(n) | O(n) |
| Binary heap | O(log n) | O(log n) | O(log n) | O(n) |
| Fibonacci heap | O(1)* | O(log n)* | O(1)* | O(1)* |

*amortised

---

## Slide 03 -- Binary Heap -- Structure Property

### Complete binary tree

A binary heap is a **complete binary tree** -- every level is fully filled except possibly the last, which is filled left to right.

```
Complete binary tree (valid heap shape):

         ○
       /   \
      ○     ○
     / \   / \
    ○   ○ ○   ○
   / \
  ○   ○

Height = ⌊log₂ n⌋   (always balanced)
```

### Why completeness matters

- Height is always `O(log n)` -- guarantees worst-case operation bounds
- No wasted space in the array representation -- no pointers needed
- Cache-friendly contiguous memory layout
- Predictable shape makes implementation simple

> A binary heap is **not** a binary search tree. There is no ordering between left and right children -- only between parent and child.

---

## Slide 04 -- Binary Heap -- Heap Property

### Min-heap property

Every node's key is **less than or equal to** both of its children's keys.

```
Min-heap:

         2
       /   \
      5     3
     / \   / \
    8   7 9   4
   / \
  10  12

Parent ≤ Children at every node
Root = minimum element
```

### Max-heap property

Every node's key is **greater than or equal to** both of its children's keys.

```
Max-heap:

         50
       /    \
      30     40
     / \    / \
    10  20 35  25

Parent ≥ Children at every node
Root = maximum element
```

> The heap property is weaker than the BST property. Siblings have no ordering relative to each other. This weaker invariant is what makes heap operations faster for priority queue use cases.

---

## Slide 05 -- Array Representation

### Implicit tree in an array

A complete binary tree maps perfectly to a flat array with zero overhead:

```
Index:   0   1   2   3   4   5   6   7   8
Value: [ 2 | 5 | 3 | 8 | 7 | 9 | 4 | 10| 12]

Tree structure (same data):

            2 [0]
          /       \
       5 [1]     3 [2]
       /   \     /   \
    8 [3] 7 [4] 9 [5] 4 [6]
    /  \
 10[7] 12[8]
```

### Navigation formulas (0-indexed)

| Relationship | Formula |
|-------------|---------|
| Parent of node i | `(i - 1) / 2` (integer division) |
| Left child of node i | `2i + 1` |
| Right child of node i | `2i + 2` |
| Is node i a leaf? | `i >= n / 2` |
| Last internal node | `n/2 - 1` |

> No pointers, no node allocations, no fragmented memory. The array representation is why binary heaps dominate in practice despite theoretically inferior bounds to Fibonacci heaps.

---

## Slide 06 -- Sift-Up (Insert)

### Algorithm

1. Append the new element at the **end** of the array (next available leaf position)
2. Compare with parent -- if smaller (min-heap), swap
3. Repeat up the tree until heap property is restored or root is reached

```
Insert 1 into min-heap:

Step 0: Append at end          Step 1: Swap with 8
         2                              2
       /   \                          /   \
      5     3                        5     3
     / \   /                        / \   /
    8   7 9                        1   7 9
   /                              /
  1                              8

Step 2: Swap with 5            Step 3: Swap with 2
         2                              1
       /   \                          /   \
      1     3                        2     3
     / \   /                        / \   /
    8   7 9                        8   7 9
   /                              /
  5                              5
```

**Time complexity:** O(log n) -- at most one swap per level.

---

## Slide 07 -- Sift-Down (Extract)

### Algorithm (extract-min)

1. Return the root element (the minimum)
2. Move the **last** element to the root position
3. Compare with children -- swap with the **smaller** child
4. Repeat down the tree until heap property is restored or a leaf is reached

```
Extract-min from heap [2, 5, 3, 8, 7, 9, 4]:

Step 0: Remove root, move last    Step 1: Swap with smaller child (3)
         4                                  3
       /   \                              /   \
      5     3                            5     4
     / \   /                            / \   /
    8   7 9                            8   7 9

Step 2: 4 > 9? No. Done.
         3
       /   \
      5     4
     / \   /
    8   7 9
```

**Time complexity:** O(log n) -- at most one swap per level.

> Sift-down is also the core subroutine of build-heap and heap sort.

---

## Slide 08 -- Build-Heap in O(n)

### The naive approach

Insert elements one by one: `n` insertions x `O(log n)` each = `O(n log n)`.

### Floyd's algorithm (bottom-up)

Start from the last internal node and sift-down each node, working backwards to the root.

```python
def build_heap(arr):
    n = len(arr)
    # Start from last internal node, work backward to root
    for i in range(n // 2 - 1, -1, -1):
        sift_down(arr, i, n)
```

### Why this is O(n), not O(n log n)

Most nodes are near the bottom. Nodes at the bottom do almost no work:

| Level (from bottom) | Nodes at level | Sift-down cost | Total work |
|---------------------|---------------|----------------|------------|
| 0 (leaves) | n/2 | 0 | 0 |
| 1 | n/4 | 1 | n/4 |
| 2 | n/8 | 2 | n/4 |
| k | n/2^(k+1) | k | kn/2^(k+1) |

---

## Slide 09 -- Build-Heap -- Proof of O(n)

### Summing the work

Total work = sum over all levels:

```
T(n) = Σ (k=0 to h) ⌊n / 2^(k+1)⌋ · k

     = n · Σ (k=0 to ∞) k / 2^(k+1)

     = n · Σ (k=0 to ∞) k · x^k   where x = 1/2
```

### The key identity

Using the power series identity `Σ k·x^k = x / (1-x)^2` for |x| < 1:

```
Σ (k=0 to ∞) k / 2^k = (1/2) / (1 - 1/2)^2
                       = (1/2) / (1/4)
                       = 2
```

Therefore: `T(n) = n · 2 / 2 = n`

### Build-heap is O(n)

This is a tight bound -- you cannot build a heap faster than O(n) because you must at least read every element.

> This result is frequently tested in interviews and exams. The intuition: half the nodes are leaves (zero work), a quarter do one swap, an eighth do two swaps -- the geometric series converges.

---

## Slide 10 -- Heap Sort

### Algorithm

1. **Build a max-heap** from the input array -- O(n)
2. Repeatedly **extract the maximum** and place it at the end:
   - Swap root (max) with last unsorted element
   - Shrink the heap by one
   - Sift-down the new root

```python
def heap_sort(arr):
    n = len(arr)
    # Phase 1: build max-heap -- O(n)
    for i in range(n // 2 - 1, -1, -1):
        sift_down(arr, i, n)

    # Phase 2: extract max repeatedly -- O(n log n)
    for end in range(n - 1, 0, -1):
        arr[0], arr[end] = arr[end], arr[0]
        sift_down(arr, 0, end)
```

### Properties

| Property | Value |
|----------|-------|
| Time (worst, average, best) | O(n log n) |
| Space | O(1) -- in-place |
| Stable? | No -- relative order of equal elements not preserved |
| Adaptive? | No -- always O(n log n) regardless of input order |

> Heap sort's worst case beats quicksort's O(n^2) worst case. In practice, quicksort wins on average due to better cache behaviour and smaller constant factors. Heap sort shines when guaranteed O(n log n) and O(1) space are both required.

---

## Slide 11 -- Min-Heap vs Max-Heap

### Structural difference

The only difference is the direction of the comparison:

| Property | Min-heap | Max-heap |
|----------|----------|----------|
| Root contains | Smallest element | Largest element |
| Parent vs child | parent.key <= child.key | parent.key >= child.key |
| extract operation | Returns minimum | Returns maximum |
| Primary use | Priority queues (lowest = highest priority) | Heap sort, top-K largest |

### Converting between them

Negate all keys: a min-heap on `-key` behaves as a max-heap on `key`.

```python
# Max-heap using Python's heapq (which is min-heap only):
import heapq
max_heap = []
heapq.heappush(max_heap, -value)       # negate on insert
largest = -heapq.heappop(max_heap)     # negate on extract
```

### Language defaults

| Language | Default | Module / Class |
|----------|---------|---------------|
| Python | Min-heap | `heapq` |
| Java | Min-heap | `PriorityQueue` |
| C++ | Max-heap | `priority_queue` |
| Go | Min-heap (with interface) | `container/heap` |

---

## Slide 12 -- D-ary Heaps

### Generalising the branching factor

A **d-ary heap** is a complete d-ary tree satisfying the heap property. A binary heap is the special case d = 2.

```
4-ary min-heap (d=4):

              2
        /   |   \    \
       5    3    7    4
     / | \
    8  9  6
```

### Navigation (0-indexed)

| Relationship | Formula |
|-------------|---------|
| Parent of node i | `(i - 1) / d` |
| j-th child of i | `d * i + j + 1` (j = 0..d-1) |

### Trade-offs

| Operation | Binary (d=2) | D-ary |
|-----------|-------------|-------|
| insert (sift-up) | O(log_2 n) | O(log_d n) -- **faster** |
| extract-min (sift-down) | O(log_2 n) | O(d · log_d n) -- **slower** |
| decrease-key | O(log_2 n) | O(log_d n) -- **faster** |

> For Dijkstra's algorithm where decrease-key dominates, a 4-ary heap often outperforms a binary heap in practice. The shallower tree also improves cache performance.

---

## Slide 13 -- Binomial Heaps

### Binomial trees

A binomial tree `B_k` is defined recursively:

- `B_0` is a single node
- `B_k` is formed by linking two `B_(k-1)` trees -- one becomes the leftmost child of the other's root

```
B_0    B_1     B_2         B_3
 ○      ○       ○           ○
        |      / \        / | \
        ○     ○   ○      ○  ○  ○
              |         / \  |
              ○        ○  ○  ○
                       |
                       ○

Nodes: 1    2       4           8  (always 2^k)
```

### Binomial heap

A binomial heap is a collection of binomial trees satisfying:

- Each tree satisfies the min-heap property
- At most **one** tree of each order (like binary representation of n)
- Trees stored in a linked list sorted by order

| Operation | Time |
|-----------|------|
| insert | O(log n) amortised O(1) |
| extract-min | O(log n) |
| merge | O(log n) |
| decrease-key | O(log n) |

> The key advantage over binary heaps is **O(log n) merge**. Binary heaps require O(n) to merge.

---

## Slide 14 -- Fibonacci Heaps -- Structure

### Lazy mergeable heap

A Fibonacci heap is a collection of heap-ordered trees (not necessarily binomial) with:

- A **min pointer** to the root with smallest key
- Trees stored in a **circular doubly-linked list** of roots
- Each node tracks its **degree** (number of children) and a **mark** bit

```
Fibonacci heap (3 trees in root list):

  min
   ↓
   1 ←→ 5 ←→ 3
   |         |
   8         7 ←→ 4
  / \
 12  10
```

### Key operations

- **insert:** create a single-node tree, add to root list, update min -- O(1)
- **merge:** concatenate root lists, update min -- O(1)
- **extract-min:** remove min node, add its children to root list, then **consolidate** trees by degree -- O(log n) amortised
- **decrease-key:** cut the node from its parent, add to root list; cascade cuts if parent was already marked -- O(1) amortised

> The magic is in the lazy approach: defer cleanup to extract-min. This makes decrease-key O(1) amortised, which is critical for Dijkstra's algorithm.

---

## Slide 15 -- Fibonacci Heaps -- Amortised Analysis

### Potential function

Define the potential `Phi = t + 2m` where:

- `t` = number of trees in the root list
- `m` = number of marked nodes

### Amortised costs

| Operation | Actual cost | Potential change | Amortised cost |
|-----------|------------|-----------------|----------------|
| insert | O(1) | +1 (one new tree) | O(1) |
| merge | O(1) | 0 | O(1) |
| extract-min | O(t + log n) | -(t - log n) | O(log n) |
| decrease-key | O(c) cuts | -(c - 2) | O(1) |

### Why O(log n) for extract-min?

During consolidation, trees of equal degree are linked. After consolidation, at most one tree of each degree remains. The maximum degree is O(log n) because:

- A node of degree `k` has at least `F_(k+2)` descendants (Fibonacci numbers)
- `F_(k+2) >= phi^k` where `phi = (1 + sqrt(5)) / 2`
- Therefore `k <= log_phi(n)` = O(log n)

> Fibonacci heaps achieve the theoretically optimal amortised bounds for a comparison-based priority queue. The constant factors are large, so they rarely outperform binary heaps in practice except for very dense graphs.

---

## Slide 16 -- Pairing Heaps

### A simpler alternative to Fibonacci heaps

Pairing heaps achieve similar practical performance to Fibonacci heaps with a much simpler implementation.

### Structure

A pairing heap is a heap-ordered multi-way tree. Each node stores:

- A key
- A pointer to its leftmost child
- A pointer to its next sibling

### Operations

| Operation | Time |
|-----------|------|
| insert | O(1) |
| find-min | O(1) |
| merge | O(1) -- link roots, smaller becomes parent |
| extract-min | O(log n) amortised |
| decrease-key | O(log log n) amortised (conjectured O(1)) |

### Extract-min strategy (two-pass pairing)

1. Remove the root
2. **Left-to-right pass:** pair adjacent siblings, merge each pair
3. **Right-to-left pass:** merge the resulting trees into one

```
Children after removing root:  5  3  8  2  7  6

Pass 1 (pair):  [3,5]  [2,8]  [6,7]
Pass 2 (merge right-to-left):  [2, [3,5], [6,7], 8]
```

> In practice, pairing heaps often outperform Fibonacci heaps due to simpler structure and better cache behaviour. They are the go-to "advanced" heap when binary heaps are insufficient.

---

## Slide 17 -- Indexed Priority Queues

### The decrease-key problem

A standard binary heap has no way to locate an arbitrary element -- you can only access the root efficiently. Graph algorithms like Dijkstra's need to update priorities of elements already in the queue.

### Indexed priority queue

Augment a binary heap with a **position map** (index table) that tracks where each element lives in the heap array:

```
Heap array:    [A:2, C:5, B:3, D:8, E:7]
Position map:  { A→0, B→2, C→1, D→3, E→4 }
Inverse map:   { 0→A, 1→C, 2→B, 3→D, 4→E }
```

### Operations

| Operation | Time | How |
|-----------|------|-----|
| insert(id, priority) | O(log n) | Add to end, sift-up, update maps |
| extract-min() | O(log n) | Swap root with last, sift-down, update maps |
| decrease-key(id, p) | O(log n) | Look up position in O(1), sift-up |
| contains(id) | O(1) | Check position map |

### Swap with bookkeeping

Every swap in sift-up or sift-down must also update both the position map and inverse map -- constant overhead per swap.

> Indexed priority queues are the standard implementation for Dijkstra's and Prim's algorithms. Java's `PriorityQueue` does not support decrease-key -- you must implement your own or use a workaround (lazy deletion).

---

## Slide 18 -- Application -- Dijkstra's Algorithm

### Shortest paths with a priority queue

Dijkstra's algorithm finds shortest paths from a source vertex to all other vertices in a graph with non-negative edge weights.

```python
def dijkstra(graph, source):
    dist = {v: float('inf') for v in graph}
    dist[source] = 0
    pq = MinHeap()
    pq.insert(source, 0)

    while not pq.is_empty():
        u = pq.extract_min()
        for v, weight in graph[u]:
            new_dist = dist[u] + weight
            if new_dist < dist[v]:
                dist[v] = new_dist
                pq.decrease_key(v, new_dist)  # critical!
    return dist
```

### Complexity depends on the heap

| Heap type | Dijkstra's total time |
|-----------|-----------------------|
| Binary heap | O((V + E) log V) |
| D-ary heap (d = E/V) | O(E · log_(E/V) V) |
| Fibonacci heap | O(V log V + E) |
| Unsorted array (no heap) | O(V^2) |

> For dense graphs (E ~ V^2), the Fibonacci heap bound O(V^2 + V log V) ~ O(V^2) matches the unsorted-array approach. For sparse graphs, binary or d-ary heaps are preferable due to lower constant factors.

---

## Slide 19 -- Applications -- Huffman, Median, Scheduling

### Huffman coding

Build an optimal prefix-free code by repeatedly extracting the two lowest-frequency symbols and merging them:

- Uses a min-heap on symbol frequencies
- `O(n log n)` for `n` symbols
- Greedy algorithm -- the heap provides the greedy choice efficiently

### Running median maintenance

Maintain two heaps to track the median of a data stream:

- **Max-heap** for the lower half of values
- **Min-heap** for the upper half of values
- Balance sizes so they differ by at most 1
- Median is always at one of the two roots -- O(1) query, O(log n) insert

### Task scheduling

Priority queues are the backbone of:

- **OS process scheduling** -- highest-priority process runs next
- **Event-driven simulation** -- next event by timestamp
- **Job queues** -- priority-based task dispatch (Celery, Sidekiq)
- **A* search** -- open list sorted by f(n) = g(n) + h(n)
- **K-way merge** -- merge K sorted lists using a min-heap of size K

---

## Slide 20 -- Summary & Further Reading

### Key takeaways

- A priority queue is an ADT; a binary heap is its most common implementation
- Binary heaps use an implicit array representation -- no pointers, O(1) space overhead
- Build-heap is O(n) via Floyd's bottom-up algorithm, not O(n log n)
- Heap sort is O(n log n) worst-case, in-place, but not stable
- Fibonacci heaps offer O(1) amortised decrease-key -- theoretically optimal for Dijkstra's
- Pairing heaps are simpler than Fibonacci heaps and often faster in practice
- Indexed priority queues solve the decrease-key lookup problem for graph algorithms
- D-ary heaps (d=4) often beat binary heaps in practice due to cache effects

### Recommended reading

| Source | Description |
|--------|------------|
| **CLRS** | *Introduction to Algorithms* -- chapters 6 (heapsort), 19 (Fibonacci heaps), 20 (van Emde Boas) |
| **Sedgewick** | *Algorithms* -- clear binary heap and indexed PQ implementations |
| **Tarjan** | *Data Structures and Network Algorithms* -- original Fibonacci heap analysis |
| **Fredman & Tarjan** | "Fibonacci Heaps and Their Uses in Improved Network Optimization Algorithms" (1987) |
| **Fredman et al.** | "The Pairing Heap: A New Form of Self-Adjusting Heap" (1986) |
