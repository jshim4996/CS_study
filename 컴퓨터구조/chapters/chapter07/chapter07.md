# Chapter 07. Cache-Friendly Programming (캐시 친화적 프로그래밍)

> Learn how to write code that maximizes cache efficiency for optimal performance.

---

## Cache Miss Impact on Performance (캐시 미스가 성능에 미치는 영향)

### Concept (개념)

**Cache misses are the silent performance killer.** Modern CPUs can execute billions of instructions per second, but a single cache miss can stall the CPU for hundreds of cycles.

**The Numbers:**
| Access Type | Latency | CPU Cycles (3 GHz) |
|-------------|---------|-------------------|
| L1 Cache Hit | ~1 ns | ~3 cycles |
| L2 Cache Hit | ~4 ns | ~12 cycles |
| L3 Cache Hit | ~15 ns | ~45 cycles |
| RAM Access | ~100 ns | ~300 cycles |

**The Problem:**
```
During a RAM access (~300 cycles), the CPU could have executed:
- 300+ simple instructions (add, subtract)
- 100+ complex instructions (multiply, divide)

Your "fast" algorithm might be slow due to memory access patterns!
```

### Diagram / Example

```
Cache Miss Impact Visualization:
┌────────────────────────────────────────────────────────────┐
│ Task: Sum 1 million integers                               │
│                                                            │
│ Cache-Friendly (Sequential Access):                        │
│ ┌───┬───┬───┬───┬───┬───┬───┬───┐                        │
│ │ 0 │ 1 │ 2 │ 3 │ 4 │ 5 │ 6 │ 7 │ → 8 ints in cache line │
│ └─┬─┴───┴───┴───┴───┴───┴───┴───┘                        │
│   │ Load once, use 8 times                                 │
│   └→ Hit rate: ~99%                                        │
│      Time: ~2 ms                                           │
│                                                            │
│ Cache-Unfriendly (Random Access):                          │
│ ┌───┐   ┌───┐   ┌───┐   ┌───┐                            │
│ │ 0 │   │427│   │ 91│   │853│   → Each access = new line  │
│ └─┬─┘   └─┬─┘   └─┬─┘   └─┬─┘                            │
│   │       │       │       │                                │
│   └───────┴───────┴───────┘                                │
│   Each access loads 64 bytes, uses 4 bytes                 │
│   Hit rate: ~50%                                           │
│   Time: ~50 ms (25x slower!)                               │
└────────────────────────────────────────────────────────────┘

Real-World Example - Game Loop:
┌────────────────────────────────────────────────────────────┐
│ // Bad: Random access to game entities                     │
│ for (Entity* e : all_entities) {                          │
│     if (e->is_active) {           // Cache miss!          │
│         e->position += e->velocity;  // Another miss!     │
│     }                                                      │
│ }                                                          │
│ // Each entity pointer could be anywhere in memory        │
│                                                            │
│ // Good: Contiguous arrays                                 │
│ for (int i = 0; i < active_count; i++) {                  │
│     positions[i] += velocities[i];  // Sequential access  │
│ }                                                          │
│ // All data packed together, prefetcher happy             │
└────────────────────────────────────────────────────────────┘
```

---

## Array Traversal Direction (배열 순회 방향)

### Concept (개념)

**Row-major vs Column-major** order determines how multi-dimensional arrays are laid out in memory.

**C/C++, Python (NumPy default), Java:** Row-major
```
int arr[3][4] in memory:
[0][0] [0][1] [0][2] [0][3] [1][0] [1][1] [1][2] [1][3] [2][0] ...
└───────── Row 0 ─────────┘└───────── Row 1 ─────────┘└─ Row 2
```

**Fortran, MATLAB, Julia:** Column-major
```
int arr[3][4] in memory:
[0][0] [1][0] [2][0] [0][1] [1][1] [2][1] [0][2] ...
└─── Col 0 ───┘└─── Col 1 ───┘└─── Col 2 ───┘
```

**Rule:** Always traverse in the direction of memory layout!

### Diagram / Example

```
Row-major Access Pattern (C/C++):
┌────────────────────────────────────────────────────────────┐
│ int matrix[1000][1000];                                    │
│                                                            │
│ GOOD: Row-by-row (inner loop = column)                     │
│ for (int i = 0; i < 1000; i++)        // rows             │
│     for (int j = 0; j < 1000; j++)    // columns          │
│         sum += matrix[i][j];                               │
│                                                            │
│ Memory access:                                             │
│ [0][0] → [0][1] → [0][2] → ... → [0][999] → [1][0] → ...  │
│ └──────────── Sequential! ──────────────┘                  │
│                                                            │
│ BAD: Column-by-column (inner loop = row)                   │
│ for (int j = 0; j < 1000; j++)        // columns          │
│     for (int i = 0; i < 1000; i++)    // rows             │
│         sum += matrix[i][j];                               │
│                                                            │
│ Memory access:                                             │
│ [0][0] → [1][0] → [2][0] → ...                            │
│    │        │        │                                     │
│    └────────┴────────┘ Stride = 1000 elements = 4000 bytes │
│    Each access jumps over entire rows!                     │
└────────────────────────────────────────────────────────────┘

Performance Comparison:
┌────────────────────────────────────────────────────────────┐
│ 1000x1000 matrix of integers (4 MB)                        │
│                                                            │
│ Row-major access:                                          │
│   Cache lines loaded: 4,000,000 / 64 ≈ 62,500             │
│   Each line fully utilized (16 ints per line)              │
│   Time: ~5 ms                                              │
│                                                            │
│ Column-major access:                                       │
│   Cache lines loaded: 1,000,000 (one per access!)         │
│   Each line only 1/16 utilized                             │
│   Time: ~100 ms                                            │
│                                                            │
│ Difference: 20x slower for wrong traversal order!         │
└────────────────────────────────────────────────────────────┘

Matrix Multiplication Optimization:
┌────────────────────────────────────────────────────────────┐
│ // Standard: C = A × B                                     │
│ // A[i][k] accessed row-wise (good)                        │
│ // B[k][j] accessed column-wise (bad!)                     │
│                                                            │
│ for (i) for (j) for (k)                                   │
│     C[i][j] += A[i][k] * B[k][j];                         │
│                              ↑ column access               │
│                                                            │
│ // Loop interchange: make B row-wise                       │
│ for (i) for (k) for (j)  // k and j swapped               │
│     C[i][j] += A[i][k] * B[k][j];                         │
│                              ↑ now row access!             │
│                                                            │
│ Result: 3-10x speedup for large matrices                   │
└────────────────────────────────────────────────────────────┘
```

---

## Data Structures and Cache Efficiency (데이터 구조와 캐시 효율)

### Concept (개념)

**Array of Structures (AoS) vs Structure of Arrays (SoA)**

Different layouts have different cache characteristics:

**AoS (Array of Structures):**
```c
struct Entity {
    float x, y, z;      // position
    float vx, vy, vz;   // velocity
    int health;
    int type;
};
Entity entities[1000];
```

**SoA (Structure of Arrays):**
```c
struct Entities {
    float x[1000], y[1000], z[1000];
    float vx[1000], vy[1000], vz[1000];
    int health[1000];
    int type[1000];
};
```

**When to use each:**
| Pattern | Best Layout | Example |
|---------|------------|---------|
| Access all fields of one entity | AoS | Game entity update |
| Access one field of all entities | SoA | Physics simulation |
| SIMD vectorization | SoA | Graphics, audio |

### Diagram / Example

```
Memory Layout Comparison:
┌────────────────────────────────────────────────────────────┐
│ AoS (Array of Structures):                                 │
│                                                            │
│ entities[0]: [x][y][z][vx][vy][vz][hp][type]              │
│ entities[1]: [x][y][z][vx][vy][vz][hp][type]              │
│ entities[2]: [x][y][z][vx][vy][vz][hp][type]              │
│ ...                                                        │
│                                                            │
│ To update all positions (x,y,z):                          │
│ Load entity[0] → use x,y,z, skip vx,vy,vz,hp,type        │
│ Load entity[1] → use x,y,z, skip vx,vy,vz,hp,type        │
│ Cache utilization: 12/32 = 37.5%                          │
└────────────────────────────────────────────────────────────┘

┌────────────────────────────────────────────────────────────┐
│ SoA (Structure of Arrays):                                 │
│                                                            │
│ x:    [x0][x1][x2][x3][x4][x5]...                         │
│ y:    [y0][y1][y2][y3][y4][y5]...                         │
│ z:    [z0][z1][z2][z3][z4][z5]...                         │
│ vx:   [vx0][vx1][vx2][vx3]...                             │
│ ...                                                        │
│                                                            │
│ To update all positions:                                   │
│ Load x array → use every float (100%)                     │
│ Load y array → use every float (100%)                     │
│ Load z array → use every float (100%)                     │
│ Cache utilization: 100%                                    │
└────────────────────────────────────────────────────────────┘

SIMD Benefits with SoA:
┌────────────────────────────────────────────────────────────┐
│ SoA enables SIMD vectorization:                            │
│                                                            │
│ // SoA: Process 8 entities at once (AVX-256)              │
│ x[0..7] += vx[0..7];  // Single SIMD instruction          │
│ y[0..7] += vy[0..7];  // Single SIMD instruction          │
│ z[0..7] += vz[0..7];  // Single SIMD instruction          │
│                                                            │
│ // AoS: Cannot vectorize easily                            │
│ // Data not contiguous, must gather/scatter               │
│                                                            │
│ Performance: SoA + SIMD can be 8x faster                   │
└────────────────────────────────────────────────────────────┘

Hybrid Approach (AoSoA):
┌────────────────────────────────────────────────────────────┐
│ // Best of both worlds for SIMD width 8                    │
│ struct EntityBlock {                                       │
│     float x[8], y[8], z[8];     // 8 entities grouped     │
│     float vx[8], vy[8], vz[8];                            │
│     int health[8], type[8];                               │
│ };                                                         │
│ EntityBlock blocks[125];  // 1000 entities total          │
│                                                            │
│ Benefits:                                                  │
│ + SIMD-friendly (process 8 at once)                        │
│ + All data for 8 entities fits in cache together          │
│ + Better than pure SoA for mixed access patterns          │
└────────────────────────────────────────────────────────────┘
```

---

## False Sharing Problem (False Sharing 문제)

### Concept (개념)

**False Sharing** occurs when multiple threads write to different variables that share the same cache line. Even though they're logically independent, the hardware treats them as conflicting.

**The Problem:**
```
Core 0 writes variable A  →  Invalidates entire cache line
Core 1 writes variable B  →  Invalidates entire cache line
                             (even though A and B are different!)
```

**Result:** Cache line "ping-pongs" between cores, destroying performance.

**Solution:** Pad data to ensure each thread's data is on its own cache line.

### Diagram / Example

```
False Sharing Illustrated:
┌────────────────────────────────────────────────────────────┐
│ struct Counters {                                          │
│     int count0;  // Thread 0 writes this                   │
│     int count1;  // Thread 1 writes this                   │
│     int count2;  // Thread 2 writes this                   │
│     int count3;  // Thread 3 writes this                   │
│ };  // All 4 ints fit in ONE 64-byte cache line!          │
│                                                            │
│ Cache Line: [count0|count1|count2|count3|...padding...]   │
│              ↑       ↑       ↑       ↑                     │
│              T0      T1      T2      T3                    │
│                                                            │
│ T0 writes count0 → Line invalidated in T1,T2,T3 caches   │
│ T1 writes count1 → Line invalidated in T0,T2,T3 caches   │
│ T2 writes count2 → Line invalidated in T0,T1,T3 caches   │
│ T3 writes count3 → Line invalidated in T0,T1,T2 caches   │
│                                                            │
│ Constant invalidation = terrible performance!              │
└────────────────────────────────────────────────────────────┘

Solution: Cache Line Padding
┌────────────────────────────────────────────────────────────┐
│ struct alignas(64) PaddedCounter {                         │
│     int count;                                             │
│     char padding[60];  // Fill rest of cache line          │
│ };                                                         │
│                                                            │
│ PaddedCounter counters[4];                                 │
│                                                            │
│ Memory Layout:                                             │
│ Cache Line 0: [count0|padding...                         ] │
│ Cache Line 1: [count1|padding...                         ] │
│ Cache Line 2: [count2|padding...                         ] │
│ Cache Line 3: [count3|padding...                         ] │
│                                                            │
│ Now each thread has its own cache line - no conflicts!    │
└────────────────────────────────────────────────────────────┘

Performance Impact:
┌────────────────────────────────────────────────────────────┐
│ Benchmark: 4 threads, each incrementing its counter       │
│ 100 million times                                          │
│                                                            │
│ Without Padding (False Sharing):                           │
│   Time: 2500 ms                                            │
│   Cache line transfers: ~400 million                       │
│                                                            │
│ With Padding (No False Sharing):                           │
│   Time: 50 ms                                              │
│   Cache line transfers: ~0                                 │
│                                                            │
│ Result: 50x speedup just by adding padding!               │
└────────────────────────────────────────────────────────────┘

C++17 Standard Solution:
┌────────────────────────────────────────────────────────────┐
│ #include <new>                                             │
│                                                            │
│ struct alignas(std::hardware_destructive_interference_size)│
│ Counter {                                                  │
│     std::atomic<int> count{0};                            │
│ };                                                         │
│                                                            │
│ // hardware_destructive_interference_size = cache line    │
│ // Usually 64 bytes on x86                                 │
└────────────────────────────────────────────────────────────┘
```

---

## Cache Optimization in Games/Graphics (게임/그래픽에서 캐시 최적화)

### Concept (개념)

**Game development** is particularly cache-sensitive because:
1. Large amounts of data (entities, vertices, textures)
2. Strict frame time budgets (16.67 ms for 60 FPS)
3. Predictable access patterns (update loops)

**Key Optimization Strategies:**

| Strategy | Description | Use Case |
|----------|-------------|----------|
| Data-Oriented Design | Organize data by access pattern | ECS (Entity Component System) |
| Hot/Cold Splitting | Separate frequently/rarely accessed data | Entity management |
| Prefetching | Load data before it's needed | Streaming, large arrays |
| Cache Blocking | Process data in cache-sized chunks | Matrix operations |

### Diagram / Example

```
Game Entity: Traditional OOP vs Data-Oriented
┌────────────────────────────────────────────────────────────┐
│ OOP Style (Cache Unfriendly):                              │
│                                                            │
│ class Entity {                                             │
│     Vector3 position;          // Used every frame        │
│     Vector3 velocity;          // Used every frame        │
│     Mesh* mesh;                // Used for rendering      │
│     AI* ai;                    // Used for some entities  │
│     Physics* physics;          // Used for some entities  │
│     String name;               // Rarely used             │
│     CreationDate created;      // Almost never used       │
│ };                                                         │
│                                                            │
│ vector<Entity*> entities;  // Pointers everywhere!        │
│                                                            │
│ for (auto e : entities) {                                 │
│     e->position += e->velocity;  // Two cache misses!     │
│ }                                                          │
└────────────────────────────────────────────────────────────┘

┌────────────────────────────────────────────────────────────┐
│ Data-Oriented (Cache Friendly):                            │
│                                                            │
│ // Hot data: accessed every frame                          │
│ struct TransformComponents {                               │
│     vector<Vector3> positions;   // Contiguous            │
│     vector<Vector3> velocities;  // Contiguous            │
│ };                                                         │
│                                                            │
│ // Cold data: accessed rarely                              │
│ struct MetadataComponents {                                │
│     vector<String> names;                                  │
│     vector<CreationDate> created;                          │
│ };                                                         │
│                                                            │
│ // Update loop: Sequential access!                         │
│ for (size_t i = 0; i < count; i++) {                      │
│     transforms.positions[i] += transforms.velocities[i];  │
│ }                                                          │
└────────────────────────────────────────────────────────────┘

Cache Blocking for Large Data:
┌────────────────────────────────────────────────────────────┐
│ // Process huge array in cache-friendly blocks             │
│ // L2 cache = 256 KB = 64K floats                         │
│                                                            │
│ const int BLOCK_SIZE = 32768;  // Fits in L2              │
│                                                            │
│ // Process in blocks                                       │
│ for (int block = 0; block < N; block += BLOCK_SIZE) {     │
│     int end = min(block + BLOCK_SIZE, N);                 │
│                                                            │
│     // All accesses within block hit L2 cache             │
│     for (int i = block; i < end; i++) {                   │
│         // Multiple passes over same data                  │
│         result[i] = process(data[i]);                     │
│     }                                                      │
│ }                                                          │
└────────────────────────────────────────────────────────────┘

Prefetching Hints:
┌────────────────────────────────────────────────────────────┐
│ // Manual prefetching for large sequential access          │
│                                                            │
│ for (int i = 0; i < N; i++) {                             │
│     // Prefetch data that will be needed soon             │
│     __builtin_prefetch(&data[i + 64], 0, 0);              │
│                                                            │
│     // Process current data (already in cache)             │
│     result[i] = process(data[i]);                         │
│ }                                                          │
│                                                            │
│ Note: Modern CPUs have good hardware prefetchers.          │
│ Manual prefetching helps mainly for irregular patterns.    │
└────────────────────────────────────────────────────────────┘

Practical Example: Particle System
┌────────────────────────────────────────────────────────────┐
│ // 100,000 particles at 60 FPS                             │
│                                                            │
│ // Bad: AoS with per-particle allocation                   │
│ class Particle {                                           │
│     Vector3 pos, vel, acc;                                │
│     Color color;                                           │
│     float life, size;                                      │
│     virtual void update();  // Virtual call per particle! │
│ };                                                         │
│ → ~20ms per frame (too slow for 60 FPS!)                  │
│                                                            │
│ // Good: SoA with batch processing                         │
│ struct ParticleSystem {                                    │
│     float* x, *y, *z;           // Positions              │
│     float* vx, *vy, *vz;        // Velocities             │
│     // Update with SIMD                                    │
│     void update_positions(float dt) {                     │
│         for (int i = 0; i < count; i += 8) {              │
│             // AVX processes 8 particles at once          │
│             x[i:i+8] += vx[i:i+8] * dt;                   │
│         }                                                  │
│     }                                                      │
│ };                                                         │
│ → ~0.5ms per frame (easily 60 FPS!)                       │
└────────────────────────────────────────────────────────────┘
```

---

## Summary (요약)

| Optimization | Korean | Key Point |
|--------------|--------|-----------|
| Sequential Access | 순차 접근 | 100x faster than random |
| Row-major Traversal | 행 우선 순회 | Match memory layout |
| SoA Layout | 배열들의 구조체 | Better for bulk operations |
| Cache Line Padding | 캐시 라인 패딩 | Prevent false sharing |
| Cache Blocking | 캐시 블로킹 | Keep working set in cache |

**Golden Rules for Cache Performance:**
1. **Access memory sequentially** when possible
2. **Keep related data together** in memory
3. **Avoid pointer chasing** - prefer contiguous arrays
4. **Pad multithreaded data** to prevent false sharing
5. **Profile before optimizing** - measure cache misses
6. **Think about data layout** before algorithms
