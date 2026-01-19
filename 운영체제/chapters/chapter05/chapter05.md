# Chapter 05. Synchronization Basics (동기화 기초)

> Process synchronization ensures orderly execution when multiple processes access shared resources.

---

## The Critical Section Problem (임계 구역 문제)

### Concept (개념)

When multiple processes or threads access shared data concurrently, we need to ensure data consistency. The **Critical Section (임계 구역)** is a code segment where shared resources are accessed.

**Structure of a Process:**
```
do {
    [Entry Section]     // Request to enter critical section

    [Critical Section]  // Access shared resources

    [Exit Section]      // Leave critical section

    [Remainder Section] // Other code
} while (true);
```

**Requirements for Critical Section Solution:**

| Requirement | Korean | Description |
|-------------|--------|-------------|
| **Mutual Exclusion** | 상호 배제 | Only one process in critical section at a time |
| **Progress** | 진행 | If no process in CS, selection cannot be postponed indefinitely |
| **Bounded Waiting** | 한정된 대기 | Limit on number of times others enter CS after a request |

### Diagram / Example

```
Critical Section Problem Visualization

Process P1                    Process P2
    │                             │
    │  Entry Section              │  Entry Section
    │  (request lock)             │  (request lock)
    │       │                     │       │
    │       ▼                     │       │
    │  ┌──────────────┐           │       │ (blocked)
    │  │   Critical   │           │       │
    │  │   Section    │           │       │
    │  │ (x = x + 1)  │           │       │
    │  └──────────────┘           │       │
    │       │                     │       │
    │  Exit Section               │       │
    │  (release lock)             │       │
    │       │                     │       ▼
    │       │                     │  ┌──────────────┐
    │       │                     │  │   Critical   │
    │       │                     │  │   Section    │
    │                             │  │ (x = x + 2)  │
    │                             │  └──────────────┘
    ▼                             ▼

Correct! Only one process in CS at a time.


Requirements Illustration:

1. Mutual Exclusion (상호 배제)
   ┌───────────────────────────────────┐
   │          Critical Section         │
   │                                   │
   │  Only ONE process allowed here    │
   │                                   │
   │         ┌─────┐                   │
   │         │  P1 │                   │
   │         └─────┘                   │
   │                                   │
   │   P2: ✗ BLOCKED                   │
   │   P3: ✗ BLOCKED                   │
   └───────────────────────────────────┘

2. Progress (진행)
   ┌───────────────────────────────────┐
   │          Critical Section         │
   │                                   │
   │           (EMPTY)                 │
   │                                   │
   └───────────────────────────────────┘
              │
              ▼
   P1, P2 both want to enter
   → Decision MUST be made (no indefinite delay)
   → Cannot be affected by processes in remainder section

3. Bounded Waiting (한정된 대기)
   ┌──────────────────────────────────────────┐
   │  After P1 requests entry:                │
   │                                          │
   │  P2 enters CS ─┐                         │
   │  P2 enters CS ─┤ At most N times        │
   │  P2 enters CS ─┘ before P1 gets in      │
   │                                          │
   │  Then P1 MUST get a turn                 │
   └──────────────────────────────────────────┘
```

---

## Race Condition (경쟁 조건)

### Concept (개념)

A **Race Condition (경쟁 조건)** occurs when the outcome of an operation depends on the timing or order of uncontrolled events. It happens when multiple processes access and manipulate shared data concurrently.

**Classic Example - Counter Increment:**
```c
// Shared variable
int counter = 5;

// Process P1             // Process P2
counter++;                counter--;

// Each high-level operation consists of multiple machine instructions:
// counter++              // counter--
// R1 = counter           // R2 = counter
// R1 = R1 + 1            // R2 = R2 - 1
// counter = R1           // counter = R2
```

If these instructions interleave incorrectly, the result may be wrong.

### Diagram / Example

```
Race Condition Example (경쟁 조건 예제)

Initial: counter = 5

Correct Execution (Sequential):
P1: counter++ → counter = 6
P2: counter-- → counter = 5
Final: counter = 5 ✓

Race Condition (Interleaved):
┌─────────────────────────────────────────────────────────┐
│  Time │ P1 (counter++)    │ P2 (counter--)   │ counter │
├───────┼───────────────────┼──────────────────┼─────────┤
│  T0   │ R1 = counter (5)  │                  │    5    │
│  T1   │                   │ R2 = counter (5) │    5    │
│  T2   │ R1 = R1 + 1 (6)   │                  │    5    │
│  T3   │                   │ R2 = R2 - 1 (4)  │    5    │
│  T4   │ counter = R1      │                  │    6    │
│  T5   │                   │ counter = R2     │    4    │
└───────┴───────────────────┴──────────────────┴─────────┘

Final: counter = 4 ✗ (Should be 5!)

The increment was LOST because of interleaving!


Another Interleaving:
┌─────────────────────────────────────────────────────────┐
│  Time │ P1 (counter++)    │ P2 (counter--)   │ counter │
├───────┼───────────────────┼──────────────────┼─────────┤
│  T0   │ R1 = counter (5)  │                  │    5    │
│  T1   │ R1 = R1 + 1 (6)   │                  │    5    │
│  T2   │                   │ R2 = counter (5) │    5    │
│  T3   │ counter = R1      │                  │    6    │
│  T4   │                   │ R2 = R2 - 1 (4)  │    6    │
│  T5   │                   │ counter = R2     │    4    │
└───────┴───────────────────┴──────────────────┴─────────┘

Final: counter = 4 ✗ (Still wrong!)


Bank Account Race Condition:

Account Balance: $1000

Thread A: Withdraw $500      Thread B: Withdraw $300
─────────────────────       ─────────────────────
read balance (1000)
                            read balance (1000)
balance = 1000 - 500
                            balance = 1000 - 300
write balance (500)
                            write balance (700)

Final Balance: $700 (Should be $200!)
Customer withdrew $800 but only $300 deducted!
```

---

## Mutual Exclusion (상호 배제)

### Concept (개념)

**Mutual Exclusion (상호 배제)** ensures that if one process is executing in its critical section, no other process can be executing in its critical section.

**Software Approaches:**
- Peterson's Solution
- Bakery Algorithm

**Hardware Approaches:**
- Disabling interrupts
- Atomic instructions (test-and-set, compare-and-swap)

**Higher-Level Constructs:**
- Mutex locks
- Semaphores
- Monitors

### Diagram / Example

```
Mutual Exclusion Concept (상호 배제 개념)

WITHOUT Mutual Exclusion:
┌─────────────────────────────────────────────┐
│                Shared Resource              │
│  ┌─────────────────────────────────────┐   │
│  │                                      │   │
│  │    P1 ────────►                      │   │
│  │                    CONFLICT!         │   │
│  │    P2 ────────►                      │   │
│  │                                      │   │
│  └─────────────────────────────────────┘   │
└─────────────────────────────────────────────┘

WITH Mutual Exclusion:
┌─────────────────────────────────────────────┐
│                Shared Resource              │
│  ┌─────────────────────────────────────┐   │
│  │                                      │   │
│  │    P1 ────────►   OK!                │   │
│  │                                      │   │
│  │    P2 ─────X BLOCKED (waiting)       │   │
│  │                                      │   │
│  └─────────────────────────────────────┘   │
└─────────────────────────────────────────────┘


Implementation Approaches:

1. Software Solutions (소프트웨어 해결책)
   ┌──────────────────────────────────────────┐
   │  • Algorithm using shared variables      │
   │  • No special hardware support needed    │
   │  • Examples: Peterson's, Bakery          │
   │  • Complex, may have issues              │
   └──────────────────────────────────────────┘

2. Hardware Solutions (하드웨어 해결책)
   ┌──────────────────────────────────────────┐
   │  • Special atomic instructions           │
   │  • Test-and-Set, Compare-and-Swap       │
   │  • Simple and efficient                  │
   │  • May cause busy waiting               │
   └──────────────────────────────────────────┘

3. OS/Language Support (OS/언어 지원)
   ┌──────────────────────────────────────────┐
   │  • Mutex, Semaphore, Monitor            │
   │  • Easier to use correctly              │
   │  • May involve blocking/sleeping        │
   │  • Chapter 6 covers these               │
   └──────────────────────────────────────────┘
```

---

## Peterson's Solution (피터슨의 해결책)

### Concept (개념)

**Peterson's Solution** is a classic software-based solution for the critical section problem for two processes. It uses two shared variables:

- `int turn`: Indicates whose turn it is to enter CS
- `bool flag[2]`: Indicates if process is ready to enter CS

**Algorithm for Process Pi:**
```c
do {
    flag[i] = true;          // I want to enter
    turn = j;                // Give turn to other process
    while (flag[j] && turn == j)
        ;                    // Wait

    // Critical Section

    flag[i] = false;         // I'm done

    // Remainder Section
} while (true);
```

**Why It Works:**
- Mutual Exclusion: Both can't be in CS (turn can only be 0 or 1)
- Progress: If Pj doesn't want to enter, Pi can proceed
- Bounded Waiting: After Pi sets turn=j, if Pj enters once, it will set turn=i

**Limitation:** Requires atomic load/store, may not work on modern processors due to instruction reordering.

### Diagram / Example

```
Peterson's Solution for Two Processes

Shared Variables:
┌────────────────────────────────────┐
│  int turn;        // 0 or 1        │
│  bool flag[2];    // intentions    │
└────────────────────────────────────┘

Process P0:                    Process P1:
─────────────                  ─────────────
flag[0] = true;                flag[1] = true;
turn = 1;                      turn = 0;
while (flag[1] && turn == 1)   while (flag[0] && turn == 0)
    ; // wait                      ; // wait

// Critical Section             // Critical Section

flag[0] = false;               flag[1] = false;


Execution Trace - Both Want to Enter:

┌────────────────────────────────────────────────────────────┐
│ Time │ P0 Action          │ P1 Action         │ Variables │
├──────┼────────────────────┼───────────────────┼───────────┤
│  1   │ flag[0] = true     │                   │ f[0]=T    │
│  2   │ turn = 1           │                   │ turn=1    │
│  3   │                    │ flag[1] = true    │ f[1]=T    │
│  4   │                    │ turn = 0          │ turn=0    │
│  5   │ Check: f[1]&&t==1? │                   │           │
│      │ T && F = FALSE     │                   │           │
│      │ P0 enters CS!      │                   │           │
│  6   │                    │ Check: f[0]&&t==0?│           │
│      │                    │ T && T = TRUE     │           │
│      │                    │ P1 waits...       │           │
│  7   │ (in CS)            │ (waiting)         │           │
│  8   │ flag[0] = false    │                   │ f[0]=F    │
│  9   │                    │ Check: f[0]&&t==0?│           │
│      │                    │ F && T = FALSE    │           │
│      │                    │ P1 enters CS!     │           │
└──────┴────────────────────┴───────────────────┴───────────┘


Proof of Correctness:

Mutual Exclusion:
┌──────────────────────────────────────────────────────────┐
│ Assume both P0 and P1 are in CS simultaneously.         │
│                                                          │
│ For P0 to be in CS: NOT(flag[1] && turn==1)             │
│   → flag[1]==false OR turn==0                           │
│                                                          │
│ For P1 to be in CS: NOT(flag[0] && turn==0)             │
│   → flag[0]==false OR turn==1                           │
│                                                          │
│ But if both are in CS:                                   │
│   flag[0]==true AND flag[1]==true (they set these)      │
│                                                          │
│ So turn must be 0 AND turn must be 1                    │
│ CONTRADICTION! ∴ Mutual exclusion holds.                │
└──────────────────────────────────────────────────────────┘
```

---

## Hardware Solutions (하드웨어 해결책)

### Concept (개념)

Modern processors provide special atomic hardware instructions that can be used to implement mutual exclusion:

**1. Disabling Interrupts (인터럽트 비활성화)**
```c
disable_interrupts();
// Critical Section
enable_interrupts();
```
- Simple but only works on single-processor systems
- User programs shouldn't disable interrupts

**2. Test-and-Set (검사와 지정)**
```c
bool test_and_set(bool *target) {
    bool rv = *target;  // Read current value
    *target = true;     // Set to true
    return rv;          // Return old value
}
// These operations are ATOMIC (indivisible)
```

**3. Compare-and-Swap (비교와 교환, CAS)**
```c
int compare_and_swap(int *value, int expected, int new_value) {
    int temp = *value;
    if (*value == expected)
        *value = new_value;
    return temp;  // Return old value
}
// Atomic operation
```

### Diagram / Example

```
Test-and-Set Lock Implementation

Shared Variable:
┌─────────────────────┐
│ bool lock = false;  │
└─────────────────────┘

Process using Test-and-Set:
do {
    while (test_and_set(&lock))
        ; // Busy wait (spin)

    // Critical Section

    lock = false;

    // Remainder Section
} while (true);


How It Works:

Process P1                    Process P2
    │                             │
    │ test_and_set(&lock)         │
    │ returns false (was false)   │
    │ lock is now true            │
    │       │                     │
    │       ▼                     │
    │ ┌──────────────┐            │
    │ │   Critical   │            │ test_and_set(&lock)
    │ │   Section    │            │ returns true (was true)
    │ │              │            │ loops... (busy wait)
    │ └──────────────┘            │      │
    │       │                     │      │ (spinning)
    │ lock = false               │      │
    │       │                     │      ▼
    │                             │ test_and_set(&lock)
    │                             │ returns false!
    │                             │ enters CS
    ▼                             ▼


Compare-and-Swap Lock Implementation

do {
    while (compare_and_swap(&lock, 0, 1) != 0)
        ; // Busy wait

    // Critical Section

    lock = 0;

    // Remainder Section
} while (true);


Atomic Instructions in x86:

┌────────────────────────────────────────────────────────┐
│  XCHG (Exchange)                                       │
│  ─────────────────                                     │
│  xchg eax, [mem]   ; Atomically swap eax with memory  │
│                                                        │
│  CMPXCHG (Compare and Exchange)                        │
│  ──────────────────────────────                        │
│  cmpxchg [mem], ebx  ; If [mem]==eax, then [mem]=ebx  │
│                      ; Atomic compare-and-swap         │
│                                                        │
│  LOCK prefix                                           │
│  ───────────                                           │
│  lock inc [mem]    ; Atomic increment                  │
│  lock add [mem], 5 ; Atomic add                        │
└────────────────────────────────────────────────────────┘


Bounded Waiting with Test-and-Set:

// Shared variables
bool lock = false;
bool waiting[n];  // All initialized to false

// Process Pi
do {
    waiting[i] = true;
    bool key = true;
    while (waiting[i] && key)
        key = test_and_set(&lock);
    waiting[i] = false;

    // Critical Section

    // Find next waiting process
    int j = (i + 1) % n;
    while (j != i && !waiting[j])
        j = (j + 1) % n;

    if (j == i)
        lock = false;      // No one waiting
    else
        waiting[j] = false; // Let j enter

    // Remainder Section
} while (true);
```

---

## Busy Waiting and Spinlocks (바쁜 대기와 스핀락)

### Concept (개념)

**Busy Waiting (바쁜 대기)**, also called **spinning**, is when a process repeatedly checks a condition in a loop, wasting CPU cycles.

**Spinlock (스핀락)**
- A lock that uses busy waiting
- Process "spins" while waiting for lock
- Efficient when wait time is short (avoids context switch overhead)
- Wasteful when wait time is long

**When to Use Spinlocks:**
- Multiprocessor systems (other CPUs can do work)
- Short critical sections
- When blocking overhead exceeds spinning cost

**When NOT to Use:**
- Single processor systems (spinning wastes all CPU time)
- Long critical sections
- When many processes compete for the lock

### Diagram / Example

```
Busy Waiting vs Blocking (바쁜 대기 vs 블로킹)

Busy Waiting (Spinlock):
┌────────────────────────────────────────────────────────┐
│  Process P1 (holding lock)  │  Process P2 (waiting)   │
│  ┌───────────────────────┐  │  while(locked) {        │
│  │    Critical Section   │  │      ; // spin          │
│  │                       │  │  }                      │
│  │    (doing work)       │  │                         │
│  └───────────────────────┘  │  CPU cycles wasted!     │
│                             │  ████████████████████   │
└────────────────────────────────────────────────────────┘

Blocking (Sleep/Wake):
┌────────────────────────────────────────────────────────┐
│  Process P1 (holding lock)  │  Process P2 (blocked)   │
│  ┌───────────────────────┐  │                         │
│  │    Critical Section   │  │  sleep(); // blocked    │
│  │                       │  │                         │
│  │    (doing work)       │  │  zzz... (no CPU usage) │
│  └───────────────────────┘  │                         │
│        │                    │                         │
│        │ unlock             │                         │
│        └────────────────────┼───► wakeup()            │
└────────────────────────────────────────────────────────┘


When Spinlocks Make Sense:

Multiprocessor System:
┌────────────────────────────────────────────────────────┐
│   CPU 0              CPU 1              CPU 2          │
│  ┌──────┐          ┌──────┐          ┌──────┐         │
│  │ P1   │          │ P2   │          │ P3   │         │
│  │ (CS) │          │(spin)│          │(work)│         │
│  └──────┘          └──────┘          └──────┘         │
│                                                        │
│  P1 doing useful work in CS                           │
│  P2 spinning, but other CPUs still working            │
│  If CS is short, P2 will get lock soon               │
└────────────────────────────────────────────────────────┘

Single Processor System:
┌────────────────────────────────────────────────────────┐
│                        CPU                             │
│                      ┌──────┐                         │
│                      │ P2   │                         │
│                      │(spin)│                         │
│                      └──────┘                         │
│                                                        │
│  P2 spinning, P1 cannot run to release lock!          │
│  Entire CPU time wasted on spinning!                  │
│  BAD! Use blocking instead.                           │
└────────────────────────────────────────────────────────┘


Cost Comparison:

┌─────────────────────────────────────────────────────────┐
│                                                         │
│  Spinlock Cost = Spin Time                             │
│  Blocking Cost = Context Switch × 2                    │
│                                                         │
│  If (Expected Wait Time < 2 × Context Switch Time):    │
│      Use Spinlock                                      │
│  Else:                                                 │
│      Use Blocking                                      │
│                                                         │
│  Typical context switch: ~1000-10000 cycles            │
│  Short CS: < 1000 cycles → Spinlock OK                │
│  Long CS: > 10000 cycles → Use Blocking               │
└─────────────────────────────────────────────────────────┘
```

---

## Summary (요약)

| Concept | Description |
|---------|-------------|
| Critical Section (임계 구역) | Code segment accessing shared resources |
| Race Condition (경쟁 조건) | Result depends on timing of events |
| Mutual Exclusion (상호 배제) | Only one process in CS at a time |
| Progress (진행) | Selection cannot be postponed indefinitely |
| Bounded Waiting (한정된 대기) | Limit on waiting time |
| Peterson's Solution (피터슨 해결책) | Software solution for 2 processes |
| Test-and-Set (검사와 지정) | Atomic hardware instruction |
| Compare-and-Swap (비교와 교환) | Atomic compare and exchange |
| Spinlock (스핀락) | Lock using busy waiting |

---

## Key Terms (주요 용어)

- **Atomic Operation (원자적 연산)**: Indivisible operation that completes entirely or not at all
- **Busy Waiting (바쁜 대기)**: Continuously checking condition in a loop
- **Lock (락)**: Mechanism to enforce mutual exclusion
- **Preemption (선점)**: Taking CPU away from running process
- **Starvation (기아)**: Process indefinitely denied access to resource
