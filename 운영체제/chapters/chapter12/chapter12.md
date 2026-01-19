# Chapter 12. I/O and Protection (입출력과 보호)

## Overview (개요)

This chapter covers the hardware and software mechanisms for performing I/O operations, as well as the protection mechanisms that ensure system security and integrity.

이 장에서는 I/O 작업을 수행하기 위한 하드웨어 및 소프트웨어 메커니즘과 시스템 보안 및 무결성을 보장하는 보호 메커니즘을 다룹니다.

---

## 1. I/O Hardware Concepts (입출력 하드웨어 개념)

### 1.1 I/O Devices Classification (입출력 장치 분류)

I/O devices vary widely in their characteristics and requirements.

I/O 장치는 특성과 요구 사항이 매우 다양합니다.

```
+------------------------------------------------------------------+
|                   I/O DEVICE CLASSIFICATION                      |
|                   (입출력 장치 분류)                              |
+------------------------------------------------------------------+
|                                                                  |
|   By Data Transfer Mode (데이터 전송 모드별):                    |
|                                                                  |
|   +------------------------+  +------------------------+         |
|   |   Block Devices        |  |   Character Devices    |         |
|   |   (블록 장치)          |  |   (문자 장치)          |         |
|   +------------------------+  +------------------------+         |
|   | - Transfer data in     |  | - Transfer one char    |         |
|   |   fixed-size blocks    |  |   at a time            |         |
|   | - 고정 크기 블록으로   |  | - 한 번에 한 문자씩    |         |
|   |   데이터 전송          |  |   전송                 |         |
|   | - Addressable          |  | - Not addressable      |         |
|   | - 주소 지정 가능       |  | - 주소 지정 불가       |         |
|   +------------------------+  +------------------------+         |
|   | Examples (예):         |  | Examples (예):         |         |
|   | - Hard disk (하드디스크)|  | - Keyboard (키보드)    |         |
|   | - SSD                  |  | - Mouse (마우스)       |         |
|   | - USB drive (USB 드라이브)|  | - Printer (프린터)   |         |
|   +------------------------+  +------------------------+         |
|                                                                  |
|   By Transfer Speed (전송 속도별):                               |
|   +----------------------------------------------------------+   |
|   |  Slow                                            Fast    |   |
|   |  (느림)                                          (빠름)  |   |
|   |  |                                                  |    |   |
|   |  Keyboard  Mouse  Printer  Network  Disk   GPU  SSD     |   |
|   |  (1 B/s)   (100)  (200KB)  (1GB/s) (200MB) (16GB)(3GB)  |   |
|   +----------------------------------------------------------+   |
|                                                                  |
+------------------------------------------------------------------+
```

### 1.2 I/O Hardware Components (입출력 하드웨어 구성 요소)

```
+------------------------------------------------------------------+
|                I/O SYSTEM ARCHITECTURE                           |
|                (입출력 시스템 아키텍처)                           |
+------------------------------------------------------------------+
|                                                                  |
|                         +-------+                                |
|                         |  CPU  |                                |
|                         +---+---+                                |
|                             |                                    |
|                    +--------+--------+                           |
|                    |    System Bus   |                           |
|                    |    (시스템 버스) |                           |
|                    +--------+--------+                           |
|                             |                                    |
|          +------------------+------------------+                  |
|          |                  |                  |                  |
|    +-----+-----+     +------+------+    +------+------+          |
|    |  Memory   |     | I/O Bridge  |    | I/O Bridge  |          |
|    |  (메모리) |     | (입출력브릿지)|    | (입출력브릿지)|          |
|    +-----------+     +------+------+    +------+------+          |
|                             |                  |                  |
|                    +--------+--------+  +------+------+          |
|                    |    PCI Bus      |  |   USB Bus   |          |
|                    +--------+--------+  +------+------+          |
|                             |                  |                  |
|               +-------------+-------------+    |                  |
|               |             |             |    |                  |
|         +-----+----+  +-----+----+  +-----+----+-----+           |
|         |  SCSI    |  | Graphics |  | USB    | USB  |            |
|         |Controller|  |   Card   |  |Device 1|Dev 2 |            |
|         +-----+----+  +----------+  +--------+------+            |
|               |                                                  |
|         +-----+-----+                                            |
|         | Disk 1    |                                            |
|         | Disk 2    |                                            |
|         +-----------+                                            |
|                                                                  |
+------------------------------------------------------------------+
```

### 1.3 Device Controllers (장치 컨트롤러)

Device controllers are the hardware interface between the CPU and I/O devices.

장치 컨트롤러는 CPU와 I/O 장치 사이의 하드웨어 인터페이스입니다.

```
+------------------------------------------------------------------+
|                  DEVICE CONTROLLER                               |
|                  (장치 컨트롤러)                                  |
+------------------------------------------------------------------+
|                                                                  |
|   +-----------------------------------------------------------+  |
|   |                    Device Controller                      |  |
|   |                    (장치 컨트롤러)                         |  |
|   +-----------------------------------------------------------+  |
|   |  +-------------+  +-------------+  +------------------+   |  |
|   |  | Data-in     |  | Data-out    |  | Control Register |   |  |
|   |  | Register    |  | Register    |  | (제어 레지스터)  |   |  |
|   |  | (입력 레지) |  | (출력 레지) |  +------------------+   |  |
|   |  +-------------+  +-------------+  | Command (명령)   |   |  |
|   |        |               |           | Status (상태)    |   |  |
|   |        +-------+-------+           +------------------+   |  |
|   |                |                                          |  |
|   |         +------+------+                                   |  |
|   |         | Local Buffer|                                   |  |
|   |         | (로컬 버퍼) |                                   |  |
|   |         +------+------+                                   |  |
|   |                |                                          |  |
|   +----------------+------------------------------------------+  |
|                    |                                             |
|                    v                                             |
|            +---------------+                                     |
|            |    Device     |                                     |
|            |    (장치)     |                                     |
|            +---------------+                                     |
|                                                                  |
|   Controller Registers (컨트롤러 레지스터):                      |
|   - Status Register: Device state (ready, busy, error)           |
|     상태 레지스터: 장치 상태 (준비, 바쁨, 오류)                   |
|   - Control Register: Commands to device                         |
|     제어 레지스터: 장치에 대한 명령                               |
|   - Data Registers: Data being transferred                       |
|     데이터 레지스터: 전송 중인 데이터                             |
|                                                                  |
+------------------------------------------------------------------+
```

### 1.4 Memory-Mapped I/O vs Port-Mapped I/O

```
+------------------------------------------------------------------+
|            I/O ADDRESSING METHODS                                |
|            (입출력 주소 지정 방법)                                |
+------------------------------------------------------------------+
|                                                                  |
|   PORT-MAPPED I/O                  MEMORY-MAPPED I/O             |
|   (포트 매핑 I/O)                  (메모리 매핑 I/O)              |
|                                                                  |
|   Memory Space    I/O Space        Memory Space                  |
|   +----------+   +----------+      +----------------+            |
|   |          |   |          |      |                |            |
|   |  Memory  |   |  Ports   |      |    Memory      |            |
|   |          |   | (포트들) |      |                |            |
|   |          |   +----------+      |----------------|            |
|   +----------+                     | Device Regs    |            |
|                                    | (장치 레지스터) |            |
|                                    +----------------+            |
|                                                                  |
|   - Separate address spaces        - Single address space        |
|   - 별도의 주소 공간               - 단일 주소 공간              |
|   - Special IN/OUT instructions    - Standard load/store         |
|   - 특별한 IN/OUT 명령어           - 표준 load/store 명령어      |
|   - Example: x86 (예: x86)         - Example: ARM (예: ARM)      |
|                                                                  |
|   Port-Mapped Example:             Memory-Mapped Example:        |
|   IN AL, 0x60  ; read from port    MOV R0, [0xFFFF0000]          |
|   OUT 0x60, AL ; write to port     STR R1, [0xFFFF0004]          |
|                                                                  |
+------------------------------------------------------------------+
```

---

## 2. I/O Methods (입출력 방식)

There are three main techniques for performing I/O operations:

I/O 작업을 수행하는 세 가지 주요 기술이 있습니다:

### 2.1 Polling (Programmed I/O) (폴링/프로그램 I/O)

CPU continuously checks device status in a loop.

CPU가 루프에서 장치 상태를 지속적으로 확인합니다.

```
+------------------------------------------------------------------+
|                      POLLING METHOD                              |
|                      (폴링 방식)                                  |
+------------------------------------------------------------------+
|                                                                  |
|   CPU                              Device Controller             |
|   +--------+                       +------------------+          |
|   |        |   1. Check status     |  Status: BUSY   |          |
|   |        |---------------------->|                  |          |
|   |        |<----------------------|  "I'm busy"      |          |
|   |        |                       |                  |          |
|   | (loop) |   2. Check again      +------------------+          |
|   |        |---------------------->|  Status: BUSY   |          |
|   |        |<----------------------|  "Still busy"   |          |
|   |        |                       |                  |          |
|   | (loop) |   3. Check again      +------------------+          |
|   |        |---------------------->|  Status: READY  |          |
|   |        |<----------------------|  "I'm ready!"   |          |
|   |        |                       |                  |          |
|   |        |   4. Transfer data    +------------------+          |
|   |        |<--------------------->|  Data transfer  |          |
|   +--------+                       +------------------+          |
|                                                                  |
|   Pseudocode (의사코드):                                         |
|   +----------------------------------------------------------+   |
|   |  while (device_status != READY) {                        |   |
|   |      // busy wait (바쁜 대기)                            |   |
|   |  }                                                       |   |
|   |  // Transfer data (데이터 전송)                          |   |
|   |  for (i = 0; i < data_size; i++) {                       |   |
|   |      write_to_device(data[i]);                           |   |
|   |      while (device_status != READY);  // wait for each   |   |
|   |  }                                                       |   |
|   +----------------------------------------------------------+   |
|                                                                  |
|   Advantages (장점):                                             |
|   + Simple to implement (구현이 간단함)                          |
|   + Good for fast devices (빠른 장치에 적합)                     |
|   + Predictable timing (예측 가능한 타이밍)                      |
|                                                                  |
|   Disadvantages (단점):                                          |
|   - CPU waste: busy waiting (CPU 낭비: 바쁜 대기)                |
|   - Cannot do other work (다른 작업 불가)                        |
|   - Inefficient for slow devices (느린 장치에 비효율적)          |
|                                                                  |
+------------------------------------------------------------------+
```

### 2.2 Interrupt-Driven I/O (인터럽트 기반 I/O)

Device interrupts CPU when it's ready, freeing CPU for other work.

장치가 준비되면 CPU를 인터럽트하여 CPU가 다른 작업을 수행할 수 있게 합니다.

```
+------------------------------------------------------------------+
|                 INTERRUPT-DRIVEN I/O                             |
|                 (인터럽트 기반 I/O)                               |
+------------------------------------------------------------------+
|                                                                  |
|   CPU                              Device Controller             |
|   +--------+                       +------------------+          |
|   |        |   1. Issue command    |                  |          |
|   |        |---------------------->|  Start operation |          |
|   |        |                       |                  |          |
|   |        |   2. Do other work    |                  |          |
|   |  ....  |   (다른 작업 수행)    |  (processing)   |          |
|   |  ....  |                       |  (처리 중)      |          |
|   |  ....  |                       |                  |          |
|   |        |   3. Interrupt!       +------------------+          |
|   |        |<~~~~~~~~~~~~~~~~~~~~~~|  "I'm done!"    |          |
|   |        |                       |                  |          |
|   |        |   4. Handle interrupt |                  |          |
|   |        |   (인터럽트 처리)     |                  |          |
|   |        |                       |                  |          |
|   |        |   5. Transfer data    +------------------+          |
|   |        |<--------------------->|  Data transfer  |          |
|   +--------+                       +------------------+          |
|                                                                  |
|   INTERRUPT HANDLING FLOW (인터럽트 처리 흐름):                  |
|                                                                  |
|   +----------------------------------------------------------+   |
|   |  1. Device raises interrupt signal                       |   |
|   |     장치가 인터럽트 신호 발생                             |   |
|   |                    |                                     |   |
|   |                    v                                     |   |
|   |  2. CPU finishes current instruction                     |   |
|   |     CPU가 현재 명령어 완료                                |   |
|   |                    |                                     |   |
|   |                    v                                     |   |
|   |  3. CPU saves state (PC, registers)                      |   |
|   |     CPU가 상태 저장 (PC, 레지스터)                        |   |
|   |                    |                                     |   |
|   |                    v                                     |   |
|   |  4. Jump to Interrupt Service Routine (ISR)              |   |
|   |     인터럽트 서비스 루틴(ISR)으로 점프                    |   |
|   |                    |                                     |   |
|   |                    v                                     |   |
|   |  5. ISR handles the I/O (transfer data)                  |   |
|   |     ISR이 I/O 처리 (데이터 전송)                         |   |
|   |                    |                                     |   |
|   |                    v                                     |   |
|   |  6. Restore state and resume                             |   |
|   |     상태 복원 및 재개                                     |   |
|   +----------------------------------------------------------+   |
|                                                                  |
|   Advantages (장점):                                             |
|   + CPU free for other work (CPU가 다른 작업 가능)               |
|   + More efficient than polling (폴링보다 효율적)                |
|   + Good for slow devices (느린 장치에 적합)                     |
|                                                                  |
|   Disadvantages (단점):                                          |
|   - Interrupt handling overhead (인터럽트 처리 오버헤드)         |
|   - Still CPU involved in data transfer                          |
|     (여전히 데이터 전송에 CPU 관여)                              |
|   - Can be slow for large transfers (대용량 전송에 느릴 수 있음) |
|                                                                  |
+------------------------------------------------------------------+
```

### 2.3 DMA (Direct Memory Access) (직접 메모리 접근)

DMA controller transfers data directly between device and memory without CPU.

DMA 컨트롤러가 CPU 없이 장치와 메모리 사이에서 직접 데이터를 전송합니다.

```
+------------------------------------------------------------------+
|                    DMA TRANSFER                                  |
|                    (DMA 전송)                                     |
+------------------------------------------------------------------+
|                                                                  |
|   +-------+          +-------+          +------------------+     |
|   |  CPU  |          |  DMA  |          | Device Controller|     |
|   +---+---+          +---+---+          +--------+---------+     |
|       |                  |                       |               |
|       |  1. Setup DMA    |                       |               |
|       |  (DMA 설정)      |                       |               |
|       |----------------->|                       |               |
|       |  - Source addr   |                       |               |
|       |  - Dest addr     |                       |               |
|       |  - Count         |                       |               |
|       |                  |                       |               |
|       |                  |  2. Request data      |               |
|       |                  |---------------------->|               |
|       |                  |                       |               |
|       |                  |  3. Data transfer     |               |
|   +------+               |<=====================>|               |
|   |Memory|<==============| (Direct to memory)    |               |
|   +------+               | (메모리로 직접)       |               |
|       |                  |                       |               |
|       |  4. Interrupt    |                       |               |
|       |  (DMA complete)  |                       |               |
|       |<-----------------|                       |               |
|       |                  |                       |               |
|                                                                  |
+------------------------------------------------------------------+
|                                                                  |
|   DMA CONTROLLER REGISTERS (DMA 컨트롤러 레지스터):              |
|                                                                  |
|   +---------------------------+                                  |
|   | Memory Address Register   |  Starting address in memory      |
|   | (메모리 주소 레지스터)    |  메모리의 시작 주소              |
|   +---------------------------+                                  |
|   | Count Register            |  Number of bytes to transfer     |
|   | (카운트 레지스터)         |  전송할 바이트 수                |
|   +---------------------------+                                  |
|   | Control Register          |  Direction, mode settings        |
|   | (제어 레지스터)           |  방향, 모드 설정                 |
|   +---------------------------+                                  |
|                                                                  |
|   DMA Transfer Modes (DMA 전송 모드):                            |
|   +---------------------------+----------------------------------+
|   | Burst Mode (버스트 모드)  | Transfers entire block at once   |
|   |                           | 전체 블록을 한 번에 전송         |
|   +---------------------------+----------------------------------+
|   | Cycle Stealing            | Transfers one word at a time     |
|   | (사이클 스틸링)           | 한 번에 한 워드 전송             |
|   +---------------------------+----------------------------------+
|   | Transparent Mode          | Uses bus only when CPU idle      |
|   | (투명 모드)               | CPU가 유휴 상태일 때만 버스 사용 |
|   +---------------------------+----------------------------------+
|                                                                  |
+------------------------------------------------------------------+
|                                                                  |
|   Advantages (장점):                                             |
|   + CPU completely free during transfer (전송 중 CPU 완전 자유)  |
|   + High-speed large data transfers (고속 대용량 데이터 전송)    |
|   + Essential for disk, network I/O (디스크, 네트워크 I/O에 필수)|
|                                                                  |
|   Disadvantages (단점):                                          |
|   - Additional hardware (DMA controller) (추가 하드웨어 필요)    |
|   - Bus contention with CPU (CPU와 버스 경합)                    |
|   - Setup overhead for small transfers (작은 전송에 설정 오버헤드)|
|                                                                  |
+------------------------------------------------------------------+
```

### I/O Methods Comparison (입출력 방식 비교)

| Feature (특징) | Polling (폴링) | Interrupt (인터럽트) | DMA |
|---------------|----------------|---------------------|-----|
| CPU Usage (CPU 사용) | High (높음) | Medium (중간) | Low (낮음) |
| Data Transfer (데이터 전송) | CPU | CPU | DMA Controller |
| Best For (적합한 용도) | Fast, simple devices | Moderate speed | Large transfers |
| Complexity (복잡도) | Low (낮음) | Medium (중간) | High (높음) |
| Latency (지연) | Predictable (예측 가능) | Variable (가변적) | Setup overhead |

---

## 3. Buffering, Caching, and Spooling (버퍼링, 캐싱, 스풀링)

### 3.1 Buffering (버퍼링)

A buffer is a memory area that stores data being transferred between two devices or between a device and an application.

버퍼는 두 장치 사이 또는 장치와 응용 프로그램 사이에서 전송되는 데이터를 저장하는 메모리 영역입니다.

```
+------------------------------------------------------------------+
|                        BUFFERING                                 |
|                        (버퍼링)                                   |
+------------------------------------------------------------------+
|                                                                  |
|   SINGLE BUFFERING (단일 버퍼링):                                |
|                                                                  |
|   Device -----> [Buffer] -----> Process                          |
|   (장치)        (버퍼)          (프로세스)                        |
|                                                                  |
|   Problem: Process must wait while buffer fills                  |
|   문제: 버퍼가 채워지는 동안 프로세스가 대기해야 함               |
|                                                                  |
+------------------------------------------------------------------+
|                                                                  |
|   DOUBLE BUFFERING (이중 버퍼링):                                |
|                                                                  |
|   Device -----> [Buffer A] -----> Process                        |
|          \                       /                               |
|           ----> [Buffer B] -----                                 |
|                                                                  |
|   Time     |  Buffer A    |  Buffer B    |  Process              |
|   ---------|--------------|--------------|-------------          |
|   T1       |  Filling     |  Empty       |  Idle                 |
|   T2       |  Ready       |  Filling     |  Processing A         |
|   T3       |  Filling     |  Ready       |  Processing B         |
|   T4       |  Ready       |  Filling     |  Processing A         |
|                                                                  |
|   Advantage: Overlap I/O and processing                          |
|   장점: I/O와 처리를 중첩                                        |
|                                                                  |
+------------------------------------------------------------------+
|                                                                  |
|   CIRCULAR BUFFERING (순환 버퍼링):                              |
|                                                                  |
|              +---+                                               |
|          +-->| 0 |--+                                            |
|          |   +---+  |                                            |
|       +--+--+    +--+--+                                         |
|       |  7  |    |  1  |                                         |
|       +-----+    +-----+                                         |
|                                                                  |
|       +-----+    +-----+                                         |
|       |  6  |    |  2  |   Producer -->  [write pointer]         |
|       +--+--+    +--+--+   Consumer -->  [read pointer]          |
|          |   +---+  |                                            |
|          +-->| 5 |<-+      Multiple buffers in a ring            |
|              +---+         링에 있는 여러 버퍼                    |
|           +--+   +--+                                            |
|           | 4 |<-| 3 |                                           |
|           +---+  +---+                                           |
|                                                                  |
+------------------------------------------------------------------+
|                                                                  |
|   Why Buffering? (왜 버퍼링인가?):                               |
|   1. Speed mismatch (속도 불일치): Device slower than CPU        |
|   2. Data size mismatch (데이터 크기 불일치): Block vs byte     |
|   3. Copy semantics (복사 의미론): Keep original while sending   |
|                                                                  |
+------------------------------------------------------------------+
```

### 3.2 Caching (캐싱)

A cache is a region of fast memory holding copies of data. Unlike buffers, cache holds copies of data that also exist elsewhere.

캐시는 데이터 복사본을 보유하는 빠른 메모리 영역입니다. 버퍼와 달리 캐시는 다른 곳에도 존재하는 데이터의 복사본을 보유합니다.

```
+------------------------------------------------------------------+
|                         CACHING                                  |
|                         (캐싱)                                    |
+------------------------------------------------------------------+
|                                                                  |
|   Cache Hierarchy (캐시 계층):                                   |
|                                                                  |
|   +----------------------------------------------------------+   |
|   |           Access Time        Size        Cost per Byte   |   |
|   +----------------------------------------------------------+   |
|   |                                                          |   |
|   |  Registers    ~1 ns          < 1KB       $$$$$$          |   |
|   |  (레지스터)   (약 1 나노초)                               |   |
|   |      |                                                   |   |
|   |      v                                                   |   |
|   |  L1 Cache     ~2 ns          32-64KB     $$$$$           |   |
|   |  (L1 캐시)    (약 2 나노초)                               |   |
|   |      |                                                   |   |
|   |      v                                                   |   |
|   |  L2 Cache     ~10 ns         256KB-1MB   $$$$            |   |
|   |  (L2 캐시)    (약 10 나노초)                              |   |
|   |      |                                                   |   |
|   |      v                                                   |   |
|   |  L3 Cache     ~30 ns         4-32MB      $$$             |   |
|   |  (L3 캐시)    (약 30 나노초)                              |   |
|   |      |                                                   |   |
|   |      v                                                   |   |
|   |  Main Memory  ~100 ns        8-64GB      $$              |   |
|   |  (주메모리)   (약 100 나노초)                             |   |
|   |      |                                                   |   |
|   |      v                                                   |   |
|   |  SSD          ~100 us        256GB-4TB   $               |   |
|   |               (약 100 마이크로초)                         |   |
|   |      |                                                   |   |
|   |      v                                                   |   |
|   |  HDD          ~10 ms         1TB-16TB    ¢               |   |
|   |               (약 10 밀리초)                              |   |
|   +----------------------------------------------------------+   |
|                                                                  |
|   Buffer Cache / Page Cache (버퍼 캐시 / 페이지 캐시):           |
|                                                                  |
|   +----------+     +------------------+     +--------+           |
|   | Process  |<--->|   Page Cache     |<--->|  Disk  |           |
|   | (프로세스)|     | (in main memory) |     | (디스크)|           |
|   +----------+     | (주메모리에 존재) |     +--------+           |
|                    +------------------+                          |
|                                                                  |
|   - Frequently accessed disk blocks kept in memory               |
|   - 자주 접근하는 디스크 블록을 메모리에 유지                     |
|   - Reduces disk I/O significantly                               |
|   - 디스크 I/O를 크게 줄임                                       |
|                                                                  |
+------------------------------------------------------------------+
```

### 3.3 Spooling (스풀링)

SPOOL = Simultaneous Peripheral Operations On-Line. A buffer that holds output for a device that cannot accept interleaved data.

SPOOL = 온라인 동시 주변장치 작업. 인터리브된 데이터를 받아들일 수 없는 장치의 출력을 보유하는 버퍼입니다.

```
+------------------------------------------------------------------+
|                        SPOOLING                                  |
|                        (스풀링)                                   |
+------------------------------------------------------------------+
|                                                                  |
|   Problem: Printer is slow and can only handle one job at once   |
|   문제: 프린터가 느리고 한 번에 하나의 작업만 처리 가능           |
|                                                                  |
|   Without Spooling (스풀링 없이):                                |
|   +--------+                                                     |
|   |Process1|----+                                                |
|   +--------+    |    +----------+                                |
|                 +--->| Printer  |  BLOCKED until done            |
|   +--------+    |    +----------+  완료될 때까지 차단됨          |
|   |Process2|----+                                                |
|   +--------+         (must wait! 기다려야 함!)                   |
|                                                                  |
|   With Spooling (스풀링 있음):                                   |
|                                                                  |
|   +--------+         +------------------+                        |
|   |Process1|-------->|                  |                        |
|   +--------+         |   Spool Queue    |     +---------+        |
|                      |   (스풀 큐)      |---->| Printer |        |
|   +--------+         |                  |     +---------+        |
|   |Process2|-------->| [Job1][Job2][...]|                        |
|   +--------+         +------------------+                        |
|                                                                  |
|   Benefits (이점):                                               |
|   + Processes complete quickly (프로세스가 빨리 완료됨)          |
|   + Jobs processed in order (작업이 순서대로 처리됨)             |
|   + Better CPU utilization (더 나은 CPU 활용)                    |
|                                                                  |
|   Common Uses (일반적인 사용):                                   |
|   - Print spooling (인쇄 스풀링)                                 |
|   - Email queuing (이메일 대기열)                                |
|   - Batch job processing (배치 작업 처리)                        |
|                                                                  |
+------------------------------------------------------------------+
```

### Comparison Table (비교 표)

| Concept (개념) | Purpose (목적) | Location (위치) | Data Persistence (데이터 지속성) |
|---------------|---------------|-----------------|--------------------------------|
| Buffer (버퍼) | Speed matching (속도 맞춤) | Memory | Temporary (임시) |
| Cache (캐시) | Fast access to copies (복사본 빠른 접근) | Memory/Hardware | Temporary (임시) |
| Spool (스풀) | Queue for device (장치 대기열) | Disk | Until processed (처리될 때까지) |

---

## 4. Protection Basics (보호 기본)

Protection refers to mechanisms for controlling access of programs, processes, or users to resources.

보호는 프로그램, 프로세스 또는 사용자의 자원 접근을 제어하는 메커니즘을 의미합니다.

### 4.1 Goals of Protection (보호의 목표)

```
+------------------------------------------------------------------+
|                   PROTECTION GOALS                               |
|                   (보호 목표)                                     |
+------------------------------------------------------------------+
|                                                                  |
|   1. PREVENT UNAUTHORIZED ACCESS (무단 접근 방지)                |
|      - Malicious users (악의적인 사용자)                         |
|      - Buggy programs (버그가 있는 프로그램)                     |
|                                                                  |
|   2. ENSURE PROPER RESOURCE USAGE (적절한 자원 사용 보장)        |
|      - Fair allocation (공정한 할당)                             |
|      - Prevent resource hogging (자원 독점 방지)                 |
|                                                                  |
|   3. DETECT ACCESS VIOLATIONS (접근 위반 감지)                   |
|      - Logging and auditing (로깅 및 감사)                       |
|      - Alert on suspicious activity (의심스러운 활동 알림)       |
|                                                                  |
|   4. PROVIDE MECHANISM FOR POLICY IMPLEMENTATION                 |
|      정책 구현을 위한 메커니즘 제공                               |
|      - Separation of mechanism and policy                        |
|      - 메커니즘과 정책의 분리                                    |
|                                                                  |
+------------------------------------------------------------------+
```

### 4.2 Protection Domains (보호 도메인)

A domain defines a set of objects and the operations that can be performed on them.

도메인은 객체 집합과 그 객체에 대해 수행할 수 있는 작업을 정의합니다.

```
+------------------------------------------------------------------+
|                  PROTECTION DOMAINS                              |
|                  (보호 도메인)                                    |
+------------------------------------------------------------------+
|                                                                  |
|   Domain = Set of <object, access-rights> pairs                  |
|   도메인 = <객체, 접근-권한> 쌍의 집합                            |
|                                                                  |
|   +----------------------------------------------------------+   |
|   |  Domain D1 (도메인 D1):                                  |   |
|   |  +------------------------------------------------+      |   |
|   |  | Object        | Rights (권한)                  |      |   |
|   |  |---------------|-------------------------------|      |   |
|   |  | File1 (파일1) | Read, Write (읽기, 쓰기)      |      |   |
|   |  | File2 (파일2) | Read (읽기)                   |      |   |
|   |  | Printer       | Print (인쇄)                  |      |   |
|   |  +------------------------------------------------+      |   |
|   +----------------------------------------------------------+   |
|                                                                  |
|   +----------------------------------------------------------+   |
|   |  Domain D2 (도메인 D2):                                  |   |
|   |  +------------------------------------------------+      |   |
|   |  | Object        | Rights (권한)                  |      |   |
|   |  |---------------|-------------------------------|      |   |
|   |  | File1 (파일1) | Execute (실행)                |      |   |
|   |  | File3 (파일3) | Read, Write, Execute          |      |   |
|   |  | Network       | Send (전송)                   |      |   |
|   |  +------------------------------------------------+      |   |
|   +----------------------------------------------------------+   |
|                                                                  |
|   Domain Association (도메인 연관):                              |
|   - Static: Process always in same domain                        |
|     정적: 프로세스가 항상 같은 도메인에 있음                      |
|   - Dynamic: Process can switch domains                          |
|     동적: 프로세스가 도메인을 전환할 수 있음                      |
|                                                                  |
|   Examples of Domains (도메인 예):                               |
|   - User mode vs Kernel mode (사용자 모드 vs 커널 모드)          |
|   - Different user accounts (다른 사용자 계정)                   |
|   - Process privileges (프로세스 권한)                           |
|                                                                  |
+------------------------------------------------------------------+
```

### 4.3 Access Matrix (접근 행렬)

The access matrix is a general model for protection, defining rights of each domain for each object.

접근 행렬은 각 도메인의 각 객체에 대한 권한을 정의하는 보호의 일반적인 모델입니다.

```
+------------------------------------------------------------------+
|                     ACCESS MATRIX                                |
|                     (접근 행렬)                                   |
+------------------------------------------------------------------+
|                                                                  |
|                  Objects (객체)                                  |
|            +--------+--------+--------+--------+--------+        |
|            | File1  | File2  | File3  |Printer | Domain |        |
|            | (파일1)| (파일2)| (파일3)|(프린터)| Switch |        |
|   ---------+--------+--------+--------+--------+--------+        |
|   Domain 1 | read   |        | read   | print  | switch |        |
|   (도메인1)| write  |        | exec   |        | to D2  |        |
|   ---------+--------+--------+--------+--------+--------+        |
|   Domain 2 | read   | read   |        |        | switch |        |
|   (도메인2)|        | write  |        |        | to D3  |        |
|   ---------+--------+--------+--------+--------+--------+        |
|   Domain 3 |        | read   | read   | print  |        |        |
|   (도메인3)|        |        | write  |        |        |        |
|   ---------+--------+--------+--------+--------+--------+        |
|                                                                  |
|   Matrix Operations (행렬 연산):                                 |
|   - Copy: Transfer rights to another domain                      |
|     복사: 다른 도메인에 권한 전달                                 |
|   - Owner: Can grant/revoke rights                               |
|     소유자: 권한 부여/취소 가능                                   |
|   - Control: Can modify other domain's rights                    |
|     제어: 다른 도메인의 권한 수정 가능                            |
|                                                                  |
+------------------------------------------------------------------+
|                                                                  |
|   IMPLEMENTATION METHODS (구현 방법):                            |
|                                                                  |
|   1. GLOBAL TABLE (전역 테이블):                                 |
|      +---------------------------------------------------+       |
|      | Domain  | Object  | Rights                        |       |
|      |---------|---------|-------------------------------|       |
|      | D1      | File1   | read, write                   |       |
|      | D1      | File3   | read, exec                    |       |
|      | D1      | Printer | print                         |       |
|      | D2      | File1   | read                          |       |
|      | D2      | File2   | read, write                   |       |
|      | ...     | ...     | ...                           |       |
|      +---------------------------------------------------+       |
|      - Simple but large table                                    |
|      - 단순하지만 큰 테이블                                       |
|                                                                  |
|   2. ACCESS CONTROL LIST (ACL) - Column-based (열 기반):         |
|      +----------------------------------------------------------+|
|      | File1: [(D1, rw), (D2, r)]                               ||
|      | File2: [(D2, rw), (D3, r)]                               ||
|      | File3: [(D1, rx), (D3, rw)]                              ||
|      +----------------------------------------------------------+|
|      - List per object (객체당 목록)                             |
|      - Easy to determine who has access to an object             |
|      - 누가 객체에 접근 가능한지 쉽게 판단                        |
|      - Used in: UNIX file permissions, Windows NTFS              |
|                                                                  |
|   3. CAPABILITY LIST - Row-based (행 기반):                      |
|      +----------------------------------------------------------+|
|      | D1: [(File1, rw), (File3, rx), (Printer, print)]        ||
|      | D2: [(File1, r), (File2, rw)]                           ||
|      | D3: [(File2, r), (File3, rw), (Printer, print)]         ||
|      +----------------------------------------------------------+|
|      - List per domain (도메인당 목록)                           |
|      - Easy to determine what a domain can access                |
|      - 도메인이 무엇에 접근 가능한지 쉽게 판단                    |
|      - Used in: Capability-based systems                         |
|                                                                  |
+------------------------------------------------------------------+
```

### 4.4 ACL vs Capability Comparison (ACL vs 능력 비교)

```
+------------------------------------------------------------------+
|              ACL vs CAPABILITY COMPARISON                        |
|              (ACL vs 능력 비교)                                   |
+------------------------------------------------------------------+
|                                                                  |
|   ACCESS CONTROL LIST (ACL)          CAPABILITY LIST             |
|   접근 제어 목록                     능력 목록                    |
|                                                                  |
|   +------------------------+        +------------------------+   |
|   |       File             |        |       Process          |   |
|   +------------------------+        +------------------------+   |
|   | User1: read, write     |        | Cap1: (File, read)     |   |
|   | User2: read            |        | Cap2: (Printer, print) |   |
|   | Group: read            |        | Cap3: (File2, write)   |   |
|   +------------------------+        +------------------------+   |
|                                                                  |
|   "Who can access this object?"     "What can this process do?"  |
|   "누가 이 객체에 접근 가능?"        "이 프로세스가 무엇 가능?"   |
|                                                                  |
+------------------------------------------------------------------+
|                                                                  |
|   Feature          |    ACL              |    Capability         |
|   특징             |                     |                       |
|   -----------------|---------------------|---------------------- |
|   Stored with      | Object (객체)       | Subject (주체)        |
|   저장 위치        |                     |                       |
|   -----------------|---------------------|---------------------- |
|   Revocation       | Easy (쉬움)         | Hard (어려움)         |
|   취소             | Change ACL          | Find all capabilities |
|   -----------------|---------------------|---------------------- |
|   Sharing          | Easy (쉬움)         | Easy (쉬움)           |
|   공유             | Add to ACL          | Pass capability       |
|   -----------------|---------------------|---------------------- |
|   Review access    | Easy (쉬움)         | Hard (어려움)         |
|   접근 검토        | Check object's ACL  | Check all processes   |
|   -----------------|---------------------|---------------------- |
|   Determine all    | Hard (어려움)       | Easy (쉬움)           |
|   accessible objs  | Check all objects   | Check process caps    |
|   접근 가능 객체   |                     |                       |
|                                                                  |
+------------------------------------------------------------------+
```

### 4.5 Protection Rings (보호 링)

Many systems use protection rings to provide different privilege levels.

많은 시스템이 다른 권한 수준을 제공하기 위해 보호 링을 사용합니다.

```
+------------------------------------------------------------------+
|                    PROTECTION RINGS                              |
|                    (보호 링)                                      |
+------------------------------------------------------------------+
|                                                                  |
|                    +-------------------+                         |
|                    |    Ring 0         |   Kernel (커널)         |
|                    |    (Most Privileged)| 가장 높은 권한        |
|                    |                   |                         |
|                +---+-------------------+---+                     |
|                |        Ring 1            |  Device Drivers      |
|                |   (High Privilege)       |  장치 드라이버       |
|                |                          |                      |
|           +----+--------------------------+----+                 |
|           |            Ring 2                  |  OS Services    |
|           |   (Medium Privilege)               |  OS 서비스      |
|           |                                    |                 |
|      +----+------------------------------------+----+            |
|      |              Ring 3                         |             |
|      |   (Least Privileged - User Mode)            |             |
|      |   가장 낮은 권한 - 사용자 모드              |             |
|      |                                             |             |
|      |     User Applications (사용자 응용프로그램)  |             |
|      +--------------------------------------------|+            |
|                                                                  |
|   Rules (규칙):                                                  |
|   1. Inner rings have more privileges                            |
|      안쪽 링이 더 많은 권한을 가짐                               |
|   2. Code can call into lower-numbered rings                     |
|      코드가 더 낮은 번호의 링을 호출할 수 있음                    |
|   3. Transition requires special gates/instructions              |
|      전환에 특별한 게이트/명령어 필요                            |
|                                                                  |
|   Intel x86 Implementation (인텔 x86 구현):                      |
|   - Ring 0: Kernel mode (커널 모드)                              |
|   - Ring 3: User mode (사용자 모드)                              |
|   - Rings 1, 2: Rarely used (거의 사용 안함)                     |
|                                                                  |
+------------------------------------------------------------------+
```

---

## Summary (요약)

```
+==================================================================+
|                    CHAPTER 12 SUMMARY                            |
|                    (12장 요약)                                    |
+==================================================================+

1. I/O HARDWARE (입출력 하드웨어)
   - Devices: Block (disk) vs Character (keyboard)
   - 장치: 블록 (디스크) vs 문자 (키보드)
   - Controllers: Interface between CPU and devices
   - 컨트롤러: CPU와 장치 사이의 인터페이스
   - I/O addressing: Port-mapped vs Memory-mapped
   - I/O 주소 지정: 포트 매핑 vs 메모리 매핑

2. I/O METHODS (입출력 방식)
   +-------------+-----------------+------------------------+
   | Method      | CPU Involvement | Best Use Case          |
   | (방식)      | (CPU 관여)      | (최적 사용 사례)       |
   +-------------+-----------------+------------------------+
   | Polling     | High (높음)     | Fast, simple devices   |
   | (폴링)      |                 | (빠르고 단순한 장치)   |
   +-------------+-----------------+------------------------+
   | Interrupt   | Medium (중간)   | Moderate speed devices |
   | (인터럽트)  |                 | (중간 속도 장치)       |
   +-------------+-----------------+------------------------+
   | DMA         | Low (낮음)      | Large data transfers   |
   |             |                 | (대용량 데이터 전송)   |
   +-------------+-----------------+------------------------+

3. BUFFERING, CACHING, SPOOLING
   - Buffer: Temporary storage for speed matching (속도 맞춤)
   - Cache: Fast copies of frequently accessed data (빠른 복사본)
   - Spool: Queue for devices that can't share (공유 불가 장치 큐)

4. PROTECTION (보호)
   - Domain: Set of access rights (접근 권한 집합)
   - Access Matrix: Objects x Domains = Rights (객체 x 도메인 = 권한)
   - Implementation:
     * ACL: Stored with objects (객체에 저장)
     * Capability: Stored with subjects (주체에 저장)
   - Protection Rings: Privilege levels (권한 수준)
     Ring 0 (kernel) to Ring 3 (user)

+==================================================================+
```

### Quick Reference (빠른 참조)

| Topic (주제) | Key Point (핵심 포인트) |
|--------------|------------------------|
| Polling (폴링) | CPU repeatedly checks device status (CPU가 반복적으로 장치 상태 확인) |
| Interrupt (인터럽트) | Device signals CPU when ready (장치가 준비되면 CPU에 신호) |
| DMA | Direct memory-device transfer, CPU free (직접 메모리-장치 전송, CPU 자유) |
| Buffer (버퍼) | Temporary holding area between transfers (전송 사이 임시 저장 영역) |
| Cache (캐시) | Fast copy of data (데이터의 빠른 복사본) |
| Spool (스풀) | Queue for serialized access (순차적 접근을 위한 큐) |
| Domain (도메인) | Collection of access rights (접근 권한의 모음) |
| ACL | Per-object access list (객체별 접근 목록) |
| Capability | Per-subject access token (주체별 접근 토큰) |
| Ring 0 | Kernel mode - highest privilege (커널 모드 - 최고 권한) |
| Ring 3 | User mode - lowest privilege (사용자 모드 - 최저 권한) |
