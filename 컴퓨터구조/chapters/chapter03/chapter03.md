# Chapter 03. CPU Basics (CPU 기초)

> Understanding the internal components of the CPU and how it executes instructions to write more efficient code.

---

## CPU Components (CPU 구성 요소)

### Concept (개념)

The CPU consists of three main components:

**1. ALU (Arithmetic Logic Unit / 산술논리장치)**
- Performs all arithmetic operations (+, -, *, /)
- Performs all logical operations (AND, OR, NOT, XOR)
- Performs comparisons (<, >, ==)
- Contains status flags (Zero, Carry, Overflow, Negative)

**2. Registers (레지스터)**
- Fastest storage in the computer (access time: ~0.3 ns)
- Limited number (typically 16-32 general-purpose registers)
- Hold data being actively processed
- Types include: general-purpose, special-purpose, status registers

**3. Control Unit (제어장치)**
- Fetches instructions from memory
- Decodes instructions to determine operations
- Generates control signals to coordinate all components
- Manages the instruction cycle timing

### Diagram / Example

```
CPU Internal Architecture:
┌─────────────────────────────────────────────────────────────┐
│                         CPU                                  │
│  ┌─────────────────────────────────────────────────────┐    │
│  │              Control Unit (제어장치)                  │    │
│  │  ┌───────────┐  ┌───────────┐  ┌───────────────┐   │    │
│  │  │ Instruction│  │  Decoder  │  │Control Signal │   │    │
│  │  │  Register │→│  (해독기)  │→│  Generator   │   │    │
│  │  │    (IR)   │  │           │  │              │   │    │
│  │  └───────────┘  └───────────┘  └───────────────┘   │    │
│  └─────────────────────────────────────────────────────┘    │
│                              ↓ control signals               │
│  ┌────────────────┐      ┌─────────────────────────────┐    │
│  │   Registers    │      │    ALU (산술논리장치)         │    │
│  │  (레지스터)     │ ←──→ │                             │    │
│  │                │      │  ┌─────┐    ┌─────────┐    │    │
│  │  R0: 00000042  │      │  │ Add │    │  Flags  │    │    │
│  │  R1: 0000000A  │      │  │ Sub │    │ Z N C V │    │    │
│  │  R2: 00000000  │      │  │ AND │    │ 0 0 0 0 │    │    │
│  │  ...          │      │  │ OR  │    └─────────┘    │    │
│  │  PC: 00001000  │      │  │ ... │                   │    │
│  │  SP: 0000FFF0  │      │  └─────┘                   │    │
│  └────────────────┘      └─────────────────────────────┘    │
│         ↑↓                        ↑↓                         │
└─────────┼─────────────────────────┼─────────────────────────┘
          └──────────────┬──────────┘
                         ↓
               Memory Bus (메모리 버스)
```

---

## Register Types (레지스터 종류)

### Concept (개념)

**General Purpose Registers (범용 레지스터):**
- Hold data for computation
- Can be used for any purpose by programs
- Examples: R0-R15, EAX, EBX, ECX, EDX (x86)

**Special Purpose Registers (특수 목적 레지스터):**

| Register | Korean | Purpose |
|----------|--------|---------|
| PC (Program Counter) | 프로그램 카운터 | Address of next instruction |
| IR (Instruction Register) | 명령어 레지스터 | Current instruction being executed |
| SP (Stack Pointer) | 스택 포인터 | Top of the stack |
| BP/FP (Base/Frame Pointer) | 베이스/프레임 포인터 | Current stack frame |
| MAR (Memory Address Register) | 메모리 주소 레지스터 | Address for memory access |
| MDR (Memory Data Register) | 메모리 데이터 레지스터 | Data to/from memory |

**Status/Flag Register (상태 레지스터):**
- Zero Flag (Z): Result was zero
- Negative/Sign Flag (N/S): Result was negative
- Carry Flag (C): Unsigned overflow occurred
- Overflow Flag (V/O): Signed overflow occurred

### Diagram / Example

```
x86-64 Register Layout:
┌────────────────────────────────────────────────────────────┐
│  64-bit        32-bit      16-bit   8-bit high  8-bit low  │
│  ┌────────────┬───────────┬────────┬─────────┬─────────┐  │
│  │    RAX     │    EAX    │   AX   │   AH    │   AL    │  │
│  │    RBX     │    EBX    │   BX   │   BH    │   BL    │  │
│  │    RCX     │    ECX    │   CX   │   CH    │   CL    │  │
│  │    RDX     │    EDX    │   DX   │   DH    │   DL    │  │
│  └────────────┴───────────┴────────┴─────────┴─────────┘  │
│  │←────────────── 64 bits ───────────────────────────→│   │
│                │←─ 32 ──→│                                 │
│                          │← 16 →│                          │
│                                  │← 8→│←8→│                │
└────────────────────────────────────────────────────────────┘

Flag Register Example:
┌────────────────────────────────────────────────────────┐
│ Operation: 5 - 5 = 0                                   │
│ Flags:     Z=1 (result zero), N=0, C=0, V=0           │
├────────────────────────────────────────────────────────┤
│ Operation: 3 - 5 = -2                                  │
│ Flags:     Z=0, N=1 (negative), C=1 (borrow), V=0     │
├────────────────────────────────────────────────────────┤
│ Operation: 127 + 1 = -128 (8-bit signed)              │
│ Flags:     Z=0, N=1, C=0, V=1 (overflow!)             │
└────────────────────────────────────────────────────────┘

Performance Impact:
┌─────────────────────────────────────────────────────────┐
│ Storage Type    │ Access Time │ Relative Speed         │
├─────────────────┼─────────────┼────────────────────────┤
│ Register        │   ~0.3 ns   │ 1x (baseline)          │
│ L1 Cache        │   ~1 ns     │ 3x slower              │
│ L2 Cache        │   ~4 ns     │ 13x slower             │
│ L3 Cache        │   ~15 ns    │ 50x slower             │
│ Main Memory     │   ~100 ns   │ 300x slower            │
└─────────────────┴─────────────┴────────────────────────┘
```

---

## Instruction Cycle (명령어 사이클)

### Concept (개념)

Every instruction goes through the **Fetch-Decode-Execute Cycle**:

**1. Fetch (인출 단계)**
- Read instruction from memory at address in PC
- Load instruction into Instruction Register (IR)
- Increment PC to point to next instruction

**2. Decode (해독 단계)**
- Control Unit interprets the instruction
- Identify operation type (opcode)
- Identify operands (source and destination)
- Generate control signals

**3. Execute (실행 단계)**
- ALU performs the operation
- May involve memory read/write
- Results stored in register or memory
- Flags updated based on result

**Extended Cycle (for memory operations):**
- **Memory Access (메모리 접근)**: Load/Store data from/to memory
- **Write Back (결과 저장)**: Write result to destination register

### Diagram / Example

```
Instruction Cycle Flow:
┌─────────────────────────────────────────────────────────┐
│                  FETCH (인출)                            │
│  1. MAR ← PC          (Send address to memory)          │
│  2. IR ← Memory[MAR]  (Read instruction)                │
│  3. PC ← PC + 4       (Point to next instruction)       │
└─────────────────────────┬───────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────┐
│                  DECODE (해독)                           │
│  1. Analyze opcode (determine operation)                │
│  2. Identify registers/memory addresses                 │
│  3. Read operand values from registers                  │
└─────────────────────────┬───────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────┐
│                  EXECUTE (실행)                          │
│  1. ALU performs operation                              │
│  2. Calculate memory address (if needed)                │
│  3. Access memory (if load/store)                       │
│  4. Write result to register                            │
│  5. Update flags                                        │
└─────────────────────────┬───────────────────────────────┘
                          ↓
                    (Repeat for next instruction)

Example: ADD R1, R2, R3  (R1 = R2 + R3)
┌─────────────────────────────────────────────────────────┐
│ Clock 1 (Fetch):                                        │
│   PC = 0x1000                                           │
│   IR ← Memory[0x1000] = "ADD R1, R2, R3"               │
│   PC ← 0x1004                                           │
├─────────────────────────────────────────────────────────┤
│ Clock 2 (Decode):                                       │
│   Opcode: ADD (addition)                                │
│   Destination: R1                                       │
│   Sources: R2 (value=5), R3 (value=3)                  │
├─────────────────────────────────────────────────────────┤
│ Clock 3 (Execute):                                      │
│   ALU: 5 + 3 = 8                                        │
│   R1 ← 8                                                │
│   Flags: Z=0, N=0, C=0, V=0                            │
└─────────────────────────────────────────────────────────┘
```

---

## Instruction Format (명령어 형식)

### Concept (개념)

Instructions are encoded in binary with specific formats:

**Instruction Components:**
- **Opcode (연산 코드)**: What operation to perform
- **Operands (피연산자)**: Data or addresses to operate on
- **Addressing Mode (주소 지정 방식)**: How to interpret operands

**Common Addressing Modes:**

| Mode | Korean | Example | Meaning |
|------|--------|---------|---------|
| Immediate | 즉시 주소 | ADD R1, #5 | Use constant value 5 |
| Register | 레지스터 | ADD R1, R2 | Use value in R2 |
| Direct | 직접 주소 | LOAD R1, [1000] | Use value at address 1000 |
| Indirect | 간접 주소 | LOAD R1, [R2] | Address is in R2 |
| Indexed | 인덱스 주소 | LOAD R1, [R2+4] | Base + offset |

**Instruction Types:**
1. **Data Transfer**: MOV, LOAD, STORE
2. **Arithmetic**: ADD, SUB, MUL, DIV
3. **Logical**: AND, OR, XOR, NOT
4. **Control Flow**: JMP, CALL, RET, conditional branches
5. **Comparison**: CMP, TEST

### Diagram / Example

```
MIPS R-Type Instruction Format (32-bit):
┌────────┬───────┬───────┬───────┬───────┬────────┐
│ opcode │  rs   │  rt   │  rd   │ shamt │ funct  │
│ 6 bits │5 bits │5 bits │5 bits │5 bits │ 6 bits │
└────────┴───────┴───────┴───────┴───────┴────────┘

Example: ADD $t0, $s1, $s2
opcode = 0 (R-type)
rs = $s1 (register 17)
rt = $s2 (register 18)
rd = $t0 (register 8)
shamt = 0 (not a shift)
funct = 32 (ADD function)

Binary: 000000 10001 10010 01000 00000 100000

MIPS I-Type Instruction Format:
┌────────┬───────┬───────┬──────────────────────┐
│ opcode │  rs   │  rt   │     immediate        │
│ 6 bits │5 bits │5 bits │      16 bits         │
└────────┴───────┴───────┴──────────────────────┘

Example: ADDI $t0, $s1, 100  (t0 = s1 + 100)
opcode = 8 (ADDI)
rs = $s1 (source register)
rt = $t0 (destination register)
immediate = 100

x86 Variable-Length Instruction:
┌──────────────────────────────────────────────┐
│ x86 instructions: 1 to 15 bytes              │
│                                              │
│ Simple: INC EAX (1 byte: 0x40)              │
│                                              │
│ Complex: MOV [RBP+RDI*8+0x12345678], RAX    │
│          (up to 15 bytes with prefixes)     │
└──────────────────────────────────────────────┘

Performance Implication:
┌─────────────────────────────────────────────────────────┐
│ Fixed-length instructions (RISC):                       │
│   + Simpler decoding                                    │
│   + Easier pipelining                                   │
│   - More memory for code                                │
│                                                         │
│ Variable-length instructions (CISC/x86):                │
│   + Compact code                                        │
│   - Complex decoding                                    │
│   - Harder to pipeline (modern x86 converts to micro-ops)│
└─────────────────────────────────────────────────────────┘
```

---

## Summary (요약)

| Component | Korean | Function |
|-----------|--------|----------|
| ALU | 산술논리장치 | Performs computations |
| Registers | 레지스터 | Fast temporary storage |
| Control Unit | 제어장치 | Coordinates execution |
| PC | 프로그램 카운터 | Tracks current instruction |
| IR | 명령어 레지스터 | Holds current instruction |

**Instruction Cycle Steps:**
1. Fetch (인출) - Load instruction from memory
2. Decode (해독) - Interpret the instruction
3. Execute (실행) - Perform the operation

**Key Takeaways for Performance:**
1. Minimize memory accesses - keep data in registers
2. Simple instructions execute faster than complex ones
3. Understand addressing modes to write efficient code
4. Modern CPUs optimize by converting complex instructions to simpler micro-operations
5. Register allocation by compilers significantly impacts performance - use `-O2` or higher optimization levels
