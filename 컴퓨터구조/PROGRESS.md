# Computer Architecture Study Plan (컴퓨터 구조 학습 계획) - Condensed Version

> Goal: Understand core concepts needed for game/app performance optimization
> 목표: 게임/앱 성능 최적화에 필요한 핵심 개념만 이해하는 수준

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

## STEP 1. Computer Architecture Basics (컴퓨터 구조 기초)

### Chapter 01. Overview (개요)
> **doc**: `chapters/chapter01/chapter01.md`
> **examples**: `chapters/chapter01/examples/`

- [ ] Computer Components: CPU, Memory, I/O (컴퓨터 구성 요소)
- [ ] Von Neumann Architecture (폰 노이만 아키텍처)
- [ ] Instruction and Data Flow (명령어와 데이터의 흐름)
- [ ] Clock and Performance Metrics (클럭과 성능 지표)

---

### Chapter 02. Data Representation (데이터 표현)
> **doc**: `chapters/chapter02/chapter02.md`
> **examples**: `chapters/chapter02/examples/`

- [ ] Binary and Hexadecimal (2진수와 16진수)
- [ ] Integer Representation: Sign Bit, Two's Complement (정수 표현)
- [ ] Floating Point: IEEE 754 (부동소수점 표현)
- [ ] Floating Point Precision Errors (부동소수점 오차)

---

## STEP 2. CPU

### Chapter 03. CPU Basics (CPU 기초)
> **doc**: `chapters/chapter03/chapter03.md`
> **examples**: `chapters/chapter03/examples/`

- [ ] CPU Components: ALU, Registers, Control Unit (CPU 구성 요소)
- [ ] Instruction Cycle: Fetch-Decode-Execute (명령어 사이클)
- [ ] Register Types: General, Special (레지스터 종류)
- [ ] Instruction Format (명령어 형식)

---

### Chapter 04. Pipeline (파이프라인)
> **doc**: `chapters/chapter04/chapter04.md`
> **examples**: `chapters/chapter04/examples/`

- [ ] Pipeline Concept (파이프라인 개념)
- [ ] Pipeline Stages: IF, ID, EX, MEM, WB (파이프라인 단계)
- [ ] Pipeline Hazards: Structural, Data, Control (파이프라인 해저드)
- [ ] Branch Prediction (분기 예측)
- [ ] Why Branches are Slow: Practical Perspective (왜 분기문이 느린가)

---

## STEP 3. Memory Hierarchy (메모리 계층)

### Chapter 05. Memory Hierarchy Structure (메모리 계층 구조)
> **doc**: `chapters/chapter05/chapter05.md`
> **examples**: `chapters/chapter05/examples/`

- [ ] Memory Hierarchy Concept (메모리 계층 개념)
- [ ] Speed vs Capacity vs Cost Trade-offs (속도 vs 용량 vs 비용)
- [ ] Locality Principle: Temporal, Spatial (지역성 원리)

---

### Chapter 06. Cache Memory (캐시 메모리)
> **doc**: `chapters/chapter06/chapter06.md`
> **examples**: `chapters/chapter06/examples/`

- [ ] Cache Concept and Purpose (캐시 개념과 필요성)
- [ ] Cache Hit vs Cache Miss (캐시 히트 vs 캐시 미스)
- [ ] Cache Line (Block) (캐시 라인)
- [ ] Mapping: Direct, Fully Associative, Set Associative (매핑 방식)
- [ ] Replacement Policies: LRU (캐시 교체 정책)
- [ ] Write Policies: Write-through, Write-back (쓰기 정책)

---

### Chapter 07. Cache-Friendly Programming (캐시 친화적 프로그래밍)
> **doc**: `chapters/chapter07/chapter07.md`
> **examples**: `chapters/chapter07/examples/`

- [ ] Cache Miss Impact on Performance (캐시 미스가 성능에 미치는 영향)
- [ ] Array Traversal: Row-major vs Column-major (배열 순회 방향)
- [ ] Data Structures and Cache Efficiency (데이터 구조와 캐시 효율)
- [ ] False Sharing Problem (False Sharing 문제)
- [ ] Cache Optimization in Games/Graphics (게임/그래픽에서 캐시 최적화)

---

## Practical Connections (실무 연결 포인트)

| Architecture Concept | Practical Application |
|---------------------|----------------------|
| Floating Point | Precision errors, Game physics |
| Pipeline/Branch Prediction | Minimize branches, Optimization |
| Cache/Locality | Memory access pattern optimization |
| Cache-Friendly Coding | Game loops, Large data processing |

---

## Skipped Topics (스킵하는 내용)

| Topic | Why Skip |
|-------|----------|
| Instruction Sets: RISC, CISC | Not needed unless embedded/compiler dev |
| I/O Details: Interrupts, DMA | Not needed unless driver dev |
| Bus Architecture | Not needed unless hardware design |

---

## For Later (나중에 필요할 때)

| Topic | When |
|-------|------|
| GPU Architecture | Game graphics, GPGPU |
| SIMD: SSE, AVX | Game engine optimization |
| Multi-core/Threading | Covered in OS |
