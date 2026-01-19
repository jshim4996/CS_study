# Chapter 06. Cache Memory (캐시 메모리)

> Deep dive into cache architecture to understand how data movement affects program performance.

---

## Cache Concept (캐시 개념)

### Concept (개념)

**Cache Memory (캐시 메모리)** is a small, fast memory that stores copies of frequently accessed data from main memory.

**Purpose:**
- Bridge the speed gap between CPU and RAM
- Exploit locality of reference
- Make average memory access time closer to cache speed

**Cache Characteristics:**
- **Transparent**: Programs don't explicitly manage cache (hardware handles it)
- **Automatic**: CPU automatically checks cache before main memory
- **Inclusive/Exclusive**: Various policies for multi-level caches

**Basic Operation:**
1. CPU requests data at memory address X
2. Check if X is in cache (cache lookup)
3. If found (hit): Return data from cache (fast)
4. If not found (miss): Fetch from main memory (slow), store in cache

### Diagram / Example

```
Cache Operation Flow:
┌─────────────────────────────────────────────────────────────┐
│                                                              │
│   CPU Request: Read address 0x1000                          │
│   ┌───────────┐                                             │
│   │    CPU    │                                             │
│   │           │ ──────────────────────┐                     │
│   └───────────┘                       ↓                     │
│                                 ┌───────────┐               │
│                                 │   Cache   │               │
│                                 │  Lookup   │               │
│                                 └─────┬─────┘               │
│                                       │                     │
│                          ┌────────────┴────────────┐        │
│                          ↓                         ↓        │
│                    ┌──────────┐              ┌──────────┐   │
│                    │ Cache Hit│              │Cache Miss│   │
│                    │  (적중)  │              │  (미스)   │   │
│                    └────┬─────┘              └────┬─────┘   │
│                         │                         │         │
│                         ↓                         ↓         │
│                  ┌────────────┐           ┌────────────┐    │
│                  │Return data │           │ Fetch from │    │
│                  │ (~1-4 ns)  │           │ RAM (~100ns)│   │
│                  └────────────┘           │ Store in   │    │
│                                           │ cache      │    │
│                                           └────────────┘    │
└─────────────────────────────────────────────────────────────┘

Modern CPU Cache Hierarchy:
┌─────────────────────────────────────────────────────────────┐
│                         CPU Die                              │
│   ┌─────────┐   ┌─────────┐   ┌─────────┐   ┌─────────┐    │
│   │ Core 0  │   │ Core 1  │   │ Core 2  │   │ Core 3  │    │
│   │┌───┬───┐│   │┌───┬───┐│   │┌───┬───┐│   │┌───┬───┐│    │
│   ││L1I│L1D││   ││L1I│L1D││   ││L1I│L1D││   ││L1I│L1D││    │
│   │└───┴───┘│   │└───┴───┘│   │└───┴───┘│   │└───┴───┘│    │
│   │ ┌─────┐ │   │ ┌─────┐ │   │ ┌─────┐ │   │ ┌─────┐ │    │
│   │ │ L2  │ │   │ │ L2  │ │   │ │ L2  │ │   │ │ L2  │ │    │
│   │ └─────┘ │   │ └─────┘ │   │ └─────┘ │   │ └─────┘ │    │
│   └────┬────┘   └────┬────┘   └────┬────┘   └────┬────┘    │
│        └─────────────┴─────────────┴─────────────┘          │
│                           │                                  │
│                    ┌──────┴──────┐                          │
│                    │  Shared L3  │                          │
│                    │  (8-32 MB)  │                          │
│                    └─────────────┘                          │
└─────────────────────────────────────────────────────────────┘
```

---

## Cache Hit vs Miss (캐시 적중 vs 미스)

### Concept (개념)

**Cache Hit (캐시 적중):**
- Requested data is found in cache
- Fast access (L1: ~1ns, L2: ~4ns, L3: ~15ns)
- Goal: Maximize hit rate

**Cache Miss (캐시 미스):**
- Requested data not in cache
- Must fetch from lower level (slower)
- Types of misses:

| Miss Type | Korean | Cause | Solution |
|-----------|--------|-------|----------|
| Compulsory | 필수적 미스 | First access to data | Prefetching |
| Capacity | 용량 미스 | Cache too small | Larger cache, better algorithm |
| Conflict | 충돌 미스 | Multiple addresses map to same location | Higher associativity |

**Hit Rate and Miss Rate:**
```
Hit Rate = Hits / (Hits + Misses)
Miss Rate = 1 - Hit Rate = Misses / (Hits + Misses)

Typical values:
L1: 95-99% hit rate
L2: 90-97% hit rate
L3: 80-95% hit rate
```

**Average Memory Access Time (AMAT):**
```
AMAT = Hit Time + (Miss Rate × Miss Penalty)

Example:
L1 Hit Time = 1 ns
L1 Miss Rate = 5%
L2 Hit Time = 4 ns
L2 Miss Rate = 10%
Memory Access = 100 ns

AMAT = 1 + 0.05 × (4 + 0.10 × 100)
     = 1 + 0.05 × 14
     = 1.7 ns
```

### Diagram / Example

```
Cache Miss Impact:
┌────────────────────────────────────────────────────────────┐
│ Scenario: Process 1 million integers                       │
│                                                            │
│ With 99% hit rate (sequential access):                     │
│   Hits: 990,000 × 1 ns = 990,000 ns                       │
│   Misses: 10,000 × 100 ns = 1,000,000 ns                  │
│   Total: 1,990,000 ns ≈ 2 ms                              │
│                                                            │
│ With 50% hit rate (random access):                         │
│   Hits: 500,000 × 1 ns = 500,000 ns                       │
│   Misses: 500,000 × 100 ns = 50,000,000 ns                │
│   Total: 50,500,000 ns ≈ 50 ms                            │
│                                                            │
│ Random access is 25x slower!                               │
└────────────────────────────────────────────────────────────┘

Miss Type Examples:
┌────────────────────────────────────────────────────────────┐
│ Compulsory Miss (필수적 미스):                              │
│   int x = arr[0];  // First access - must load from RAM   │
│                                                            │
│ Capacity Miss (용량 미스):                                  │
│   // Array too large for cache                             │
│   for (int i = 0; i < 10000000; i++)                      │
│       sum += huge_array[i];  // Can't fit all in cache    │
│                                                            │
│ Conflict Miss (충돌 미스):                                  │
│   // Two arrays map to same cache location                 │
│   for (int i = 0; i < N; i++)                             │
│       arr1[i] = arr2[i];  // May evict each other         │
└────────────────────────────────────────────────────────────┘
```

---

## Cache Line (캐시 라인)

### Concept (개념)

**Cache Line (캐시 라인)** is the unit of data transfer between cache and memory.

**Key Points:**
- Typical size: 64 bytes (modern Intel/AMD CPUs)
- When cache miss occurs, entire line is loaded
- Exploits spatial locality

**Cache Line Structure:**
```
┌────────────────────────────────────────────────────┐
│                    Memory Address                   │
│  ┌─────────────┬─────────────┬─────────────────┐  │
│  │    Tag      │   Index     │   Block Offset  │  │
│  │  (which     │  (which     │  (which byte    │  │
│  │   block)    │   set)      │   in line)      │  │
│  └─────────────┴─────────────┴─────────────────┘  │
└────────────────────────────────────────────────────┘
```

**Performance Implications:**
- Access one byte → entire 64-byte line loaded
- Sequential access: Next 63 bytes are "free"
- Stride access: May waste loaded data

### Diagram / Example

```
Cache Line Loading:
┌────────────────────────────────────────────────────────────┐
│ int arr[16];  // 16 × 4 bytes = 64 bytes = 1 cache line   │
│                                                            │
│ Access arr[0]:                                             │
│ ┌───────────────────────────────────────────────────────┐ │
│ │ Memory: arr[0] arr[1] arr[2] ... arr[14] arr[15]      │ │
│ │         ↓                                              │ │
│ │         └─────────────────┬───────────────────────────┤ │
│ │                           ↓                            │ │
│ │ Cache Line: [arr[0-15] loaded]  ← All 64 bytes!       │ │
│ └───────────────────────────────────────────────────────┘ │
│                                                            │
│ Subsequent accesses to arr[1-15]: Cache hits! (FREE)      │
└────────────────────────────────────────────────────────────┘

Stride Access Problem:
┌────────────────────────────────────────────────────────────┐
│ int matrix[1000][1000];                                    │
│                                                            │
│ Row-major access (stride 1):                               │
│ for (i) for (j) sum += matrix[i][j];                      │
│ Access pattern: [0][0], [0][1], [0][2], ...               │
│ → Excellent! Uses all bytes in each cache line            │
│                                                            │
│ Column-major access (stride 1000):                         │
│ for (j) for (i) sum += matrix[i][j];                      │
│ Access pattern: [0][0], [1][0], [2][0], ...               │
│ → Terrible! Loads 64 bytes, uses only 4 bytes             │
│                                                            │
│ Cache efficiency:                                          │
│ Row-major: 64/64 = 100%                                   │
│ Column-major: 4/64 = 6.25%                                │
└────────────────────────────────────────────────────────────┘

False Sharing Problem (거짓 공유):
┌────────────────────────────────────────────────────────────┐
│ // Two threads accessing different variables              │
│ // but same cache line                                     │
│                                                            │
│ struct {                                                   │
│     int counter1;  // Thread 1 uses this                  │
│     int counter2;  // Thread 2 uses this                  │
│ } shared;  // Both fit in same 64-byte cache line!       │
│                                                            │
│ Thread 1 writes counter1 → Invalidates entire line        │
│ Thread 2 writes counter2 → Invalidates entire line        │
│                                                            │
│ Result: Constant cache line bouncing between cores        │
│ Solution: Pad to separate cache lines                      │
│                                                            │
│ struct {                                                   │
│     alignas(64) int counter1;  // Own cache line          │
│     alignas(64) int counter2;  // Own cache line          │
│ } shared;                                                  │
└────────────────────────────────────────────────────────────┘
```

---

## Cache Mapping (캐시 매핑)

### Concept (개념)

**Cache Mapping** determines where a memory block can be stored in cache.

**1. Direct Mapped (직접 매핑)**
- Each memory block maps to exactly one cache location
- Simple, fast, but conflict misses
- Index = (Block Address) mod (Number of Cache Lines)

**2. Fully Associative (완전 연관 매핑)**
- Any memory block can go in any cache location
- No conflict misses
- Expensive: Must search all entries

**3. Set Associative (집합 연관 매핑)**
- Compromise: Block can go in any location within a set
- N-way associative = N possible locations
- Most common: 4-way, 8-way associative

### Diagram / Example

```
Direct Mapped Cache (직접 매핑):
┌────────────────────────────────────────────────────────────┐
│ Memory blocks: 0, 1, 2, 3, 4, 5, 6, 7, ...                │
│ Cache lines: 4                                             │
│ Mapping: Block i → Cache line (i mod 4)                   │
│                                                            │
│ Memory:   0  1  2  3  4  5  6  7  8  9  10 11 ...        │
│           │  │  │  │  │  │  │  │  │  │  │  │             │
│           ↓  ↓  ↓  ↓  ↓  ↓  ↓  ↓  ↓  ↓  ↓  ↓             │
│ Cache: ┌────┬────┬────┬────┐                              │
│        │ 0  │ 1  │ 2  │ 3  │  Line index                  │
│        │0,4,│1,5,│2,6,│3,7,│  Blocks that map here        │
│        │8...│9...│10..│11..│                              │
│        └────┴────┴────┴────┘                              │
│                                                            │
│ Problem: Blocks 0, 4, 8 all map to line 0 → conflicts!   │
└────────────────────────────────────────────────────────────┘

2-Way Set Associative (2-웨이 집합 연관):
┌────────────────────────────────────────────────────────────┐
│ Memory blocks: 0, 1, 2, 3, 4, 5, 6, 7, ...                │
│ Cache: 4 lines organized as 2 sets × 2 ways               │
│                                                            │
│ Set 0:  ┌────────┬────────┐                               │
│         │ Way 0  │ Way 1  │  Can hold 2 blocks            │
│         └────────┴────────┘                               │
│ Set 1:  ┌────────┬────────┐                               │
│         │ Way 0  │ Way 1  │  Can hold 2 blocks            │
│         └────────┴────────┘                               │
│                                                            │
│ Mapping: Block i → Set (i mod 2), any way                 │
│ Block 0, 2, 4, 6 → Set 0 (pick any of 2 ways)            │
│ Block 1, 3, 5, 7 → Set 1 (pick any of 2 ways)            │
│                                                            │
│ Now blocks 0 and 4 can coexist in Set 0!                  │
└────────────────────────────────────────────────────────────┘

Fully Associative (완전 연관):
┌────────────────────────────────────────────────────────────┐
│ Any block can go anywhere:                                 │
│                                                            │
│ Cache: ┌────┬────┬────┬────┐                              │
│        │ ?  │ ?  │ ?  │ ?  │  Any block can go here      │
│        └────┴────┴────┴────┘                              │
│                                                            │
│ + No conflict misses                                       │
│ - Must compare tag with ALL entries (expensive)           │
│ - Used only for small caches (TLB, etc.)                  │
└────────────────────────────────────────────────────────────┘

Trade-offs:
┌─────────────────┬────────────┬────────────┬──────────────┐
│ Type            │ Hit Time   │ Miss Rate  │ Hardware     │
├─────────────────┼────────────┼────────────┼──────────────┤
│ Direct Mapped   │ Fastest    │ Highest    │ Simplest     │
│ 2-Way           │ Fast       │ Medium     │ Moderate     │
│ 4-Way           │ Medium     │ Low        │ Moderate     │
│ 8-Way           │ Slower     │ Lower      │ Complex      │
│ Fully Assoc.    │ Slowest    │ Lowest     │ Most Complex │
└─────────────────┴────────────┴────────────┴──────────────┘
```

---

## Replacement Policies (교체 정책)

### Concept (개념)

When cache is full, which line to evict? **Replacement Policy (교체 정책)** decides.

**Common Policies:**

| Policy | Korean | Description | Pros/Cons |
|--------|--------|-------------|-----------|
| LRU | 최근 최소 사용 | Evict least recently used | Good hit rate, expensive to track |
| FIFO | 선입선출 | Evict oldest entry | Simple, can evict hot data |
| Random | 무작위 | Evict random entry | Simple, unpredictable |
| LFU | 최소 빈도 사용 | Evict least frequently used | Good for skewed access, complex |

**LRU (Least Recently Used):**
- Most common policy
- Based on temporal locality assumption
- Hardware implementations often approximate (pseudo-LRU)

### Diagram / Example

```
LRU Example (4-way set associative):
┌────────────────────────────────────────────────────────────┐
│ Time  │ Access │ Cache State        │ LRU Order           │
├───────┼────────┼────────────────────┼─────────────────────┤
│ t1    │ A      │ [A, _, _, _]       │ A                   │
│ t2    │ B      │ [A, B, _, _]       │ A, B                │
│ t3    │ C      │ [A, B, C, _]       │ A, B, C             │
│ t4    │ D      │ [A, B, C, D]       │ A, B, C, D          │
│ t5    │ B      │ [A, B, C, D]       │ A, C, D, B  (B→MRU) │
│ t6    │ E      │ [E, B, C, D]       │ C, D, B, E (evict A)│
│ t7    │ A      │ [E, B, A, D]       │ D, B, E, A (evict C)│
└───────┴────────┴────────────────────┴─────────────────────┘

Pseudo-LRU Tree (cheaper approximation):
┌────────────────────────────────────────────────────────────┐
│ 4-way cache with tree-based pseudo-LRU:                    │
│                                                            │
│                    [0]  ← Points to less-recently-used half│
│                   /   \                                    │
│                [0]     [1]  ← Second-level pointers        │
│               /  \    /  \                                 │
│            Way0  Way1  Way2  Way3                          │
│                                                            │
│ To find LRU: Follow 0s down the tree                      │
│ After access: Update path bits to point away               │
│                                                            │
│ Approximates LRU with only log2(N) bits per set           │
└────────────────────────────────────────────────────────────┘
```

---

## Write Policies (쓰기 정책)

### Concept (개념)

When CPU writes data, how should cache and memory be updated?

**Write Hit Policies (쓰기 적중):**

| Policy | Korean | Description |
|--------|--------|-------------|
| Write-Through | 즉시 쓰기 | Write to both cache and memory immediately |
| Write-Back | 나중 쓰기 | Write only to cache; update memory when evicted |

**Write Miss Policies (쓰기 미스):**

| Policy | Korean | Description |
|--------|--------|-------------|
| Write-Allocate | 쓰기 할당 | Load block into cache, then write |
| No-Write-Allocate | 쓰기 비할당 | Write directly to memory, don't cache |

**Common Combinations:**
- Write-Back + Write-Allocate (most common for performance)
- Write-Through + No-Write-Allocate (simpler, for coherency)

### Diagram / Example

```
Write-Through (즉시 쓰기):
┌────────────────────────────────────────────────────────────┐
│ CPU writes X=5:                                            │
│                                                            │
│   CPU ──┬──→ Cache ──→ Write 5 immediately                │
│         │                                                  │
│         └──→ Memory ──→ Write 5 immediately               │
│                                                            │
│ + Memory always up-to-date                                │
│ + Simple cache coherence                                   │
│ - Slow (every write goes to memory)                        │
│ - Write buffer needed to hide latency                      │
└────────────────────────────────────────────────────────────┘

Write-Back (나중 쓰기):
┌────────────────────────────────────────────────────────────┐
│ CPU writes X=5:                                            │
│                                                            │
│   CPU ──→ Cache ──→ Write 5, mark "dirty"                 │
│              │                                             │
│              └── Memory NOT updated yet                    │
│                                                            │
│ Later, when cache line evicted:                            │
│   Cache ──→ Memory ──→ Write dirty data                   │
│                                                            │
│ + Fast (most writes stay in cache)                         │
│ + Multiple writes to same location → 1 memory write       │
│ - Complex (need dirty bits, write-back on eviction)        │
│ - Memory may be stale                                      │
└────────────────────────────────────────────────────────────┘

Write-Allocate vs No-Write-Allocate:
┌────────────────────────────────────────────────────────────┐
│ Write to address not in cache:                             │
│                                                            │
│ Write-Allocate (쓰기 할당):                                 │
│   1. Load block from memory into cache                     │
│   2. Write to cache                                        │
│   → Good if will access this data again (spatial locality) │
│                                                            │
│ No-Write-Allocate (쓰기 비할당):                            │
│   1. Write directly to memory                              │
│   2. Don't load into cache                                 │
│   → Good for streaming writes (video encoding, etc.)       │
└────────────────────────────────────────────────────────────┘

Performance Impact:
┌────────────────────────────────────────────────────────────┐
│ Example: Writing 1 million integers sequentially           │
│                                                            │
│ Write-Through:                                              │
│   1,000,000 × 100 ns (memory write) = 100 ms              │
│                                                            │
│ Write-Back:                                                 │
│   ~16,000 cache line evictions × 100 ns = 1.6 ms          │
│   (Most writes absorbed by cache)                          │
│                                                            │
│ Write-Back is ~60x faster for this pattern!               │
└────────────────────────────────────────────────────────────┘
```

---

## Summary (요약)

| Concept | Korean | Description |
|---------|--------|-------------|
| Cache Hit | 캐시 적중 | Data found in cache |
| Cache Miss | 캐시 미스 | Data not in cache |
| Cache Line | 캐시 라인 | Unit of transfer (64 bytes) |
| Direct Mapped | 직접 매핑 | One possible location |
| Set Associative | 집합 연관 | N possible locations per set |
| LRU | 최근 최소 사용 | Evict least recently used |
| Write-Back | 나중 쓰기 | Write to cache only |
| Write-Through | 즉시 쓰기 | Write to cache and memory |

**Key Takeaways for Performance:**
1. **Cache misses are expensive** - 100x slower than hits
2. **Cache line size matters** - access data in 64-byte chunks
3. **Avoid false sharing** - pad data structures in multithreaded code
4. **Higher associativity reduces conflicts** - but increases hit time
5. **Write-back is faster** - for write-heavy workloads
6. **Understand your cache sizes** - keep working set in L1/L2
