# Chapter 08. Memory Management Basics (메모리 관리 기초)

> Memory management is crucial for efficient system operation, ensuring processes have access to the memory they need while protecting them from each other.

---

## Memory Hierarchy (메모리 계층 구조)

### Concept (개념)

Computer systems use a **Memory Hierarchy (메모리 계층 구조)** to balance speed, capacity, and cost. Faster memory is more expensive and smaller, while cheaper memory is slower but larger.

**Hierarchy Levels (from fastest to slowest):**

| Level | Type | Speed | Size | Cost/Byte |
|-------|------|-------|------|-----------|
| 1 | Registers | < 1 ns | < 1 KB | Highest |
| 2 | Cache (L1/L2/L3) | 1-10 ns | KB-MB | High |
| 3 | Main Memory (RAM) | 50-100 ns | GB | Medium |
| 4 | SSD/Flash | 10-100 us | GB-TB | Low |
| 5 | Hard Disk | 5-10 ms | TB | Lowest |

**Principle of Locality:**
- **Temporal Locality (시간적 지역성)**: Recently accessed data likely accessed again soon
- **Spatial Locality (공간적 지역성)**: Data near recently accessed data likely accessed soon

### Diagram / Example

```
Memory Hierarchy Pyramid (메모리 계층 피라미드)

┌─────────────────────────────────────────────────────────┐
│                                                          │
│                         /\                               │
│                        /  \                              │
│                       / CPU\     ◄── Registers          │
│                      / Regs \        (< 1 KB)           │
│                     /────────\                          │
│                    /  Cache   \  ◄── L1/L2/L3 Cache    │
│                   /  (L1/L2/L3)\     (KB - MB)         │
│                  /──────────────\                       │
│                 /   Main Memory  \ ◄── RAM             │
│                /      (RAM)       \    (GB)            │
│               /────────────────────\                    │
│              /    Secondary Storage \ ◄── SSD/HDD      │
│             /      (SSD/HDD)         \   (TB)          │
│            /──────────────────────────\                 │
│                                                          │
│           Faster ▲                   ▲ Smaller          │
│           Expensive                  Closer to CPU      │
│                                                          │
│           Slower ▼                   ▼ Larger           │
│           Cheaper                    Further from CPU   │
│                                                          │
└─────────────────────────────────────────────────────────┘


Speed vs Size Trade-off:

┌────────────────────────────────────────────────────────┐
│                                                         │
│  Access Time          Size                              │
│       │                 │                               │
│   1 ns│ ● Registers    │ 1 KB                          │
│       │                 │                               │
│  10 ns│ ● Cache        │ 1 MB                          │
│       │                 │                               │
│ 100 ns│ ● RAM          │ 16 GB                         │
│       │                 │                               │
│  1 ms │ ● SSD          │ 512 GB                        │
│       │                 │                               │
│ 10 ms │ ● HDD          │ 4 TB                          │
│       ▼                 ▼                               │
│                                                         │
└────────────────────────────────────────────────────────┘


Principle of Locality (지역성 원리):

Temporal Locality (시간적 지역성):
┌────────────────────────────────────────────────────────┐
│                                                         │
│  for (i = 0; i < 1000; i++) {                          │
│      sum += array[i];        ◄── 'sum' accessed        │
│  }                               1000 times            │
│                                                         │
│  Variable 'sum' accessed repeatedly                    │
│  Keep 'sum' in fast memory (cache/register)           │
│                                                         │
└────────────────────────────────────────────────────────┘

Spatial Locality (공간적 지역성):
┌────────────────────────────────────────────────────────┐
│                                                         │
│  Array in memory:                                       │
│  ┌─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┐    │
│  │ [0] │ [1] │ [2] │ [3] │ [4] │ [5] │ [6] │ [7] │    │
│  └─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┘    │
│    ▲                                                    │
│    └── Access [0], likely to access [1], [2]... soon  │
│                                                         │
│  When [0] accessed, load entire block into cache       │
│                                                         │
└────────────────────────────────────────────────────────┘


Data Movement Through Hierarchy:

┌────────────────────────────────────────────────────────┐
│                                                         │
│  CPU needs data at address X                           │
│         │                                               │
│         ▼                                               │
│  ┌─────────────────┐                                   │
│  │ Check Registers │──── Found? ──► Use it            │
│  └────────┬────────┘                                   │
│           │ Not found (miss)                           │
│           ▼                                            │
│  ┌─────────────────┐                                   │
│  │  Check L1 Cache │──── Found? ──► Copy to register  │
│  └────────┬────────┘                                   │
│           │ Not found (miss)                           │
│           ▼                                            │
│  ┌─────────────────┐                                   │
│  │  Check L2 Cache │──── Found? ──► Copy to L1       │
│  └────────┬────────┘                                   │
│           │ Not found (miss)                           │
│           ▼                                            │
│  ┌─────────────────┐                                   │
│  │  Check RAM      │──── Found? ──► Copy to cache    │
│  └────────┬────────┘                                   │
│           │ Not found (page fault)                    │
│           ▼                                            │
│  ┌─────────────────┐                                   │
│  │  Load from Disk │──► Copy to RAM                   │
│  └─────────────────┘                                   │
│                                                         │
└────────────────────────────────────────────────────────┘
```

---

## Address Binding (주소 바인딩)

### Concept (개념)

**Address Binding (주소 바인딩)** is the process of mapping program addresses to physical memory addresses. It can occur at different times.

**Types of Address Binding:**

| Time | Description | Flexibility | Modern Usage |
|------|-------------|-------------|--------------|
| **Compile Time** | Physical address known at compile time | None | Embedded systems |
| **Load Time** | Address assigned when program loads | Limited | Older systems |
| **Execution Time** | Address determined during runtime | Full | Modern systems |

**Key Benefit of Execution-Time Binding:**
- Process can be moved in memory during execution
- Requires hardware support (MMU)
- Enables virtual memory and dynamic relocation

### Diagram / Example

```
Address Binding Times (주소 바인딩 시점)

Program Lifecycle:
┌─────────────────────────────────────────────────────────┐
│                                                          │
│  Source Code       Object Code      Executable          │
│  ┌─────────┐      ┌─────────┐      ┌─────────┐         │
│  │ int x;  │      │ symbolic│      │ relocat-│         │
│  │ x = 5;  │ ───► │ address │ ───► │ able    │         │
│  │         │      │         │      │ address │         │
│  └─────────┘      └─────────┘      └─────────┘         │
│    Compile          Link            Load                │
│                                                          │
│  ───────────────────────────────────────────────────►   │
│                         Time                             │
│                                                          │
└─────────────────────────────────────────────────────────┘


1. Compile-Time Binding (컴파일 타임 바인딩):
┌─────────────────────────────────────────────────────────┐
│                                                          │
│  At compile time, physical address is known             │
│                                                          │
│  Source Code:              Generated Code:               │
│  ┌─────────────────┐      ┌─────────────────┐           │
│  │ int x = 5;      │ ───► │ MOV [1000], 5   │           │
│  │                 │      │ (absolute addr) │           │
│  └─────────────────┘      └─────────────────┘           │
│                                                          │
│  Physical Memory:                                        │
│  ┌────────────────────────────────────────┐             │
│  │ ...  │ [1000]: x │  ...                │             │
│  └────────────────────────────────────────┘             │
│                                                          │
│  ✗ Cannot move program in memory                        │
│  ✗ Must recompile if location changes                   │
│  ✓ Used in simple embedded systems                      │
│                                                          │
└─────────────────────────────────────────────────────────┘


2. Load-Time Binding (로드 타임 바인딩):
┌─────────────────────────────────────────────────────────┐
│                                                          │
│  Relocatable code generated at compile time             │
│  Actual address assigned when program loads             │
│                                                          │
│  Executable:                                             │
│  ┌─────────────────┐                                    │
│  │ MOV [base+0], 5 │  (relocatable)                    │
│  └─────────────────┘                                    │
│           │                                              │
│           │  Load at address 5000                       │
│           ▼                                              │
│  Memory:                                                 │
│  ┌─────────────────┐                                    │
│  │ MOV [5000], 5   │  base = 5000                      │
│  └─────────────────┘                                    │
│                                                          │
│  ✓ More flexible than compile-time                      │
│  ✗ Cannot move after loading                            │
│                                                          │
└─────────────────────────────────────────────────────────┘


3. Execution-Time Binding (실행 타임 바인딩):
┌─────────────────────────────────────────────────────────┐
│                                                          │
│  Address translation happens during execution           │
│  Requires hardware support (MMU)                        │
│                                                          │
│  CPU                     MMU              Physical       │
│  ┌─────┐               ┌─────┐            Memory        │
│  │     │──► Logical ──►│     │──► Physical│            │
│  │     │    Address    │     │    Address │            │
│  └─────┘    (100)      └─────┘    (14100) │            │
│                          ▲                 │            │
│                          │                 │            │
│                      Relocation           ┌▼────────┐   │
│                      Register             │ 14100   │   │
│                      (14000)              └─────────┘   │
│                                                          │
│  14000 + 100 = 14100                                    │
│                                                          │
│  ✓ Can move process during execution                    │
│  ✓ Enables virtual memory                               │
│  ✓ Modern operating systems use this                    │
│                                                          │
└─────────────────────────────────────────────────────────┘


Comparison Table:

┌────────────────┬────────────────┬────────────────────────┐
│   Binding Time │  Flexibility   │       Example          │
├────────────────┼────────────────┼────────────────────────┤
│ Compile Time   │ None           │ MS-DOS .COM files      │
│                │                │ Embedded systems       │
├────────────────┼────────────────┼────────────────────────┤
│ Load Time      │ One-time       │ Older Unix systems     │
│                │ relocation     │ Some libraries         │
├────────────────┼────────────────┼────────────────────────┤
│ Execution Time │ Full dynamic   │ Modern OS (Linux,      │
│                │ relocation     │ Windows, macOS)        │
└────────────────┴────────────────┴────────────────────────┘
```

---

## Logical vs Physical Address (논리적 주소 vs 물리적 주소)

### Concept (개념)

**Logical Address (논리적 주소)**, also called **Virtual Address (가상 주소)**, is the address generated by the CPU during program execution.

**Physical Address (물리적 주소)** is the actual address in physical memory (RAM).

**Address Translation:**
- Performed by **MMU (Memory Management Unit)**
- Transparent to the user program
- Enables memory protection and virtual memory

**Benefits of Address Translation:**
1. **Protection**: Processes cannot access each other's memory
2. **Relocation**: Process can be placed anywhere in memory
3. **Virtual Memory**: Process can use more memory than physically available

### Diagram / Example

```
Logical vs Physical Address (논리적 vs 물리적 주소)

Basic Concept:
┌─────────────────────────────────────────────────────────┐
│                                                          │
│  Logical Address Space          Physical Address Space  │
│  (What process sees)            (Actual RAM)            │
│                                                          │
│  Process P1:                    Physical Memory:        │
│  ┌─────────────────┐           ┌─────────────────┐      │
│  │ 0: code         │           │ 0: OS           │      │
│  │ 100: data       │   ─────►  │ ...             │      │
│  │ 200: stack      │   MMU     │ 5000: P1 code   │      │
│  │ ...             │           │ 5100: P1 data   │      │
│  │ max: limit      │           │ 5200: P1 stack  │      │
│  └─────────────────┘           │ ...             │      │
│                                │ 8000: P2        │      │
│  Process P2:                   │ ...             │      │
│  ┌─────────────────┐           └─────────────────┘      │
│  │ 0: code         │                                    │
│  │ ...             │   ─────►  Mapped to 8000+         │
│  └─────────────────┘                                    │
│                                                          │
│  Each process thinks it has the whole address space!    │
│                                                          │
└─────────────────────────────────────────────────────────┘


MMU (Memory Management Unit):
┌─────────────────────────────────────────────────────────┐
│                                                          │
│         CPU                    MMU                       │
│  ┌─────────────────┐    ┌─────────────────────┐         │
│  │                 │    │                     │         │
│  │  Generates      │    │  Relocation         │         │
│  │  Logical        │───►│  Register           │         │
│  │  Address        │    │  ┌─────────┐        │         │
│  │                 │    │  │  14000  │        │         │
│  │  (e.g., 346)    │    │  └────┬────┘        │         │
│  │                 │    │       │             │         │
│  └─────────────────┘    │       ▼             │         │
│                         │  ┌─────────┐        │         │
│                         │  │   +     │        │         │
│                         │  └────┬────┘        │         │
│                         │       │             │         │
│                         │       ▼             │         │
│                         │  Physical Address   │         │
│                         │  (14346)            │───►Memory
│                         │                     │         │
│                         └─────────────────────┘         │
│                                                          │
│  Logical 346 + Base 14000 = Physical 14346             │
│                                                          │
└─────────────────────────────────────────────────────────┘


Memory Protection with Base and Limit:
┌─────────────────────────────────────────────────────────┐
│                                                          │
│  For each process, OS maintains:                        │
│  • Base register: Starting physical address             │
│  • Limit register: Size of address space                │
│                                                          │
│         CPU                     MMU                      │
│  ┌─────────────────┐     ┌─────────────────────────┐    │
│  │                 │     │                         │    │
│  │  Logical Addr   │     │   ┌─────────────────┐  │    │
│  │     (LA)        │────►│   │    Limit        │  │    │
│  │                 │     │   │    Check        │  │    │
│  └─────────────────┘     │   └────────┬────────┘  │    │
│                          │            │           │    │
│                          │     LA < Limit?        │    │
│                          │       │        │       │    │
│                          │      YES       NO      │    │
│                          │       │        │       │    │
│                          │       ▼        ▼       │    │
│                          │   ┌──────┐  ┌──────┐  │    │
│                          │   │ + Base│  │ TRAP │  │    │
│                          │   └──┬───┘  │ Error│  │    │
│                          │      │      └──────┘  │    │
│                          │      ▼                 │    │
│                          │  Physical              │    │
│                          │  Address               │    │
│                          └─────────────────────────┘    │
│                                                          │
│  Example:                                                │
│  Base = 10000, Limit = 5000                             │
│  LA = 3000 → 3000 < 5000? YES → PA = 13000 ✓          │
│  LA = 6000 → 6000 < 5000? NO → TRAP (error) ✗         │
│                                                          │
└─────────────────────────────────────────────────────────┘


Address Space Isolation:
┌─────────────────────────────────────────────────────────┐
│                                                          │
│  Physical Memory:                                        │
│  ┌────────────────────────────────────────────────────┐ │
│  │        │           │           │           │       │ │
│  │   OS   │    P1     │    P2     │    P3     │ Free  │ │
│  │        │           │           │           │       │ │
│  └────────────────────────────────────────────────────┘ │
│  0      1000        5000        9000       12000        │
│                                                          │
│  Process P1:                 Process P2:                │
│  Base = 1000                 Base = 5000                │
│  Limit = 4000                Limit = 4000               │
│                                                          │
│  P1 sees: 0-3999             P2 sees: 0-3999           │
│  Maps to: 1000-4999          Maps to: 5000-8999        │
│                                                          │
│  P1 cannot access P2's memory (protected by MMU)       │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

---

## Contiguous Memory Allocation (연속 메모리 할당)

### Concept (개념)

**Contiguous Memory Allocation (연속 메모리 할당)** assigns each process a single contiguous block of memory.

**Allocation Strategies:**

| Strategy | Korean | Description | Pros/Cons |
|----------|--------|-------------|-----------|
| **First Fit** | 최초 적합 | Use first hole big enough | Fast, but leaves small fragments at front |
| **Best Fit** | 최적 적합 | Use smallest hole that fits | Minimizes waste but slow, leaves tiny holes |
| **Worst Fit** | 최악 적합 | Use largest hole | Leaves largest remaining hole but slow |

**Studies show** First Fit is generally fastest and performs as well as Best Fit.

### Diagram / Example

```
Contiguous Memory Allocation (연속 메모리 할당)

Memory Layout:
┌─────────────────────────────────────────────────────────┐
│                                                          │
│  ┌─────────────────────────────────────────────────┐    │
│  │ OS │ Process 1 │ Hole │ Process 2 │ Hole │ P3 │    │
│  └─────────────────────────────────────────────────┘    │
│  0   100        400    600        900   1100  1400      │
│                                                          │
│  Holes: 600-400=200, 1100-900=200                       │
│                                                          │
└─────────────────────────────────────────────────────────┘


Allocation Strategies Example:

Memory with Holes:
┌─────────────────────────────────────────────────────────┐
│                                                          │
│  ┌────┬────┬────────┬────┬──────┬────┬──────────┐       │
│  │ OS │ P1 │ Hole A │ P2 │Hole B│ P3 │  Hole C  │       │
│  │    │    │ 100 KB │    │50 KB │    │  200 KB  │       │
│  └────┴────┴────────┴────┴──────┴────┴──────────┘       │
│                                                          │
│  New process needs 80 KB                                │
│                                                          │
└─────────────────────────────────────────────────────────┘

First Fit (최초 적합):
┌─────────────────────────────────────────────────────────┐
│  Search from beginning, use first hole that fits        │
│                                                          │
│  Hole A (100 KB) >= 80 KB? YES → Use Hole A            │
│                                                          │
│  Result:                                                 │
│  ┌────┬────┬────┬────┬────┬──────┬────┬──────────┐      │
│  │ OS │ P1 │ P4 │ 20 │ P2 │Hole B│ P3 │  Hole C  │      │
│  │    │    │80KB│ KB │    │50 KB │    │  200 KB  │      │
│  └────┴────┴────┴────┴────┴──────┴────┴──────────┘      │
│                                                          │
│  ✓ Fast - stops at first fit                            │
│                                                          │
└─────────────────────────────────────────────────────────┘

Best Fit (최적 적합):
┌─────────────────────────────────────────────────────────┐
│  Search all holes, use smallest that fits               │
│                                                          │
│  Hole A: 100 KB (fits, leftover 20 KB)                 │
│  Hole B: 50 KB  (doesn't fit)                          │
│  Hole C: 200 KB (fits, leftover 120 KB)                │
│                                                          │
│  Best: Hole A (smallest that fits)                      │
│                                                          │
│  Result: Same as First Fit in this case                │
│                                                          │
│  ✓ Minimizes wasted space in chosen hole               │
│  ✗ Must search entire list                              │
│  ✗ Tends to create many tiny unusable holes            │
│                                                          │
└─────────────────────────────────────────────────────────┘

Worst Fit (최악 적합):
┌─────────────────────────────────────────────────────────┐
│  Search all holes, use largest                          │
│                                                          │
│  Hole A: 100 KB                                         │
│  Hole B: 50 KB                                          │
│  Hole C: 200 KB ◄── Largest                            │
│                                                          │
│  Use Hole C                                              │
│                                                          │
│  Result:                                                 │
│  ┌────┬────┬────────┬────┬──────┬────┬────┬──────┐      │
│  │ OS │ P1 │ Hole A │ P2 │Hole B│ P3 │ P4 │120KB │      │
│  │    │    │ 100 KB │    │50 KB │    │80KB│      │      │
│  └────┴────┴────────┴────┴──────┴────┴────┴──────┘      │
│                                                          │
│  ✓ Leaves large remaining hole                          │
│  ✗ Must search entire list                              │
│  ✗ Tends to break up large holes                        │
│                                                          │
└─────────────────────────────────────────────────────────┘


Performance Comparison:

┌────────────────────────────────────────────────────────┐
│                                                         │
│  Speed:     First Fit > Best Fit = Worst Fit           │
│                                                         │
│  Memory     First Fit ≈ Best Fit > Worst Fit           │
│  Usage:     (First Fit and Best Fit similar)           │
│                                                         │
│  Recommendation: Use First Fit                         │
│                                                         │
└────────────────────────────────────────────────────────┘
```

---

## Fragmentation (단편화)

### Concept (개념)

**Fragmentation (단편화)** is wasted memory that cannot be used. Two types exist:

**Internal Fragmentation (내부 단편화):**
- Wasted space INSIDE an allocated block
- Caused by fixed-size allocation units
- Memory allocated but not fully used

**External Fragmentation (외부 단편화):**
- Wasted space BETWEEN allocated blocks
- Total free memory exists but is not contiguous
- Cannot satisfy request even though enough total memory exists

**Solutions:**
- Compaction: Move processes to create contiguous free space
- Paging/Segmentation: Non-contiguous allocation

### Diagram / Example

```
Types of Fragmentation (단편화 유형)

Internal Fragmentation (내부 단편화):
┌─────────────────────────────────────────────────────────┐
│                                                          │
│  Fixed allocation units (e.g., 4 KB blocks)            │
│                                                          │
│  Process needs 18 KB → Allocated 5 blocks (20 KB)      │
│                                                          │
│  ┌──────────────────────────────────────────┐           │
│  │  Block 1  │  Block 2  │  Block 3  │  Block 4 │Block 5│
│  │   4 KB    │   4 KB    │   4 KB    │   4 KB   │ 4 KB  │
│  │   Used    │   Used    │   Used    │   Used   │ Used  │
│  │           │           │           │   2KB    │ 2KB   │
│  │           │           │           │  wasted! │wasted!│
│  └──────────────────────────────────────────┘           │
│                                                          │
│  Allocated: 20 KB                                        │
│  Needed:    18 KB                                        │
│  Wasted:     2 KB (internal fragmentation)              │
│                                                          │
│  Even simpler example:                                   │
│  ┌─────────────────────┐                                │
│  │     Allocated       │                                │
│  │  ┌───────────────┐  │                                │
│  │  │   Actual Use  │▓▓│  ◄── ▓▓ = Internal            │
│  │  │               │▓▓│       Fragmentation           │
│  │  └───────────────┘  │                                │
│  └─────────────────────┘                                │
│                                                          │
└─────────────────────────────────────────────────────────┘


External Fragmentation (외부 단편화):
┌─────────────────────────────────────────────────────────┐
│                                                          │
│  Memory after processes arrive and leave:               │
│                                                          │
│  ┌────┬────────┬────┬──────┬────┬────┬──────────┐       │
│  │ OS │ Hole   │ P1 │ Hole │ P2 │Hole│   P3     │       │
│  │    │ 30 KB  │    │ 40KB │    │20KB│          │       │
│  └────┴────────┴────┴──────┴────┴────┴──────────┘       │
│                                                          │
│  Total free: 30 + 40 + 20 = 90 KB                      │
│                                                          │
│  New process needs 80 KB                                 │
│                                                          │
│  Can we allocate? NO!                                    │
│  Even though total free (90 KB) > needed (80 KB)        │
│  No single contiguous hole is large enough!             │
│                                                          │
│  This is EXTERNAL FRAGMENTATION                         │
│                                                          │
└─────────────────────────────────────────────────────────┘


Solution 1: Compaction (압축)
┌─────────────────────────────────────────────────────────┐
│                                                          │
│  Before Compaction:                                      │
│  ┌────┬────────┬────┬──────┬────┬────┬──────────┐       │
│  │ OS │ Hole   │ P1 │ Hole │ P2 │Hole│   P3     │       │
│  │    │ 30 KB  │    │ 40KB │    │20KB│          │       │
│  └────┴────────┴────┴──────┴────┴────┴──────────┘       │
│                                                          │
│  After Compaction:                                       │
│  ┌────┬────┬────┬──────────┬───────────────────────┐    │
│  │ OS │ P1 │ P2 │   P3     │     Free (90 KB)      │    │
│  │    │    │    │          │                       │    │
│  └────┴────┴────┴──────────┴───────────────────────┘    │
│                                                          │
│  Now can allocate 80 KB!                                │
│                                                          │
│  Problems with compaction:                               │
│  • Requires relocation at execution time                │
│  • Very expensive (move lots of memory)                 │
│  • I/O may be disrupted                                 │
│                                                          │
└─────────────────────────────────────────────────────────┘


Solution 2: Non-contiguous Allocation (비연속 할당)
┌─────────────────────────────────────────────────────────┐
│                                                          │
│  Instead of requiring contiguous memory,                │
│  allow process to be split into pieces:                 │
│                                                          │
│  ┌────┬────────┬────┬──────┬────┬────┬──────────┐       │
│  │ OS │P4-part1│ P1 │P4-pt2│ P2 │P4-3│   P3     │       │
│  │    │ 30 KB  │    │ 40KB │    │20KB│          │       │
│  └────┴────────┴────┴──────┴────┴────┴──────────┘       │
│                                                          │
│  P4 uses: 30 + 40 + 20 = 90 KB (split into 3 parts)   │
│                                                          │
│  Techniques:                                             │
│  • Paging: Fixed-size pieces (pages)                    │
│  • Segmentation: Variable-size pieces (segments)        │
│                                                          │
└─────────────────────────────────────────────────────────┘


Fragmentation Summary:

┌────────────────────────────────────────────────────────┐
│                                                         │
│  Type        │ Where      │ Cause           │ Solution │
│  ────────────┼────────────┼─────────────────┼──────────│
│  Internal    │ Inside     │ Fixed-size      │ Smaller  │
│              │ allocated  │ allocation      │ units    │
│              │ block      │ units           │          │
│  ────────────┼────────────┼─────────────────┼──────────│
│  External    │ Between    │ Variable-size   │ Paging,  │
│              │ allocated  │ allocation,     │ Compac-  │
│              │ blocks     │ dynamic memory  │ tion     │
│                                                         │
└────────────────────────────────────────────────────────┘
```

---

## Paging Concept (페이징 개념)

### Concept (개념)

**Paging (페이징)** divides memory into fixed-size blocks:
- **Physical memory** divided into **frames (프레임)**
- **Logical memory** divided into **pages (페이지)**
- Page size = Frame size (typically 4 KB)

**Key Features:**
- Eliminates external fragmentation
- Allows non-contiguous allocation
- May have internal fragmentation (last page)
- Uses page table for address translation

**Address Translation:**
- Logical address = (page number, offset)
- Physical address = (frame number, offset)
- Page table maps page numbers to frame numbers

### Diagram / Example

```
Paging Concept (페이징 개념)

Basic Idea:
┌─────────────────────────────────────────────────────────┐
│                                                          │
│  Logical Memory               Physical Memory            │
│  (Pages)                      (Frames)                   │
│                                                          │
│  ┌─────────┐                  ┌─────────┐               │
│  │ Page 0  │───────────┐      │Frame 0  │               │
│  ├─────────┤           │      ├─────────┤               │
│  │ Page 1  │─────────┐ └────► │Frame 1  │ ◄── Page 0   │
│  ├─────────┤         │        ├─────────┤               │
│  │ Page 2  │───────┐ │        │Frame 2  │               │
│  ├─────────┤       │ └──────► │Frame 3  │ ◄── Page 1   │
│  │ Page 3  │─────┐ │          ├─────────┤               │
│  └─────────┘     │ └────────► │Frame 4  │ ◄── Page 2   │
│                  │            ├─────────┤               │
│                  └──────────► │Frame 5  │ ◄── Page 3   │
│                               ├─────────┤               │
│                               │Frame 6  │               │
│                               ├─────────┤               │
│                               │Frame 7  │               │
│                               └─────────┘               │
│                                                          │
│  Pages can be anywhere in physical memory!              │
│  No external fragmentation!                              │
│                                                          │
└─────────────────────────────────────────────────────────┘


Page Table (페이지 테이블):
┌─────────────────────────────────────────────────────────┐
│                                                          │
│  Process P1 Page Table:                                 │
│                                                          │
│  Page Number │ Frame Number                             │
│  ────────────┼──────────────                            │
│       0      │      1                                   │
│       1      │      3                                   │
│       2      │      4                                   │
│       3      │      5                                   │
│                                                          │
│  Each process has its own page table                    │
│                                                          │
└─────────────────────────────────────────────────────────┘


Address Translation (주소 변환):
┌─────────────────────────────────────────────────────────┐
│                                                          │
│  Logical Address Format (32-bit, 4KB pages):           │
│                                                          │
│  ┌─────────────────────────────┬────────────────┐       │
│  │   Page Number (20 bits)     │ Offset (12 bits)│       │
│  └─────────────────────────────┴────────────────┘       │
│                                                          │
│  Page size = 4 KB = 2^12 bytes → 12-bit offset         │
│  Address space = 2^32 → 20-bit page number             │
│                                                          │
│                                                          │
│  Translation Process:                                    │
│                                                          │
│  Logical Address: 0x00002A3C                            │
│                                                          │
│  ┌───────────────────────────────────────────────────┐  │
│  │  0x00002A3C = 0000 0000 0000 0010 1010 0011 1100  │  │
│  │               ├─────────────────┤├────────────┤   │  │
│  │               Page Number: 2     Offset: 0xA3C   │  │
│  └───────────────────────────────────────────────────┘  │
│                                                          │
│  Page Table lookup:                                      │
│  Page 2 → Frame 4                                       │
│                                                          │
│  Physical Address:                                       │
│  Frame 4 * 4096 + 0xA3C = 0x4A3C                       │
│                                                          │
│  ┌──────────────────┬────────────────┐                 │
│  │ Frame Number: 4  │ Offset: 0xA3C  │                 │
│  └──────────────────┴────────────────┘                 │
│                                                          │
└─────────────────────────────────────────────────────────┘


Address Translation Hardware:
┌─────────────────────────────────────────────────────────┐
│                                                          │
│         CPU                                              │
│         │                                                │
│         ▼                                                │
│  ┌──────────────────┐                                   │
│  │ Logical Address  │                                   │
│  │ ┌──────┬───────┐ │                                   │
│  │ │Page #│Offset │ │                                   │
│  │ └──┬───┴───────┘ │                                   │
│  └────┼─────────────┘                                   │
│       │                                                  │
│       │    ┌─────────────────┐                          │
│       └───►│   Page Table    │                          │
│            │  ┌─────┬──────┐ │                          │
│            │  │  0  │  1   │ │                          │
│            │  │  1  │  3   │ │                          │
│            │  │  2  │  4   │◄┼── Page 2 maps to Frame 4│
│            │  │  3  │  5   │ │                          │
│            │  └─────┴──────┘ │                          │
│            └────────┬────────┘                          │
│                     │                                    │
│                     ▼                                    │
│  ┌──────────────────────────────┐                       │
│  │     Physical Address         │                       │
│  │ ┌───────────┬───────┐        │                       │
│  │ │Frame #: 4 │Offset │        │                       │
│  │ └───────────┴───────┘        │                       │
│  └──────────────────────────────┘                       │
│                     │                                    │
│                     ▼                                    │
│               Physical Memory                            │
│                                                          │
└─────────────────────────────────────────────────────────┘


Internal Fragmentation in Paging:
┌─────────────────────────────────────────────────────────┐
│                                                          │
│  Process needs 18,500 bytes                             │
│  Page size = 4,096 bytes                                │
│                                                          │
│  Pages needed: ceil(18500 / 4096) = 5 pages            │
│  Space allocated: 5 * 4096 = 20,480 bytes              │
│                                                          │
│  ┌────────┬────────┬────────┬────────┬────────┐        │
│  │ Page 0 │ Page 1 │ Page 2 │ Page 3 │ Page 4 │        │
│  │ 4096B  │ 4096B  │ 4096B  │ 4096B  │ 2212B  │        │
│  │ full   │ full   │ full   │ full   │ used   │        │
│  │        │        │        │        │ ▓▓▓▓▓▓ │        │
│  └────────┴────────┴────────┴────────┴────────┘        │
│                                      ▓▓ = 1884B wasted  │
│                                      (internal frag)    │
│                                                          │
│  Average internal fragmentation = PageSize / 2         │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

---

## Segmentation (세그멘테이션)

### Concept (개념)

**Segmentation (세그멘테이션)** divides memory into logical segments based on program structure.

**Segments typically include:**
- Code segment (text)
- Data segment
- Stack segment
- Heap segment

**Key Features:**
- Variable-size segments (unlike fixed-size pages)
- Matches programmer's view of memory
- Provides logical separation
- May cause external fragmentation

**Address Translation:**
- Logical address = (segment number, offset)
- Segment table maps segment number to (base, limit)

### Diagram / Example

```
Segmentation Concept (세그멘테이션 개념)

Program's Logical View:
┌─────────────────────────────────────────────────────────┐
│                                                          │
│  Program Structure:                                      │
│                                                          │
│  ┌─────────────────────────────────────────────┐        │
│  │           Main Program                       │        │
│  │  ┌─────────────────────────────────────┐    │        │
│  │  │     Code (Segment 0)                │    │        │
│  │  │     - function1()                   │    │        │
│  │  │     - function2()                   │    │        │
│  │  │     - main()                        │    │        │
│  │  └─────────────────────────────────────┘    │        │
│  │                                             │        │
│  │  ┌─────────────────────────────────────┐    │        │
│  │  │     Data (Segment 1)                │    │        │
│  │  │     - global variables              │    │        │
│  │  │     - static variables              │    │        │
│  │  └─────────────────────────────────────┘    │        │
│  │                                             │        │
│  │  ┌─────────────────────────────────────┐    │        │
│  │  │     Stack (Segment 2)               │    │        │
│  │  │     - local variables               │    │        │
│  │  │     - return addresses              │    │        │
│  │  └─────────────────────────────────────┘    │        │
│  │                                             │        │
│  │  ┌─────────────────────────────────────┐    │        │
│  │  │     Heap (Segment 3)                │    │        │
│  │  │     - dynamically allocated         │    │        │
│  │  └─────────────────────────────────────┘    │        │
│  │                                             │        │
│  └─────────────────────────────────────────────┘        │
│                                                          │
└─────────────────────────────────────────────────────────┘


Segment Table (세그먼트 테이블):
┌─────────────────────────────────────────────────────────┐
│                                                          │
│  Segment Table for Process P1:                          │
│                                                          │
│  Segment │ Base    │ Limit                              │
│  ────────┼─────────┼────────                            │
│     0    │  1400   │  1000   (Code)                     │
│     1    │  6300   │   400   (Data)                     │
│     2    │  4300   │   400   (Stack)                    │
│     3    │  3200   │  1100   (Heap)                     │
│                                                          │
│  Base: Starting address in physical memory              │
│  Limit: Length of segment                               │
│                                                          │
└─────────────────────────────────────────────────────────┘


Physical Memory Layout:
┌─────────────────────────────────────────────────────────┐
│                                                          │
│  Physical Memory:                                        │
│  ┌─────────────────────────────────────────────────┐    │
│  │     │          │       │       │      │         │    │
│  │     │ Seg 0    │       │Seg 3  │Seg 2 │  Seg 1  │    │
│  │     │ (Code)   │       │(Heap) │(Stack│  (Data) │    │
│  │     │          │       │       │      │         │    │
│  └─────────────────────────────────────────────────┘    │
│  0   1400      2400    3200    4300   4700    6300 6700 │
│                                                          │
│  Segments can be anywhere! (non-contiguous)            │
│                                                          │
└─────────────────────────────────────────────────────────┘


Address Translation:
┌─────────────────────────────────────────────────────────┐
│                                                          │
│  Logical Address: <Segment 2, Offset 53>               │
│                                                          │
│         CPU                     Segment Table           │
│  ┌──────────────┐              ┌───────────────┐        │
│  │ Seg: 2       │              │ 0 │1400│ 1000 │        │
│  │ Offset: 53   │─────────────►│ 1 │6300│  400 │        │
│  └──────────────┘              │ 2 │4300│  400 │◄─Look up
│                                │ 3 │3200│ 1100 │        │
│                                └───────────────┘        │
│                                        │                │
│                                        ▼                │
│                                Base = 4300              │
│                                Limit = 400              │
│                                                          │
│  Step 1: Check offset < limit                           │
│          53 < 400? YES (valid)                          │
│                                                          │
│  Step 2: Physical Address = Base + Offset              │
│          = 4300 + 53 = 4353                            │
│                                                          │
│  If offset >= limit → Segmentation Fault!              │
│                                                          │
└─────────────────────────────────────────────────────────┘


Segmentation Hardware:
┌─────────────────────────────────────────────────────────┐
│                                                          │
│         CPU                                              │
│         │                                                │
│         ▼                                                │
│  ┌──────────────────┐                                   │
│  │ Logical Address  │                                   │
│  │ ┌─────┬────────┐ │                                   │
│  │ │Seg# │ Offset │ │                                   │
│  │ └──┬──┴────┬───┘ │                                   │
│  └────┼───────┼─────┘                                   │
│       │       │                                          │
│       │       │           Segment Table                 │
│       │       │          ┌──────┬──────┐                │
│       │       │       s  │ base │limit │                │
│       └───────┼─────────►│      │      │                │
│               │          └──┬───┴──┬───┘                │
│               │             │      │                    │
│               │             ▼      ▼                    │
│               │          ┌────┐ ┌─────────┐            │
│               │          │base│ │   <     │            │
│               │          └──┬─┘ └────┬────┘            │
│               │             │        │                  │
│               │             │    yes │ no               │
│               │             │        │                  │
│               │             ▼        ▼                  │
│               │          ┌────┐   ┌──────┐             │
│               └─────────►│ +  │   │ TRAP │             │
│                          └──┬─┘   │ fault│             │
│                             │     └──────┘             │
│                             ▼                           │
│                     Physical Address                    │
│                                                          │
└─────────────────────────────────────────────────────────┘


Paging vs Segmentation Comparison:

┌────────────────────────────────────────────────────────┐
│                                                         │
│  Aspect          │ Paging        │ Segmentation        │
│  ────────────────┼───────────────┼─────────────────────│
│  Unit Size       │ Fixed         │ Variable            │
│  ────────────────┼───────────────┼─────────────────────│
│  View            │ Physical      │ Logical/Programmer  │
│  ────────────────┼───────────────┼─────────────────────│
│  External Frag   │ None          │ Yes                 │
│  ────────────────┼───────────────┼─────────────────────│
│  Internal Frag   │ Yes (last pg) │ None                │
│  ────────────────┼───────────────┼─────────────────────│
│  Table Entry     │ Frame number  │ Base + Limit        │
│  ────────────────┼───────────────┼─────────────────────│
│  Protection      │ Per page      │ Per segment (more   │
│                  │               │ meaningful)         │
│  ────────────────┼───────────────┼─────────────────────│
│  Sharing         │ Page level    │ Segment level       │
│                  │               │ (easier)            │
│                                                         │
└────────────────────────────────────────────────────────┘
```

---

## Summary (요약)

| Concept | Description |
|---------|-------------|
| Memory Hierarchy (메모리 계층) | Registers → Cache → RAM → Disk (speed vs size trade-off) |
| Address Binding (주소 바인딩) | Mapping logical addresses to physical addresses |
| Compile-Time Binding | Fixed addresses at compile time |
| Load-Time Binding | Addresses assigned when loading |
| Execution-Time Binding | Dynamic translation during runtime (modern) |
| Logical Address (논리적 주소) | Address generated by CPU (virtual address) |
| Physical Address (물리적 주소) | Actual address in RAM |
| MMU | Hardware that performs address translation |
| Contiguous Allocation | Process gets single contiguous block |
| Internal Fragmentation (내부 단편화) | Wasted space inside allocated block |
| External Fragmentation (외부 단편화) | Wasted space between blocks |
| Paging (페이징) | Fixed-size pages/frames, no external fragmentation |
| Segmentation (세그멘테이션) | Variable-size segments, logical separation |

---

## Key Terms (주요 용어)

- **Frame (프레임)**: Fixed-size block in physical memory
- **Page (페이지)**: Fixed-size block in logical memory
- **Page Table (페이지 테이블)**: Maps pages to frames
- **Segment Table (세그먼트 테이블)**: Maps segments to base+limit
- **Compaction (압축)**: Moving processes to create contiguous free space
- **Base Register**: Starting address of process in memory
- **Limit Register**: Size of process address space
