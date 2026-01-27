# 컴퓨터 구조 학습 계획 (압축 버전)

> 목표: 게임/앱 성능 최적화에 필요한 핵심 개념만 이해하는 수준

---

## AI 학습 가이드

### 역할
- AI는 이 커리큘럼으로 학습을 시켜주는 **시니어 개발자이자 교수**

### 학습 진행 흐름 (5단계)

**1단계: 진행도 확인** - "오늘 어디 할 차례야"
**2단계: 설명** - "학습 내용 보여줘" → 해당 챕터 .md 읽기
**3단계: 예제** - "예제 보여줘" → examples/ 폴더 확인
**4단계: 과제** - "과제 줘"
**5단계: 평가** - [답안 제출] → 통과 = [x] 체크

### 평가 기준
□ 학습한 개념을 이해하고 적용했는가

---

## STEP 1. 컴퓨터 구조 기초

### Chapter 01. 개요
> **doc**: `chapters/chapter01/chapter01.md`
> **examples**: `chapters/chapter01/examples/`

- [ ] 컴퓨터 구성 요소: CPU, 메모리, I/O
- [ ] 폰 노이만 아키텍처
- [ ] 명령어와 데이터의 흐름
- [ ] 클럭과 성능 지표

---

### Chapter 02. 데이터 표현
> **doc**: `chapters/chapter02/chapter02.md`
> **examples**: `chapters/chapter02/examples/`

- [ ] 2진수와 16진수
- [ ] 정수 표현: 부호 비트, 2의 보수
- [ ] 부동소수점 표현: IEEE 754
- [ ] 부동소수점 오차

---

## STEP 2. CPU

### Chapter 03. CPU 기초
> **doc**: `chapters/chapter03/chapter03.md`
> **examples**: `chapters/chapter03/examples/`

- [ ] CPU 구성 요소: ALU, 레지스터, 제어 장치
- [ ] 명령어 사이클: Fetch-Decode-Execute
- [ ] 레지스터 종류: 범용, 특수 목적
- [ ] 명령어 형식

---

### Chapter 04. 파이프라인
> **doc**: `chapters/chapter04/chapter04.md`
> **examples**: `chapters/chapter04/examples/`

- [ ] 파이프라인 개념
- [ ] 파이프라인 단계: IF, ID, EX, MEM, WB
- [ ] 파이프라인 해저드: 구조적, 데이터, 제어
- [ ] 분기 예측
- [ ] 왜 분기문이 느린가: 실무 관점

---

## STEP 3. 메모리 계층

### Chapter 05. 메모리 계층 구조
> **doc**: `chapters/chapter05/chapter05.md`
> **examples**: `chapters/chapter05/examples/`

- [ ] 메모리 계층 개념
- [ ] 속도 vs 용량 vs 비용 트레이드오프
- [ ] 지역성 원리: 시간적, 공간적

---

### Chapter 06. 캐시 메모리
> **doc**: `chapters/chapter06/chapter06.md`
> **examples**: `chapters/chapter06/examples/`

- [ ] 캐시 개념과 필요성
- [ ] 캐시 히트 vs 캐시 미스
- [ ] 캐시 라인(블록)
- [ ] 매핑 방식: 직접, 완전 연관, 집합 연관
- [ ] 캐시 교체 정책: LRU
- [ ] 쓰기 정책: Write-through, Write-back

---

### Chapter 07. 캐시 친화적 프로그래밍
> **doc**: `chapters/chapter07/chapter07.md`
> **examples**: `chapters/chapter07/examples/`

- [ ] 캐시 미스가 성능에 미치는 영향
- [ ] 배열 순회 방향: Row-major vs Column-major
- [ ] 데이터 구조와 캐시 효율
- [ ] False Sharing 문제
- [ ] 게임/그래픽에서 캐시 최적화

---

## 실무 연결 포인트

| 구조 개념 | 실무 적용 |
|----------|----------|
| 부동소수점 | 정밀도 오류, 게임 물리 |
| 파이프라인/분기 예측 | 분기 최소화, 최적화 |
| 캐시/지역성 | 메모리 접근 패턴 최적화 |
| 캐시 친화적 코딩 | 게임 루프, 대용량 데이터 처리 |

---

## 스킵하는 내용

| 주제 | 스킵 이유 |
|------|----------|
| 명령어 집합: RISC, CISC | 임베디드/컴파일러 개발 아니면 불필요 |
| I/O 상세: 인터럽트, DMA | 드라이버 개발 아니면 불필요 |
| 버스 아키텍처 | 하드웨어 설계 아니면 불필요 |

---

## 나중에 필요할 때

| 주제 | 언제 |
|------|------|
| GPU 아키텍처 | 게임 그래픽스, GPGPU |
| SIMD: SSE, AVX | 게임 엔진 최적화 |
| 멀티코어/스레딩 | OS에서 다룸 |
