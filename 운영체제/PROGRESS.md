# 운영체제 학습 계획

> 목표: OS 동작 원리를 이해하고 개발에 적용하는 수준

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

## STEP 1. 운영체제 기초

### Chapter 01. 운영체제 개요
> **doc**: `chapters/chapter01/chapter01.md`
> **examples**: `chapters/chapter01/examples/`

- [ ] 운영체제란 무엇인가
- [ ] 운영체제의 역할: 자원 관리, 추상화
- [ ] 커널 모드 vs 사용자 모드
- [ ] 시스템 콜
- [ ] 운영체제 구조: 모놀리식, 마이크로커널

---

## STEP 2. 프로세스와 스레드

### Chapter 02. 프로세스
> **doc**: `chapters/chapter02/chapter02.md`
> **examples**: `chapters/chapter02/examples/`

- [ ] 프로세스 개념
- [ ] 프로세스 상태: New, Ready, Running, Waiting, Terminated
- [ ] PCB - Process Control Block
- [ ] 컨텍스트 스위칭
- [ ] 프로세스 생성: fork, exec
- [ ] 프로세스 종료

---

### Chapter 03. 스레드
> **doc**: `chapters/chapter03/chapter03.md`
> **examples**: `chapters/chapter03/examples/`

- [ ] 스레드 개념
- [ ] 프로세스 vs 스레드
- [ ] 멀티스레딩의 장점
- [ ] 사용자 스레드 vs 커널 스레드
- [ ] 스레드 모델: 1:1, N:1, N:M

---

### Chapter 04. CPU 스케줄링
> **doc**: `chapters/chapter04/chapter04.md`
> **examples**: `chapters/chapter04/examples/`

- [ ] 스케줄링 개념
- [ ] 스케줄링 기준: CPU 사용률, 처리량 등
- [ ] FCFS - First Come First Served
- [ ] SJF - Shortest Job First
- [ ] 우선순위 스케줄링
- [ ] Round Robin
- [ ] 다단계 큐

---

## STEP 3. 프로세스 동기화

### Chapter 05. 동기화 기초
> **doc**: `chapters/chapter05/chapter05.md`
> **examples**: `chapters/chapter05/examples/`

- [ ] 임계 영역 문제
- [ ] 경쟁 조건 (Race Condition)
- [ ] 상호 배제 조건
- [ ] 피터슨 해결책
- [ ] 하드웨어 해결책: Test-and-Set, Compare-and-Swap

---

### Chapter 06. 동기화 도구
> **doc**: `chapters/chapter06/chapter06.md`
> **examples**: `chapters/chapter06/examples/`

- [ ] 뮤텍스 (Mutex)
- [ ] 세마포어 (Semaphore)
- [ ] 모니터 (Monitor)
- [ ] 조건 변수
- [ ] 동기화 고전 문제

---

### Chapter 07. 데드락
> **doc**: `chapters/chapter07/chapter07.md`
> **examples**: `chapters/chapter07/examples/`

- [ ] 데드락 개념
- [ ] 데드락 조건: 상호 배제, 점유 대기 등
- [ ] 자원 할당 그래프
- [ ] 데드락 예방
- [ ] 데드락 회피: Banker's Algorithm
- [ ] 데드락 탐지와 회복

---

## STEP 4. 메모리 관리

### Chapter 08. 메모리 관리 기초
> **doc**: `chapters/chapter08/chapter08.md`
> **examples**: `chapters/chapter08/examples/`

- [ ] 메모리 계층
- [ ] 주소 바인딩: 컴파일, 로드, 실행 시점
- [ ] 논리 주소 vs 물리 주소
- [ ] 연속 메모리 할당
- [ ] 단편화: 내부, 외부
- [ ] 페이징 개념
- [ ] 세그멘테이션

---

### Chapter 09. 가상 메모리
> **doc**: `chapters/chapter09/chapter09.md`
> **examples**: `chapters/chapter09/examples/`

- [ ] 가상 메모리 개념
- [ ] 요구 페이징
- [ ] 페이지 폴트
- [ ] 페이지 교체 알고리즘: FIFO, LRU, Optimal
- [ ] 스래싱
- [ ] 워킹 셋

---

## STEP 5. 파일 시스템

### Chapter 10. 파일 시스템 인터페이스
> **doc**: `chapters/chapter10/chapter10.md`
> **examples**: `chapters/chapter10/examples/`

- [ ] 파일 개념
- [ ] 파일 속성과 연산
- [ ] 디렉토리 구조
- [ ] 파일 접근 방법
- [ ] 파일 보호

---

### Chapter 11. 파일 시스템 구현
> **doc**: `chapters/chapter11/chapter11.md`
> **examples**: `chapters/chapter11/examples/`

- [ ] 파일 시스템 구조
- [ ] 디스크 할당 방법: 연속, 연결, 인덱스
- [ ] 빈 공간 관리
- [ ] 디스크 스케줄링: FCFS, SSTF, SCAN, C-SCAN

---

### Chapter 12. 입출력과 보호
> **doc**: `chapters/chapter12/chapter12.md`
> **examples**: `chapters/chapter12/examples/`

- [ ] 입출력 하드웨어
- [ ] 입출력 방식: 폴링, 인터럽트, DMA
- [ ] 버퍼링, 캐싱, 스풀링
- [ ] 보호 기초

---

## 실무 연결 포인트

| OS 개념 | 실무 적용 |
|---------|----------|
| 프로세스/스레드 | 멀티프로세싱, 비동기 프로그래밍 |
| 동기화 | 동시성 프로그래밍, 락 처리 |
| 메모리 관리 | 메모리 누수, 성능 최적화 |
| 파일 시스템 | 파일 I/O, 스토리지 설계 |

---

## 나중에 필요할 때

| 주제 | 언제 |
|------|------|
| 분산 시스템 | 대규모 서비스 |
| 가상화, 컨테이너 | DevOps, Cloud |
| 실시간 OS | 임베디드, 게임 엔진 |
