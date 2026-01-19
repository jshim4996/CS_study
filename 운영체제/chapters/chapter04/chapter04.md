# Chapter 04. CPU Scheduling (CPU 스케줄링)

> CPU scheduling determines which process runs when, maximizing CPU utilization and system performance.

---

## Scheduling Concepts (스케줄링 개념)

### Concept (개념)

**CPU Scheduling (CPU 스케줄링)** is the basis of multiprogrammed operating systems. By switching the CPU among processes, the OS makes the computer more productive.

**CPU-I/O Burst Cycle:**
- Process execution alternates between CPU execution and I/O wait
- **CPU Burst (CPU 버스트)**: Time spent executing instructions
- **I/O Burst (I/O 버스트)**: Time spent waiting for I/O
- CPU-bound processes: Long CPU bursts
- I/O-bound processes: Short CPU bursts

**When CPU Scheduling Decisions Occur:**
1. Process switches from running to waiting (I/O request)
2. Process switches from running to ready (interrupt, preemption)
3. Process switches from waiting to ready (I/O completion)
4. Process terminates

**Non-preemptive vs Preemptive:**
- **Non-preemptive (비선점형)**: Process keeps CPU until it voluntarily releases (cases 1, 4)
- **Preemptive (선점형)**: OS can take CPU away from a process (all cases)

### Diagram / Example

```
CPU-I/O Burst Cycle (CPU-I/O 버스트 사이클)

┌───────────────────────────────────────────────────────────┐
│                    Process Execution                       │
│                                                           │
│  ┌─────────┐   ┌─────────┐   ┌─────────┐   ┌─────────┐   │
│  │  CPU    │   │   I/O   │   │  CPU    │   │   I/O   │   │
│  │  Burst  │──►│   Wait  │──►│  Burst  │──►│   Wait  │──►│
│  │         │   │         │   │         │   │         │   │
│  └─────────┘   └─────────┘   └─────────┘   └─────────┘   │
│      5ms          15ms          8ms           20ms        │
└───────────────────────────────────────────────────────────┘

CPU Burst Distribution (CPU 버스트 분포)

Frequency
    │
    │ ██
    │ ██
    │ ██ ██
    │ ██ ██
    │ ██ ██ ██
    │ ██ ██ ██ ██
    │ ██ ██ ██ ██ ██
    │ ██ ██ ██ ██ ██ ██ ██
    └──────────────────────────► CPU Burst Duration
      Short         Long

Most processes are I/O bound (short bursts)

Scheduling Decision Points (스케줄링 결정 시점)

         ┌─────────┐
         │   New   │
         └────┬────┘
              │ admitted
              ▼
         ┌─────────┐        ②              ┌───────────┐
    ┌───►│  Ready  │◄─────interrupt────────│  Running  │
    │    └────┬────┘                       └─────┬─────┘
    │         │                                  │
    │         │ ① scheduler dispatch            │ ① I/O or
    │         └──────────────────────────────────► event wait
    │                                            │
    │                     ③ I/O done             ▼
    └───────────────────────────────────────┌─────────┐
                                            │ Waiting │
                                            └─────────┘
                           ④ exit
                              │
                              ▼
                        ┌────────────┐
                        │ Terminated │
                        └────────────┘

① Non-preemptive decisions (비선점)
② Preemptive decisions (선점)
```

---

## Scheduling Criteria (스케줄링 기준)

### Concept (개념)

| Criterion | Korean | Description | Goal |
|-----------|--------|-------------|------|
| **CPU Utilization** | CPU 이용률 | Percentage of time CPU is busy | Maximize |
| **Throughput** | 처리량 | Number of processes completed per time unit | Maximize |
| **Turnaround Time** | 총처리 시간 | Time from submission to completion | Minimize |
| **Waiting Time** | 대기 시간 | Time spent waiting in ready queue | Minimize |
| **Response Time** | 응답 시간 | Time from submission to first response | Minimize |

**Formulas:**
- Turnaround Time = Completion Time - Arrival Time
- Waiting Time = Turnaround Time - Burst Time
- Response Time = First Response Time - Arrival Time

**Trade-offs:**
- Optimizing one criterion may hurt another
- Interactive systems prioritize response time
- Batch systems prioritize throughput

### Diagram / Example

```
Scheduling Criteria Visualization

Process Timeline Example:

Arrival ──────────► First Run ─────────► Completion
   │                    │                     │
   │◄───Response Time───►│                    │
   │                                          │
   │◄────────── Turnaround Time ─────────────►│
   │                                          │
        │◄─Waiting─►│◄──Running──►│◄─Waiting─►│


Example Calculation:

Process   Arrival   Burst   Completion   Turnaround   Waiting
                    Time       Time         Time        Time
────────────────────────────────────────────────────────────
  P1        0        24         24           24          0
  P2        0         3         27           27         24
  P3        0         3         30           30         27
────────────────────────────────────────────────────────────
Average:                                    27.0       17.0


Throughput Visualization:

Time ──────────────────────────────────────────────────►
      0    5    10   15   20   25   30   35   40
      │    │    │    │    │    │    │    │    │
      ├────┼────┼────┼────┼────┼────┼────┼────┤
      │ P1 │ P2 │ P3 │ P4 │    │    │    │    │

      4 processes in 20 time units
      Throughput = 4/20 = 0.2 processes/time unit
```

---

## First-Come, First-Served (FCFS, 선입선처리)

### Concept (개념)

**FCFS (First-Come, First-Served)**, also known as FIFO, is the simplest scheduling algorithm. Processes are executed in the order they arrive.

**Characteristics:**
- Non-preemptive (비선점형)
- Easy to implement using FIFO queue
- Average waiting time can be long
- Convoy Effect (호송 효과): Short processes wait behind long ones

**Convoy Effect:**
When a CPU-bound process holds the CPU, many I/O-bound processes wait, leading to poor device utilization.

### Diagram / Example

```
FCFS Scheduling Example

Process   Arrival Time   Burst Time
   P1          0            24
   P2          0             3
   P3          0             3

Order of Arrival: P1, P2, P3

Gantt Chart (간트 차트):
┌────────────────────────────────┬─────┬─────┐
│              P1                │ P2  │ P3  │
└────────────────────────────────┴─────┴─────┘
0                               24    27    30

Results:
Process   Completion   Turnaround   Waiting
                Time       Time       Time
  P1         24           24          0
  P2         27           27         24
  P3         30           30         27
────────────────────────────────────────────
Average                  27.0       17.0

Different Order (P2, P3, P1):
┌─────┬─────┬────────────────────────────────┐
│ P2  │ P3  │              P1                │
└─────┴─────┴────────────────────────────────┘
0     3     6                               30

Results:
Process   Completion   Turnaround   Waiting
                Time       Time       Time
  P1         30           30          6
  P2          3            3          0
  P3          6            6          3
────────────────────────────────────────────
Average                  13.0        3.0

Order matters significantly!

Convoy Effect (호송 효과):
┌────────────────────────────────────────────────────┐
│        CPU-bound process holding CPU               │
│  ┌────────────────────────────────────────┐       │
│  │          Long CPU Burst                 │       │
│  └────────────────────────────────────────┘       │
│                    │                              │
│    ┌───┐ ┌───┐ ┌───┐ (waiting I/O-bound)         │
│    │ P2│ │ P3│ │ P4│                             │
│    └───┘ └───┘ └───┘                             │
│    Short processes starving!                      │
└────────────────────────────────────────────────────┘
```

---

## Shortest Job First (SJF, 최단 작업 우선)

### Concept (개념)

**SJF (Shortest Job First)** schedules the process with the smallest CPU burst next. It is provably optimal for minimizing average waiting time.

**Two Versions:**
1. **Non-preemptive SJF**: Once CPU is given, process runs to completion
2. **Preemptive SJF (SRTF)**: If new process has shorter burst than remaining time of current process, preempt (Shortest Remaining Time First)

**Challenge:**
- Cannot know exact burst time in advance
- Must predict using exponential averaging:
  τ(n+1) = α × t(n) + (1-α) × τ(n)
  - τ(n+1): Predicted next burst
  - t(n): Actual last burst
  - τ(n): Predicted last burst
  - α: Weight (typically 0.5)

### Diagram / Example

```
Non-preemptive SJF Example

Process   Arrival Time   Burst Time
   P1          0             7
   P2          2             4
   P3          4             1
   P4          5             4

Gantt Chart:
┌───────────────┬───┬──────────┬──────────┐
│      P1       │P3 │    P2    │    P4    │
└───────────────┴───┴──────────┴──────────┘
0               7   8         12         16

At time 0: Only P1 available → Run P1
At time 7: P2(4), P3(1), P4(4) available → P3 shortest

Results:
Process   Completion   Turnaround   Waiting
  P1          7            7          0
  P2         12           10          6
  P3          8            4          3
  P4         16           11          7
────────────────────────────────────────────
Average                   8.0        4.0


Preemptive SJF (SRTF) Example

Process   Arrival Time   Burst Time
   P1          0             7
   P2          2             4
   P3          4             1
   P4          5             4

Gantt Chart:
┌───┬──────┬───┬───┬───────────┬──────────┐
│P1 │  P2  │P3 │P2 │    P4     │    P1    │
└───┴──────┴───┴───┴───────────┴──────────┘
0   2      4   5   7          11         16

Time 0: P1 starts (remaining: 7)
Time 2: P2 arrives (4 < 5 remaining) → preempt, run P2
Time 4: P3 arrives (1 < 2 remaining) → preempt, run P3
Time 5: P3 done, P4 arrives, P2 remaining(2) < P4(4) → run P2
Time 7: P2 done, P4(4) < P1(5) → run P4
Time 11: P4 done → run P1

Results:
Process   Completion   Turnaround   Waiting
  P1         16           16          9
  P2          7            5          1
  P3          5            1          0
  P4         11            6          2
────────────────────────────────────────────
Average                   7.0        3.0


Burst Time Prediction (버스트 시간 예측):

τ(n+1) = α × t(n) + (1-α) × τ(n)

Example with α = 0.5:
┌──────┬────────┬──────────┬────────────────┐
│  n   │  t(n)  │   τ(n)   │    τ(n+1)      │
├──────┼────────┼──────────┼────────────────┤
│  1   │   6    │   10     │ 0.5×6+0.5×10=8 │
│  2   │   4    │    8     │ 0.5×4+0.5×8=6  │
│  3   │   6    │    6     │ 0.5×6+0.5×6=6  │
│  4   │   4    │    6     │ 0.5×4+0.5×6=5  │
└──────┴────────┴──────────┴────────────────┘
```

---

## Priority Scheduling (우선순위 스케줄링)

### Concept (개념)

**Priority Scheduling (우선순위 스케줄링)** assigns a priority number to each process. The CPU is allocated to the process with the highest priority.

**Priority Assignment:**
- Internal: Based on measurable factors (time limits, memory needs, I/O ratio)
- External: Based on importance, department, payment

**Conventions:**
- Some systems: Lower number = higher priority
- Other systems: Higher number = higher priority

**Problem - Starvation (기아, 무한대기):**
Low-priority processes may never execute.

**Solution - Aging (에이징):**
Gradually increase priority of waiting processes over time.

### Diagram / Example

```
Priority Scheduling Example (Lower number = Higher priority)

Process   Arrival   Burst   Priority
   P1        0        10       3
   P2        0         1       1
   P3        0         2       4
   P4        0         1       5
   P5        0         5       2

Gantt Chart:
┌───┬─────────┬────────────────┬────┬───┐
│P2 │   P5    │       P1       │ P3 │P4 │
└───┴─────────┴────────────────┴────┴───┘
0   1         6               16   18  19

Execution Order by Priority:
P2 (priority 1) → P5 (priority 2) → P1 (priority 3)
→ P3 (priority 4) → P4 (priority 5)

Results:
Process   Completion   Turnaround   Waiting
  P1         16           16          6
  P2          1            1          0
  P3         18           18         16
  P4         19           19         18
  P5          6            6          1
────────────────────────────────────────────
Average                  12.0        8.2


Starvation Problem (기아 문제):

High Priority Processes Keep Arriving
         │
         ▼
┌────┐ ┌────┐ ┌────┐ ┌────┐
│ HP │ │ HP │ │ HP │ │ HP │  ──► Always running
└────┘ └────┘ └────┘ └────┘

         ┌────┐
         │ LP │  ──► Never gets CPU! (Starvation)
         └────┘


Aging Solution (에이징 해결책):

Time    Process   Original    Waiting    Effective
                  Priority    Time       Priority
────────────────────────────────────────────────────
  0       P1         10         0          10
 10       P1         10        10          10-1=9
 20       P1         10        20          10-2=8
 30       P1         10        30          10-3=7
 ...      ...        ...       ...         ...

Eventually low-priority process gets high enough priority to run!
```

---

## Round Robin (RR, 라운드 로빈)

### Concept (개념)

**Round Robin (RR)** is designed for time-sharing systems. Each process gets a small unit of CPU time called a **time quantum (시간 할당량)** or **time slice (타임 슬라이스)**.

**Characteristics:**
- Preemptive version of FCFS
- Process runs for at most one quantum, then moves to back of ready queue
- Fair: Each process gets equal share of CPU
- Good response time for interactive systems

**Time Quantum Selection:**
- Too large: Behaves like FCFS
- Too small: Too much context switch overhead
- Rule of thumb: 80% of CPU bursts should be shorter than quantum
- Typical values: 10-100 milliseconds

### Diagram / Example

```
Round Robin Example (Time Quantum = 4)

Process   Arrival Time   Burst Time
   P1          0            24
   P2          0             3
   P3          0             3

Ready Queue Evolution:
Time 0:  [P1, P2, P3] → Run P1 for 4
Time 4:  [P2, P3, P1] → Run P2 for 3 (completes)
Time 7:  [P3, P1] → Run P3 for 3 (completes)
Time 10: [P1] → Run P1 for 4
Time 14: [P1] → Run P1 for 4
...

Gantt Chart:
┌────┬───┬───┬────┬────┬────┬────┬────┐
│ P1 │P2 │P3 │ P1 │ P1 │ P1 │ P1 │ P1 │
└────┴───┴───┴────┴────┴────┴────┴────┘
0    4   7  10   14   18   22   26   30

Results:
Process   Completion   Turnaround   Waiting
  P1         30           30          6
  P2          7            7          4
  P3         10           10          7
────────────────────────────────────────────
Average                  15.67       5.67

Compared to FCFS: Better response time!


Time Quantum Effect (시간 할당량 효과):

Quantum too small (q = 1):
┌─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┐...
│P│P│P│P│P│P│P│P│P│P│P│P│
│1│2│3│1│2│3│1│2│3│1│2│3│
└─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┘
Too many context switches! (오버헤드 증가)

Quantum too large (q = ∞):
┌──────────────────────────┬───┬───┐
│           P1             │P2 │P3 │
└──────────────────────────┴───┴───┘
Same as FCFS! (FCFS와 동일)

Quantum just right:
┌────────┬────────┬────────┬────────┐
│   P1   │   P2   │   P3   │   P1   │...
└────────┴────────┴────────┴────────┘
Balance between fairness and overhead


Rule of 80%:
┌─────────────────────────────────────────────┐
│                                             │
│  CPU Burst Distribution                     │
│      │                                      │
│  Freq│ ██                                   │
│      │ ██ ██                                │
│      │ ██ ██ ██                             │
│      │ ██ ██ ██ ██ ██                       │
│      └────────────┼─────────────────►       │
│                   │                         │
│              Time Quantum                   │
│                   │                         │
│    ◄── 80% ──►    │                         │
│    complete in    │                         │
│    one quantum    │                         │
└─────────────────────────────────────────────┘
```

---

## Multilevel Queue Scheduling (다단계 큐 스케줄링)

### Concept (개념)

**Multilevel Queue Scheduling** partitions the ready queue into separate queues based on process properties. Each queue can have its own scheduling algorithm.

**Common Queue Structure:**
1. System processes - highest priority
2. Interactive processes
3. Interactive editing processes
4. Batch processes
5. Student processes - lowest priority

**Scheduling Between Queues:**
- Fixed priority: Serve all from higher queue before lower
- Time slice: Each queue gets percentage of CPU time (e.g., 80% interactive, 20% batch)

**Multilevel Feedback Queue:**
- Processes can move between queues
- Processes using too much CPU time → lower queue
- I/O-bound and interactive processes → higher queue
- Aging can be implemented

### Diagram / Example

```
Multilevel Queue Scheduling (다단계 큐)

                           highest priority
┌──────────────────────────────────────────────┐
│    System Processes      │ Priority, RR(q=8) │◄──┐
├──────────────────────────────────────────────┤   │
│    Interactive Processes │ RR (quantum=4)    │   │
├──────────────────────────────────────────────┤   │ Fixed
│    Interactive Editing   │ RR (quantum=2)    │   │ Priority
├──────────────────────────────────────────────┤   │
│    Batch Processes       │ FCFS              │   │
├──────────────────────────────────────────────┤   │
│    Student Processes     │ FCFS              │◄──┘
└──────────────────────────────────────────────┘
                           lowest priority

Process stays in assigned queue permanently.


Multilevel Feedback Queue (다단계 피드백 큐)

┌─────────────────────────────────────────────────────┐
│  Queue 0 (RR, quantum=8)    Highest Priority        │
│  ┌────────────────────────────────────────────┐     │
│  │  P1 ──► P3 ──► P5                          │     │
│  └────────────────────────────────────────────┘     │
│              │ timeout (didn't finish in 8ms)       │
│              ▼                                      │
│  Queue 1 (RR, quantum=16)                          │
│  ┌────────────────────────────────────────────┐     │
│  │  P2 ──► P7                                 │     │
│  └────────────────────────────────────────────┘     │
│              │ timeout (didn't finish in 16ms)      │
│              ▼                                      │
│  Queue 2 (FCFS)             Lowest Priority         │
│  ┌────────────────────────────────────────────┐     │
│  │  P4 ──► P6 ──► P8   (CPU-bound processes)  │     │
│  └────────────────────────────────────────────┘     │
└─────────────────────────────────────────────────────┘

New processes enter Queue 0
If they don't finish in time quantum, move to lower queue
I/O-bound processes stay in high queues (good response)
CPU-bound processes sink to lower queues


Example Execution:

Process P enters system (new process)
         │
         ▼
┌─────────────────┐
│ Queue 0 (q=8)   │
│     P runs      │────► Finished in 8ms? ─Yes─► Done!
└────────┬────────┘            │
         │                     No
         │ demoted             │
         ▼                     ▼
┌─────────────────┐
│ Queue 1 (q=16)  │
│     P runs      │────► Finished in 16ms? ─Yes─► Done!
└────────┬────────┘            │
         │                     No
         │ demoted             │
         ▼                     ▼
┌─────────────────┐
│ Queue 2 (FCFS)  │
│  P waits, runs  │────► Finished eventually
└─────────────────┘


Parameters to Define:
1. Number of queues
2. Scheduling algorithm for each queue
3. Method to determine when to promote/demote
4. Method to determine which queue for new process
```

---

## Scheduling Algorithm Comparison (스케줄링 알고리즘 비교)

### Concept (개념)

| Algorithm | Preemptive | Starvation | Overhead | Best For |
|-----------|------------|------------|----------|----------|
| **FCFS** | No | No | Low | Batch systems |
| **SJF** | No/Yes | Yes | Medium | Shortest wait time |
| **SRTF** | Yes | Yes | High | Minimum wait time |
| **Priority** | No/Yes | Yes | Medium | Prioritized workloads |
| **RR** | Yes | No | High | Time-sharing |
| **Multilevel Queue** | Yes | Yes | Medium | Mixed workloads |
| **Multilevel Feedback** | Yes | No (aging) | High | General purpose |

### Diagram / Example

```
Algorithm Selection Guide (알고리즘 선택 가이드)

                    What are your requirements?
                              │
              ┌───────────────┼───────────────┐
              │               │               │
          Batch           Time-sharing    Mixed/Complex
          System             System          System
              │               │               │
              ▼               ▼               ▼
         ┌────────┐      ┌────────┐     ┌──────────────┐
         │  FCFS  │      │   RR   │     │  Multilevel  │
         │  or    │      │        │     │  Feedback    │
         │  SJF   │      │        │     │    Queue     │
         └────────┘      └────────┘     └──────────────┘


Performance Comparison (Average Waiting Time):

Given: P1(burst=10), P2(burst=1), P3(burst=2), P4(burst=1), P5(burst=5)
All arrive at time 0

FCFS (order: P1,P2,P3,P4,P5):
┌──────────────┬───┬────┬───┬───────┐
│      P1      │P2 │ P3 │P4 │  P5   │
└──────────────┴───┴────┴───┴───────┘
0             10  11   13  14      19
Avg Wait = (0+10+11+13+14)/5 = 9.6

SJF (order: P2,P4,P3,P5,P1):
┌───┬───┬────┬───────┬──────────────┐
│P2 │P4 │ P3 │  P5   │      P1      │
└───┴───┴────┴───────┴──────────────┘
0   1   2    4       9             19
Avg Wait = (9+0+2+1+4)/5 = 3.2

RR (q=2):
┌──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┐
│P1│P2│P3│P4│P5│P1│P5│P1│P5│P1│P1│P1│
└──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┘
0  2  3  5  6  8 10 12 14 15 17 19
(P2, P3, P4 complete early; P5 at 15; P1 at 19)
Avg Wait = ...

Summary:
┌────────────┬───────────────┐
│ Algorithm  │ Avg Wait Time │
├────────────┼───────────────┤
│ FCFS       │     9.6       │
│ SJF        │     3.2       │ ◄── Optimal
│ RR (q=2)   │     ~6.0      │
└────────────┴───────────────┘
```

---

## Summary (요약)

| Concept | Description |
|---------|-------------|
| CPU Scheduling (CPU 스케줄링) | Selecting which process runs on CPU |
| Preemptive (선점형) | OS can take CPU from running process |
| Non-preemptive (비선점형) | Process keeps CPU until it releases |
| FCFS (선입선처리) | First come, first served |
| SJF (최단작업우선) | Shortest burst time first |
| Priority (우선순위) | Highest priority first |
| RR (라운드로빈) | Time quantum-based fair scheduling |
| Multilevel Queue (다단계 큐) | Multiple queues with different algorithms |
| Starvation (기아) | Process never gets CPU |
| Aging (에이징) | Increase priority over time |

---

## Key Terms (주요 용어)

- **Dispatcher (디스패처)**: Module that gives CPU control to selected process
- **Dispatch Latency (디스패치 지연)**: Time for dispatcher to stop one process and start another
- **Time Quantum (시간 할당량)**: Fixed time slice in RR scheduling
- **Convoy Effect (호송 효과)**: Short processes waiting behind long one in FCFS
- **Turnaround Time (총처리 시간)**: Total time from submission to completion
