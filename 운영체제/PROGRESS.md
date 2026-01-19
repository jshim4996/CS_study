# Operating Systems Study Plan (운영체제 학습 계획)

> Goal: Understand how OS works and apply it to development
> 목표: OS 동작 원리를 이해하고 개발에 적용하는 수준

---

## AI Study Guide (AI 학습 가이드)

### Role (역할)
- AI is a **senior developer and professor** teaching this curriculum
- AI는 이 커리큘럼으로 학습을 시켜주는 **시니어 개발자이자 교수**

### Language Rule (언어 규칙)
- **Default: Explain in English (기본: 영어로 설명)**
- When user says "번역" or "translate" → Provide Korean translation

### Study Flow (학습 진행 흐름) - 5 Steps

**Step 1: Check Progress** - "What's next?"
**Step 2: Explanation** - "Explain this" → Read chapter .md
**Step 3: Example** - "Show example" → Check examples/ folder
**Step 4: Assignment** - "Give me an assignment"
**Step 5: Evaluation** - [Submit answer] → Pass = [x] check

### Evaluation Criteria (평가 기준)
□ Did you understand and apply the concept? (학습한 개념을 이해하고 적용했는가)

---

## STEP 1. OS Fundamentals (운영체제 기초)

### Chapter 01. Introduction to Operating Systems (운영체제 개요)
> **doc**: `chapters/chapter01/chapter01.md`
> **examples**: `chapters/chapter01/examples/`

- [ ] What is an Operating System (운영체제란 무엇인가)
- [ ] OS Roles: Resource Management, Abstraction (운영체제의 역할)
- [ ] Kernel vs User Mode (커널 모드 vs 사용자 모드)
- [ ] System Calls (시스템 콜)
- [ ] OS Structure: Monolithic, Microkernel (운영체제 구조)

---

## STEP 2. Processes and Threads (프로세스와 스레드)

### Chapter 02. Processes (프로세스)
> **doc**: `chapters/chapter02/chapter02.md`
> **examples**: `chapters/chapter02/examples/`

- [ ] Process Concept (프로세스 개념)
- [ ] Process States: New, Ready, Running, Waiting, Terminated (프로세스 상태)
- [ ] PCB - Process Control Block
- [ ] Context Switching (컨텍스트 스위칭)
- [ ] Process Creation: fork, exec (프로세스 생성)
- [ ] Process Termination (프로세스 종료)

---

### Chapter 03. Threads (스레드)
> **doc**: `chapters/chapter03/chapter03.md`
> **examples**: `chapters/chapter03/examples/`

- [ ] Thread Concept (스레드 개념)
- [ ] Process vs Thread (프로세스 vs 스레드)
- [ ] Multi-threading Benefits (멀티스레딩의 장점)
- [ ] User Thread vs Kernel Thread (사용자 스레드 vs 커널 스레드)
- [ ] Thread Models: 1:1, N:1, N:M (스레드 모델)

---

### Chapter 04. CPU Scheduling (CPU 스케줄링)
> **doc**: `chapters/chapter04/chapter04.md`
> **examples**: `chapters/chapter04/examples/`

- [ ] Scheduling Concept (스케줄링 개념)
- [ ] Scheduling Criteria: CPU Utilization, Throughput, etc. (스케줄링 기준)
- [ ] FCFS - First Come First Served
- [ ] SJF - Shortest Job First
- [ ] Priority Scheduling (우선순위 스케줄링)
- [ ] Round Robin
- [ ] Multilevel Queue (다단계 큐)

---

## STEP 3. Process Synchronization (프로세스 동기화)

### Chapter 05. Synchronization Basics (동기화 기초)
> **doc**: `chapters/chapter05/chapter05.md`
> **examples**: `chapters/chapter05/examples/`

- [ ] Critical Section Problem (임계 영역 문제)
- [ ] Race Condition (경쟁 조건)
- [ ] Mutual Exclusion Requirements (상호 배제 조건)
- [ ] Peterson's Solution (피터슨 해결책)
- [ ] Hardware Solutions: Test-and-Set, Compare-and-Swap (하드웨어 해결책)

---

### Chapter 06. Synchronization Tools (동기화 도구)
> **doc**: `chapters/chapter06/chapter06.md`
> **examples**: `chapters/chapter06/examples/`

- [ ] Mutex (뮤텍스)
- [ ] Semaphore (세마포어)
- [ ] Monitor (모니터)
- [ ] Condition Variables (조건 변수)
- [ ] Classic Synchronization Problems (동기화 고전 문제)

---

### Chapter 07. Deadlock (데드락)
> **doc**: `chapters/chapter07/chapter07.md`
> **examples**: `chapters/chapter07/examples/`

- [ ] Deadlock Concept (데드락 개념)
- [ ] Deadlock Conditions: Mutual Exclusion, Hold and Wait, etc. (데드락 조건)
- [ ] Resource Allocation Graph (자원 할당 그래프)
- [ ] Deadlock Prevention (데드락 예방)
- [ ] Deadlock Avoidance: Banker's Algorithm (데드락 회피)
- [ ] Deadlock Detection and Recovery (데드락 탐지와 회복)

---

## STEP 4. Memory Management (메모리 관리)

### Chapter 08. Memory Management Basics (메모리 관리 기초)
> **doc**: `chapters/chapter08/chapter08.md`
> **examples**: `chapters/chapter08/examples/`

- [ ] Memory Hierarchy (메모리 계층)
- [ ] Address Binding: Compile, Load, Execution Time (주소 바인딩)
- [ ] Logical vs Physical Address (논리 주소 vs 물리 주소)
- [ ] Contiguous Memory Allocation (연속 메모리 할당)
- [ ] Fragmentation: Internal, External (단편화)
- [ ] Paging Concept (페이징 개념)
- [ ] Segmentation (세그멘테이션)

---

### Chapter 09. Virtual Memory (가상 메모리)
> **doc**: `chapters/chapter09/chapter09.md`
> **examples**: `chapters/chapter09/examples/`

- [ ] Virtual Memory Concept (가상 메모리 개념)
- [ ] Demand Paging (요구 페이징)
- [ ] Page Fault (페이지 폴트)
- [ ] Page Replacement Algorithms: FIFO, LRU, Optimal (페이지 교체 알고리즘)
- [ ] Thrashing (스래싱)
- [ ] Working Set (워킹 셋)

---

## STEP 5. File Systems (파일 시스템)

### Chapter 10. File System Interface (파일 시스템 인터페이스)
> **doc**: `chapters/chapter10/chapter10.md`
> **examples**: `chapters/chapter10/examples/`

- [ ] File Concept (파일 개념)
- [ ] File Attributes and Operations (파일 속성과 연산)
- [ ] Directory Structure (디렉토리 구조)
- [ ] File Access Methods (파일 접근 방법)
- [ ] File Protection (파일 보호)

---

### Chapter 11. File System Implementation (파일 시스템 구현)
> **doc**: `chapters/chapter11/chapter11.md`
> **examples**: `chapters/chapter11/examples/`

- [ ] File System Structure (파일 시스템 구조)
- [ ] Disk Allocation Methods: Contiguous, Linked, Indexed (디스크 할당 방법)
- [ ] Free Space Management (빈 공간 관리)
- [ ] Disk Scheduling: FCFS, SSTF, SCAN, C-SCAN (디스크 스케줄링)

---

### Chapter 12. I/O and Protection (입출력과 보호)
> **doc**: `chapters/chapter12/chapter12.md`
> **examples**: `chapters/chapter12/examples/`

- [ ] I/O Hardware (입출력 하드웨어)
- [ ] I/O Methods: Polling, Interrupt, DMA (입출력 방식)
- [ ] Buffering, Caching, Spooling (버퍼링, 캐싱, 스풀링)
- [ ] Protection Basics (보호 기초)

---

## Practical Connections (실무 연결 포인트)

| OS Concept | Practical Application |
|------------|----------------------|
| Process/Thread | Multiprocessing, Async programming |
| Synchronization | Concurrent programming, Lock handling |
| Memory Management | Memory leaks, Performance optimization |
| File System | File I/O, Storage design |

---

## For Later (나중에 필요할 때)

| Topic | When |
|-------|------|
| Distributed Systems | Large-scale services |
| Virtualization, Containers | DevOps, Cloud |
| Real-time OS | Embedded, Game engines |
