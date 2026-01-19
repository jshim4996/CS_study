# Chapter 03. Threads (스레드)

> A thread is the basic unit of CPU utilization, enabling parallel execution within a single process.

---

## Thread Concept (스레드 개념)

### Concept (개념)

A **Thread (스레드)**, also called a Lightweight Process (LWP, 경량 프로세스), is the smallest unit of CPU scheduling. A thread belongs to a process and shares the process's resources.

**What a Thread Contains (Own Resources):**
- Thread ID
- Program Counter (PC)
- Register Set
- Stack

**What Threads Share (from Process):**
- Code section (text)
- Data section
- Heap
- Open files and signals

**Single-threaded vs Multi-threaded:**
- Traditional process: Single thread of control
- Modern applications: Multiple threads within one process
- Example: Web browser with separate threads for display, network, user input

### Diagram / Example

```
Single-threaded vs Multi-threaded Process

Single-threaded Process          Multi-threaded Process
┌────────────────────┐          ┌────────────────────────────────┐
│   Code   │  Data   │          │      Code      │     Data      │
├──────────┴─────────┤          ├────────────────┴───────────────┤
│                    │          │             Heap               │
│       Heap         │          ├──────────┬──────────┬──────────┤
│                    │          │ Thread 1 │ Thread 2 │ Thread 3 │
├────────────────────┤          │┌────────┐│┌────────┐│┌────────┐│
│      Stack         │          ││Register│││Register│││Register││
│                    │          ││   PC   │││   PC   │││   PC   ││
│    Registers       │          │├────────┤│├────────┤│├────────┤│
│       PC           │          ││ Stack  │││ Stack  │││ Stack  ││
└────────────────────┘          │└────────┘│└────────┘│└────────┘│
                                └──────────┴──────────┴──────────┘

Thread Structure Detail

Process (PID: 1000)
┌─────────────────────────────────────────────────────────┐
│  Shared Resources:                                       │
│  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌───────────────┐  │
│  │  Code   │ │  Data   │ │  Heap   │ │  Open Files   │  │
│  └─────────┘ └─────────┘ └─────────┘ └───────────────┘  │
├─────────────────────────────────────────────────────────┤
│  Thread 1         Thread 2          Thread 3            │
│  ┌──────────┐    ┌──────────┐      ┌──────────┐        │
│  │ TID: 1   │    │ TID: 2   │      │ TID: 3   │        │
│  │ PC: 0x100│    │ PC: 0x200│      │ PC: 0x150│        │
│  │ Regs:... │    │ Regs:... │      │ Regs:... │        │
│  │ Stack    │    │ Stack    │      │ Stack    │        │
│  └──────────┘    └──────────┘      └──────────┘        │
└─────────────────────────────────────────────────────────┘
```

---

## Process vs Thread (프로세스 vs 스레드)

### Concept (개념)

| Aspect | Process (프로세스) | Thread (스레드) |
|--------|-------------------|-----------------|
| **Definition** | Independent execution unit | Execution unit within a process |
| **Memory** | Own address space | Shares process address space |
| **Creation overhead** | High (copy entire address space) | Low (share existing resources) |
| **Context switch** | Expensive | Cheap |
| **Communication** | IPC required (pipes, sockets) | Direct sharing through memory |
| **Independence** | Isolated from other processes | Depends on other threads |
| **Crash impact** | Doesn't affect other processes | May crash entire process |

**Benefits of Threads:**
1. **Responsiveness (응답성)**: Application continues running even if part is blocked
2. **Resource Sharing (자원 공유)**: Threads share memory and resources easily
3. **Economy (경제성)**: Creating/switching threads is faster than processes
4. **Scalability (확장성)**: Can take advantage of multicore architectures

### Diagram / Example

```
Process vs Thread Comparison

Creating New Process (fork)        Creating New Thread
┌─────────────────────┐           ┌─────────────────────┐
│    Parent Process   │           │      Process        │
│  ┌───┐ ┌───┐ ┌───┐  │           │  ┌───┐ ┌───┐ ┌───┐  │
│  │Cod│ │Dat│ │Heap│ │           │  │Cod│ │Dat│ │Heap│ │
│  └───┘ └───┘ └───┘  │           │  └───┘ └───┘ └───┘  │
│      │Stack │       │           │  ┌─────┐ ┌─────┐    │
│      └──────┘       │           │  │Thrd1│ │Thrd2│    │
└─────────────────────┘           │  │Stack│ │Stack│    │
         │ fork()                 │  └─────┘ └─────┘    │
         │ (copy ALL)             └─────────────────────┘
         ▼                                 │
┌─────────────────────┐                    │ create thread
│    Child Process    │                    │ (share, add stack)
│  ┌───┐ ┌───┐ ┌───┐  │                    ▼
│  │Cod│ │Dat│ │Heap│ │           ┌─────────────────────┐
│  └───┘ └───┘ └───┘  │           │      Process        │
│      │Stack │       │           │  ┌───┐ ┌───┐ ┌───┐  │
│      └──────┘       │           │  │Cod│ │Dat│ │Heap│ │
└─────────────────────┘           │  └───┘ └───┘ └───┘  │
                                  │  ┌─────┐┌─────┐┌────┐│
Separate memory spaces!           │  │Thrd1││Thrd2││Thr3││
                                  │  │Stack││Stack││Stck││
                                  │  └─────┘└─────┘└────┘│
                                  └─────────────────────┘

                                  Shared memory space!

Context Switch Cost Comparison

Process Context Switch:           Thread Context Switch:
• Save all registers              • Save registers
• Save memory maps               • Save stack pointer
• Flush TLB                      • (No TLB flush)
• Switch page tables             • (Same address space)
• Restore memory maps            • Restore stack pointer
• Restore all registers          • Restore registers

Time: ~1000-10000 cycles         Time: ~100-1000 cycles
```

---

## Multi-threading Models (다중 스레드 모델)

### Concept (개념)

Threads can be implemented at two levels:

**User Threads (사용자 스레드):**
- Managed by user-level thread library
- Kernel unaware of their existence
- Fast to create and manage
- Examples: POSIX Pthreads, Java threads

**Kernel Threads (커널 스레드):**
- Managed directly by the OS kernel
- Kernel-aware and can be scheduled
- Slower to create but more powerful
- Examples: Windows threads, Linux threads

**Multi-threading Models:**

| Model | Description | Pros | Cons |
|-------|-------------|------|------|
| **Many-to-One (다대일)** | Many user threads → One kernel thread | Efficient, portable | No parallelism, blocking issues |
| **One-to-One (일대일)** | One user thread → One kernel thread | True parallelism | Overhead of creating kernel threads |
| **Many-to-Many (다대다)** | Many user threads → Many kernel threads | Flexible, good parallelism | Complex to implement |

### Diagram / Example

```
Multi-threading Models

1. Many-to-One Model (다대일 모델)
   - Green threads, GNU Portable Threads

   User Space     Kernel Space
   ┌─────────┐
   │ U U U U │    ┌───┐
   │ T T T T │───►│ K │───► CPU
   │ 1 2 3 4 │    │ T │
   └─────────┘    └───┘

   Problem: If one thread blocks, ALL block!

2. One-to-One Model (일대일 모델)
   - Linux, Windows

   User Space     Kernel Space
   ┌───┐          ┌───┐
   │UT1│─────────►│KT1│───┐
   └───┘          └───┘   │
   ┌───┐          ┌───┐   │
   │UT2│─────────►│KT2│───┼──► CPU(s)
   └───┘          └───┘   │
   ┌───┐          ┌───┐   │
   │UT3│─────────►│KT3│───┘
   └───┘          └───┘

   True parallelism! Each thread can run on different CPU

3. Many-to-Many Model (다대다 모델)
   - Solaris, Windows (ThreadFiber)

   User Space     Kernel Space
   ┌─────────┐    ┌───────┐
   │ U U U U │    │ K K K │
   │ T T T T │◄──►│ T T T │───► CPU(s)
   │ 1 2 3 4 │    │ 1 2 3 │
   └─────────┘    └───────┘

   Multiplexes user threads onto kernel threads

4. Two-Level Model (이단계 모델)
   - Similar to M:M but allows binding

   User Space     Kernel Space
   ┌─────────┐    ┌───────┐
   │ U U U   │    │ K K   │
   │ T T T   │◄──►│ T T   │───► CPU(s)
   │ 1 2 3   │    │ 1 2   │
   └─────────┘    └───────┘
       │              │
       └──────────────┘
         UT bound to KT
```

---

## User-Level vs Kernel-Level Threads (사용자 수준 vs 커널 수준 스레드)

### Concept (개념)

**User-Level Threads (ULT, 사용자 수준 스레드):**

| Advantages | Disadvantages |
|------------|---------------|
| Fast creation (no system call) | Kernel sees one thread |
| Fast context switch | Blocking call blocks all threads |
| Customizable scheduling | Cannot use multiprocessors |
| Portable across OS | Page fault blocks all threads |

**Kernel-Level Threads (KLT, 커널 수준 스레드):**

| Advantages | Disadvantages |
|------------|---------------|
| True parallelism | Slower creation (system call) |
| Blocking doesn't affect others | Slower context switch |
| Can utilize multiprocessors | Less portable |
| Kernel can schedule optimally | Limited by kernel resources |

### Diagram / Example

```
User-Level Threads (사용자 수준 스레드)

┌─────────────────────────────────────────┐
│              User Space                  │
│  ┌───────────────────────────────────┐  │
│  │         Application               │  │
│  │   ┌────┐ ┌────┐ ┌────┐ ┌────┐    │  │
│  │   │ T1 │ │ T2 │ │ T3 │ │ T4 │    │  │
│  │   └──┬─┘ └──┬─┘ └──┬─┘ └──┬─┘    │  │
│  │      └──────┴───┬──┴──────┘       │  │
│  │                 │                 │  │
│  │    ┌────────────▼─────────────┐   │  │
│  │    │   Thread Library         │   │  │
│  │    │   (Runtime System)       │   │  │
│  │    │   • Thread table         │   │  │
│  │    │   • Scheduler            │   │  │
│  │    └────────────┬─────────────┘   │  │
│  └─────────────────┼─────────────────┘  │
├────────────────────┼────────────────────┤
│  Kernel Space      │                    │
│             ┌──────▼──────┐             │
│             │   Process   │             │
│             │  (1 thread  │             │
│             │  to kernel) │             │
│             └─────────────┘             │
└─────────────────────────────────────────┘

Kernel-Level Threads (커널 수준 스레드)

┌─────────────────────────────────────────┐
│              User Space                  │
│   ┌────┐ ┌────┐ ┌────┐ ┌────┐          │
│   │ T1 │ │ T2 │ │ T3 │ │ T4 │          │
│   └──┬─┘ └──┬─┘ └──┬─┘ └──┬─┘          │
│      │      │      │      │            │
├──────┼──────┼──────┼──────┼────────────┤
│      │      │      │      │  Kernel    │
│      ▼      ▼      ▼      ▼  Space     │
│   ┌────┐ ┌────┐ ┌────┐ ┌────┐          │
│   │KT1 │ │KT2 │ │KT3 │ │KT4 │          │
│   └──┬─┘ └──┬─┘ └──┬─┘ └──┬─┘          │
│      │      │      │      │            │
│      ▼      ▼      ▼      ▼            │
│   ┌────────────────────────────┐        │
│   │     Kernel Scheduler       │        │
│   └────────────┬───────────────┘        │
│                │                        │
│         ┌──────┼──────┐                │
│         ▼      ▼      ▼                │
│       CPU0   CPU1   CPU2    True       │
│                             Parallelism│
└─────────────────────────────────────────┘
```

---

## Thread Libraries and APIs (스레드 라이브러리)

### Concept (개념)

Common thread libraries:

1. **POSIX Pthreads**: POSIX standard, works on UNIX/Linux/macOS
2. **Windows Threads**: Native Windows API
3. **Java Threads**: Platform-independent, JVM-managed

**Pthread API Basics:**
- `pthread_create()`: Create a new thread
- `pthread_join()`: Wait for thread termination
- `pthread_exit()`: Terminate calling thread
- `pthread_cancel()`: Request thread cancellation
- `pthread_self()`: Get thread ID

### Diagram / Example

```c
// POSIX Pthreads Example (Pthread 예제)

#include <pthread.h>
#include <stdio.h>
#include <stdlib.h>

#define NUM_THREADS 5

// Thread function (스레드 함수)
void *thread_function(void *arg) {
    int thread_num = *(int *)arg;
    printf("Thread %d: Hello from thread!\n", thread_num);

    // Do some work...
    int sum = 0;
    for (int i = 0; i < 1000000; i++) {
        sum += i;
    }

    printf("Thread %d: Sum = %d\n", thread_num, sum);
    pthread_exit(NULL);
}

int main() {
    pthread_t threads[NUM_THREADS];
    int thread_args[NUM_THREADS];
    int rc;

    // Create threads (스레드 생성)
    for (int i = 0; i < NUM_THREADS; i++) {
        thread_args[i] = i;
        printf("Main: Creating thread %d\n", i);

        rc = pthread_create(&threads[i],    // thread ID
                           NULL,             // attributes (default)
                           thread_function,  // function to run
                           &thread_args[i]); // argument

        if (rc) {
            fprintf(stderr, "Error: pthread_create returned %d\n", rc);
            exit(1);
        }
    }

    // Wait for threads to complete (스레드 종료 대기)
    for (int i = 0; i < NUM_THREADS; i++) {
        pthread_join(threads[i], NULL);
        printf("Main: Thread %d completed\n", i);
    }

    printf("Main: All threads completed\n");
    return 0;
}

// Compile: gcc -pthread thread_example.c -o thread_example

/*
Pthread Functions Summary (Pthread 함수 요약)

Creation & Termination:
┌─────────────────────┬────────────────────────────────────┐
│ pthread_create()    │ Create new thread                  │
│ pthread_exit()      │ Terminate calling thread           │
│ pthread_join()      │ Wait for thread to terminate       │
│ pthread_detach()    │ Detach thread (no join needed)     │
└─────────────────────┴────────────────────────────────────┘

Thread Attributes:
┌─────────────────────┬────────────────────────────────────┐
│ pthread_self()      │ Get thread ID                      │
│ pthread_equal()     │ Compare thread IDs                 │
│ pthread_attr_init() │ Initialize thread attributes       │
└─────────────────────┴────────────────────────────────────┘
*/


// Java Threads Example (Java 스레드 예제)

// Method 1: Extending Thread class
class MyThread extends Thread {
    private int threadNum;

    public MyThread(int num) {
        this.threadNum = num;
    }

    @Override
    public void run() {
        System.out.println("Thread " + threadNum + ": Running");
    }
}

// Method 2: Implementing Runnable interface
class MyRunnable implements Runnable {
    private int threadNum;

    public MyRunnable(int num) {
        this.threadNum = num;
    }

    @Override
    public void run() {
        System.out.println("Runnable " + threadNum + ": Running");
    }
}

// Usage
public class ThreadExample {
    public static void main(String[] args) {
        // Using Thread class
        MyThread t1 = new MyThread(1);
        t1.start();

        // Using Runnable interface
        Thread t2 = new Thread(new MyRunnable(2));
        t2.start();

        // Using lambda (Java 8+)
        Thread t3 = new Thread(() -> {
            System.out.println("Lambda thread: Running");
        });
        t3.start();
    }
}
```

---

## Threading Issues (스레드 이슈)

### Concept (개념)

**1. fork() and exec() Semantics:**
- If a thread calls fork(), does the child duplicate all threads or just one?
- UNIX provides two versions: duplicate all threads or only calling thread
- exec() replaces entire process including all threads

**2. Signal Handling (시그널 처리):**
- Synchronous signals: delivered to thread causing the signal
- Asynchronous signals: delivered to any thread
- Options: deliver to specific thread, all threads, or designated thread

**3. Thread Cancellation (스레드 취소):**
- Asynchronous cancellation: immediate termination
- Deferred cancellation: thread checks for cancellation points

**4. Thread-Local Storage (TLS, 스레드 지역 저장소):**
- Data that is unique to each thread
- Similar to static data but per-thread

**5. Thread Pools (스레드 풀):**
- Pre-created pool of threads waiting for work
- Reduces overhead of thread creation
- Limits maximum number of threads

### Diagram / Example

```
Thread Pool Concept (스레드 풀)

Without Thread Pool:
Request 1 ──► Create Thread ──► Process ──► Destroy Thread
Request 2 ──► Create Thread ──► Process ──► Destroy Thread
Request 3 ──► Create Thread ──► Process ──► Destroy Thread
(High overhead for creation/destruction)

With Thread Pool:
                    ┌─────────────────────────┐
                    │      Thread Pool        │
                    │  ┌───┐ ┌───┐ ┌───┐     │
                    │  │ T1│ │ T2│ │ T3│     │
                    │  │   │ │   │ │   │     │
                    │  │wai│ │wai│ │wai│     │
Request ──► Queue ──┼─►│ t │ │ t │ │ t │     │
                    │  └───┘ └───┘ └───┘     │
                    └─────────────────────────┘

Thread Cancellation Points (스레드 취소 지점)

┌────────────────────────────────────────────────────────┐
│                  Thread Execution                       │
│                                                        │
│  ──────►[Work]────►[Cancel Point]────►[Work]────►     │
│                          │                             │
│                          │ Check for                   │
│                          ▼ cancellation                │
│                    ┌──────────┐                        │
│                    │Cancelled?│                        │
│                    └────┬─────┘                        │
│                  No     │      Yes                     │
│                  ┌──────┴──────┐                       │
│                  │             │                       │
│                  ▼             ▼                       │
│              Continue      Cleanup &                   │
│                           Terminate                    │
└────────────────────────────────────────────────────────┘

Deferred Cancellation Points:
• pthread_testcancel()
• pthread_cond_wait()
• pthread_cond_timedwait()
• sleep(), read(), write() (on some systems)

Signal Handling in Multi-threaded Programs

Process with Multiple Threads
┌──────────────────────────────────────────┐
│           Process                        │
│  ┌────┐  ┌────┐  ┌────┐  ┌────┐        │
│  │ T1 │  │ T2 │  │ T3 │  │ T4 │        │
│  │    │  │    │  │    │  │(sig│        │
│  │    │  │    │  │    │  │hnd)│        │
│  └────┘  └────┘  └────┘  └──▲─┘        │
│                              │          │
│  Signal ─────────────────────┘          │
│  (delivered to designated handler)      │
└──────────────────────────────────────────┘

// Setting signal handler for specific thread
pthread_kill(thread_id, SIGUSR1);  // Send signal to specific thread
```

---

## Summary (요약)

| Concept | Description |
|---------|-------------|
| Thread (스레드) | Basic unit of CPU utilization within a process |
| User Thread (사용자 스레드) | Managed by user-level library, kernel unaware |
| Kernel Thread (커널 스레드) | Managed by OS kernel, true parallelism |
| Many-to-One (다대일) | Many user threads to one kernel thread |
| One-to-One (일대일) | Each user thread maps to kernel thread |
| Many-to-Many (다대다) | Many user threads to many kernel threads |
| Thread Pool (스레드 풀) | Pre-created threads waiting for work |
| TLS (스레드 지역 저장소) | Per-thread unique data storage |

---

## Key Terms (주요 용어)

- **Concurrent (동시성)**: Tasks making progress at the same time (may be interleaved)
- **Parallel (병렬성)**: Tasks literally running simultaneously on multiple cores
- **Thread-safe (스레드 안전)**: Code that functions correctly when called from multiple threads
- **Race Condition (경쟁 조건)**: Bug from unsynchronized access to shared data
- **Lightweight Process (경량 프로세스)**: Another name for thread
