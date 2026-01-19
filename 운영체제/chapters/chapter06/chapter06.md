# Chapter 06. Synchronization Tools (동기화 도구)

> Synchronization tools provide mechanisms for processes to coordinate access to shared resources safely.

---

## Mutex Locks (뮤텍스 락)

### Concept (개념)

A **Mutex (Mutual Exclusion Lock, 상호 배제 락)** is the simplest synchronization tool. It protects critical sections by requiring that a process acquire a lock before entering and release it when exiting.

**Key Operations:**
- `acquire()`: Get the lock (blocks if already held)
- `release()`: Release the lock

**Properties:**
- Only the owner can release the lock
- Binary state: locked or unlocked
- Provides mutual exclusion

**Implementation:**
```c
// Mutex structure
typedef struct {
    bool locked;
} mutex_t;

void acquire(mutex_t *m) {
    while (test_and_set(&m->locked))
        ;  // Spin until lock is free
}

void release(mutex_t *m) {
    m->locked = false;
}

// Usage
mutex_t lock;
acquire(&lock);
// Critical Section
release(&lock);
```

### Diagram / Example

```
Mutex Lock Operation (뮤텍스 락 동작)

Initial State:
┌─────────────────────────────────────────┐
│           Mutex Lock                    │
│        ┌───────────┐                    │
│        │ UNLOCKED  │                    │
│        └───────────┘                    │
└─────────────────────────────────────────┘

Process P1 acquires:
┌─────────────────────────────────────────┐
│           Mutex Lock                    │
│        ┌───────────┐                    │
│        │  LOCKED   │◄─── Owner: P1      │
│        └───────────┘                    │
│                                         │
│  P1: In Critical Section                │
└─────────────────────────────────────────┘

Process P2 tries to acquire:
┌─────────────────────────────────────────┐
│           Mutex Lock                    │
│        ┌───────────┐                    │
│        │  LOCKED   │◄─── Owner: P1      │
│        └───────────┘                    │
│              ▲                          │
│              │ (blocked)                │
│  P2: ────────┘ Waiting...               │
│                                         │
│  P1: In Critical Section                │
└─────────────────────────────────────────┘

P1 releases, P2 acquires:
┌─────────────────────────────────────────┐
│           Mutex Lock                    │
│        ┌───────────┐                    │
│        │  LOCKED   │◄─── Owner: P2      │
│        └───────────┘                    │
│                                         │
│  P2: In Critical Section                │
│  P1: Continuing other work              │
└─────────────────────────────────────────┘


Mutex vs Spinlock Comparison:

Spinlock (Busy Waiting):
┌────────────────────────────────────────────────┐
│  while (locked) {                              │
│      // Keep checking (wastes CPU cycles)     │
│  }                                             │
│                                                │
│  CPU Usage: ████████████████████ (100%)       │
│  Good for: Short critical sections, SMP       │
└────────────────────────────────────────────────┘

Mutex (Blocking):
┌────────────────────────────────────────────────┐
│  if (locked) {                                 │
│      block();  // Sleep, yield CPU            │
│  }                                             │
│                                                │
│  CPU Usage: ░░░░░░░░░░░░░░░░░░░░ (0%)         │
│  Good for: Long critical sections, UP         │
└────────────────────────────────────────────────┘


Real-World Analogy - Bathroom Key:

┌─────────────────────────────────────────────────┐
│                   BATHROOM                       │
│                  (Critical Section)              │
│                                                  │
│    ┌─────┐                                       │
│    │ KEY │  ◄─── Only ONE key exists            │
│    └─────┘                                       │
│       │                                          │
│       │  Take key                                │
│       ▼                                          │
│  ┌─────────┐                                     │
│  │ Person  │  ◄─── Using bathroom               │
│  │    A    │       (in critical section)        │
│  └─────────┘                                     │
│                                                  │
│  Person B: Waiting for key                       │
│  Person C: Waiting for key                       │
│                                                  │
│  When A finishes → returns key → B gets it      │
└─────────────────────────────────────────────────┘
```

---

## Semaphores (세마포어)

### Concept (개념)

A **Semaphore (세마포어)** is a more sophisticated synchronization tool invented by Dijkstra. It's an integer variable accessed only through two atomic operations.

**Operations:**
- `wait(S)` or `P(S)`: Decrement S; if S < 0, block
- `signal(S)` or `V(S)`: Increment S; if S <= 0, wake a blocked process

**Types:**
| Type | Value Range | Use Case |
|------|-------------|----------|
| **Binary Semaphore** | 0 or 1 | Mutual exclusion (like mutex) |
| **Counting Semaphore** | 0 to N | Controlling access to N resources |

**Implementation:**
```c
typedef struct {
    int value;
    queue_t waiting_list;
} semaphore_t;

void wait(semaphore_t *S) {
    S->value--;
    if (S->value < 0) {
        // Add this process to S->waiting_list
        block();
    }
}

void signal(semaphore_t *S) {
    S->value++;
    if (S->value <= 0) {
        // Remove a process P from S->waiting_list
        wakeup(P);
    }
}
```

### Diagram / Example

```
Semaphore Operations (세마포어 연산)

Binary Semaphore (S = 1):

Initial: S = 1

P1: wait(S)     →  S = 0 (P1 enters CS)
P2: wait(S)     →  S = -1 (P2 blocks, added to queue)
P3: wait(S)     →  S = -2 (P3 blocks, added to queue)
P1: signal(S)   →  S = -1 (P2 wakes up, enters CS)
P2: signal(S)   →  S = 0 (P3 wakes up, enters CS)
P3: signal(S)   →  S = 1 (no one waiting)


Visual Representation:

┌──────────────────────────────────────────────────────┐
│  Semaphore S                                         │
│  ┌─────────────┐                                     │
│  │  Value: 1   │                                     │
│  └─────────────┘                                     │
│  ┌─────────────────────────────────────┐             │
│  │ Waiting Queue: [empty]              │             │
│  └─────────────────────────────────────┘             │
└──────────────────────────────────────────────────────┘

After P1 wait(S):
┌──────────────────────────────────────────────────────┐
│  Semaphore S                                         │
│  ┌─────────────┐                                     │
│  │  Value: 0   │  P1 in Critical Section            │
│  └─────────────┘                                     │
│  ┌─────────────────────────────────────┐             │
│  │ Waiting Queue: [empty]              │             │
│  └─────────────────────────────────────┘             │
└──────────────────────────────────────────────────────┘

After P2, P3 wait(S):
┌──────────────────────────────────────────────────────┐
│  Semaphore S                                         │
│  ┌─────────────┐                                     │
│  │  Value: -2  │  P1 in Critical Section            │
│  └─────────────┘                                     │
│  ┌─────────────────────────────────────┐             │
│  │ Waiting Queue: [P2] → [P3]          │             │
│  └─────────────────────────────────────┘             │
│                                                      │
│  |Value| = Number of waiting processes               │
└──────────────────────────────────────────────────────┘


Counting Semaphore Example (Resource Pool):

5 Printers Available (S = 5)

┌─────────────────────────────────────────────────────┐
│     PRINTER POOL                                     │
│  ┌───────┬───────┬───────┬───────┬───────┐          │
│  │Print 1│Print 2│Print 3│Print 4│Print 5│          │
│  └───────┴───────┴───────┴───────┴───────┘          │
│                                                      │
│  Semaphore S = 5                                     │
└─────────────────────────────────────────────────────┘

Process P1: wait(S) → S = 4, gets Printer 1
Process P2: wait(S) → S = 3, gets Printer 2
Process P3: wait(S) → S = 2, gets Printer 3
Process P4: wait(S) → S = 1, gets Printer 4
Process P5: wait(S) → S = 0, gets Printer 5
Process P6: wait(S) → S = -1, BLOCKS! (no printer available)

┌─────────────────────────────────────────────────────┐
│     PRINTER POOL                                     │
│  ┌───────┬───────┬───────┬───────┬───────┐          │
│  │ [P1]  │ [P2]  │ [P3]  │ [P4]  │ [P5]  │          │
│  └───────┴───────┴───────┴───────┴───────┘          │
│  All printers in use!                                │
│                                                      │
│  Semaphore S = -1                                    │
│  Waiting Queue: [P6]                                 │
└─────────────────────────────────────────────────────┘

When P1 finishes: signal(S) → S = 0, P6 wakes up


Semaphore for Process Synchronization:

Enforce S1 before S2:
┌─────────────────────────────────────────────────────┐
│  semaphore sync = 0;                                 │
│                                                      │
│  Process P1:              Process P2:                │
│  ──────────               ──────────                 │
│  S1;                      wait(sync);                │
│  signal(sync);            S2;                        │
│                                                      │
│  S2 will always execute after S1!                    │
└─────────────────────────────────────────────────────┘

Timeline:
┌─────────────────────────────────────────────────────┐
│  P1: ───[S1]───[signal]───────►                     │
│                    │                                 │
│                    └─────────┐                       │
│                              ▼                       │
│  P2: ───[wait]──────────────[S2]────►               │
│         (blocked)                                    │
└─────────────────────────────────────────────────────┘
```

---

## Monitors (모니터)

### Concept (개념)

A **Monitor (모니터)** is a high-level synchronization construct that combines data, procedures, and synchronization into a single unit. It was developed to address the difficulty of using semaphores correctly.

**Key Features:**
- Only one process can be active inside the monitor at a time
- Automatic mutual exclusion
- Condition variables for process synchronization

**Structure:**
```
monitor MonitorName {
    // Shared data

    // Condition variables
    condition x, y;

    // Procedures (only way to access data)
    procedure P1(...) { ... }
    procedure P2(...) { ... }

    // Initialization code
    initialization_code(...) { ... }
}
```

**Monitor vs Semaphore:**
| Aspect | Semaphore | Monitor |
|--------|-----------|---------|
| Level | Low-level | High-level |
| Ease of use | Error-prone | Safer |
| Mutual exclusion | Manual | Automatic |
| Language support | OS primitives | Language construct |

### Diagram / Example

```
Monitor Structure (모니터 구조)

┌─────────────────────────────────────────────────────────┐
│                       MONITOR                            │
│  ┌───────────────────────────────────────────────────┐  │
│  │              Shared Data                          │  │
│  │         (encapsulated, private)                   │  │
│  └───────────────────────────────────────────────────┘  │
│                                                          │
│  ┌─────────────────────┐  ┌─────────────────────┐       │
│  │   Procedure 1       │  │   Procedure 2       │       │
│  │   ─────────────     │  │   ─────────────     │       │
│  │   Operations on     │  │   Operations on     │       │
│  │   shared data       │  │   shared data       │       │
│  └─────────────────────┘  └─────────────────────┘       │
│                                                          │
│  ┌───────────────────────────────────────────────────┐  │
│  │           Initialization Code                     │  │
│  └───────────────────────────────────────────────────┘  │
│                                                          │
│                    ▲                                     │
│                    │ Only ONE process                    │
│                    │ can be inside                       │
│                    │                                     │
│  Entry Queue: [P1] [P2] [P3]                            │
│               (waiting to enter)                         │
└─────────────────────────────────────────────────────────┘


Monitor Mutual Exclusion (모니터 상호 배제):

Process P1 enters monitor:
┌─────────────────────────────────────────────────────────┐
│                       MONITOR                            │
│  ┌─────────────────────────────────────────────────┐    │
│  │                                                   │   │
│  │    ┌────┐                                        │   │
│  │    │ P1 │  ◄── Active inside monitor            │   │
│  │    └────┘                                        │   │
│  │                                                   │   │
│  └─────────────────────────────────────────────────┘    │
│                                                          │
│  Entry Queue: [P2] [P3]  ← Blocked, waiting              │
│                                                          │
└─────────────────────────────────────────────────────────┘

Automatic mutual exclusion - no explicit lock needed!


Java Monitor Example:

┌─────────────────────────────────────────────────────────┐
│  class BoundedBuffer {                                   │
│      private Object[] buffer;                            │
│      private int count, in, out;                        │
│                                                          │
│      public synchronized void insert(Object item) {     │
│          while (count == BUFFER_SIZE)                    │
│              wait();  // Buffer full                     │
│          buffer[in] = item;                              │
│          in = (in + 1) % BUFFER_SIZE;                   │
│          count++;                                        │
│          notifyAll();                                    │
│      }                                                   │
│                                                          │
│      public synchronized Object remove() {               │
│          while (count == 0)                              │
│              wait();  // Buffer empty                    │
│          Object item = buffer[out];                      │
│          out = (out + 1) % BUFFER_SIZE;                 │
│          count--;                                        │
│          notifyAll();                                    │
│          return item;                                    │
│      }                                                   │
│  }                                                       │
│                                                          │
│  'synchronized' keyword = monitor in Java               │
└─────────────────────────────────────────────────────────┘


Monitor with Condition Variables:

┌─────────────────────────────────────────────────────────┐
│                       MONITOR                            │
│                                                          │
│  Condition Variables:                                    │
│  ┌─────────┐  ┌─────────┐                               │
│  │ cond x  │  │ cond y  │                               │
│  └────┬────┘  └────┬────┘                               │
│       │            │                                     │
│       │   Queues for each condition:                    │
│       │   ┌───────────────────────┐                     │
│       └──►│ x.queue: [P1] [P2]   │                     │
│           └───────────────────────┘                     │
│           ┌───────────────────────┐                     │
│           │ y.queue: [P3]         │                     │
│           └───────────────────────┘                     │
│                                                          │
│  Entry Queue: [P4] [P5]                                 │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

---

## Condition Variables (조건 변수)

### Concept (개념)

**Condition Variables (조건 변수)** allow processes inside a monitor to wait for specific conditions. They are used with monitors to provide synchronization beyond simple mutual exclusion.

**Operations:**
- `wait(c)`: Release monitor lock and sleep until signaled
- `signal(c)`: Wake up one waiting process (if any)
- `broadcast(c)`: Wake up all waiting processes

**Signal Semantics:**
| Type | Description |
|------|-------------|
| **Signal and Wait** | Signaler waits; awakened process runs immediately |
| **Signal and Continue** | Signaler continues; awakened process waits for monitor |

**Important Rules:**
- `wait()` always blocks the caller
- `signal()` does nothing if no one is waiting
- Use `while` loops, not `if`, when checking conditions

### Diagram / Example

```
Condition Variable Operations (조건 변수 연산)

wait(condition) Operation:
┌─────────────────────────────────────────────────────────┐
│  Process P1 inside monitor                               │
│                                                          │
│  if (condition not met) {                                │
│      wait(x);  ───────┐                                  │
│  }                    │                                  │
│                       ▼                                  │
│  1. Release monitor lock                                 │
│  2. Add P1 to x's waiting queue                         │
│  3. P1 blocks (sleeps)                                   │
│  4. When signaled: reacquire lock and continue          │
│                                                          │
└─────────────────────────────────────────────────────────┘

signal(condition) Operation:
┌─────────────────────────────────────────────────────────┐
│  Process P2 inside monitor                               │
│                                                          │
│  // Make condition true                                  │
│  signal(x);  ─────────┐                                  │
│                       │                                  │
│                       ▼                                  │
│  If x's queue is not empty:                              │
│      Wake up one waiting process                         │
│  Else:                                                   │
│      Do nothing (signal is lost)                         │
│                                                          │
└─────────────────────────────────────────────────────────┘


Signal and Wait vs Signal and Continue:

Signal and Wait (Hoare semantics):
┌─────────────────────────────────────────────────────────┐
│  P2 signals x                                            │
│       │                                                  │
│       ▼                                                  │
│  P2 waits (gives up monitor)                            │
│  P1 wakes up and runs immediately                       │
│       │                                                  │
│       ▼                                                  │
│  P1 finishes, P2 resumes                                │
│                                                          │
│  Timeline:                                               │
│  P2: ──[signal]─────────[wait]──────[resume]───►        │
│  P1: ───────────────────[runs]──────[done]              │
└─────────────────────────────────────────────────────────┘

Signal and Continue (Mesa semantics - more common):
┌─────────────────────────────────────────────────────────┐
│  P2 signals x                                            │
│       │                                                  │
│       ▼                                                  │
│  P2 continues executing                                  │
│  P1 is moved to ready queue                             │
│       │                                                  │
│       ▼                                                  │
│  P2 exits monitor, P1 can enter                         │
│                                                          │
│  Timeline:                                               │
│  P2: ──[signal]────[continues]────[exit]───►            │
│  P1: ───────────────[ready]───────[runs]───►            │
└─────────────────────────────────────────────────────────┘


Why Use while Instead of if:

┌─────────────────────────────────────────────────────────┐
│  WRONG (using if):                                       │
│  ───────────────────                                     │
│  if (count == 0)      // Check once                     │
│      wait(notEmpty);  // Wake up                        │
│  remove();            // May fail! Another process      │
│                       // might have removed item        │
│                                                          │
│  CORRECT (using while):                                  │
│  ────────────────────                                    │
│  while (count == 0)   // Keep checking                  │
│      wait(notEmpty);  // Wake up                        │
│  remove();            // Safe! Condition rechecked      │
│                                                          │
└─────────────────────────────────────────────────────────┘

Scenario showing the problem:
┌─────────────────────────────────────────────────────────┐
│  Buffer has 1 item                                       │
│  P1 and P2 both waiting on notEmpty                     │
│                                                          │
│  Producer adds item and signals                          │
│       │                                                  │
│       ▼                                                  │
│  Both P1 and P2 wake up (broadcast or race)             │
│  P1 runs first, removes item                            │
│  P2 runs next... buffer is empty!                       │
│                                                          │
│  With 'while': P2 rechecks, waits again ✓               │
│  With 'if': P2 crashes trying to remove from empty ✗    │
└─────────────────────────────────────────────────────────┘


Condition Variable Example - Bounded Buffer:

┌─────────────────────────────────────────────────────────┐
│  monitor BoundedBuffer {                                 │
│      item buffer[N];                                     │
│      int count = 0, in = 0, out = 0;                    │
│      condition notFull, notEmpty;                        │
│                                                          │
│      procedure insert(item x) {                          │
│          while (count == N)        // Buffer full       │
│              wait(notFull);                              │
│          buffer[in] = x;                                 │
│          in = (in + 1) % N;                             │
│          count++;                                        │
│          signal(notEmpty);         // Wake consumer     │
│      }                                                   │
│                                                          │
│      procedure remove() {                                │
│          while (count == 0)        // Buffer empty      │
│              wait(notEmpty);                             │
│          item x = buffer[out];                          │
│          out = (out + 1) % N;                           │
│          count--;                                        │
│          signal(notFull);          // Wake producer     │
│          return x;                                       │
│      }                                                   │
│  }                                                       │
└─────────────────────────────────────────────────────────┘
```

---

## Classic Synchronization Problems (동기화 고전 문제)

### The Producer-Consumer Problem (생산자-소비자 문제)

#### Concept (개념)

The **Producer-Consumer Problem** (also known as the Bounded-Buffer Problem) involves two types of processes sharing a fixed-size buffer.

**Participants:**
- **Producer (생산자)**: Creates items and puts them in the buffer
- **Consumer (소비자)**: Removes items from the buffer

**Constraints:**
1. Producer cannot add to a full buffer
2. Consumer cannot remove from an empty buffer
3. Only one process can access the buffer at a time

#### Diagram / Example

```
Producer-Consumer Problem (생산자-소비자 문제)

The Setup:
┌─────────────────────────────────────────────────────────┐
│                                                          │
│  Producer                   Buffer                Consumer
│  ┌─────────┐           ┌─────────────┐          ┌─────────┐
│  │         │           │ [ ][ ][ ][ ]│          │         │
│  │ Creates │──────────►│             │─────────►│ Uses    │
│  │  Items  │           │ (size = N)  │          │  Items  │
│  │         │           │             │          │         │
│  └─────────┘           └─────────────┘          └─────────┘
│                                                          │
└─────────────────────────────────────────────────────────┘


Semaphore Solution:

┌─────────────────────────────────────────────────────────┐
│  // Shared resources                                     │
│  semaphore mutex = 1;      // Buffer access             │
│  semaphore empty = N;      // Empty slots               │
│  semaphore full = 0;       // Filled slots              │
│  item buffer[N];                                         │
│  int in = 0, out = 0;                                   │
│                                                          │
│  // Producer                    // Consumer              │
│  while (true) {                 while (true) {           │
│      item = produce();              wait(full);          │
│      wait(empty);                   wait(mutex);         │
│      wait(mutex);                   item = buffer[out];  │
│      buffer[in] = item;             out = (out+1) % N;   │
│      in = (in+1) % N;               signal(mutex);       │
│      signal(mutex);                 signal(empty);       │
│      signal(full);                  consume(item);       │
│  }                              }                        │
└─────────────────────────────────────────────────────────┘


Execution Trace (Buffer size = 3):

Initial: empty=3, full=0, mutex=1

Producer P1:                 Semaphores
──────────────              ────────────
wait(empty)                  empty=2
wait(mutex)                  mutex=0
buffer[0] = A
signal(mutex)                mutex=1
signal(full)                 full=1

Buffer: [A][ ][ ]

Producer P2:                 Consumer C1:
────────────                 ────────────
wait(empty)  empty=1         wait(full)   full=0
wait(mutex)  mutex=0         wait(mutex)  (blocked)

P2 in critical section...

signal(mutex) mutex=1        (C1 unblocked)
signal(full)  full=1         wait(mutex)  mutex=0
                             item = buffer[0] = A
                             signal(mutex) mutex=1
                             signal(empty) empty=2

Buffer: [ ][B][ ]  (A consumed, B produced)


Visual Timeline:

┌────────────────────────────────────────────────────────┐
│  Time    0   1   2   3   4   5   6   7   8   9         │
│  ────────────────────────────────────────────────────  │
│                                                         │
│  Buffer: [ ][ ][ ]                                     │
│              │                                          │
│  P1 produces │   [A][ ][ ]                             │
│              │       │                                  │
│  P2 produces │       │   [A][B][ ]                     │
│              │       │       │                          │
│  C1 consumes │       │       │   [ ][B][ ]             │
│              │       │       │       │                  │
│  P3 produces │       │       │       │   [ ][B][C]     │
│              │       │       │       │       │          │
│  C2 consumes │       │       │       │       │   [ ][ ][C]
│              ▼       ▼       ▼       ▼       ▼          │
└────────────────────────────────────────────────────────┘
```

### The Readers-Writers Problem (독자-저자 문제)

#### Concept (개념)

The **Readers-Writers Problem** involves access to a shared database or file.

**Participants:**
- **Readers (독자)**: Only read data; can read simultaneously
- **Writers (저자)**: Modify data; require exclusive access

**Variants:**
1. **First Readers-Writers**: Readers have priority (may starve writers)
2. **Second Readers-Writers**: Writers have priority (may starve readers)
3. **Fair**: No starvation for either

#### Diagram / Example

```
Readers-Writers Problem (독자-저자 문제)

Rules:
┌─────────────────────────────────────────────────────────┐
│                                                          │
│  Multiple Readers       ✓  Can read simultaneously      │
│  Reader + Writer        ✗  Cannot access together       │
│  Multiple Writers       ✗  Cannot write simultaneously  │
│                                                          │
│       DATABASE                                           │
│      ┌────────┐                                          │
│      │        │  ◄── R1 (reading)                       │
│      │  DATA  │  ◄── R2 (reading)   OK!                │
│      │        │  ◄── R3 (reading)                       │
│      └────────┘                                          │
│                                                          │
│      ┌────────┐                                          │
│      │        │  ◄── W1 (writing)                       │
│      │  DATA  │  ✗── R1 (blocked)   NOT OK!            │
│      │        │  ✗── W2 (blocked)                       │
│      └────────┘                                          │
│                                                          │
└─────────────────────────────────────────────────────────┘


First Readers-Writers Solution (Readers Priority):

┌─────────────────────────────────────────────────────────┐
│  // Shared variables                                     │
│  semaphore rw_mutex = 1;   // Writer or first reader    │
│  semaphore mutex = 1;      // Protect read_count        │
│  int read_count = 0;       // Number of readers         │
│                                                          │
│  // Writer                     // Reader                 │
│  while (true) {                while (true) {            │
│      wait(rw_mutex);               wait(mutex);          │
│      // Writing...                 read_count++;         │
│      signal(rw_mutex);             if (read_count == 1)  │
│  }                                     wait(rw_mutex);   │
│                                    signal(mutex);        │
│                                    // Reading...         │
│                                    wait(mutex);          │
│                                    read_count--;         │
│                                    if (read_count == 0)  │
│                                        signal(rw_mutex); │
│                                    signal(mutex);        │
│                                }                         │
└─────────────────────────────────────────────────────────┘


Execution Scenario:

Initial: rw_mutex=1, mutex=1, read_count=0

Step 1: R1 arrives
┌────────────────────────────────────────────────────────┐
│  R1: wait(mutex)           mutex=0                      │
│  R1: read_count++ = 1                                   │
│  R1: wait(rw_mutex)        rw_mutex=0  ◄── First reader │
│  R1: signal(mutex)         mutex=1        locks rw      │
│  R1: READING...                                         │
│                                                          │
│  rw_mutex=0, read_count=1                               │
└────────────────────────────────────────────────────────┘

Step 2: R2 arrives (while R1 reading)
┌────────────────────────────────────────────────────────┐
│  R2: wait(mutex)           mutex=0                      │
│  R2: read_count++ = 2                                   │
│  R2: read_count != 1, skip wait(rw_mutex)              │
│  R2: signal(mutex)         mutex=1                      │
│  R2: READING...            ◄── No need to wait!        │
│                                                          │
│  rw_mutex=0, read_count=2                               │
└────────────────────────────────────────────────────────┘

Step 3: W1 arrives (while R1, R2 reading)
┌────────────────────────────────────────────────────────┐
│  W1: wait(rw_mutex)        BLOCKED!                     │
│                                                          │
│  Writer must wait for all readers to finish            │
└────────────────────────────────────────────────────────┘

Step 4: R1 finishes
┌────────────────────────────────────────────────────────┐
│  R1: wait(mutex)           mutex=0                      │
│  R1: read_count-- = 1                                   │
│  R1: read_count != 0, skip signal(rw_mutex)            │
│  R1: signal(mutex)         mutex=1                      │
│                                                          │
│  R2 still reading, W1 still blocked                    │
└────────────────────────────────────────────────────────┘

Step 5: R2 finishes
┌────────────────────────────────────────────────────────┐
│  R2: wait(mutex)           mutex=0                      │
│  R2: read_count-- = 0                                   │
│  R2: signal(rw_mutex)      rw_mutex=1  ◄── Last reader │
│  R2: signal(mutex)         mutex=1        unlocks rw   │
│                                                          │
│  W1 unblocked! W1: WRITING...                          │
└────────────────────────────────────────────────────────┘


Writer Starvation Problem:

┌────────────────────────────────────────────────────────┐
│  Time ──────────────────────────────────────────►      │
│                                                         │
│  R1: ─────────[reading]────────────────────────►       │
│  R2:     ────────[reading]─────────────────────►       │
│  R3:         ──────[reading]───────────────────►       │
│  R4:             ────────[reading]─────────────►       │
│                                                         │
│  W1:     ┬──────────────[waiting]──────────────►       │
│          │                                              │
│          └── Writer never gets a chance!               │
│              (Readers keep arriving)                    │
└────────────────────────────────────────────────────────┘
```

### The Dining Philosophers Problem (식사하는 철학자 문제)

#### Concept (개념)

The **Dining Philosophers Problem** is a classic synchronization problem illustrating deadlock and resource allocation challenges.

**Setup:**
- 5 philosophers sit at a round table
- Each philosopher alternates between thinking and eating
- 5 chopsticks, one between each pair of philosophers
- A philosopher needs BOTH chopsticks to eat

**Challenges:**
- Deadlock: All pick up left chopstick, wait for right forever
- Starvation: A philosopher may never get both chopsticks

#### Diagram / Example

```
Dining Philosophers Problem (식사하는 철학자 문제)

The Setup:
┌─────────────────────────────────────────────────────────┐
│                                                          │
│                      [P0]                                │
│                    /      \                              │
│                 C4          C0                           │
│                /              \                          │
│             [P4]              [P1]                       │
│              |                  |                        │
│             C3                 C1                        │
│              |                  |                        │
│             [P3]──────C2──────[P2]                       │
│                                                          │
│  Pi = Philosopher i                                      │
│  Ci = Chopstick i                                        │
│                                                          │
│  Each philosopher needs chopsticks on both sides        │
│  P0 needs C4 and C0                                     │
│  P1 needs C0 and C1                                     │
│  ...                                                     │
└─────────────────────────────────────────────────────────┘


Simple (Deadlock-Prone) Solution:

┌─────────────────────────────────────────────────────────┐
│  semaphore chopstick[5] = {1, 1, 1, 1, 1};              │
│                                                          │
│  // Philosopher i                                        │
│  while (true) {                                          │
│      think();                                            │
│      wait(chopstick[i]);           // Pick up left      │
│      wait(chopstick[(i+1) % 5]);   // Pick up right     │
│      eat();                                              │
│      signal(chopstick[i]);         // Put down left     │
│      signal(chopstick[(i+1) % 5]); // Put down right    │
│  }                                                       │
└─────────────────────────────────────────────────────────┘


Deadlock Scenario:

Step 1: All philosophers get hungry simultaneously

┌─────────────────────────────────────────────────────────┐
│                      [P0]                                │
│                     picks                                │
│                    /      \                              │
│                 C4↑         ↑C0                          │
│                /              \                          │
│             [P4]              [P1]                       │
│            picks↑              ↑picks                    │
│             C3                 C1                        │
│              ↑                  ↑                        │
│           picks              picks                       │
│             [P3]──────C2──────[P2]                       │
│                      ↑                                   │
│                    picks                                 │
│                                                          │
│  Each philosopher picks up LEFT chopstick               │
│  P0: has C4, waiting for C0                             │
│  P1: has C0, waiting for C1                             │
│  P2: has C1, waiting for C2                             │
│  P3: has C2, waiting for C3                             │
│  P4: has C3, waiting for C4                             │
│                                                          │
│  CIRCULAR WAIT → DEADLOCK!                              │
└─────────────────────────────────────────────────────────┘


Solutions to Prevent Deadlock:

Solution 1: Allow at most 4 philosophers to sit
┌─────────────────────────────────────────────────────────┐
│  semaphore room = 4;  // Only 4 can sit                 │
│                                                          │
│  while (true) {                                          │
│      wait(room);                   // Enter room        │
│      wait(chopstick[i]);                                │
│      wait(chopstick[(i+1) % 5]);                        │
│      eat();                                              │
│      signal(chopstick[i]);                              │
│      signal(chopstick[(i+1) % 5]);                      │
│      signal(room);                 // Leave room        │
│  }                                                       │
│                                                          │
│  With only 4 philosophers, at least one can eat!        │
└─────────────────────────────────────────────────────────┘

Solution 2: Asymmetric - Odd picks left first, Even picks right first
┌─────────────────────────────────────────────────────────┐
│  // Philosopher i                                        │
│  if (i % 2 == 0) {  // Even philosopher                 │
│      wait(chopstick[(i+1) % 5]);  // Right first        │
│      wait(chopstick[i]);          // Then left          │
│  } else {           // Odd philosopher                  │
│      wait(chopstick[i]);          // Left first         │
│      wait(chopstick[(i+1) % 5]);  // Then right         │
│  }                                                       │
│  eat();                                                  │
│  // Release both                                         │
│                                                          │
│  Breaks circular wait!                                   │
└─────────────────────────────────────────────────────────┘

Solution 3: Pick up both chopsticks atomically (Monitor)
┌─────────────────────────────────────────────────────────┐
│  monitor DiningPhilosophers {                            │
│      enum {THINKING, HUNGRY, EATING} state[5];          │
│      condition self[5];                                  │
│                                                          │
│      void pickup(int i) {                                │
│          state[i] = HUNGRY;                              │
│          test(i);           // Try to eat               │
│          if (state[i] != EATING)                        │
│              self[i].wait();                             │
│      }                                                   │
│                                                          │
│      void putdown(int i) {                               │
│          state[i] = THINKING;                            │
│          test((i+4) % 5);   // Check left neighbor      │
│          test((i+1) % 5);   // Check right neighbor     │
│      }                                                   │
│                                                          │
│      void test(int i) {                                  │
│          if (state[(i+4)%5] != EATING &&                │
│              state[i] == HUNGRY &&                       │
│              state[(i+1)%5] != EATING) {                │
│              state[i] = EATING;                          │
│              self[i].signal();                           │
│          }                                               │
│      }                                                   │
│  }                                                       │
└─────────────────────────────────────────────────────────┘


State Transition Diagram:

┌───────────────────────────────────────────────────────┐
│                                                        │
│              ┌──────────┐                              │
│              │ THINKING │                              │
│              └────┬─────┘                              │
│                   │                                    │
│                   │ pickup()                           │
│                   ▼                                    │
│              ┌──────────┐                              │
│         ┌────│  HUNGRY  │────┐                        │
│         │    └──────────┘    │                        │
│         │                    │                        │
│    Neighbors          Both chopsticks                  │
│    eating             available                        │
│         │                    │                        │
│         ▼                    ▼                        │
│    ┌────────┐          ┌──────────┐                   │
│    │ WAIT   │          │  EATING  │                   │
│    └────┬───┘          └────┬─────┘                   │
│         │                   │                         │
│    signal()                 │ putdown()               │
│    from neighbor            │                         │
│         │                   ▼                         │
│         └──────────► ┌──────────┐                     │
│                      │ THINKING │                     │
│                      └──────────┘                     │
│                                                        │
└───────────────────────────────────────────────────────┘
```

---

## Summary (요약)

| Concept | Description |
|---------|-------------|
| Mutex (뮤텍스) | Binary lock for mutual exclusion; owner must release |
| Semaphore (세마포어) | Integer counter with wait/signal operations |
| Binary Semaphore | Semaphore with value 0 or 1 (like mutex) |
| Counting Semaphore | Semaphore for controlling N resources |
| Monitor (모니터) | High-level construct with automatic mutual exclusion |
| Condition Variable (조건 변수) | Wait/signal mechanism within monitors |
| Producer-Consumer (생산자-소비자) | Shared buffer problem |
| Readers-Writers (독자-저자) | Multiple readers OR one writer |
| Dining Philosophers (식사하는 철학자) | Circular resource allocation problem |

---

## Key Terms (주요 용어)

- **Acquire/Release (획득/해제)**: Mutex lock/unlock operations
- **Wait/Signal (대기/시그널)**: Semaphore P/V operations (also called P/V, down/up)
- **Blocking (블로킹)**: Process sleeps until condition is met
- **Busy Waiting (바쁜 대기)**: Continuously checking condition (spinning)
- **Deadlock (데드락)**: Circular wait where no process can proceed
- **Starvation (기아)**: Process indefinitely denied resources
- **Priority Inversion (우선순위 역전)**: High-priority process blocked by low-priority process
