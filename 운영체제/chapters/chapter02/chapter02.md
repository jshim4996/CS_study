# Chapter 02. Processes (프로세스)

> A process is a program in execution, the fundamental unit of work in an operating system.

---

## Process Concept (프로세스 개념)

### Concept (개념)

A **Process (프로세스)** is a program in execution. While a program is a passive entity (executable file on disk), a process is an active entity with a program counter and associated resources.

**Process Memory Layout:**

| Section | Description |
|---------|-------------|
| **Text Section (코드 영역)** | Program code (executable instructions) |
| **Data Section (데이터 영역)** | Global and static variables |
| **Heap (힙)** | Dynamically allocated memory (malloc, new) |
| **Stack (스택)** | Temporary data (function parameters, local variables, return addresses) |

**Process vs Program:**
- Program: Static, stored on disk
- Process: Dynamic, loaded in memory
- One program can have multiple processes (multiple instances)

### Diagram / Example

```
Process Memory Layout (프로세스 메모리 구조)

High Address
┌─────────────────────┐
│       Stack         │ ← Local variables, function calls
│         ↓           │   (grows downward)
│                     │
│         ↑           │
│        Heap         │ ← Dynamic allocation (malloc)
│                     │   (grows upward)
├─────────────────────┤
│   Uninitialized     │
│   Data (BSS)        │ ← Uninitialized global/static vars
├─────────────────────┤
│   Initialized       │
│   Data              │ ← Initialized global/static vars
├─────────────────────┤
│       Text          │ ← Program code (read-only)
└─────────────────────┘
Low Address

// Example in C
int global_var = 100;        // Data section
int uninit_global;           // BSS section

int main() {
    int local_var = 10;      // Stack
    static int static_var;   // BSS section

    int *ptr = malloc(100);  // Heap

    return 0;                // Text section contains this code
}
```

---

## Process States (프로세스 상태)

### Concept (개념)

A process transitions through various states during its lifetime:

| State | Korean | Description |
|-------|--------|-------------|
| **New** | 생성 | Process is being created |
| **Ready** | 준비 | Process waiting to be assigned to CPU |
| **Running** | 실행 | Instructions are being executed |
| **Waiting** | 대기 | Process waiting for an event (I/O completion) |
| **Terminated** | 종료 | Process has finished execution |

**State Transitions:**
- New → Ready: Admitted to ready queue
- Ready → Running: Scheduler dispatches process
- Running → Ready: Interrupt (timer, preemption)
- Running → Waiting: I/O request or event wait
- Waiting → Ready: I/O completion or event occurs
- Running → Terminated: Process completes or exits

### Diagram / Example

```
Process State Diagram (프로세스 상태 다이어그램)

                    admitted
        ┌────────────────────────┐
        │                        ▼
    ┌───────┐               ┌─────────┐
    │  New  │               │  Ready  │◄──────────┐
    └───────┘               └────┬────┘           │
                                 │                │
                    scheduler    │                │ I/O or event
                    dispatch     │                │ completion
                                 ▼                │
                           ┌──────────┐           │
              ┌────────────│ Running  │───────────┤
              │            └────┬─────┘           │
              │                 │                 │
    interrupt │                 │ I/O or         │
   (preemption)                 │ event wait     │
              │                 ▼                │
              │          ┌───────────┐           │
              └─────────►│  Waiting  │───────────┘
                         └───────────┘

                    exit │
                         ▼
                  ┌────────────┐
                  │ Terminated │
                  └────────────┘
```

---

## Process Control Block (프로세스 제어 블록, PCB)

### Concept (개념)

The **Process Control Block (PCB)**, also called Task Control Block, is a data structure maintained by the OS for each process. It contains all information needed to manage the process.

**PCB Contents:**

| Field | Description |
|-------|-------------|
| **Process State (프로세스 상태)** | Current state (new, ready, running, etc.) |
| **Program Counter (프로그램 카운터)** | Address of next instruction to execute |
| **CPU Registers (CPU 레지스터)** | Contents of all process-centric registers |
| **CPU Scheduling Info (스케줄링 정보)** | Priority, scheduling queue pointers |
| **Memory Management Info (메모리 관리 정보)** | Page tables, segment tables, base/limit registers |
| **Accounting Info (계정 정보)** | CPU time used, time limits, process numbers |
| **I/O Status Info (I/O 상태 정보)** | List of open files, I/O devices allocated |

### Diagram / Example

```
Process Control Block (PCB) Structure

┌─────────────────────────────────────┐
│         Process Control Block       │
├─────────────────────────────────────┤
│  Process ID (PID): 1234             │
├─────────────────────────────────────┤
│  Process State: Running             │
├─────────────────────────────────────┤
│  Program Counter: 0x0040156A        │
├─────────────────────────────────────┤
│  CPU Registers:                     │
│    EAX: 0x00000005                  │
│    EBX: 0x00001000                  │
│    ESP: 0x7FFFFFFF                  │
│    ...                              │
├─────────────────────────────────────┤
│  Scheduling Info:                   │
│    Priority: 10                     │
│    Queue Pointer: 0x...             │
├─────────────────────────────────────┤
│  Memory Info:                       │
│    Base Register: 0x00100000        │
│    Limit Register: 0x00050000       │
│    Page Table Ptr: 0x...            │
├─────────────────────────────────────┤
│  I/O Status:                        │
│    Open Files: [fd0, fd1, fd2...]   │
├─────────────────────────────────────┤
│  Accounting:                        │
│    CPU Time: 00:05:32               │
│    Start Time: 2024-01-15 10:30:00  │
└─────────────────────────────────────┘

// Linux: View process info
$ cat /proc/[pid]/status
$ ps aux
```

---

## Context Switching (문맥 교환)

### Concept (개념)

**Context Switching (문맥 교환)** is the process of saving the state of a currently running process and restoring the state of another process. This allows multiple processes to share a single CPU.

**Context Switch Steps:**
1. Save context of currently running process to its PCB
2. Move current process to appropriate queue (ready/waiting)
3. Select next process from ready queue (scheduler decision)
4. Load context of selected process from its PCB
5. Update memory management structures
6. Resume execution of new process

**Context Switch Overhead:**
- Pure overhead (no useful work during switch)
- Time depends on hardware support (register sets, memory speed)
- Typical time: microseconds to milliseconds
- Frequent switches reduce throughput

### Diagram / Example

```
Context Switch Between Process P0 and P1

Process P0          Operating System         Process P1
    │                     │                      │
    │   executing         │                      │
    │──────────────►      │                      │
    │                     │                      │
    │   interrupt or      │                      │
    │   system call       │                      │
    │────────────────────►│                      │
    │                     │                      │
    │               save state to PCB0           │
    │                     │                      │
    │               ┌─────┴─────┐                │
    │               │  Context  │                │
    │               │  Switch   │                │
    │               └─────┬─────┘                │
    │                     │                      │
    │              load state from PCB1          │
    │                     │                      │
    │                     │────────────────────► │
    │                     │                      │
    │                 idle│       executing      │
    │                     │◄──────────────────── │
    │                     │                      │
    │                     │   interrupt or       │
    │                     │◄────────────────────│
    │                     │   system call        │
    │                     │                      │
    │               save state to PCB1           │
    │                     │                      │
    │               ┌─────┴─────┐                │
    │               │  Context  │                │
    │               │  Switch   │                │
    │               └─────┬─────┘                │
    │                     │                      │
    │              load state from PCB0          │
    │◄────────────────────│                      │
    │   executing         │                 idle │
    ▼                     ▼                      ▼
```

---

## Process Creation: fork() and exec() (프로세스 생성)

### Concept (개념)

**fork() System Call:**
- Creates a new process (child) by duplicating the calling process (parent)
- Child gets a copy of parent's address space
- Returns: 0 to child, child's PID to parent, -1 on error
- After fork(), both processes continue from the next instruction

**exec() Family:**
- Replaces current process image with a new program
- Does not create a new process (same PID)
- Variants: execl(), execv(), execle(), execve(), execlp(), execvp()

**Process Creation Options:**
1. Resource sharing: parent and child share all, some, or no resources
2. Execution: parent and child execute concurrently or parent waits
3. Address space: child duplicates parent or child loads new program

**Process Termination:**
- exit(): Process terminates and returns status to parent
- wait(): Parent waits for child to terminate
- Orphan process (고아 프로세스): Parent terminates before child
- Zombie process (좀비 프로세스): Child terminates but parent hasn't called wait()

### Diagram / Example

```
fork() and exec() Visualization

Parent Process (PID: 100)
┌─────────────────────────┐
│   int main() {          │
│       pid_t pid;        │
│       pid = fork();     │──────────┐
│       ...               │          │ fork() creates
│   }                     │          │ child process
└─────────────────────────┘          │
           │                         │
           │                         ▼
           │              Child Process (PID: 101)
           │              ┌─────────────────────────┐
           │              │   (copy of parent)      │
           │              │   pid = 0 (in child)    │
           │              │   exec("/bin/ls")       │──┐
           │              └─────────────────────────┘  │
           │                         │                 │
           │                         │ exec() replaces │
           │                         ▼                 │
           │              ┌─────────────────────────┐  │
           │              │   /bin/ls program       │◄─┘
           │              │   (same PID: 101)       │
           │              └─────────────────────────┘
           ▼

// Complete example: fork() and exec()
#include <stdio.h>
#include <unistd.h>
#include <sys/wait.h>

int main() {
    pid_t pid = fork();

    if (pid < 0) {
        // Fork failed (포크 실패)
        fprintf(stderr, "Fork failed\n");
        return 1;
    }
    else if (pid == 0) {
        // Child process (자식 프로세스)
        printf("Child: PID = %d\n", getpid());
        execlp("/bin/ls", "ls", "-l", NULL);
        // If exec returns, it failed
        fprintf(stderr, "Exec failed\n");
        return 1;
    }
    else {
        // Parent process (부모 프로세스)
        printf("Parent: Child PID = %d\n", pid);
        int status;
        wait(&status);  // Wait for child (자식 대기)
        printf("Child completed\n");
    }

    return 0;
}

Process Tree Example (프로세스 트리)

              init (PID 1)
                   │
        ┌──────────┼──────────┐
        │          │          │
      sshd       cron      systemd
        │
    ┌───┴───┐
    │       │
  bash    bash
    │
   vim

// View process tree in Linux
$ pstree
$ ps -ef --forest
```

---

## Process Scheduling Queues (프로세스 스케줄링 큐)

### Concept (개념)

The OS maintains several queues to manage processes:

**Job Queue (작업 큐)**
- Contains all processes in the system

**Ready Queue (준비 큐)**
- Contains processes that are ready and waiting for CPU
- Usually implemented as a linked list of PCBs

**Device Queues (장치 큐)**
- Processes waiting for a particular I/O device
- Each device has its own queue

**Schedulers:**
- **Long-term Scheduler (장기 스케줄러)**: Selects processes from job pool to ready queue (controls degree of multiprogramming)
- **Short-term Scheduler (단기 스케줄러, CPU 스케줄러)**: Selects process from ready queue for CPU execution (runs frequently)
- **Medium-term Scheduler (중기 스케줄러)**: Swaps processes in/out of memory (swapping)

### Diagram / Example

```
Process Scheduling Queues

                    ┌─────────────┐
                    │  Job Queue  │
                    │ (all jobs)  │
                    └──────┬──────┘
                           │
              Long-term    │
              Scheduler    ▼
                    ┌─────────────┐
        ┌──────────►│ Ready Queue │◄──────────────────┐
        │           └──────┬──────┘                   │
        │                  │                          │
        │     Short-term   │                          │
        │     Scheduler    ▼                          │
        │           ┌─────────────┐                   │
        │           │     CPU     │                   │
        │           └──────┬──────┘                   │
        │                  │                          │
        │     ┌────────────┼────────────┐             │
        │     │            │            │             │
        │     ▼            ▼            ▼             │
        │ ┌───────┐  ┌───────┐  ┌───────────┐        │
        │ │I/O Req│  │Time   │  │ Child     │        │
        │ │       │  │Expired│  │ Executes  │        │
        │ └───┬───┘  └───┬───┘  └─────┬─────┘        │
        │     │          │            │              │
        │     ▼          │            ▼              │
        │ ┌───────┐      │     ┌───────────┐         │
        │ │Device │      └────►│  Wait for │         │
        │ │Queue  │            │  Interrupt│─────────┘
        │ └───┬───┘            └───────────┘
        │     │
        │     │ I/O Complete
        └─────┘

Medium-term Scheduler (Swapping)
┌────────────┐         ┌────────────┐
│   Memory   │ ◄─────► │    Disk    │
│            │  Swap   │  (Backing  │
│  Processes │  In/Out │   Store)   │
└────────────┘         └────────────┘
```

---

## Summary (요약)

| Concept | Description |
|---------|-------------|
| Process (프로세스) | Program in execution with its own address space |
| PCB (프로세스 제어 블록) | Data structure containing all process information |
| Process State (프로세스 상태) | New, Ready, Running, Waiting, Terminated |
| Context Switch (문맥 교환) | Saving and restoring process states |
| fork() | Creates child process by duplicating parent |
| exec() | Replaces process image with new program |
| Zombie (좀비) | Terminated child waiting for parent's wait() |
| Orphan (고아) | Child whose parent has terminated |

---

## Key Terms (주요 용어)

- **PID (Process ID, 프로세스 ID)**: Unique identifier for each process
- **Parent Process (부모 프로세스)**: Process that created another process
- **Child Process (자식 프로세스)**: Process created by another process
- **Multiprogramming (다중 프로그래밍)**: Multiple processes in memory simultaneously
- **Time Sharing (시분할)**: CPU time divided among processes
