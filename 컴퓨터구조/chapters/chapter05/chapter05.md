# Chapter 05. Memory Hierarchy Structure (메모리 계층 구조)

> Understanding memory hierarchy to bridge the speed gap between fast CPUs and slow memory.

---

## Memory Hierarchy Concept (메모리 계층 개념)

### Concept (개념)

**The Problem:**
- CPU speed doubles every ~2 years (Moore's Law)
- Memory speed improves only ~7% per year
- Result: **Memory Wall (메모리 벽)** - CPU waits for memory

**The Solution: Memory Hierarchy (메모리 계층)**
A series of storage levels with different speed, size, and cost characteristics:

```
Speed    ←─────────────────────────────→    Capacity
Fast/Expensive                           Slow/Cheap

Registers → L1 Cache → L2 Cache → L3 Cache → Main Memory → Storage
```

**Key Principle:**
Keep frequently used data in faster, smaller memories close to the CPU.

**Why It Works:**
Programs exhibit **locality of reference (참조 지역성)** - they tend to access the same data and nearby data repeatedly.

### Diagram / Example

```
Memory Hierarchy Pyramid:
                    ┌───────────┐
                    │ Registers │  ~0.3 ns, ~1 KB
                    │  (레지스터) │
                    └─────┬─────┘
                    ┌─────┴─────┐
                    │  L1 Cache │  ~1 ns, ~64 KB
                    │ (L1 캐시)  │
                    └─────┬─────┘
                ┌─────────┴─────────┐
                │     L2 Cache      │  ~4 ns, ~256 KB
                │    (L2 캐시)       │
                └─────────┬─────────┘
            ┌─────────────┴─────────────┐
            │        L3 Cache           │  ~15 ns, ~8-32 MB
            │       (L3 캐시)            │
            └─────────────┬─────────────┘
        ┌─────────────────┴─────────────────┐
        │         Main Memory (RAM)         │  ~100 ns, ~16-64 GB
        │           (주기억장치)               │
        └─────────────────┬─────────────────┘
    ┌─────────────────────┴─────────────────────┐
    │          SSD / Storage (저장장치)          │  ~100 μs, ~1 TB
    └───────────────────────────────────────────┘

Typical Access Times (Modern Desktop ~2024):
┌─────────────────┬─────────────────┬───────────────┐
│ Level           │ Access Time     │ CPU Cycles    │
├─────────────────┼─────────────────┼───────────────┤
│ Register        │ ~0.3 ns         │ 1             │
│ L1 Cache        │ ~1 ns           │ 3-4           │
│ L2 Cache        │ ~4 ns           │ 12-15         │
│ L3 Cache        │ ~15 ns          │ 40-50         │
│ Main Memory     │ ~100 ns         │ 200-300       │
│ SSD             │ ~100,000 ns     │ 300,000       │
│ HDD             │ ~10,000,000 ns  │ 30,000,000    │
└─────────────────┴─────────────────┴───────────────┘
```

---

## Speed vs Capacity vs Cost (속도 vs 용량 vs 비용)

### Concept (개념)

**Trade-off Triangle:**
You cannot have all three: fast AND large AND cheap.

| Property | Registers | L1 Cache | L3 Cache | RAM | SSD |
|----------|-----------|----------|----------|-----|-----|
| Speed | Fastest | Very Fast | Fast | Moderate | Slow |
| Capacity | ~1 KB | ~64 KB | ~32 MB | ~64 GB | ~1 TB |
| Cost/GB | Priceless | ~$1000 | ~$100 | ~$5 | ~$0.10 |

**Why Smaller is Faster:**
1. **Physical distance**: Electrons travel shorter distances
2. **Simpler addressing**: Fewer bits to decode
3. **Faster technology**: SRAM vs DRAM vs Flash

**Memory Technologies:**

| Type | Korean | Speed | Use Case |
|------|--------|-------|----------|
| SRAM | 정적 RAM | Very fast | Cache |
| DRAM | 동적 RAM | Moderate | Main memory |
| Flash/SSD | 플래시 | Slow | Storage |
| HDD | 하드디스크 | Very slow | Bulk storage |

### Diagram / Example

```
Speed vs Size Trade-off:
Access Time (log scale)
│
│  10 ms ─┼─────────────────────────────────────● HDD
│         │
│  100 μs ─┼──────────────────────────● SSD
│         │
│  100 ns ─┼────────────────● RAM
│         │
│   15 ns ─┼─────────● L3
│    4 ns ─┼────● L2
│    1 ns ─┼──● L1
│  0.3 ns ─┼● Reg
│         │
└─────────┴───────────────────────────────────────
          1KB   64KB  256KB  32MB  64GB   1TB
                     Capacity (log scale)

Cost Analysis (approximate 2024 prices):
┌────────────────────────────────────────────────────────────┐
│ Storage Type  │ $/GB      │ 1 GB would cost...            │
├───────────────┼───────────┼───────────────────────────────┤
│ SRAM (cache)  │ ~$1000    │ As much as a laptop!          │
│ DRAM          │ ~$5       │ Cup of coffee                 │
│ SSD           │ ~$0.10    │ Fraction of a cent            │
│ HDD           │ ~$0.02    │ Almost free                   │
└───────────────┴───────────┴───────────────────────────────┘

Why We Need Hierarchy:
┌────────────────────────────────────────────────────────────┐
│ Imagine if all memory were as fast as L1 cache:           │
│                                                            │
│ 64 GB at $1000/GB = $64,000 just for RAM!                 │
│                                                            │
│ With hierarchy:                                            │
│ - 64 KB L1: ~$64 (built into CPU)                        │
│ - 256 KB L2: ~$256 (built into CPU)                      │
│ - 32 MB L3: ~$3,200 (built into CPU)                     │
│ - 64 GB RAM: ~$320                                        │
│                                                            │
│ Total: ~$3,840 (and we get the speed benefits!)          │
└────────────────────────────────────────────────────────────┘
```

---

## Locality Principle (지역성 원리)

### Concept (개념)

**Locality of Reference (참조 지역성)** is why memory hierarchy works. Programs tend to access data in predictable patterns.

**1. Temporal Locality (시간적 지역성)**
- Recently accessed data will likely be accessed again soon
- Examples:
  - Loop counters
  - Frequently called functions
  - Hot variables

**2. Spatial Locality (공간적 지역성)**
- Data near recently accessed data will likely be accessed soon
- Examples:
  - Array elements accessed sequentially
  - Struct fields accessed together
  - Code executed in sequence

**Why Locality Exists:**
- Loops reuse the same instructions and data
- Data structures store related data together
- Programs execute instructions sequentially (mostly)

### Diagram / Example

```
Temporal Locality (시간적 지역성):
┌────────────────────────────────────────────────────────────┐
│ for (int i = 0; i < 1000; i++) {                          │
│     sum += arr[i];  // 'sum' accessed 1000 times          │
│ }                   // Loop variable 'i' accessed 1000x   │
│                                                            │
│ Time:  t0    t1    t2    t3    ...   t999                 │
│ Access: sum   sum   sum   sum  ...   sum                  │
│         ↑     ↑     ↑     ↑          ↑                    │
│         └─────┴─────┴─────┴──────────┘                    │
│              Same location accessed repeatedly             │
└────────────────────────────────────────────────────────────┘

Spatial Locality (공간적 지역성):
┌────────────────────────────────────────────────────────────┐
│ int arr[1000];                                             │
│ for (int i = 0; i < 1000; i++) {                          │
│     sum += arr[i];  // Sequential access                  │
│ }                                                          │
│                                                            │
│ Memory:                                                    │
│ ┌─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┐        │
│ │arr[0]│arr[1]│arr[2]│arr[3]│arr[4]│arr[5]│arr[6]│arr[7]│        │
│ └─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┘        │
│    ↑     ↑     ↑     ↑     ↑     ↑     ↑     ↑           │
│    └─────┴─────┴─────┴─────┴─────┴─────┴─────┘           │
│           Adjacent locations accessed in sequence          │
└────────────────────────────────────────────────────────────┘

Cache Line Prefetch Benefit:
┌────────────────────────────────────────────────────────────┐
│ When CPU accesses arr[0]:                                  │
│ - Cache loads entire cache line (64 bytes)                │
│ - For int[16], that's arr[0] through arr[15]              │
│                                                            │
│ ┌─────────────────── 64-byte cache line ──────────────────┐│
│ │arr[0]│arr[1]│arr[2]│...│arr[14]│arr[15]│                ││
│ └─────────────────────────────────────────────────────────┘│
│                                                            │
│ Access arr[1-15]: Already in cache! (FREE)                │
└────────────────────────────────────────────────────────────┘

Good vs Bad Locality Examples:
┌────────────────────────────────────────────────────────────┐
│ GOOD: Sequential array access                              │
│ for (i = 0; i < N; i++)                                   │
│     sum += arr[i];                                        │
│ // Excellent spatial locality                              │
│                                                            │
│ BAD: Random access                                         │
│ for (i = 0; i < N; i++)                                   │
│     sum += arr[rand() % N];                               │
│ // No spatial locality, each access likely a cache miss   │
│                                                            │
│ Performance difference: 10-100x slower with random access! │
└────────────────────────────────────────────────────────────┘

Struct Layout and Locality:
┌────────────────────────────────────────────────────────────┐
│ // Good: Related fields together (hot/cold separation)    │
│ struct Good {                                              │
│     float x, y, z;      // Position - often accessed      │
│     float vx, vy, vz;   // Velocity - often accessed      │
│     char name[32];      // Rarely accessed                │
│     int id;             // Rarely accessed                │
│ };                                                         │
│                                                            │
│ // Better for iteration:                                   │
│ struct Position { float x, y, z; };                       │
│ struct Velocity { float vx, vy, vz; };                    │
│ Position positions[1000];  // Iterate these together      │
│ Velocity velocities[1000]; // Iterate these together      │
└────────────────────────────────────────────────────────────┘
```

---

## Summary (요약)

| Level | Korean | Typical Size | Access Time | Technology |
|-------|--------|--------------|-------------|------------|
| Registers | 레지스터 | ~1 KB | ~0.3 ns | Flip-flops |
| L1 Cache | L1 캐시 | ~64 KB | ~1 ns | SRAM |
| L2 Cache | L2 캐시 | ~256 KB | ~4 ns | SRAM |
| L3 Cache | L3 캐시 | ~32 MB | ~15 ns | SRAM |
| RAM | 주기억장치 | ~64 GB | ~100 ns | DRAM |
| SSD | 저장장치 | ~1 TB | ~100 μs | Flash |

**Locality Types:**

| Type | Korean | Description | Example |
|------|--------|-------------|---------|
| Temporal | 시간적 지역성 | Same data reused | Loop counter |
| Spatial | 공간적 지역성 | Nearby data used | Array traversal |

**Key Takeaways for Performance:**
1. **Exploit temporal locality** - reuse data while it's in cache
2. **Exploit spatial locality** - access data sequentially
3. **Understand cache line size** (typically 64 bytes) - access data in chunks
4. **Keep working set small** - fit hot data in L1/L2 cache
5. **Memory access is the bottleneck** - optimize for cache, not just CPU
6. **Data layout matters** - organize data for access patterns, not just logical grouping
