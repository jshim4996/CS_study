# Chapter 07. Deadlock (데드락)

> Deadlock occurs when processes wait indefinitely for resources held by each other, resulting in a system standstill.

---

## Deadlock Concept (데드락 개념)

### Concept (개념)

A **Deadlock (데드락, 교착 상태)** is a situation where two or more processes are waiting for each other to release resources, and none can proceed. It's a permanent blocking of a set of processes.

**Formal Definition:**
A set of processes is deadlocked if each process in the set is waiting for an event that only another process in the set can cause.

**Simple Example:**
- Process P1 holds Resource A, needs Resource B
- Process P2 holds Resource B, needs Resource A
- Neither can proceed - DEADLOCK!

**Real-world Analogy:**
Think of a narrow bridge where cars from both directions enter simultaneously. Neither can proceed or back up, creating a gridlock.

### Diagram / Example

```
Deadlock Visualization (데드락 시각화)

Simple Deadlock:
┌─────────────────────────────────────────────────────────┐
│                                                          │
│    Process P1                    Process P2              │
│    ┌─────────┐                  ┌─────────┐             │
│    │         │                  │         │             │
│    │  Holds  │◄────────────────►│  Holds  │             │
│    │   R1    │     wants        │   R2    │             │
│    │         │                  │         │             │
│    └─────────┘                  └─────────┘             │
│        │                            │                    │
│        │ wants                      │ wants              │
│        ▼                            ▼                    │
│    ┌─────────┐                  ┌─────────┐             │
│    │         │                  │         │             │
│    │   R2    │                  │   R1    │             │
│    │ (held   │                  │ (held   │             │
│    │  by P2) │                  │  by P1) │             │
│    └─────────┘                  └─────────┘             │
│                                                          │
│    P1 waits for P2, P2 waits for P1 → DEADLOCK!        │
│                                                          │
└─────────────────────────────────────────────────────────┘


Traffic Deadlock Analogy:

┌─────────────────────────────────────────────────────────┐
│                                                          │
│                    │ │ │ │ │                            │
│                    │ │▼│ │ │                            │
│                    │ │ │ │ │                            │
│  ─────────────     │ │ │ │ │     ─────────────          │
│  ◄── ◄── ◄── │     │ │ │ │ │     │ ──► ──► ──►          │
│  ─────────────     │ │ │ │ │     ─────────────          │
│                    │ │ │ │ │                            │
│              ▲     │ │ │ │ │     │                      │
│              │     │ │ │ │ │     ▼                      │
│                                                          │
│  Cars from all directions blocked at intersection       │
│  None can move → Traffic Deadlock!                      │
│                                                          │
└─────────────────────────────────────────────────────────┘


System Resource Deadlock:

┌─────────────────────────────────────────────────────────┐
│  Process States:                                         │
│                                                          │
│  P1: "I have Printer, waiting for Scanner..."           │
│  P2: "I have Scanner, waiting for Printer..."           │
│  P3: "I have Disk, waiting for Memory..."               │
│  P4: "I have Memory, waiting for Disk..."               │
│                                                          │
│                                                          │
│        P1 ◄───────────────► P2                          │
│       holds Printer    holds Scanner                     │
│       wants Scanner    wants Printer                     │
│            │                │                            │
│            └───── DEADLOCK ─┘                           │
│                                                          │
│        P3 ◄───────────────► P4                          │
│       holds Disk       holds Memory                      │
│       wants Memory     wants Disk                        │
│            │                │                            │
│            └───── DEADLOCK ─┘                           │
│                                                          │
└─────────────────────────────────────────────────────────┘


Deadlock vs Starvation vs Livelock:

┌────────────────────────────────────────────────────────┐
│  Deadlock (데드락):                                     │
│  ─────────────────                                      │
│  • Processes blocked forever                            │
│  • Circular wait condition                              │
│  • No progress possible                                 │
│  • Example: P1 waits for P2, P2 waits for P1           │
│                                                         │
│  Starvation (기아):                                     │
│  ────────────────                                       │
│  • Process waits indefinitely                           │
│  • Resources available but always given to others      │
│  • Progress possible for others                        │
│  • Example: Low priority process never scheduled       │
│                                                         │
│  Livelock (라이브락):                                   │
│  ───────────────                                        │
│  • Processes active but no progress                    │
│  • Constantly changing state                           │
│  • Like two people in hallway moving same direction   │
│  • Example: Both release and retry simultaneously     │
└────────────────────────────────────────────────────────┘
```

---

## Four Necessary Conditions (데드락의 4가지 필요 조건)

### Concept (개념)

A deadlock can occur only if ALL FOUR conditions hold simultaneously. These are called the **Coffman Conditions**.

| Condition | Korean | Description |
|-----------|--------|-------------|
| **Mutual Exclusion** | 상호 배제 | At least one resource must be non-shareable |
| **Hold and Wait** | 점유와 대기 | Process holding resource(s) can request more |
| **No Preemption** | 비선점 | Resources cannot be forcibly taken away |
| **Circular Wait** | 순환 대기 | Circular chain of processes waiting for resources |

**Key Insight:**
If we can prevent ANY ONE of these conditions, deadlock cannot occur. This is the basis for deadlock prevention strategies.

### Diagram / Example

```
Four Necessary Conditions (4가지 필요 조건)

1. Mutual Exclusion (상호 배제)
┌─────────────────────────────────────────────────────────┐
│                                                          │
│  Resource can only be held by ONE process at a time     │
│                                                          │
│       ┌──────────┐                                       │
│       │ Printer  │ ◄── Only one process can use        │
│       │          │                                       │
│       │  [P1]    │     P2, P3 must wait                 │
│       └──────────┘                                       │
│                                                          │
│  Shareable resources (read-only files) don't cause      │
│  deadlock for this condition.                           │
│                                                          │
└─────────────────────────────────────────────────────────┘

2. Hold and Wait (점유와 대기)
┌─────────────────────────────────────────────────────────┐
│                                                          │
│  Process holds some resources while waiting for others  │
│                                                          │
│       Process P1                                         │
│       ┌─────────────┐                                    │
│       │ Holding:    │                                    │
│       │   - R1      │                                    │
│       │   - R2      │                                    │
│       │             │                                    │
│       │ Waiting for:│                                    │
│       │   - R3      │ ◄── Won't release R1, R2         │
│       └─────────────┘                                    │
│                                                          │
└─────────────────────────────────────────────────────────┘

3. No Preemption (비선점)
┌─────────────────────────────────────────────────────────┐
│                                                          │
│  Resources cannot be forcibly taken from a process      │
│                                                          │
│       Process P1 holds R1                               │
│       ┌─────────────┐                                    │
│       │     R1      │                                    │
│       └─────────────┘                                    │
│              │                                           │
│              │  OS cannot force P1 to release R1        │
│              │  (until P1 voluntarily releases it)      │
│              ▼                                           │
│       P1 must explicitly release R1                     │
│                                                          │
└─────────────────────────────────────────────────────────┘

4. Circular Wait (순환 대기)
┌─────────────────────────────────────────────────────────┐
│                                                          │
│  Circular chain: each process waits for next in chain   │
│                                                          │
│       P1 ─────────────► P2                              │
│       ▲  waits for       │                              │
│       │                  │ waits for                    │
│       │                  ▼                              │
│       │                 P3                              │
│       │                  │                              │
│       │                  │ waits for                    │
│       │                  ▼                              │
│       └───────────────  P4                              │
│          waits for                                       │
│                                                          │
│  P1→P2→P3→P4→P1 (circular chain)                       │
│                                                          │
└─────────────────────────────────────────────────────────┘


All Four Conditions Together:

┌─────────────────────────────────────────────────────────┐
│                      DEADLOCK!                           │
│                                                          │
│     P1                              P2                   │
│  ┌──────┐                        ┌──────┐               │
│  │Holds │ ──── wants R2 ────►    │Holds │               │
│  │ R1   │                        │ R2   │               │
│  │      │ ◄─── wants R1 ────     │      │               │
│  └──────┘                        └──────┘               │
│                                                          │
│  ✓ Mutual Exclusion: R1, R2 non-shareable              │
│  ✓ Hold and Wait: P1 holds R1, waits for R2            │
│  ✓ No Preemption: Cannot take R1 from P1               │
│  ✓ Circular Wait: P1→P2→P1                             │
│                                                          │
│  All 4 conditions met → Deadlock occurred!              │
│                                                          │
└─────────────────────────────────────────────────────────┘


Breaking ANY Condition Prevents Deadlock:

┌────────────────────────────────────────────────────────┐
│                                                         │
│  Condition        │ Break it by...                     │
│  ──────────────────────────────────────────────────   │
│  Mutual Exclusion │ Use shareable resources (hard)    │
│  Hold and Wait    │ Request all at once               │
│  No Preemption    │ Allow resource preemption         │
│  Circular Wait    │ Order resources, request in order │
│                                                         │
└────────────────────────────────────────────────────────┘
```

---

## Resource Allocation Graph (자원 할당 그래프)

### Concept (개념)

A **Resource Allocation Graph (RAG, 자원 할당 그래프)** is a directed graph used to describe and detect deadlocks.

**Graph Components:**
- **Nodes:**
  - Process nodes (circles): P1, P2, ...
  - Resource nodes (rectangles): R1, R2, ...
  - Instance dots: Each dot represents one instance of a resource
- **Edges:**
  - Request edge (Pi → Rj): Process Pi is requesting resource Rj
  - Assignment edge (Rj → Pi): Resource Rj is assigned to process Pi

**Deadlock Detection with RAG:**
- If graph has NO cycle → No deadlock
- If graph has cycle:
  - Single instance per resource → Deadlock exists
  - Multiple instances per resource → Deadlock MAY exist

### Diagram / Example

```
Resource Allocation Graph (자원 할당 그래프)

Graph Notation:

┌────────────────────────────────────────────────────────┐
│  Process Node:           Resource Node:                 │
│  ┌───────┐              ┌─────────────┐                │
│  │       │              │  •  •  •    │                │
│  │  Pi   │              │     Rj      │                │
│  │       │              │  (3 units)  │                │
│  └───────┘              └─────────────┘                │
│                                                         │
│  Request Edge:          Assignment Edge:                │
│  Pi ──────────► Rj      Rj ──────────► Pi              │
│  (Pi wants Rj)          (Rj allocated to Pi)           │
└────────────────────────────────────────────────────────┘


Example 1: No Deadlock

┌────────────────────────────────────────────────────────┐
│                                                         │
│      ┌────┐                ┌────┐                      │
│      │ P1 │                │ P2 │                      │
│      └──┬─┘                └─┬──┘                      │
│         │                    │                         │
│         │  request           │  request                │
│         ▼                    ▼                         │
│      ┌──────┐            ┌──────┐                      │
│      │ • R1 │            │ • R2 │                      │
│      └──────┘            └──────┘                      │
│                                                         │
│  No cycle → No Deadlock                                │
│  P1 and P2 can get their resources                     │
│                                                         │
└────────────────────────────────────────────────────────┘


Example 2: Deadlock (Single Instance)

┌────────────────────────────────────────────────────────┐
│                                                         │
│      ┌────┐                ┌────┐                      │
│      │ P1 │◄───────────────│ R2 │ (R2 assigned to P1) │
│      └──┬─┘                └──▲─┘                      │
│         │                     │                        │
│         │ request             │ assigned               │
│         ▼                     │                        │
│      ┌──────┐             ┌────┐                       │
│      │ • R1 │─────────────►│ P2 │ (R1 assigned to P2) │
│      └──────┘             └──┬─┘                       │
│         ▲                    │                         │
│         │                    │ request                 │
│         │                    ▼                         │
│         └────────────────────┘                         │
│                                                         │
│  Cycle: P1 → R1 → P2 → R2 → P1                        │
│  Single instance per resource → DEADLOCK!              │
│                                                         │
└────────────────────────────────────────────────────────┘


Example 3: Cycle but No Deadlock (Multiple Instances)

┌────────────────────────────────────────────────────────┐
│                                                         │
│      ┌────┐         ┌────┐         ┌────┐             │
│      │ P1 │         │ P2 │         │ P3 │             │
│      └─┬──┘         └─┬──┘         └─┬──┘             │
│        │              │              │                 │
│        │      ┌───────┴───────┐      │                 │
│        ▼      ▼               ▼      ▼                 │
│      ┌───────────┐       ┌───────────┐                │
│      │ •  •  R1  │       │ •  •  R2  │                │
│      └─────┬─────┘       └─────┬─────┘                │
│            │                   │                       │
│            │      ┌────────────┘                       │
│            ▼      ▼                                    │
│      ┌────┐  ┌────┐                                   │
│      │ P1 │  │ P3 │                                   │
│      └────┘  └────┘                                   │
│                                                         │
│  R1 has 2 instances, R2 has 2 instances                │
│  P1: holds R1(1), requests R2                          │
│  P2: holds R1(1), R2(1), requests nothing              │
│  P3: holds R2(1), requests R1                          │
│                                                         │
│  Cycle exists: P1 → R2 → P3 → R1 → P1                 │
│  But P2 can release R1 or R2, breaking the cycle!     │
│  → NO DEADLOCK                                         │
│                                                         │
└────────────────────────────────────────────────────────┘


RAG Summary:

┌────────────────────────────────────────────────────────┐
│                                                         │
│  Cycle in RAG?                                          │
│        │                                                │
│        ├── No  ────────► No Deadlock                   │
│        │                                                │
│        └── Yes ───┐                                     │
│                   │                                     │
│                   ▼                                     │
│        Resource Instances?                              │
│               │                                         │
│               ├── Single ────► Deadlock EXISTS         │
│               │                                         │
│               └── Multiple ──► Deadlock MAY exist      │
│                                (need further analysis)  │
│                                                         │
└────────────────────────────────────────────────────────┘
```

---

## Deadlock Prevention (데드락 예방)

### Concept (개념)

**Deadlock Prevention (데드락 예방)** ensures that at least one of the four necessary conditions cannot hold. This is a proactive approach that prevents deadlock from ever occurring.

**Strategies:**

| Condition | Prevention Method | Drawbacks |
|-----------|------------------|-----------|
| Mutual Exclusion | Make resources shareable | Not always possible |
| Hold and Wait | Request all resources at once | Low utilization, starvation |
| No Preemption | Allow preemption | May cause rollback, data loss |
| Circular Wait | Impose resource ordering | Inconvenient, limits flexibility |

### Diagram / Example

```
Deadlock Prevention Strategies (데드락 예방 전략)

1. Breaking Mutual Exclusion (상호 배제 깨기)
┌─────────────────────────────────────────────────────────┐
│                                                          │
│  Make resources shareable where possible                │
│                                                          │
│  Read-only files:         Printers:                     │
│  ┌────────────────┐       ┌────────────────┐            │
│  │    File.txt    │       │    Printer     │            │
│  │                │       │                │            │
│  │  P1 reads ───► │       │ P1 uses ─────► │            │
│  │  P2 reads ───► │       │ P2 waits ✗     │            │
│  │  P3 reads ───► │       │                │            │
│  │                │       │                │            │
│  │  (shareable)   │       │ (not shareable)│            │
│  └────────────────┘       └────────────────┘            │
│                                                          │
│  Limitation: Many resources are inherently non-shareable│
│                                                          │
└─────────────────────────────────────────────────────────┘


2. Breaking Hold and Wait (점유와 대기 깨기)
┌─────────────────────────────────────────────────────────┐
│                                                          │
│  Method A: Request ALL resources at once                │
│  ────────────────────────────────────────               │
│                                                          │
│  Process P1 needs: Printer, Scanner, Disk               │
│                                                          │
│  Before:                    After:                       │
│  ┌──────────────────┐      ┌──────────────────┐         │
│  │ Get Printer      │      │ Request ALL:     │         │
│  │ Get Scanner      │  ──► │ Printer+Scanner  │         │
│  │ Get Disk         │      │ +Disk at once    │         │
│  └──────────────────┘      └──────────────────┘         │
│                                                          │
│  If any unavailable → wait without holding anything     │
│                                                          │
│  Method B: Release before requesting                    │
│  ───────────────────────────────────                    │
│  If holding resources and need more:                    │
│  1. Release all currently held resources                │
│  2. Request all resources again (old + new)             │
│                                                          │
│  Drawbacks:                                              │
│  • Low resource utilization                              │
│  • Possible starvation (may never get all at once)      │
│  • Process may not know what it needs in advance        │
│                                                          │
└─────────────────────────────────────────────────────────┘


3. Breaking No Preemption (비선점 깨기)
┌─────────────────────────────────────────────────────────┐
│                                                          │
│  Allow OS to take resources from a process              │
│                                                          │
│  Scenario:                                               │
│  ┌───────────────────────────────────────────────────┐  │
│  │  P1 holds R1, requests R2 (held by P2)            │  │
│  │                                                    │  │
│  │  Option 1: Preempt P1's resources                 │  │
│  │  ───────────────────────────────────              │  │
│  │  P1 releases R1, waits for R1 + R2                │  │
│  │  P2 can now get R1 if needed                      │  │
│  │                                                    │  │
│  │  Option 2: Preempt P2's resources                 │  │
│  │  ───────────────────────────────────              │  │
│  │  If P2 is also waiting, take R2 from P2           │  │
│  │  Give R2 to P1                                    │  │
│  └───────────────────────────────────────────────────┘  │
│                                                          │
│  Works well for: CPU, Memory (can save state)           │
│  Not suitable for: Printers, Tape drives                │
│                                                          │
│  Drawback: Requires ability to save/restore state       │
│                                                          │
└─────────────────────────────────────────────────────────┘


4. Breaking Circular Wait (순환 대기 깨기)
┌─────────────────────────────────────────────────────────┐
│                                                          │
│  Impose a total ordering on all resource types          │
│  Processes must request resources in increasing order   │
│                                                          │
│  Resource Ordering:                                      │
│  ┌─────────────────────────────────────────────────┐    │
│  │  R1 (Scanner)     < order 1                     │    │
│  │  R2 (Printer)     < order 2                     │    │
│  │  R3 (Disk)        < order 3                     │    │
│  │  R4 (Tape Drive)  < order 4                     │    │
│  └─────────────────────────────────────────────────┘    │
│                                                          │
│  Rule: Can only request Rj if j > all currently held    │
│                                                          │
│  Valid Request Sequence:                                 │
│  ┌────────────────────────────────────────┐             │
│  │  P1: Request R1 ✓                      │             │
│  │  P1: Request R3 ✓ (3 > 1)              │             │
│  │  P1: Request R2 ✗ (2 < 3, NOT allowed) │             │
│  └────────────────────────────────────────┘             │
│                                                          │
│  Why this prevents circular wait:                        │
│  ┌────────────────────────────────────────────────────┐ │
│  │  Assume circular wait: P1→R1→P2→R2→...→Pn→Rn→P1  │ │
│  │                                                     │ │
│  │  P1 holds R_a, wants R_b (so b > a)               │ │
│  │  P2 holds R_b, wants R_c (so c > b)               │ │
│  │  ...                                               │ │
│  │  Pn holds R_x, wants R_a (so a > x)               │ │
│  │                                                     │ │
│  │  This means: a < b < c < ... < x < a              │ │
│  │  CONTRADICTION! (a cannot be less than itself)    │ │
│  │  Therefore, circular wait cannot exist.            │ │
│  └────────────────────────────────────────────────────┘ │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

---

## Deadlock Avoidance (데드락 회피)

### Concept (개념)

**Deadlock Avoidance (데드락 회피)** allows the system to avoid deadlock by making careful resource allocation decisions. The system uses additional information about how resources will be requested.

**Key Concept - Safe State:**
- A state is **safe** if there exists a sequence of all processes that can complete without deadlock
- A state is **unsafe** if no such sequence exists
- Unsafe state MAY lead to deadlock (not guaranteed)

**Approach:**
- Process must declare maximum resources it may need
- System grants request only if it remains in a safe state
- If granting would lead to unsafe state, process must wait

### Diagram / Example

```
Safe State vs Unsafe State (안전 상태 vs 불안전 상태)

Safe State Example:
┌─────────────────────────────────────────────────────────┐
│  Total Resources: 12 tape drives                         │
│                                                          │
│  Process │ Maximum Need │ Current Hold │ Can Finish?    │
│  ────────┼──────────────┼──────────────┼───────────────  │
│    P0    │      10      │      5       │                 │
│    P1    │       4      │      2       │                 │
│    P2    │       9      │      2       │                 │
│  ────────┴──────────────┴──────────────┴───────────────  │
│  Total held: 9        Available: 3                       │
│                                                          │
│  Safe Sequence: <P1, P0, P2>                            │
│                                                          │
│  Step 1: Give 2 to P1 (needs 4-2=2, have 3)            │
│          P1 finishes, releases 4 → Available: 5        │
│                                                          │
│  Step 2: Give 5 to P0 (needs 10-5=5, have 5)           │
│          P0 finishes, releases 10 → Available: 12      │
│                                                          │
│  Step 3: Give 7 to P2 (needs 9-2=7, have 12)           │
│          P2 finishes → ALL COMPLETE!                    │
│                                                          │
│  SAFE STATE: All processes can finish                   │
│                                                          │
└─────────────────────────────────────────────────────────┘


Unsafe State Example:
┌─────────────────────────────────────────────────────────┐
│  Total Resources: 12 tape drives                         │
│                                                          │
│  Process │ Maximum Need │ Current Hold │                │
│  ────────┼──────────────┼──────────────┤                │
│    P0    │      10      │      5       │                │
│    P1    │       4      │      2       │                │
│    P2    │       9      │      3       │  ◄── +1 more   │
│  ────────┴──────────────┴──────────────┘                │
│  Total held: 10       Available: 2                       │
│                                                          │
│  Try to find safe sequence:                              │
│                                                          │
│  P0: needs 5 more (have 2) ✗ Cannot finish              │
│  P1: needs 2 more (have 2) ✓ Can finish!                │
│                                                          │
│  After P1: Available = 2 + 2 = 4                        │
│                                                          │
│  P0: needs 5 (have 4) ✗ Cannot finish                   │
│  P2: needs 6 (have 4) ✗ Cannot finish                   │
│                                                          │
│  NO SAFE SEQUENCE! → UNSAFE STATE                       │
│  (May lead to deadlock)                                  │
│                                                          │
└─────────────────────────────────────────────────────────┘


State Space Diagram:

┌─────────────────────────────────────────────────────────┐
│                                                          │
│  ┌─────────────────────────────────────────────────┐    │
│  │                 ALL STATES                       │    │
│  │                                                   │    │
│  │   ┌───────────────────────────────────────────┐  │    │
│  │   │             SAFE STATES                   │  │    │
│  │   │                                           │  │    │
│  │   │   System operates here                    │  │    │
│  │   │   (all processes can complete)            │  │    │
│  │   │                                           │  │    │
│  │   └───────────────────────────────────────────┘  │    │
│  │                      │                            │    │
│  │                      │ Entering unsafe zone      │    │
│  │                      ▼                            │    │
│  │   ┌───────────────────────────────────────────┐  │    │
│  │   │            UNSAFE STATES                  │  │    │
│  │   │                                           │  │    │
│  │   │   ┌─────────────────────────────────┐    │  │    │
│  │   │   │         DEADLOCK                 │    │  │    │
│  │   │   │                                  │    │  │    │
│  │   │   │   No process can proceed        │    │  │    │
│  │   │   └─────────────────────────────────┘    │  │    │
│  │   │                                           │  │    │
│  │   │   May or may not reach deadlock          │  │    │
│  │   └───────────────────────────────────────────┘  │    │
│  │                                                   │    │
│  └─────────────────────────────────────────────────┘    │
│                                                          │
│  Avoidance: Never leave safe states!                    │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

---

## Banker's Algorithm (은행원 알고리즘)

### Concept (개념)

The **Banker's Algorithm** is the most famous deadlock avoidance algorithm. Named after the way a banker allocates limited funds to customers.

**Data Structures:**
Let n = number of processes, m = number of resource types

- `Available[m]`: Available instances of each resource
- `Max[n][m]`: Maximum demand of each process
- `Allocation[n][m]`: Currently allocated to each process
- `Need[n][m]`: Remaining need (Need = Max - Allocation)

**Algorithm Components:**
1. **Safety Algorithm**: Check if current state is safe
2. **Resource-Request Algorithm**: Decide if request can be granted

### Diagram / Example

```
Banker's Algorithm (은행원 알고리즘)

Data Structures:
┌─────────────────────────────────────────────────────────┐
│                                                          │
│  Given: n processes, m resource types                   │
│                                                          │
│  Available[m]:      [A, B, C]                           │
│                     Available instances of each type    │
│                                                          │
│  Max[n][m]:         Process │  A   B   C                │
│                     ────────┼──────────────             │
│                        P0   │  7   5   3                │
│                        P1   │  3   2   2                │
│                        P2   │  9   0   2                │
│                        P3   │  2   2   2                │
│                        P4   │  4   3   3                │
│                     (Maximum each process might need)   │
│                                                          │
│  Allocation[n][m]:  Process │  A   B   C                │
│                     ────────┼──────────────             │
│                        P0   │  0   1   0                │
│                        P1   │  2   0   0                │
│                        P2   │  3   0   2                │
│                        P3   │  2   1   1                │
│                        P4   │  0   0   2                │
│                     (Currently allocated)               │
│                                                          │
│  Need[n][m] = Max - Allocation:                         │
│                     Process │  A   B   C                │
│                     ────────┼──────────────             │
│                        P0   │  7   4   3                │
│                        P1   │  1   2   2                │
│                        P2   │  6   0   0                │
│                        P3   │  0   1   1                │
│                        P4   │  4   3   1                │
│                     (Still needs)                       │
│                                                          │
│  Available = Total - sum(Allocation)                    │
│            = [10,5,7] - [7,2,5] = [3,3,2]              │
│                                                          │
└─────────────────────────────────────────────────────────┘


Safety Algorithm (안전 알고리즘):

┌─────────────────────────────────────────────────────────┐
│  1. Let Work = Available                                │
│     Let Finish[n] = false for all processes            │
│                                                          │
│  2. Find process Pi such that:                          │
│     a) Finish[i] == false                               │
│     b) Need[i] <= Work                                  │
│                                                          │
│     If no such Pi exists, go to step 4                  │
│                                                          │
│  3. Work = Work + Allocation[i]                         │
│     Finish[i] = true                                    │
│     Go to step 2                                        │
│                                                          │
│  4. If Finish[i] == true for all i,                     │
│     system is in SAFE state                             │
│                                                          │
└─────────────────────────────────────────────────────────┘


Safety Algorithm Example:

┌─────────────────────────────────────────────────────────┐
│  Initial: Work = [3, 3, 2], Finish = [F, F, F, F, F]   │
│                                                          │
│  Iteration 1: Find Pi where Need[i] <= Work            │
│  ──────────────────────────────────────────            │
│  P0: Need[7,4,3] <= [3,3,2]? NO (7>3)                  │
│  P1: Need[1,2,2] <= [3,3,2]? YES!                      │
│                                                          │
│  Execute P1:                                             │
│  Work = [3,3,2] + Allocation[1] = [3,3,2] + [2,0,0]    │
│       = [5,3,2]                                         │
│  Finish = [F, T, F, F, F]                               │
│                                                          │
│  Iteration 2:                                            │
│  ──────────────                                          │
│  P0: Need[7,4,3] <= [5,3,2]? NO                         │
│  P2: Need[6,0,0] <= [5,3,2]? NO (6>5)                  │
│  P3: Need[0,1,1] <= [5,3,2]? YES!                      │
│                                                          │
│  Execute P3:                                             │
│  Work = [5,3,2] + [2,1,1] = [7,4,3]                    │
│  Finish = [F, T, F, T, F]                               │
│                                                          │
│  Iteration 3:                                            │
│  ──────────────                                          │
│  P0: Need[7,4,3] <= [7,4,3]? YES!                      │
│                                                          │
│  Execute P0:                                             │
│  Work = [7,4,3] + [0,1,0] = [7,5,3]                    │
│  Finish = [T, T, F, T, F]                               │
│                                                          │
│  Iteration 4:                                            │
│  ──────────────                                          │
│  P2: Need[6,0,0] <= [7,5,3]? YES!                      │
│                                                          │
│  Execute P2:                                             │
│  Work = [7,5,3] + [3,0,2] = [10,5,5]                   │
│  Finish = [T, T, T, T, F]                               │
│                                                          │
│  Iteration 5:                                            │
│  ──────────────                                          │
│  P4: Need[4,3,1] <= [10,5,5]? YES!                     │
│                                                          │
│  Execute P4:                                             │
│  Work = [10,5,5] + [0,0,2] = [10,5,7]                  │
│  Finish = [T, T, T, T, T]                               │
│                                                          │
│  All Finish = True → SAFE STATE!                        │
│  Safe Sequence: <P1, P3, P0, P2, P4>                   │
│                                                          │
└─────────────────────────────────────────────────────────┘


Resource-Request Algorithm (자원 요청 알고리즘):

┌─────────────────────────────────────────────────────────┐
│  Process Pi requests Request[i]                         │
│                                                          │
│  1. If Request[i] > Need[i]                             │
│        → Error! Exceeded maximum claim                  │
│                                                          │
│  2. If Request[i] > Available                           │
│        → Pi must wait (resources not available)         │
│                                                          │
│  3. Pretend to allocate:                                │
│     Available = Available - Request[i]                  │
│     Allocation[i] = Allocation[i] + Request[i]         │
│     Need[i] = Need[i] - Request[i]                     │
│                                                          │
│  4. Run Safety Algorithm:                               │
│     If SAFE → Grant request                            │
│     If UNSAFE → Deny request, Pi waits                 │
│                 (restore old values)                    │
│                                                          │
└─────────────────────────────────────────────────────────┘


Request Example: P1 requests (1, 0, 2)

┌─────────────────────────────────────────────────────────┐
│  Request[1] = (1, 0, 2)                                 │
│                                                          │
│  Step 1: Request <= Need[1]?                            │
│          (1,0,2) <= (1,2,2)? YES                        │
│                                                          │
│  Step 2: Request <= Available?                          │
│          (1,0,2) <= (3,3,2)? YES                        │
│                                                          │
│  Step 3: Pretend to allocate:                           │
│          Available = (3,3,2) - (1,0,2) = (2,3,0)       │
│          Allocation[1] = (2,0,0) + (1,0,2) = (3,0,2)   │
│          Need[1] = (1,2,2) - (1,0,2) = (0,2,0)         │
│                                                          │
│  Step 4: Safety check with new state...                 │
│          (Run safety algorithm)                         │
│                                                          │
│          Result: Still SAFE → GRANT REQUEST!           │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

---

## Deadlock Detection and Recovery (데드락 탐지 및 복구)

### Concept (개념)

**Deadlock Detection (데드락 탐지)** allows deadlock to occur, but the system periodically checks for deadlock and recovers when detected.

**Detection Approaches:**
1. **Single Instance Resources**: Use wait-for graph (simplified RAG)
2. **Multiple Instance Resources**: Use detection algorithm (similar to safety algorithm)

**Recovery Options:**
1. **Process Termination**: Kill process(es) to break deadlock
2. **Resource Preemption**: Take resources from process(es)

### Diagram / Example

```
Deadlock Detection (데드락 탐지)

Wait-For Graph (Single Instance Resources):
┌─────────────────────────────────────────────────────────┐
│                                                          │
│  Resource Allocation Graph:        Wait-For Graph:       │
│                                                          │
│      ┌───┐    ┌───┐                  ┌───┐              │
│      │P1 │───►│R1 │                  │P1 │              │
│      └───┘    └─┬─┘                  └─┬─┘              │
│                 │                      │                 │
│                 ▼                      │ waits for       │
│              ┌───┐                     ▼                 │
│              │P2 │                  ┌───┐               │
│              └─┬─┘                  │P2 │               │
│                │                    └─┬─┘               │
│                ▼                      │ waits for       │
│      ┌───┐    ┌───┐                   ▼                 │
│      │R2 │◄───│P2 │                ┌───┐               │
│      └─┬─┘    └───┘                │P3 │               │
│        │                           └─┬─┘               │
│        ▼                             │ waits for       │
│      ┌───┐                           ▼                 │
│      │P3 │─────────────────────►  ┌───┐               │
│      └───┘     (wants R1)         │P1 │               │
│                                   └───┘               │
│                                                          │
│  Remove resource nodes, add edge Pi→Pj if Pi           │
│  waits for a resource held by Pj                        │
│                                                          │
│  Cycle in wait-for graph → DEADLOCK!                   │
│                                                          │
└─────────────────────────────────────────────────────────┘


Detection Algorithm (Multiple Instances):
┌─────────────────────────────────────────────────────────┐
│  Similar to safety algorithm but:                        │
│  - Uses current request instead of maximum need         │
│  - Optimistically assumes processes not requesting      │
│    will finish and release resources                    │
│                                                          │
│  Data Structures:                                        │
│  Available[m]:    Available resources                   │
│  Allocation[n][m]: Currently allocated                  │
│  Request[n][m]:   Current requests (not max need!)     │
│                                                          │
│  Algorithm:                                              │
│  1. Work = Available                                    │
│     Finish[i] = (Allocation[i] == 0)                   │
│                                                          │
│  2. Find Pi where Finish[i]==false and Request[i]<=Work│
│     If none found, go to step 4                         │
│                                                          │
│  3. Work = Work + Allocation[i]                         │
│     Finish[i] = true                                    │
│     Go to step 2                                        │
│                                                          │
│  4. If any Finish[i] == false → DEADLOCK!              │
│     Processes with Finish[i]==false are deadlocked     │
│                                                          │
└─────────────────────────────────────────────────────────┘


Detection Example:
┌─────────────────────────────────────────────────────────┐
│                                                          │
│  Allocation:         Request:          Available:        │
│     A  B  C             A  B  C           A  B  C        │
│  P0 0  1  0          P0 0  0  0           0  0  0        │
│  P1 2  0  0          P1 2  0  2                          │
│  P2 3  0  3          P2 0  0  0                          │
│  P3 2  1  1          P3 1  0  0                          │
│  P4 0  0  2          P4 0  0  2                          │
│                                                          │
│  Run detection algorithm:                                │
│                                                          │
│  Initial: Work = [0,0,0]                                │
│  Finish = [F,F,F,F,F] (all have allocations)           │
│                                                          │
│  Find Request[i] <= Work:                               │
│  P0: [0,0,0] <= [0,0,0]? YES                           │
│                                                          │
│  Work = [0,0,0] + [0,1,0] = [0,1,0]                    │
│  Finish = [T,F,F,F,F]                                   │
│                                                          │
│  P2: [0,0,0] <= [0,1,0]? YES                           │
│  Work = [0,1,0] + [3,0,3] = [3,1,3]                    │
│  Finish = [T,F,T,F,F]                                   │
│                                                          │
│  P1: [2,0,2] <= [3,1,3]? YES                           │
│  Work = [3,1,3] + [2,0,0] = [5,1,3]                    │
│  ... continue until all finish                          │
│                                                          │
│  All Finish = True → NO DEADLOCK                        │
│                                                          │
└─────────────────────────────────────────────────────────┘


Deadlock Recovery (데드락 복구):

Option 1: Process Termination (프로세스 종료)
┌─────────────────────────────────────────────────────────┐
│                                                          │
│  A) Abort ALL deadlocked processes                      │
│     ┌───────────────────────────────────┐               │
│     │ P1, P2, P3 in deadlock            │               │
│     │                                    │               │
│     │ TERMINATE ALL: P1 ✗  P2 ✗  P3 ✗  │               │
│     │                                    │               │
│     │ Simple but costly (lost work)     │               │
│     └───────────────────────────────────┘               │
│                                                          │
│  B) Abort ONE at a time until deadlock broken          │
│     ┌───────────────────────────────────┐               │
│     │ Select victim based on:           │               │
│     │ • Priority of process             │               │
│     │ • How long it has been running    │               │
│     │ • How many resources it holds     │               │
│     │ • How many more it needs          │               │
│     │ • Interactive vs Batch            │               │
│     └───────────────────────────────────┘               │
│                                                          │
└─────────────────────────────────────────────────────────┘

Option 2: Resource Preemption (자원 선점)
┌─────────────────────────────────────────────────────────┐
│                                                          │
│  Take resources from processes to break deadlock        │
│                                                          │
│  Issues to address:                                      │
│                                                          │
│  1. Selecting victim:                                    │
│     Which process to preempt? (minimize cost)           │
│                                                          │
│  2. Rollback:                                            │
│     ┌─────────────────────────────────────────────┐     │
│     │  Option A: Total rollback                    │     │
│     │  ───────────────────────                     │     │
│     │  Abort process, restart from beginning       │     │
│     │                                              │     │
│     │  Option B: Partial rollback (checkpoint)     │     │
│     │  ──────────────────────────────────────      │     │
│     │  Roll back to safe state (requires           │     │
│     │  checkpoint mechanism)                       │     │
│     └─────────────────────────────────────────────┘     │
│                                                          │
│  3. Starvation prevention:                               │
│     ┌─────────────────────────────────────────────┐     │
│     │  Same process may be selected repeatedly     │     │
│     │                                              │     │
│     │  Solution: Include number of rollbacks in    │     │
│     │  cost factor (limit how many times a         │     │
│     │  process can be chosen as victim)            │     │
│     └─────────────────────────────────────────────┘     │
│                                                          │
└─────────────────────────────────────────────────────────┘


When to Run Detection Algorithm?
┌─────────────────────────────────────────────────────────┐
│                                                          │
│  Option 1: Every resource request                       │
│  ─────────────────────────────────                      │
│  • Immediate detection                                   │
│  • High overhead                                         │
│  • Good if deadlocks are frequent                       │
│                                                          │
│  Option 2: Periodically (e.g., every N minutes)        │
│  ─────────────────────────────────────────────         │
│  • Lower overhead                                        │
│  • Delayed detection                                     │
│  • Balance between cost and response time               │
│                                                          │
│  Option 3: When CPU utilization drops below threshold  │
│  ────────────────────────────────────────────────────   │
│  • Deadlock often causes low CPU utilization            │
│  • Adaptive approach                                     │
│  • May miss deadlocks if system is still busy          │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

---

## Summary (요약)

| Concept | Description |
|---------|-------------|
| Deadlock (데드락) | Processes wait indefinitely for each other |
| Mutual Exclusion (상호 배제) | Resource can only be held by one process |
| Hold and Wait (점유와 대기) | Process holds resources while waiting for more |
| No Preemption (비선점) | Resources cannot be forcibly taken |
| Circular Wait (순환 대기) | Circular chain of waiting processes |
| Resource Allocation Graph (자원 할당 그래프) | Graph showing resource assignments and requests |
| Safe State (안전 상태) | State where all processes can complete |
| Banker's Algorithm (은행원 알고리즘) | Deadlock avoidance using maximum claims |
| Detection (탐지) | Find existing deadlocks |
| Recovery (복구) | Break deadlock by termination or preemption |

---

## Key Terms (주요 용어)

- **Coffman Conditions**: The four necessary conditions for deadlock
- **Safe Sequence (안전 순서)**: Order in which all processes can complete
- **Wait-For Graph (대기 그래프)**: Simplified RAG showing process dependencies
- **Victim Selection (희생자 선택)**: Choosing process to terminate/preempt
- **Rollback (롤백)**: Returning process to previous safe state
- **Checkpoint (체크포인트)**: Saved state for potential rollback
