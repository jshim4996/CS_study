# Chapter 01. Overview (개요)

> Understanding the fundamental building blocks of modern computers and how they work together to execute programs.

---

## Computer Components (컴퓨터 구성 요소)

### Concept (개념)

A computer system consists of three main components that work together:

1. **CPU (Central Processing Unit / 중앙처리장치)**: The "brain" of the computer that executes instructions
2. **Memory (메모리)**: Stores both programs and data temporarily during execution
3. **I/O Devices (입출력 장치)**: Interfaces between the computer and the outside world

These components communicate through a **System Bus (시스템 버스)**, which consists of:
- **Address Bus (주소 버스)**: Specifies memory locations
- **Data Bus (데이터 버스)**: Transfers actual data
- **Control Bus (제어 버스)**: Coordinates operations

### Diagram / Example

```
┌─────────────────────────────────────────────────────────────┐
│                      System Bus (시스템 버스)                 │
│  ┌─────────────────────────────────────────────────────┐   │
│  │            Address Bus (주소 버스)                    │   │
│  ├─────────────────────────────────────────────────────┤   │
│  │            Data Bus (데이터 버스)                     │   │
│  ├─────────────────────────────────────────────────────┤   │
│  │            Control Bus (제어 버스)                    │   │
│  └─────────────────────────────────────────────────────┘   │
│         │                    │                    │         │
│    ┌────┴────┐          ┌────┴────┐          ┌────┴────┐   │
│    │   CPU   │          │ Memory  │          │   I/O   │   │
│    │ (프로세서)│          │ (메모리) │          │(입출력) │   │
│    └─────────┘          └─────────┘          └─────────┘   │
└─────────────────────────────────────────────────────────────┘
```

---

## Von Neumann Architecture (폰 노이만 구조)

### Concept (개념)

The **Von Neumann Architecture** is the foundation of most modern computers. Its key principle is the **Stored Program Concept (저장 프로그램 개념)**:

- Both **instructions** and **data** are stored in the same memory
- Instructions are fetched and executed sequentially (unless branching occurs)
- The CPU processes one instruction at a time

**Key Characteristics:**
1. Single memory for both program and data
2. Sequential instruction execution
3. Memory is addressed linearly
4. Instructions and data are indistinguishable in memory (determined by context)

**Von Neumann Bottleneck (폰 노이만 병목):**
Since instructions and data share the same memory bus, the CPU often waits for memory access. This limitation drives modern optimizations like caching and pipelining.

### Diagram / Example

```
Von Neumann Architecture:

┌─────────────────────────────────────────┐
│              Memory (메모리)              │
│  ┌─────────────────────────────────┐    │
│  │  Instructions (명령어)           │    │
│  │  0x0000: LOAD R1, [0x1000]      │    │
│  │  0x0004: ADD R1, R2             │    │
│  │  0x0008: STORE [0x1004], R1     │    │
│  ├─────────────────────────────────┤    │
│  │  Data (데이터)                   │    │
│  │  0x1000: 42                     │    │
│  │  0x1004: 0                      │    │
│  └─────────────────────────────────┘    │
└─────────────────────────────────────────┘
            ↑↓ Single Bus (단일 버스)
┌─────────────────────────────────────────┐
│               CPU (프로세서)              │
│  ┌──────────┐  ┌──────────────────┐     │
│  │ Control  │  │    Registers     │     │
│  │  Unit    │  │  (레지스터)       │     │
│  │(제어장치) │  │  PC, R1, R2...   │     │
│  └──────────┘  └──────────────────┘     │
│       ↓              ↓                  │
│  ┌──────────────────────────────────┐   │
│  │        ALU (연산장치)             │   │
│  └──────────────────────────────────┘   │
└─────────────────────────────────────────┘
```

---

## Instruction and Data Flow (명령어와 데이터 흐름)

### Concept (개념)

Understanding how instructions and data move through the system is crucial for optimization:

**Instruction Flow (명령어 흐름):**
1. **Fetch (인출)**: CPU reads instruction from memory using Program Counter (PC)
2. **Decode (해독)**: Control Unit interprets the instruction
3. **Execute (실행)**: ALU performs the operation
4. **Store (저장)**: Results written back to register or memory

**Data Flow (데이터 흐름):**
- Data moves between Memory ↔ Registers ↔ ALU
- Registers provide fast temporary storage
- Memory provides larger but slower storage

**Performance Implication:**
Data that stays in registers is accessed ~100x faster than memory. Compilers optimize by keeping frequently used variables in registers.

### Diagram / Example

```
Instruction Execution Flow:

Step 1: Fetch (인출)
┌────────┐    address    ┌────────┐
│   PC   │──────────────→│ Memory │
│ 0x0004 │←──────────────│        │
└────────┘  instruction  └────────┘

Step 2: Decode (해독)
┌─────────────────────────────────┐
│ Instruction: ADD R1, R2, R3     │
│ Opcode: ADD (덧셈)               │
│ Source: R2, R3                  │
│ Destination: R1                 │
└─────────────────────────────────┘

Step 3: Execute (실행)
┌────────┐     ┌─────┐     ┌────────┐
│   R2   │────→│ ALU │←────│   R3   │
│   10   │     │  +  │     │   20   │
└────────┘     └──┬──┘     └────────┘
                  │ 30
                  ↓
Step 4: Store (저장)
              ┌────────┐
              │   R1   │
              │   30   │
              └────────┘
```

---

## Clock and Performance Metrics (클럭과 성능 지표)

### Concept (개념)

**Clock (클럭):**
- The CPU operates in discrete time steps called **clock cycles (클럭 사이클)**
- **Clock Speed/Frequency (클럭 속도)**: Number of cycles per second, measured in Hz
- Example: 3.5 GHz = 3.5 billion cycles per second

**Key Performance Metrics:**

1. **Clock Cycle Time (클럭 주기)**: Duration of one cycle
   - T = 1 / Frequency
   - 3.5 GHz → T = 0.286 ns

2. **CPI (Cycles Per Instruction / 명령어당 사이클 수)**:
   - Average cycles needed per instruction
   - Lower is better
   - Varies by instruction type: ADD (~1), MUL (~3-5), DIV (~10-20)

3. **IPC (Instructions Per Cycle / 사이클당 명령어 수)**:
   - IPC = 1 / CPI
   - Modern CPUs achieve IPC > 1 through parallelism

4. **CPU Execution Time (CPU 실행 시간)**:
   ```
   CPU Time = Instruction Count × CPI × Clock Cycle Time
            = Instruction Count × CPI / Clock Frequency
   ```

5. **MIPS (Million Instructions Per Second)**:
   ```
   MIPS = Instruction Count / (Execution Time × 10^6)
   ```

**Performance Optimization Implications:**
- Higher clock speed ≠ always faster (depends on CPI)
- Reducing instruction count through better algorithms often beats hardware upgrades
- Memory access dominates execution time in many programs

### Diagram / Example

```
Clock Signal:
     ┌──┐  ┌──┐  ┌──┐  ┌──┐  ┌──┐
─────┘  └──┘  └──┘  └──┘  └──┘  └──
     │←─T─→│
     T = 1/f (Clock Period / 클럭 주기)

Performance Calculation Example:

Program A:
- Instructions: 1,000,000
- CPI: 2.0
- Clock: 2 GHz

CPU Time = (1,000,000 × 2.0) / (2 × 10^9)
         = 2,000,000 / 2,000,000,000
         = 0.001 seconds = 1 ms

Program B (optimized algorithm):
- Instructions: 500,000
- CPI: 2.5
- Clock: 2 GHz

CPU Time = (500,000 × 2.5) / (2 × 10^9)
         = 1,250,000 / 2,000,000,000
         = 0.000625 seconds = 0.625 ms

→ Program B is 1.6x faster despite higher CPI!
```

**Real-World Performance Factors:**
```
┌────────────────────────────────────────────┐
│         Total Program Execution Time        │
├────────────────────────────────────────────┤
│  CPU Computation     │████████░░░░│  30%   │
│  Memory Access       │████████████│  50%   │
│  I/O Operations      │████░░░░░░░░│  20%   │
└────────────────────────────────────────────┘

Most programs are memory-bound, not CPU-bound!
```

---

## Summary (요약)

| Component | Korean | Role |
|-----------|--------|------|
| CPU | 중앙처리장치 | Executes instructions |
| Memory | 메모리 | Stores programs and data |
| I/O | 입출력 장치 | External communication |
| Clock | 클럭 | Synchronizes operations |
| CPI | 명령어당 사이클 | Efficiency metric |

**Key Takeaways for Performance:**
1. The Von Neumann bottleneck limits memory bandwidth - minimize memory access
2. Clock speed matters less than efficient algorithms
3. Keeping data in registers (through compiler optimization) dramatically improves speed
4. Understanding the instruction cycle helps write cache-friendly code
