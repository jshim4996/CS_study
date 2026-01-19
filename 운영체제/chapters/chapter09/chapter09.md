# Chapter 09. Virtual Memory (가상 메모리)

> Virtual memory allows processes to use more memory than physically available by storing parts of processes on disk.

---

## Virtual Memory Concept (가상 메모리 개념)

### Concept (개념)

**Virtual Memory (가상 메모리)** is a memory management technique that creates an illusion of a very large main memory by using disk storage as an extension of RAM.

**Key Benefits:**
1. **Large Address Space**: Processes can be larger than physical memory
2. **Memory Isolation**: Each process has its own virtual address space
3. **Memory Sharing**: Multiple processes can share pages
4. **Efficient Memory Use**: Only needed parts loaded

**How It Works:**
- Process's virtual address space divided into pages
- Only some pages reside in physical memory
- Other pages stored on disk (swap space)
- Pages loaded on demand (demand paging)

### Diagram / Example

```
Virtual Memory Concept (가상 메모리 개념)

Basic Idea:
┌─────────────────────────────────────────────────────────┐
│                                                          │
│  Virtual Address Space          Physical Memory          │
│  (per process)                  (shared by all)         │
│                                                          │
│  ┌─────────────────┐           ┌─────────────────┐      │
│  │     4 GB        │           │    Physical     │      │
│  │  (or 64-bit:    │           │    Memory       │      │
│  │   huge!)        │           │    (e.g., 8GB)  │      │
│  │                 │           │                 │      │
│  │  Process sees   │◄─────────►│  Actual RAM     │      │
│  │  large, private │   MMU     │  (limited)      │      │
│  │  address space  │           │                 │      │
│  └─────────────────┘           └─────────────────┘      │
│           │                            │                 │
│           │                            │                 │
│           │                            │                 │
│           └────────────┬───────────────┘                │
│                        │                                 │
│                        ▼                                 │
│           ┌─────────────────────────┐                   │
│           │      Disk (Swap)        │                   │
│           │    (much larger)        │                   │
│           │    e.g., 100+ GB        │                   │
│           └─────────────────────────┘                   │
│                                                          │
│  Virtual Memory = Physical RAM + Swap Space             │
│                                                          │
└─────────────────────────────────────────────────────────┘


Process View vs Reality:
┌─────────────────────────────────────────────────────────┐
│                                                          │
│  What Process Thinks:        What Actually Happens:      │
│  ─────────────────────       ──────────────────────     │
│                                                          │
│  ┌─────────────────┐         Physical Memory:           │
│  │   My Memory     │         ┌─────────────────┐        │
│  │                 │         │ P1's page 0     │        │
│  │  Page 0 ──────────────────│ P2's page 3     │        │
│  │  Page 1         │    ┌────│ P3's page 1     │        │
│  │  Page 2 ──────────────┼───│ P1's page 2     │        │
│  │  Page 3         │    │    │ (some free)     │        │
│  │  Page 4 ───────────┐ │    └─────────────────┘        │
│  │  ...            │  │ │                               │
│  └─────────────────┘  │ │    Disk (Swap):              │
│                       │ │    ┌─────────────────┐        │
│  All pages seem       │ └────│ P1's page 1     │        │
│  available!           └──────│ P1's page 4     │        │
│                              │ P2's page 0     │        │
│                              │ ...             │        │
│                              └─────────────────┘        │
│                                                          │
└─────────────────────────────────────────────────────────┘


Virtual Memory Benefits:

┌────────────────────────────────────────────────────────┐
│                                                         │
│  1. Larger Programs Can Run                            │
│  ───────────────────────────                           │
│  ┌─────────────────────────────────────────────────┐   │
│  │  Physical RAM: 8 GB                              │   │
│  │                                                   │   │
│  │  Running simultaneously:                          │   │
│  │  • Chrome: 4 GB virtual                          │   │
│  │  • IDE: 3 GB virtual                             │   │
│  │  • Database: 10 GB virtual                       │   │
│  │  • OS + Others: 5 GB virtual                     │   │
│  │  ─────────────────────────                       │   │
│  │  Total: 22 GB virtual > 8 GB physical!           │   │
│  │                                                   │   │
│  │  Works because not all pages needed at once      │   │
│  └─────────────────────────────────────────────────┘   │
│                                                         │
│  2. Memory Protection                                  │
│  ────────────────────                                  │
│  ┌─────────────────────────────────────────────────┐   │
│  │  Process P1's virtual space:                     │   │
│  │  0x0000 - 0xFFFF (isolated)                      │   │
│  │                                                   │   │
│  │  Process P2's virtual space:                     │   │
│  │  0x0000 - 0xFFFF (isolated)                      │   │
│  │                                                   │   │
│  │  Same addresses, different physical locations!   │   │
│  │  P1 cannot access P2's memory (protected)        │   │
│  └─────────────────────────────────────────────────┘   │
│                                                         │
│  3. Memory Sharing                                     │
│  ─────────────────                                     │
│  ┌─────────────────────────────────────────────────┐   │
│  │  libc.so loaded once in physical memory          │   │
│  │  Mapped into multiple processes' virtual spaces  │   │
│  │                                                   │   │
│  │  Process 1:  [libc] ──┐                          │   │
│  │  Process 2:  [libc] ──┼──► Single physical copy │   │
│  │  Process 3:  [libc] ──┘                          │   │
│  │                                                   │   │
│  │  Saves memory! (Copy-on-Write for data)         │   │
│  └─────────────────────────────────────────────────┘   │
│                                                         │
└────────────────────────────────────────────────────────┘
```

---

## Demand Paging (요구 페이징)

### Concept (개념)

**Demand Paging (요구 페이징)** loads pages into memory only when they are needed (on demand), rather than loading the entire program at startup.

**Key Concepts:**
- **Lazy Loading**: Don't load until absolutely necessary
- **Valid-Invalid Bit**: Indicates if page is in memory
  - Valid (v): Page in memory
  - Invalid (i): Page not in memory or illegal access
- **Page Fault**: Occurs when accessing page not in memory

**Benefits:**
- Fast startup (no need to load entire program)
- Efficient memory use (only used pages loaded)
- Supports larger virtual address spaces

### Diagram / Example

```
Demand Paging (요구 페이징)

Page Table with Valid-Invalid Bit:
┌─────────────────────────────────────────────────────────┐
│                                                          │
│  Page Table:                        Physical Memory:     │
│  ┌───────┬────────┬─────┐          ┌─────────────────┐  │
│  │ Page  │ Frame  │ V/I │          │                 │  │
│  ├───────┼────────┼─────┤          │  Frame 0: P.0   │  │
│  │   0   │   0    │  v  │◄─────────│  Frame 1: P.2   │  │
│  │   1   │   -    │  i  │  ✗ disk  │  Frame 2: P.5   │  │
│  │   2   │   1    │  v  │◄─────────│  Frame 3: ...   │  │
│  │   3   │   -    │  i  │  ✗ disk  │                 │  │
│  │   4   │   -    │  i  │  ✗ disk  └─────────────────┘  │
│  │   5   │   2    │  v  │                               │
│  └───────┴────────┴─────┘                               │
│                                                          │
│  v = valid (in memory)                                  │
│  i = invalid (on disk or unused)                        │
│                                                          │
└─────────────────────────────────────────────────────────┘


Memory Access with Demand Paging:
┌─────────────────────────────────────────────────────────┐
│                                                          │
│  CPU wants to access address in Page 2:                 │
│                                                          │
│  Step 1: Check page table                               │
│          Page 2 → Frame 1, Valid bit = v               │
│                                                          │
│  Step 2: Access Frame 1 directly                        │
│          No page fault! (page is in memory)            │
│                                                          │
│  ┌─────┐    ┌──────────────┐    ┌─────────────────┐    │
│  │ CPU │───►│ Page Table   │───►│ Physical Memory │    │
│  │     │    │ P2→F1, valid │    │ Frame 1         │    │
│  └─────┘    └──────────────┘    └─────────────────┘    │
│                                                          │
└─────────────────────────────────────────────────────────┘


CPU wants to access address in Page 1:
┌─────────────────────────────────────────────────────────┐
│                                                          │
│  Step 1: Check page table                               │
│          Page 1 → Invalid bit = i                       │
│                                                          │
│  Step 2: PAGE FAULT! (page not in memory)              │
│          Trap to OS                                      │
│                                                          │
│  ┌─────┐    ┌──────────────┐                            │
│  │ CPU │───►│ Page Table   │───► Page Fault!           │
│  │     │    │ P1→invalid   │                            │
│  └─────┘    └──────────────┘                            │
│                                                          │
└─────────────────────────────────────────────────────────┘


Pure Demand Paging:
┌─────────────────────────────────────────────────────────┐
│                                                          │
│  Extreme case: Start with NO pages in memory            │
│                                                          │
│  Process startup:                                        │
│  ┌─────────────────────────────────────────────────┐    │
│  │  Page Table: All entries invalid                 │    │
│  │  ┌─────┬─────┬─────┬─────┬─────┬─────┐          │    │
│  │  │  i  │  i  │  i  │  i  │  i  │  i  │          │    │
│  │  └─────┴─────┴─────┴─────┴─────┴─────┘          │    │
│  └─────────────────────────────────────────────────┘    │
│                                                          │
│  First instruction fetch:                                │
│  → Page fault for code page                             │
│  → Load code page from disk                             │
│  → Execute instruction                                   │
│                                                          │
│  First data access:                                      │
│  → Page fault for data page                             │
│  → Load data page from disk                             │
│  → Access data                                           │
│                                                          │
│  After a while, most needed pages are in memory        │
│  (working set stabilizes)                               │
│                                                          │
└─────────────────────────────────────────────────────────┘


Hardware Support Required:
┌─────────────────────────────────────────────────────────┐
│                                                          │
│  1. Page Table with Valid/Invalid Bit                   │
│     Distinguishes pages in memory vs on disk            │
│                                                          │
│  2. Secondary Storage (Swap Space)                      │
│     Holds pages not currently in memory                 │
│                                                          │
│  3. Instruction Restart Capability                      │
│     After page fault handled, restart the instruction   │
│                                                          │
│  ┌─────────────────────────────────────────────────┐    │
│  │  Example: MOV A, [address]                       │    │
│  │                                                   │    │
│  │  1. Fetch instruction ← may cause page fault     │    │
│  │  2. Decode                                        │    │
│  │  3. Fetch operand ← may cause page fault         │    │
│  │  4. Execute                                       │    │
│  │  5. Store result ← may cause page fault          │    │
│  │                                                   │    │
│  │  If page fault at any step, must be able to     │    │
│  │  restart the ENTIRE instruction after loading    │    │
│  └─────────────────────────────────────────────────┘    │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

---

## Page Fault (페이지 폴트)

### Concept (개념)

A **Page Fault (페이지 폴트)** occurs when a process tries to access a page that is not currently in physical memory. The OS must handle this by loading the required page from disk.

**Page Fault Handling Steps:**
1. Trap to OS (page fault interrupt)
2. OS checks if reference is valid
3. Find a free frame (or evict a page)
4. Load page from disk into frame
5. Update page table
6. Restart the instruction

**Types of Page Faults:**
- **Minor (Soft) Page Fault**: Page in memory but not mapped (e.g., shared library)
- **Major (Hard) Page Fault**: Page must be loaded from disk

### Diagram / Example

```
Page Fault Handling (페이지 폴트 처리)

Page Fault Steps:
┌─────────────────────────────────────────────────────────┐
│                                                          │
│  Reference to page not in memory                        │
│         │                                                │
│         ▼                                                │
│  ┌────────────────┐                                     │
│  │ 1. TRAP to OS  │  (page fault interrupt)            │
│  │    (kernel)    │                                     │
│  └───────┬────────┘                                     │
│          │                                               │
│          ▼                                               │
│  ┌────────────────┐      ┌──────────────────────────┐   │
│  │ 2. Check if    │─────►│ Invalid address?         │   │
│  │    valid ref   │      │ → Terminate process      │   │
│  └───────┬────────┘      │ → Segmentation fault     │   │
│          │               └──────────────────────────┘   │
│          │ Valid reference                              │
│          ▼                                               │
│  ┌────────────────┐      ┌──────────────────────────┐   │
│  │ 3. Get free    │─────►│ No free frame?           │   │
│  │    frame       │      │ → Page replacement       │   │
│  └───────┬────────┘      │    (evict a page)        │   │
│          │               └──────────────────────────┘   │
│          │ Have free frame                              │
│          ▼                                               │
│  ┌────────────────┐                                     │
│  │ 4. Read page   │  (disk I/O - SLOW!)                │
│  │    from disk   │  Process blocked during I/O        │
│  └───────┬────────┘                                     │
│          │                                               │
│          ▼                                               │
│  ┌────────────────┐                                     │
│  │ 5. Update      │  Set valid bit, update frame #     │
│  │    page table  │                                     │
│  └───────┬────────┘                                     │
│          │                                               │
│          ▼                                               │
│  ┌────────────────┐                                     │
│  │ 6. Restart     │  Process resumes execution         │
│  │    instruction │                                     │
│  └────────────────┘                                     │
│                                                          │
└─────────────────────────────────────────────────────────┘


Detailed Page Fault Diagram:
┌─────────────────────────────────────────────────────────┐
│                                                          │
│       ┌───────┐                                          │
│       │  CPU  │                                          │
│       └───┬───┘                                          │
│           │ ① reference                                  │
│           ▼                                              │
│       ┌───────────────────┐                             │
│       │    Page Table     │                             │
│       │  ┌─────┬───────┐  │                             │
│       │  │ ... │   i   │◄─┼── ② page fault             │
│       │  └─────┴───────┘  │                             │
│       └───────────────────┘                             │
│               │                                          │
│               │ ③ trap to OS                            │
│               ▼                                          │
│       ┌───────────────────┐                             │
│       │   Operating       │                             │
│       │     System        │                             │
│       └───────┬───────────┘                             │
│               │ ④ disk I/O request                      │
│               ▼                                          │
│       ┌───────────────────┐     ┌───────────────────┐   │
│       │     Disk          │────►│   Free Frame      │   │
│       │   (swap space)    │     │  in Memory        │   │
│       └───────────────────┘     └─────────┬─────────┘   │
│                                           │              │
│       ⑤ load page into free frame         │              │
│                                           │              │
│       ┌───────────────────┐               │              │
│       │    Page Table     │◄──────────────┘              │
│       │  ┌─────┬───────┐  │  ⑥ update page table        │
│       │  │ ... │   v   │  │     (set valid bit)         │
│       │  └─────┴───────┘  │                             │
│       └───────────────────┘                             │
│               │                                          │
│               │ ⑦ restart instruction                   │
│               ▼                                          │
│       ┌───────┐                                          │
│       │  CPU  │  (continue execution)                   │
│       └───────┘                                          │
│                                                          │
└─────────────────────────────────────────────────────────┘


Page Fault Time Impact:
┌─────────────────────────────────────────────────────────┐
│                                                          │
│  Memory access time: ~100 nanoseconds                   │
│  Disk access time:   ~10 milliseconds                   │
│                                                          │
│  That's 100,000 times slower!                           │
│                                                          │
│  Effective Access Time (EAT):                           │
│  ────────────────────────────                           │
│  EAT = (1 - p) × memory_access + p × page_fault_time   │
│                                                          │
│  Where p = page fault probability                       │
│                                                          │
│  Example:                                                │
│  memory_access = 100 ns                                 │
│  page_fault_time = 10 ms = 10,000,000 ns               │
│  p = 0.001 (1 fault per 1000 accesses)                 │
│                                                          │
│  EAT = 0.999 × 100 + 0.001 × 10,000,000               │
│      = 99.9 + 10,000                                    │
│      = 10,099.9 ns                                      │
│                                                          │
│  100× slowdown with just 0.1% page fault rate!         │
│                                                          │
│  For good performance, need p < 0.0001 (1 in 10,000)   │
│                                                          │
└─────────────────────────────────────────────────────────┘


Page Fault Frequency Graph:
┌─────────────────────────────────────────────────────────┐
│                                                          │
│  Page Fault Rate                                         │
│       │                                                  │
│       │ ●                                                │
│       │  \                                               │
│       │   \                                              │
│       │    \                                             │
│       │     ●                                            │
│       │      \                                           │
│       │       \                                          │
│       │        ●                                         │
│       │         \_________●_______●______●____          │
│       │                                                  │
│       └──────────────────────────────────────► Frames   │
│             More frames = fewer page faults             │
│                                                          │
│  Eventually plateaus (working set fits in memory)       │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

---

## Page Replacement Algorithms (페이지 교체 알고리즘)

### Concept (개념)

When a page fault occurs and no free frames are available, the OS must **evict (replace)** an existing page. **Page Replacement Algorithms** decide which page to evict.

**Goal**: Minimize page fault rate

**Common Algorithms:**
| Algorithm | Korean | Description |
|-----------|--------|-------------|
| **FIFO** | 선입선출 | Replace oldest page |
| **LRU** | 최근 최소 사용 | Replace least recently used |
| **Optimal** | 최적 | Replace page used furthest in future |
| **Clock** | 클럭 | Approximation of LRU |

### Diagram / Example

```
Page Replacement Overview (페이지 교체 개요)

When Page Fault Occurs with No Free Frame:
┌─────────────────────────────────────────────────────────┐
│                                                          │
│  Physical Memory (3 frames, all full):                  │
│  ┌─────────┬─────────┬─────────┐                        │
│  │ Frame 0 │ Frame 1 │ Frame 2 │                        │
│  │ Page A  │ Page B  │ Page C  │                        │
│  └─────────┴─────────┴─────────┘                        │
│                                                          │
│  Need to load Page D → No free frame!                   │
│                                                          │
│  Must choose victim:                                     │
│  ┌─────────────────────────────────────────┐            │
│  │ Option 1: Evict A → Load D into Frame 0 │            │
│  │ Option 2: Evict B → Load D into Frame 1 │            │
│  │ Option 3: Evict C → Load D into Frame 2 │            │
│  └─────────────────────────────────────────┘            │
│                                                          │
│  Which choice minimizes future page faults?             │
│                                                          │
└─────────────────────────────────────────────────────────┘


FIFO (First-In, First-Out) Algorithm:
┌─────────────────────────────────────────────────────────┐
│                                                          │
│  Replace the OLDEST page (first one loaded)            │
│                                                          │
│  Reference string: 7, 0, 1, 2, 0, 3, 0, 4, 2, 3        │
│  Frames: 3                                               │
│                                                          │
│  Step │ Ref │ Frame 0 │ Frame 1 │ Frame 2 │ Fault?     │
│  ─────┼─────┼─────────┼─────────┼─────────┼─────────   │
│   1   │  7  │    7    │    -    │    -    │   Yes      │
│   2   │  0  │    7    │    0    │    -    │   Yes      │
│   3   │  1  │    7    │    0    │    1    │   Yes      │
│   4   │  2  │    2    │    0    │    1    │   Yes*     │
│   5   │  0  │    2    │    0    │    1    │   No       │
│   6   │  3  │    2    │    3    │    1    │   Yes*     │
│   7   │  0  │    2    │    3    │    0    │   Yes*     │
│   8   │  4  │    4    │    3    │    0    │   Yes*     │
│   9   │  2  │    4    │    2    │    0    │   Yes*     │
│  10   │  3  │    4    │    2    │    3    │   Yes*     │
│                                                          │
│  * = page replacement occurred                          │
│                                                          │
│  Total page faults: 9                                   │
│                                                          │
│  Visualization at step 4:                               │
│  ┌───────┬───────┬───────┐                              │
│  │   7   │   0   │   1   │  ← All full                 │
│  │ oldest│       │ newest│                              │
│  └───────┴───────┴───────┘                              │
│      ▲                                                   │
│      └── FIFO evicts 7 (oldest)                        │
│                                                          │
└─────────────────────────────────────────────────────────┘


Belady's Anomaly (벨라디의 모순):
┌─────────────────────────────────────────────────────────┐
│                                                          │
│  Counter-intuitive: More frames can cause MORE faults!  │
│                                                          │
│  Reference string: 1, 2, 3, 4, 1, 2, 5, 1, 2, 3, 4, 5  │
│                                                          │
│  With 3 frames: 9 page faults                           │
│  With 4 frames: 10 page faults  ← More faults!         │
│                                                          │
│  This only happens with FIFO, not LRU or Optimal       │
│                                                          │
└─────────────────────────────────────────────────────────┘


Optimal (OPT) Algorithm:
┌─────────────────────────────────────────────────────────┐
│                                                          │
│  Replace page that won't be used for LONGEST time       │
│  (Requires future knowledge - theoretical best)         │
│                                                          │
│  Reference string: 7, 0, 1, 2, 0, 3, 0, 4, 2, 3        │
│  Frames: 3                                               │
│                                                          │
│  Step │ Ref │ Frame 0 │ Frame 1 │ Frame 2 │ Fault?     │
│  ─────┼─────┼─────────┼─────────┼─────────┼─────────   │
│   1   │  7  │    7    │    -    │    -    │   Yes      │
│   2   │  0  │    7    │    0    │    -    │   Yes      │
│   3   │  1  │    7    │    0    │    1    │   Yes      │
│   4   │  2  │    2    │    0    │    1    │   Yes*     │
│   5   │  0  │    2    │    0    │    1    │   No       │
│   6   │  3  │    2    │    0    │    3    │   Yes*     │
│   7   │  0  │    2    │    0    │    3    │   No       │
│   8   │  4  │    2    │    4    │    3    │   Yes*     │
│   9   │  2  │    2    │    4    │    3    │   No       │
│  10   │  3  │    2    │    4    │    3    │   No       │
│                                                          │
│  Total page faults: 6 (optimal!)                        │
│                                                          │
│  At step 4: Which to replace? (7, 0, 1 in memory)      │
│  Future: 0, 3, 0, 4, 2, 3                               │
│  - Page 7: Never used again                             │
│  - Page 0: Used at step 5                               │
│  - Page 1: Used at step 6 (as 3... wait no)            │
│  → Replace 7 (won't be used)                            │
│                                                          │
│  Cannot implement in practice (need future knowledge)   │
│  Used as benchmark to compare other algorithms         │
│                                                          │
└─────────────────────────────────────────────────────────┘


LRU (Least Recently Used) Algorithm:
┌─────────────────────────────────────────────────────────┐
│                                                          │
│  Replace page that hasn't been used for longest time   │
│  (Approximates optimal using past behavior)            │
│                                                          │
│  Reference string: 7, 0, 1, 2, 0, 3, 0, 4, 2, 3        │
│  Frames: 3                                               │
│                                                          │
│  Step │ Ref │ Frame 0 │ Frame 1 │ Frame 2 │ Fault?     │
│  ─────┼─────┼─────────┼─────────┼─────────┼─────────   │
│   1   │  7  │    7    │    -    │    -    │   Yes      │
│   2   │  0  │    7    │    0    │    -    │   Yes      │
│   3   │  1  │    7    │    0    │    1    │   Yes      │
│   4   │  2  │    2    │    0    │    1    │   Yes*     │
│   5   │  0  │    2    │    0    │    1    │   No       │
│   6   │  3  │    2    │    0    │    3    │   Yes*     │
│   7   │  0  │    2    │    0    │    3    │   No       │
│   8   │  4  │    4    │    0    │    3    │   Yes*     │
│   9   │  2  │    4    │    0    │    2    │   Yes*     │
│  10   │  3  │    4    │    3    │    2    │   Yes*     │
│                                                          │
│  Total page faults: 8                                   │
│                                                          │
│  At step 4: Which is LRU? (7, 0, 1 in memory)          │
│  Access history: 7, 0, 1                                │
│  - Page 7: Last used at step 1 (oldest)                │
│  - Page 0: Last used at step 2                          │
│  - Page 1: Last used at step 3 (most recent)           │
│  → Replace 7 (least recently used)                      │
│                                                          │
│  LRU uses PAST to predict FUTURE                        │
│  (temporal locality principle)                          │
│                                                          │
└─────────────────────────────────────────────────────────┘


LRU Implementation Methods:
┌─────────────────────────────────────────────────────────┐
│                                                          │
│  1. Counter-based (카운터 방식):                        │
│  ─────────────────────────────                          │
│  Each page has a "last used" timestamp                  │
│  Replace page with smallest timestamp                   │
│                                                          │
│  Page Table:                                             │
│  ┌──────┬───────┬─────────────┐                         │
│  │ Page │ Frame │ Last Access │                         │
│  ├──────┼───────┼─────────────┤                         │
│  │  A   │   0   │    1000     │  ← Oldest              │
│  │  B   │   1   │    1050     │                         │
│  │  C   │   2   │    1100     │  ← Most recent         │
│  └──────┴───────┴─────────────┘                         │
│  Replace A (smallest timestamp)                         │
│                                                          │
│  Problem: Must search all pages to find minimum        │
│                                                          │
│                                                          │
│  2. Stack-based (스택 방식):                            │
│  ─────────────────────────────                          │
│  Maintain stack of page numbers                         │
│  When page referenced, move to top                      │
│  Bottom of stack = LRU page                             │
│                                                          │
│  Stack (most recent on top):                            │
│  ┌───┐                                                   │
│  │ C │  ← Most recently used                            │
│  ├───┤                                                   │
│  │ B │                                                   │
│  ├───┤                                                   │
│  │ A │  ← Least recently used (LRU)                    │
│  └───┘                                                   │
│                                                          │
│  Problem: Expensive to update stack on every access    │
│                                                          │
└─────────────────────────────────────────────────────────┘


Algorithm Comparison:
┌─────────────────────────────────────────────────────────┐
│                                                          │
│  Reference string: 7, 0, 1, 2, 0, 3, 0, 4, 2, 3        │
│  3 Frames                                                │
│                                                          │
│  ┌───────────┬─────────────┬────────────────────────┐   │
│  │ Algorithm │ Page Faults │ Notes                  │   │
│  ├───────────┼─────────────┼────────────────────────┤   │
│  │ FIFO      │      9      │ Simple, Belady's       │   │
│  │           │             │ anomaly possible       │   │
│  ├───────────┼─────────────┼────────────────────────┤   │
│  │ Optimal   │      6      │ Best possible,         │   │
│  │           │             │ requires future        │   │
│  ├───────────┼─────────────┼────────────────────────┤   │
│  │ LRU       │      8      │ Good approximation     │   │
│  │           │             │ of optimal             │   │
│  └───────────┴─────────────┴────────────────────────┘   │
│                                                          │
│  Optimal > LRU > FIFO (in terms of performance)        │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

---

## Thrashing (스래싱)

### Concept (개념)

**Thrashing (스래싱)** occurs when a system spends more time handling page faults than executing processes. It happens when processes don't have enough frames for their working set.

**Causes:**
- Too many processes competing for limited frames
- Each process has insufficient frames
- Constant page faults → constant disk I/O

**Effects:**
- CPU utilization drops dramatically
- System becomes unresponsive
- Disk I/O increases to 100%

**Solutions:**
- Reduce degree of multiprogramming
- Add more memory
- Use working set model

### Diagram / Example

```
Thrashing (스래싱)

What Happens During Thrashing:
┌─────────────────────────────────────────────────────────┐
│                                                          │
│  Normal Operation:                                       │
│  ┌─────────────────────────────────────────────────┐    │
│  │  Process: Execute → Execute → Execute → ...      │    │
│  │                                                   │    │
│  │  Occasional page fault (handled quickly)         │    │
│  │  CPU utilization high, system responsive         │    │
│  └─────────────────────────────────────────────────┘    │
│                                                          │
│  Thrashing:                                              │
│  ┌─────────────────────────────────────────────────┐    │
│  │  Process: Fault→Wait→Fault→Wait→Fault→Wait→...  │    │
│  │                                                   │    │
│  │  Constant page faults!                           │    │
│  │  CPU utilization near 0%, system frozen          │    │
│  └─────────────────────────────────────────────────┘    │
│                                                          │
└─────────────────────────────────────────────────────────┘


CPU Utilization vs Degree of Multiprogramming:
┌─────────────────────────────────────────────────────────┐
│                                                          │
│  CPU                                                     │
│  Utilization                                             │
│       │                                                  │
│  100% │            ●●●●●                                │
│       │         ●●●     ●●●                              │
│       │       ●●            ●                            │
│       │     ●●               ●                           │
│       │    ●                  ●                          │
│       │   ●                    \                         │
│       │  ●                      \                        │
│       │ ●                        \                       │
│       │●                          \ ← Thrashing!        │
│     0 │────────────────────────────\─────────────────   │
│       └──────────────────────────────────────────────►  │
│         Few                              Many            │
│                    Degree of Multiprogramming           │
│                    (number of processes)                │
│                                                          │
│  Sweet spot: Maximum CPU utilization                    │
│  After peak: Thrashing begins, CPU drops rapidly        │
│                                                          │
└─────────────────────────────────────────────────────────┘


Why Thrashing Occurs:
┌─────────────────────────────────────────────────────────┐
│                                                          │
│  Step 1: OS sees low CPU utilization                    │
│          "CPU is idle, let's add more processes!"       │
│                                                          │
│  Step 2: More processes means less frames per process  │
│          Each process has insufficient frames           │
│                                                          │
│  Step 3: Processes start page faulting more            │
│          Pages constantly swapped in/out               │
│                                                          │
│  Step 4: Disk becomes bottleneck (100% busy)           │
│          CPU waits for disk I/O                        │
│                                                          │
│  Step 5: OS sees low CPU utilization again!            │
│          "CPU is idle, let's add more processes!"      │
│                                                          │
│  Step 6: Even more page faults! System freezes         │
│                                                          │
│  Vicious cycle:                                          │
│  ┌──────────────────────────────────────────────────┐   │
│  │                                                    │   │
│  │     Low CPU util ──► Add processes                │   │
│  │           ▲                    │                   │   │
│  │           │                    ▼                   │   │
│  │     More page       Less frames per process      │   │
│  │     faults ◄─────────────────┘                   │   │
│  │                                                    │   │
│  └──────────────────────────────────────────────────┘   │
│                                                          │
└─────────────────────────────────────────────────────────┘


Thrashing Example:
┌─────────────────────────────────────────────────────────┐
│                                                          │
│  System: 100 frames total, 3 processes                  │
│                                                          │
│  Scenario A (Good):                                      │
│  ┌──────────────────────────────────────────────────┐   │
│  │  P1: 30 frames (needs 30)  ✓                      │   │
│  │  P2: 40 frames (needs 40)  ✓                      │   │
│  │  P3: 30 frames (needs 30)  ✓                      │   │
│  │                                                    │   │
│  │  Each process has enough frames                   │   │
│  │  Few page faults, good performance               │   │
│  └──────────────────────────────────────────────────┘   │
│                                                          │
│  Scenario B (Thrashing):                                │
│  ┌──────────────────────────────────────────────────┐   │
│  │  P1: 10 frames (needs 30)  ✗ 20 short!           │   │
│  │  P2: 10 frames (needs 40)  ✗ 30 short!           │   │
│  │  P3: 10 frames (needs 30)  ✗ 20 short!           │   │
│  │  P4: 10 frames (needs 20)  ✗ added too many!     │   │
│  │  P5: 10 frames (needs 25)  ✗                      │   │
│  │  ... more processes ...                           │   │
│  │                                                    │   │
│  │  Every process page faults constantly!           │   │
│  │  System grinds to a halt                         │   │
│  └──────────────────────────────────────────────────┘   │
│                                                          │
└─────────────────────────────────────────────────────────┘


Solutions to Thrashing:
┌─────────────────────────────────────────────────────────┐
│                                                          │
│  1. Local Replacement Policy                            │
│  ────────────────────────────                           │
│  Process can only replace its own pages                 │
│  Thrashing process doesn't affect others               │
│  But: thrashing process still runs poorly              │
│                                                          │
│  2. Working Set Model                                   │
│  ──────────────────────                                 │
│  Track which pages each process is using               │
│  Only run process if working set fits in memory        │
│  Suspend process if not enough frames                  │
│                                                          │
│  3. Page Fault Frequency (PFF)                         │
│  ─────────────────────────────                         │
│  Monitor page fault rate per process                   │
│  - Too high: Give more frames or suspend              │
│  - Too low: Can take away some frames                 │
│                                                          │
│  4. Add More Physical Memory                           │
│  ─────────────────────────────                         │
│  The ultimate solution (but costs money)              │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

---

## Working Set (작업 집합)

### Concept (개념)

The **Working Set (작업 집합)** is the set of pages a process is actively using during a given time interval. It's based on the principle of locality.

**Working Set Model:**
- **WSS (Working Set Size)**: Number of pages in working set
- **Window (Δ)**: Time interval for measuring working set
- If sum of all WSS > available frames → thrashing will occur

**Uses:**
- Determine minimum frames needed per process
- Decide when to load/suspend processes
- Prevent thrashing

### Diagram / Example

```
Working Set Model (작업 집합 모델)

Working Set Definition:
┌─────────────────────────────────────────────────────────┐
│                                                          │
│  Working Set = Pages referenced in last Δ time units   │
│                                                          │
│  Reference String:                                       │
│  ...2, 6, 1, 5, 7, 7, 7, 7, 5, 1, 6, 2, 3, 4, 1, 2...  │
│                        ▲                                 │
│                        │ Current time (t)               │
│                                                          │
│  Window Δ = 10 references                               │
│                                                          │
│  Working Set at time t:                                 │
│  Look back Δ=10 references: 7, 7, 7, 7, 5, 1, 6, 2, 3, 4│
│  Unique pages: {1, 2, 3, 4, 5, 6, 7}                   │
│                                                          │
│  WSS(t) = 7 pages                                       │
│                                                          │
└─────────────────────────────────────────────────────────┘


Working Set Window (Δ):
┌─────────────────────────────────────────────────────────┐
│                                                          │
│  Reference String (time →):                             │
│  ┌───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┐     │
│  │ 1 │ 2 │ 3 │ 2 │ 1 │ 4 │ 5 │ 4 │ 1 │ 2 │ 3 │ 5 │     │
│  └───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┘     │
│  t=1  2   3   4   5   6   7   8   9  10  11  12         │
│                                                          │
│  With Δ = 5:                                            │
│                                                          │
│  At t=5:  Window = [1,2,3,2,1]  WS = {1,2,3}  WSS = 3  │
│  At t=8:  Window = [1,4,5,4,1]  WS = {1,4,5}  WSS = 3  │
│  At t=12: Window = [1,2,3,5,?]  WS = {1,2,3,5} WSS = 4 │
│                                                          │
│                                                          │
│  Choosing Δ:                                             │
│  ┌─────────────────────────────────────────────────┐    │
│  │  Δ too small:                                    │    │
│  │  → Working set too small                        │    │
│  │  → Many page faults                             │    │
│  │                                                  │    │
│  │  Δ too large:                                    │    │
│  │  → Working set includes unused pages            │    │
│  │  → Wastes memory                                │    │
│  │                                                  │    │
│  │  Δ just right:                                   │    │
│  │  → Captures locality                            │    │
│  │  → Efficient memory use                         │    │
│  └─────────────────────────────────────────────────┘    │
│                                                          │
└─────────────────────────────────────────────────────────┘


Working Set and Thrashing Prevention:
┌─────────────────────────────────────────────────────────┐
│                                                          │
│  System has M frames total                              │
│                                                          │
│  Process │ Working Set Size (WSS)                       │
│  ────────┼────────────────────────                      │
│    P1    │        25                                    │
│    P2    │        30                                    │
│    P3    │        20                                    │
│    P4    │        35                                    │
│  ────────┴────────────────────────                      │
│  Total D = Σ WSS = 110 frames needed                   │
│                                                          │
│  Case 1: M = 128 frames                                 │
│  ┌─────────────────────────────────────────────────┐    │
│  │  D (110) < M (128)                               │    │
│  │  ✓ All processes can run without thrashing      │    │
│  │  ✓ Some free frames available                   │    │
│  └─────────────────────────────────────────────────┘    │
│                                                          │
│  Case 2: M = 80 frames                                  │
│  ┌─────────────────────────────────────────────────┐    │
│  │  D (110) > M (80)                                │    │
│  │  ✗ Thrashing will occur!                        │    │
│  │  Solution: Suspend one or more processes        │    │
│  │                                                  │    │
│  │  Suspend P4 (WSS=35):                           │    │
│  │  New D = 75 < M = 80  ✓                         │    │
│  └─────────────────────────────────────────────────┘    │
│                                                          │
└─────────────────────────────────────────────────────────┘


Page Fault Frequency (PFF) Approach:
┌─────────────────────────────────────────────────────────┐
│                                                          │
│  Monitor page fault rate for each process               │
│                                                          │
│  Page Fault                                              │
│  Rate                                                    │
│       │                                                  │
│       │  ─ ─ ─ ─ Upper threshold                        │
│       │     ●                                            │
│       │    ● ●     ← Too high! Give more frames        │
│       │   ●   ●                                          │
│       │        ●●●●●●●●  ← Acceptable range            │
│       │              ●●●●                                │
│       │  ─ ─ ─ ─ Lower threshold                        │
│       │                  ●●●  ← Too low! Take frames   │
│       │                                                  │
│       └──────────────────────────────────────────────►  │
│                         Time                             │
│                                                          │
│  Actions:                                                │
│  • Above upper threshold → Allocate more frames        │
│  • Below lower threshold → Deallocate some frames      │
│  • Can't allocate (none free) → Suspend process        │
│                                                          │
└─────────────────────────────────────────────────────────┘


Working Set Visualization:
┌─────────────────────────────────────────────────────────┐
│                                                          │
│  Program Behavior Over Time:                            │
│                                                          │
│  WSS                                                     │
│   │                                                      │
│   │    ┌───────┐              ┌───────────┐             │
│   │    │       │              │           │             │
│   │    │ Phase │              │  Phase    │             │
│   │    │   A   │   ┌──────┐   │    C      │             │
│   │    │       │   │Phase │   │           │             │
│   │    │       │   │  B   │   │           │             │
│   │────┴───────┴───┴──────┴───┴───────────┴─────────►   │
│                         Time                             │
│                                                          │
│  Phase A: Loop processing array (needs 50 pages)       │
│  Phase B: I/O waiting (needs few pages)                │
│  Phase C: Different computation (needs 80 pages)       │
│                                                          │
│  Working set changes as program moves between phases   │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

---

## Summary (요약)

| Concept | Description |
|---------|-------------|
| Virtual Memory (가상 메모리) | Technique using disk as extension of RAM |
| Demand Paging (요구 페이징) | Load pages only when needed |
| Page Fault (페이지 폴트) | Trap when accessing page not in memory |
| FIFO | Replace oldest page (simple but anomaly possible) |
| LRU | Replace least recently used page |
| Optimal | Replace page not used for longest future time |
| Thrashing (스래싱) | Excessive paging causing low CPU utilization |
| Working Set (작업 집합) | Set of pages actively used by process |
| Page Fault Frequency | Monitor fault rate to allocate frames |

---

## Key Terms (주요 용어)

- **Swap Space (스왑 공간)**: Disk area for storing pages not in memory
- **Valid-Invalid Bit**: Indicates if page is in physical memory
- **Victim Page (희생 페이지)**: Page chosen for replacement
- **Dirty Bit (수정 비트)**: Indicates if page was modified (needs write back)
- **Reference Bit (참조 비트)**: Indicates if page was recently accessed
- **Locality (지역성)**: Tendency to access nearby memory addresses
- **Degree of Multiprogramming**: Number of processes in memory
- **Page Replacement Policy**: Algorithm to choose victim page
