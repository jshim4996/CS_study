# Chapter 01. Introduction to OS (운영체제 개요)

> An operating system is the software layer that manages hardware resources and provides services for application programs.

---

## What is an Operating System? (운영체제란?)

### Concept (개념)

An **Operating System (OS)** is system software that acts as an intermediary between computer hardware and users. It manages all hardware resources (CPU, memory, storage, I/O devices) and provides a convenient environment for executing programs.

Key perspectives on what an OS does:
- **Resource Manager (자원 관리자)**: Allocates and manages CPU time, memory space, storage, and I/O devices
- **Control Program (제어 프로그램)**: Controls execution of user programs to prevent errors and misuse
- **Abstraction Provider (추상화 제공자)**: Hides complex hardware details from users and applications

### Diagram / Example

```
┌─────────────────────────────────────────┐
│            User Applications            │
│    (Web Browser, Text Editor, Games)    │
├─────────────────────────────────────────┤
│           System Programs               │
│    (Compilers, Shells, Utilities)       │
├─────────────────────────────────────────┤
│          Operating System               │
│   ┌─────────────────────────────────┐   │
│   │           Kernel                │   │
│   │  (Process, Memory, File, I/O)   │   │
│   └─────────────────────────────────┘   │
├─────────────────────────────────────────┤
│             Hardware                    │
│    (CPU, Memory, Disk, Network)         │
└─────────────────────────────────────────┘
```

---

## Roles of Operating System (운영체제의 역할)

### Concept (개념)

The OS performs several critical functions:

1. **Process Management (프로세스 관리)**
   - Creating and terminating processes
   - Scheduling processes on the CPU
   - Synchronizing and communicating between processes

2. **Memory Management (메모리 관리)**
   - Tracking which parts of memory are in use
   - Allocating and deallocating memory
   - Managing virtual memory

3. **Storage Management (저장장치 관리)**
   - File system creation and management
   - Directory management
   - Disk scheduling

4. **I/O Management (입출력 관리)**
   - Buffering, caching, and spooling
   - Device driver interfaces
   - Interrupt handling

5. **Security and Protection (보안 및 보호)**
   - User authentication
   - Access control to resources
   - Protection against malicious programs

### Diagram / Example

```
              ┌─────────────────┐
              │  User Programs  │
              └────────┬────────┘
                       │ System Calls
              ┌────────▼────────┐
              │   OS Services   │
              ├─────────────────┤
              │ Process Mgmt    │──► Create, Schedule, Terminate
              │ Memory Mgmt     │──► Allocate, Protect, Virtual Memory
              │ File System     │──► Create, Read, Write, Delete
              │ I/O System      │──► Device Drivers, Buffering
              │ Protection      │──► Access Control, Authentication
              └─────────────────┘
```

---

## Kernel Mode vs User Mode (커널 모드 vs 사용자 모드)

### Concept (개념)

Modern CPUs support at least two modes of operation to protect the OS from user programs:

**User Mode (사용자 모드)**
- Limited access to hardware and memory
- Cannot execute privileged instructions
- Normal application programs run here
- Mode bit = 1

**Kernel Mode (커널 모드)** (also called Supervisor Mode, 관리자 모드)
- Full access to all hardware and memory
- Can execute any CPU instruction
- OS kernel runs here
- Mode bit = 0

**Privileged Instructions (특권 명령)**
- I/O operations
- Timer management
- Interrupt management
- Memory management (page table manipulation)

### Diagram / Example

```
┌──────────────────────────────────────────────────┐
│                  User Mode (Ring 3)              │
│  ┌────────────┐  ┌────────────┐  ┌────────────┐  │
│  │   App A    │  │   App B    │  │   App C    │  │
│  └─────┬──────┘  └─────┬──────┘  └─────┬──────┘  │
│        │               │               │         │
│        └───────────────┼───────────────┘         │
│                        │ System Call (Trap)      │
│                        ▼                         │
├──────────────────────────────────────────────────┤
│                Kernel Mode (Ring 0)              │
│  ┌──────────────────────────────────────────┐    │
│  │              OS Kernel                    │   │
│  │  • Full hardware access                   │   │
│  │  • Execute privileged instructions        │   │
│  │  • Manage all system resources            │   │
│  └──────────────────────────────────────────┘    │
└──────────────────────────────────────────────────┘

Mode Transition:
User Mode ──(System Call/Interrupt)──► Kernel Mode
Kernel Mode ──(Return from call)──► User Mode
```

---

## System Calls (시스템 호출)

### Concept (개념)

**System Calls (시스템 호출)** are the programming interface between user programs and the OS. They provide a way for user-mode programs to request services from the kernel.

**Categories of System Calls:**

| Category | Examples | Description |
|----------|----------|-------------|
| Process Control (프로세스 제어) | fork(), exec(), exit(), wait() | Create, load, terminate processes |
| File Management (파일 관리) | open(), read(), write(), close() | File operations |
| Device Management (장치 관리) | ioctl(), read(), write() | Device I/O operations |
| Information Maintenance (정보 유지) | getpid(), time(), sleep() | Get/set system info |
| Communication (통신) | pipe(), socket(), send(), recv() | Inter-process communication |
| Protection (보호) | chmod(), chown(), setuid() | Security and permissions |

**System Call Mechanism:**
1. User program invokes library function (e.g., `printf()`)
2. Library function sets up parameters and executes trap instruction
3. CPU switches to kernel mode
4. Kernel executes the requested service
5. Kernel returns result and switches back to user mode

### Diagram / Example

```
User Program                          OS Kernel
    │                                     │
    │  printf("Hello")                    │
    │       │                             │
    │       ▼                             │
    │  C Library (libc)                   │
    │       │                             │
    │       ▼                             │
    │  write(1, "Hello", 5)               │
    │       │                             │
    │       │  ┌─────────────────────┐    │
    │       └──│  System Call Table  │────┤
    │          │  (syscall #1 = write)│   │
    │          └─────────────────────┘    │
    │                    │                │
    │                    ▼                │
    │          Kernel write() handler     │
    │                    │                │
    │                    ▼                │
    │          Write to stdout            │
    │                    │                │
    │  ◄─────────────────┘                │
    │  Return value                       │
    ▼                                     ▼

// Example: fork() system call in C
#include <unistd.h>
#include <stdio.h>

int main() {
    pid_t pid = fork();  // System call to create new process

    if (pid == 0) {
        printf("Child process (자식 프로세스)\n");
    } else if (pid > 0) {
        printf("Parent process (부모 프로세스)\n");
    } else {
        printf("Fork failed!\n");
    }
    return 0;
}
```

---

## OS Structure (운영체제 구조)

### Concept (개념)

Operating systems can be structured in several ways:

**1. Monolithic Structure (일체형 구조)**
- All OS services run in kernel space
- Fast performance (no mode switches between services)
- Difficult to maintain and extend
- Examples: Traditional UNIX, Linux

**2. Layered Structure (계층 구조)**
- OS divided into layers, each built on lower layers
- Easy to debug and verify
- Overhead from layer traversal
- Example: THE operating system

**3. Microkernel Structure (마이크로커널 구조)**
- Minimal kernel (IPC, memory management, scheduling)
- Other services run in user space
- More secure and reliable
- Slower due to message passing
- Examples: Mach, MINIX 3, QNX

**4. Modular Structure (모듈 구조)**
- Core kernel with loadable modules
- Combines benefits of monolithic and microkernel
- Most modern OS use this approach
- Example: Modern Linux (with loadable kernel modules)

**5. Hybrid Structure (하이브리드 구조)**
- Combines multiple approaches
- Examples: Windows NT, macOS

### Diagram / Example

```
Monolithic (일체형)          Microkernel (마이크로커널)
┌─────────────────┐         ┌─────────────────────────┐
│   User Space    │         │      User Space         │
├─────────────────┤         │  ┌────┐ ┌────┐ ┌────┐   │
│                 │         │  │File│ │Net │ │Dev │   │
│   Entire OS     │         │  │Srv │ │Srv │ │Drv │   │
│   in Kernel     │         │  └──┬─┘ └──┬─┘ └──┬─┘   │
│   (Big kernel)  │         ├─────┼──────┼──────┼─────┤
│                 │         │     └──────┼──────┘     │
└─────────────────┘         │      Microkernel        │
                            │   (IPC, Scheduling)     │
                            └─────────────────────────┘

Layered Structure (계층 구조)
┌─────────────────────────┐
│   Layer N: User Programs│
├─────────────────────────┤
│   Layer N-1: I/O        │
├─────────────────────────┤
│   Layer N-2: Device Drv │
├─────────────────────────┤
│        ...              │
├─────────────────────────┤
│   Layer 1: Memory Mgmt  │
├─────────────────────────┤
│   Layer 0: Hardware     │
└─────────────────────────┘

Modular Structure (Linux 모듈 구조)
┌───────────────────────────────────────┐
│            Core Kernel                │
│  ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐     │
│  │Sched│ │ MM  │ │ VFS │ │ IPC │     │
│  └──┬──┘ └──┬──┘ └──┬──┘ └──┬──┘     │
│     └───────┴───────┴───────┘        │
│              │                        │
│    ┌─────────┼─────────┐             │
│    ▼         ▼         ▼             │
│ ┌──────┐ ┌──────┐ ┌──────┐          │
│ │ ext4 │ │ USB  │ │ TCP  │ Loadable │
│ │Module│ │Module│ │Module│ Modules  │
│ └──────┘ └──────┘ └──────┘          │
└───────────────────────────────────────┘
```

---

## Summary (요약)

| Concept | Description |
|---------|-------------|
| Operating System (운영체제) | Software that manages hardware and provides services for programs |
| Kernel (커널) | Core component of OS with full hardware access |
| User Mode (사용자 모드) | Restricted mode for running applications |
| Kernel Mode (커널 모드) | Privileged mode for OS operations |
| System Call (시스템 호출) | Interface for user programs to request OS services |
| Monolithic (일체형) | All OS services in one large kernel |
| Microkernel (마이크로커널) | Minimal kernel with services in user space |

---

## Key Terms (주요 용어)

- **Bootstrap Program (부트스트랩 프로그램)**: Initial program loaded at power-up
- **Interrupt (인터럽트)**: Signal to CPU for attention
- **Trap (트랩)**: Software-generated interrupt (system call, error)
- **Daemon (데몬)**: Background process running continuously
- **Shell (셸)**: Command-line interface to OS
