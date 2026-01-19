# Chapter 04. Pipeline (파이프라인)

> Understanding CPU pipelining and hazards to write code that runs efficiently on modern processors.

---

## Pipeline Concept (파이프라인 개념)

### Concept (개념)

**Pipelining (파이프라이닝)** is a technique to increase CPU throughput by overlapping instruction execution.

**Analogy - Laundry:**
```
Without pipelining (Sequential):
- Wash 1, Dry 1, Fold 1, then Wash 2, Dry 2, Fold 2...
- 3 loads take 9 time units

With pipelining (Overlapped):
- Time 1: Wash 1
- Time 2: Dry 1, Wash 2
- Time 3: Fold 1, Dry 2, Wash 3
- 3 loads take 5 time units
```

**Key Principles:**
- Each pipeline stage does different work simultaneously
- **Throughput (처리량)** increases: more instructions complete per unit time
- **Latency (지연시간)** for single instruction stays the same
- Pipeline depth = number of stages
- Ideal speedup = number of pipeline stages

**Modern CPU Pipelines:**
- Intel Skylake: ~14-19 stages
- ARM Cortex-A76: ~13 stages
- Deeper pipelines allow higher clock speeds but increase hazard penalties

### Diagram / Example

```
Non-Pipelined Execution:
Time:    1   2   3   4   5   6   7   8   9   10  11  12
Inst 1: [F] [D] [E]
Inst 2:             [F] [D] [E]
Inst 3:                         [F] [D] [E]
Inst 4:                                     [F] [D] [E]
→ 4 instructions in 12 cycles = 0.33 IPC

Pipelined Execution:
Time:    1   2   3   4   5   6
Inst 1: [F] [D] [E]
Inst 2:     [F] [D] [E]
Inst 3:         [F] [D] [E]
Inst 4:             [F] [D] [E]
→ 4 instructions in 6 cycles = 0.67 IPC

After pipeline is full:
Time:    1   2   3   4   5   6   7   8   9   10
Inst 1: [F] [D] [E]
Inst 2:     [F] [D] [E]
Inst 3:         [F] [D] [E]
Inst 4:             [F] [D] [E]
Inst 5:                 [F] [D] [E]
Inst 6:                     [F] [D] [E]
Inst 7:                         [F] [D] [E]
Inst 8:                             [F] [D] [E]
→ 1 instruction completes every cycle! (1.0 IPC)
```

---

## Pipeline Stages (파이프라인 단계)

### Concept (개념)

The classic **5-stage RISC pipeline**:

| Stage | Korean | Description |
|-------|--------|-------------|
| IF (Instruction Fetch) | 명령어 인출 | Read instruction from memory |
| ID (Instruction Decode) | 명령어 해독 | Decode instruction, read registers |
| EX (Execute) | 실행 | ALU operation or address calculation |
| MEM (Memory Access) | 메모리 접근 | Read/write data memory |
| WB (Write Back) | 결과 저장 | Write result to register |

**Stage Details:**

**IF (명령어 인출):**
- Access instruction cache
- Increment PC
- Branch prediction occurs here

**ID (명령어 해독):**
- Decode opcode
- Read register file
- Sign-extend immediate values
- Detect hazards

**EX (실행):**
- ALU operations
- Calculate memory addresses
- Evaluate branch conditions

**MEM (메모리 접근):**
- Load: read from data cache
- Store: write to data cache
- Non-memory instructions pass through

**WB (결과 저장):**
- Write results back to register file
- Update architectural state

### Diagram / Example

```
5-Stage Pipeline Detail:
┌─────────────────────────────────────────────────────────────┐
│                                                              │
│    IF          ID          EX          MEM         WB       │
│  ┌─────┐    ┌─────┐    ┌─────┐    ┌─────┐    ┌─────┐       │
│  │Fetch│ →  │Decode│ → │Execute│ → │Memory│ → │Write│       │
│  │     │    │ +Reg │    │ ALU  │    │Access│    │Back │      │
│  │     │    │ Read │    │      │    │      │    │     │      │
│  └─────┘    └─────┘    └─────┘    └─────┘    └─────┘       │
│     │          │          │          │          │           │
│  ┌──┴──┐    ┌──┴──┐    ┌──┴──┐    ┌──┴──┐    ┌──┴──┐      │
│  │IF/ID│    │ID/EX│    │EX/MEM│   │MEM/WB│    │     │      │
│  │ Reg │    │ Reg │    │ Reg │    │ Reg │    │     │       │
│  └─────┘    └─────┘    └─────┘    └─────┘    └─────┘       │
│  Pipeline registers store data between stages               │
└─────────────────────────────────────────────────────────────┘

Example: ADD R1, R2, R3 going through pipeline

┌────────────────────────────────────────────────────────────┐
│ IF:  PC=100, fetch "ADD R1, R2, R3" from memory           │
│      IR ← instruction                                      │
├────────────────────────────────────────────────────────────┤
│ ID:  Decode: opcode=ADD, dest=R1, src1=R2, src2=R3       │
│      Read R2=5, R3=3 from register file                   │
├────────────────────────────────────────────────────────────┤
│ EX:  ALU computes 5 + 3 = 8                               │
├────────────────────────────────────────────────────────────┤
│ MEM: No memory operation (pass through)                    │
├────────────────────────────────────────────────────────────┤
│ WB:  Write 8 to R1 in register file                       │
└────────────────────────────────────────────────────────────┘

Pipeline Timing (5 instructions):
         Cycle:  1    2    3    4    5    6    7    8    9
ADD R1,R2,R3:  [IF] [ID] [EX] [MEM][WB]
SUB R4,R1,R5:       [IF] [ID] [EX] [MEM][WB]
MUL R6,R4,R7:            [IF] [ID] [EX] [MEM][WB]
AND R8,R6,R9:                 [IF] [ID] [EX] [MEM][WB]
OR R10,R8,R1:                      [IF] [ID] [EX] [MEM][WB]
```

---

## Pipeline Hazards (파이프라인 해저드)

### Concept (개념)

**Hazards (해저드)** are situations that prevent the next instruction from executing in the next clock cycle.

**1. Structural Hazards (구조적 해저드)**
- Two instructions need the same hardware resource simultaneously
- Example: Single memory port for both instruction fetch and data access
- Solution: Separate instruction and data caches (Harvard architecture)

**2. Data Hazards (데이터 해저드)**
- An instruction depends on the result of a previous instruction still in the pipeline
- Three types:
  - **RAW (Read After Write)**: Most common, true dependency
  - **WAR (Write After Read)**: Anti-dependency
  - **WAW (Write After Write)**: Output dependency

**3. Control Hazards (제어 해저드)**
- Branch instructions: don't know next PC until branch is resolved
- Causes **pipeline stall (파이프라인 지연)** or **bubble (버블)**

### Diagram / Example

```
Data Hazard Example (RAW):
┌────────────────────────────────────────────────────────────┐
│ ADD R1, R2, R3    ; R1 = R2 + R3                          │
│ SUB R4, R1, R5    ; R4 = R1 - R5  ← needs R1 from ADD!   │
└────────────────────────────────────────────────────────────┘

Without forwarding (stalls):
         Cycle:  1    2    3    4    5    6    7    8
ADD R1,R2,R3:  [IF] [ID] [EX] [MEM][WB]
SUB R4,R1,R5:       [IF] [ID] [**] [**] [EX] [MEM][WB]
                          ↑ stall ↑
                     Must wait for R1 to be written!

With forwarding (데이터 전방전달):
         Cycle:  1    2    3    4    5    6
ADD R1,R2,R3:  [IF] [ID] [EX] [MEM][WB]
SUB R4,R1,R5:       [IF] [ID] [EX] [MEM][WB]
                          ↑
                    Forward result from ADD's EX stage!

Forwarding Paths:
┌─────────────────────────────────────────────────────────────┐
│     ID          EX          MEM         WB                  │
│   ┌─────┐    ┌─────┐    ┌─────┐    ┌─────┐                │
│   │     │ ←──│     │←───│     │←───│     │  Forward paths │
│   └─────┘    └─────┘    └─────┘    └─────┘                │
└─────────────────────────────────────────────────────────────┘

Load-Use Hazard (cannot forward in time):
         Cycle:  1    2    3    4    5    6    7
LOAD R1,[R2]:  [IF] [ID] [EX] [MEM][WB]
ADD R3,R1,R4:       [IF] [ID] [**] [EX] [MEM][WB]
                          ↑
                    Must stall 1 cycle (load-use delay)
                    Data available after MEM, but needed in EX
```

---

## Branch Prediction (분기 예측)

### Concept (개념)

**Control hazards** from branches are expensive. Modern CPUs use **branch prediction (분기 예측)** to speculate on branch outcomes.

**Static Prediction (정적 예측):**
- Always predict taken or not taken
- Predict backward branches taken (loops)
- Simple but limited accuracy (~60-70%)

**Dynamic Prediction (동적 예측):**
- Learn from branch history
- **Branch History Table (BHT)**: Records recent branch outcomes
- **Branch Target Buffer (BTB)**: Caches branch target addresses

**Common Predictors:**
1. **1-bit predictor**: Remember last outcome
2. **2-bit predictor**: Saturating counter (strong/weak taken/not-taken)
3. **Tournament predictor**: Choose between local and global history
4. **Neural branch predictor**: Modern CPUs use ML-based prediction

**Misprediction Penalty:**
- When prediction is wrong, must flush pipeline
- Penalty = pipeline depth (5-20+ cycles on modern CPUs)
- Critical for performance!

### Diagram / Example

```
2-Bit Saturating Counter:
┌─────────────────────────────────────────────────────────┐
│                                                          │
│  ┌──────────┐    taken    ┌──────────┐                  │
│  │ Strongly │ ←────────── │ Weakly   │                  │
│  │  Taken   │ ──────────→ │  Taken   │                  │
│  │   (11)   │  not taken  │   (10)   │                  │
│  └──────────┘             └──────────┘                  │
│       ↑                         ↓                        │
│       │ taken             not taken                      │
│       │                         ↓                        │
│  ┌──────────┐             ┌──────────┐                  │
│  │ Weakly   │ ←────────── │ Strongly │                  │
│  │Not Taken │ ──────────→ │Not Taken │                  │
│  │   (01)   │    taken    │   (00)   │                  │
│  └──────────┘             └──────────┘                  │
│                                                          │
│  Requires 2 mispredictions to change prediction!        │
└─────────────────────────────────────────────────────────┘

Branch Prediction Example (loop):
┌────────────────────────────────────────────────────────────┐
│ for (int i = 0; i < 1000; i++) { ... }                    │
│                                                            │
│ Branch: if (i < 1000) goto loop_body                      │
│                                                            │
│ Iterations:  1    2    3   ...  999  1000                 │
│ Actual:      T    T    T   ...   T     N                  │
│ Predicted:   N    T    T   ...   T     T                  │
│ Result:     Miss  Hit  Hit ...  Hit   Miss                │
│                                                            │
│ Accuracy: 998/1000 = 99.8%                                │
└────────────────────────────────────────────────────────────┘

Pipeline Flush on Misprediction:
         Cycle:  1    2    3    4    5    6    7    8    9
BEQ R1,R2,100: [IF] [ID] [EX] [MEM][WB]
wrong inst 1:       [IF] [ID] [XX] ← Flushed!
wrong inst 2:            [IF] [XX] ← Flushed!
correct inst:                 [IF] [ID] [EX] [MEM][WB]
                              ↑
                    3 cycles wasted (bubble)
```

---

## Why Branches Are Slow (분기가 느린 이유)

### Concept (개념)

Branches disrupt the pipeline flow in multiple ways:

**1. Pipeline Flush (파이프라인 플러시)**
- Wrong predictions require discarding all speculative work
- Deeper pipelines = larger penalty
- Modern CPUs: 10-20 cycle penalty per misprediction

**2. Instruction Cache Misses**
- Branch targets may not be in cache
- Especially bad for indirect branches (function pointers, virtual calls)

**3. Limits Instruction-Level Parallelism**
- CPU cannot easily reorder across unpredictable branches
- Reduces effectiveness of out-of-order execution

**Performance Impact:**
```
Assuming:
- 20% of instructions are branches
- 90% prediction accuracy
- 15 cycle misprediction penalty

Effective CPI increase = 0.20 × 0.10 × 15 = 0.3 cycles per instruction
→ 30% performance loss from branch mispredictions alone!
```

### Diagram / Example

```
Optimization Techniques:

1. Branch Elimination (분기 제거):
┌────────────────────────────────────────────────────────────┐
│ // Before (branch)                                         │
│ if (x > 0) result = a;                                    │
│ else result = b;                                          │
│                                                            │
│ // After (branchless using conditional move)              │
│ result = (x > 0) ? a : b;  // CMOV instruction           │
│                                                            │
│ // Or using arithmetic                                     │
│ int mask = -(x > 0);  // All 1s if true, all 0s if false │
│ result = (a & mask) | (b & ~mask);                        │
└────────────────────────────────────────────────────────────┘

2. Loop Unrolling (루프 풀기):
┌────────────────────────────────────────────────────────────┐
│ // Before: 1000 branches                                   │
│ for (int i = 0; i < 1000; i++)                            │
│     sum += arr[i];                                        │
│                                                            │
│ // After: 250 branches (4x unrolled)                      │
│ for (int i = 0; i < 1000; i += 4) {                       │
│     sum += arr[i];                                        │
│     sum += arr[i+1];                                      │
│     sum += arr[i+2];                                      │
│     sum += arr[i+3];                                      │
│ }                                                          │
└────────────────────────────────────────────────────────────┘

3. Predictable Branch Patterns:
┌────────────────────────────────────────────────────────────┐
│ // Unpredictable (50% taken)                              │
│ if (random() % 2 == 0) { ... }                            │
│                                                            │
│ // Predictable (mostly taken)                              │
│ if (i < n) { ... }  // Loop condition                     │
│                                                            │
│ // Sort data to make branches predictable                  │
│ sort(data);                                                │
│ for (auto x : data)                                        │
│     if (x > threshold) { ... }  // All true, then all false│
└────────────────────────────────────────────────────────────┘

Real Performance Numbers:
┌─────────────────────────────────────────────────────────────┐
│ Test: Process array with condition check                    │
│                                                             │
│ Sorted array:    ~2.5 ns per element (predictable branch)  │
│ Random array:    ~12 ns per element (unpredictable)        │
│ Branchless code: ~3 ns per element (no branch penalty)     │
│                                                             │
│ Unpredictable branches are ~5x slower!                     │
└─────────────────────────────────────────────────────────────┘
```

---

## Summary (요약)

| Concept | Korean | Description |
|---------|--------|-------------|
| Pipeline | 파이프라인 | Overlapped instruction execution |
| Hazard | 해저드 | Situation preventing next instruction |
| Structural Hazard | 구조적 해저드 | Resource conflict |
| Data Hazard | 데이터 해저드 | Data dependency |
| Control Hazard | 제어 해저드 | Branch uncertainty |
| Forwarding | 전방전달 | Bypass data to avoid stalls |
| Branch Prediction | 분기 예측 | Guess branch outcome |

**Key Takeaways for Performance:**
1. **Minimize unpredictable branches** - sort data, use branchless code
2. **Loop unrolling** reduces branch overhead
3. **Conditional moves** (CMOV) eliminate simple branches
4. **Understand misprediction cost** - on modern CPUs, ~10-20 cycles
5. **Profile your code** - identify hot branches and optimize them
6. **Data-dependent branches are slow** - consider sorting or restructuring data
