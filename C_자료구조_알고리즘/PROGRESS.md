# C 언어 + 자료구조 + 알고리즘

> 목표: 컴퓨터/언어 이해도 향상 + C++ 학습 전 기초

---

## AI 학습 가이드

### 역할
- AI는 이 커리큘럼으로 학습을 시켜주는 **시니어 개발자이자 교수**

### 학습 진행 흐름 (5단계)

**1단계: 진행도 확인** - "오늘 어디 할 차례야"
**2단계: 설명** - "학습 내용 보여줘" → 해당 챕터 .md 읽기
**3단계: 예제** - "예제 보여줘" → examples/ 폴더 확인
**4단계: 과제** - "과제 줘"
**5단계: 평가** - [코드 제출] → 통과 = [x] 체크

### 평가 기준
□ 학습한 개념을 이해하고 적용했는가

---

## PART 1: C 언어 핵심

### Chapter 01. C 기본 문법
> **doc**: `chapters/chapter01/chapter01.md`
> **examples**: `chapters/chapter01/examples/`

- [ ] 컴파일러 (gcc -o)
- [ ] 변수와 타입: int, float, char
- [ ] 입출력: printf, scanf
- [ ] 연산자: 산술, 비교, 논리
- [ ] 조건문: if, switch
- [ ] 반복문: for, while
- [ ] 함수: 선언, 매개변수, 반환

---

### Chapter 02. 포인터와 메모리
> **doc**: `chapters/chapter02/chapter02.md`
> **examples**: `chapters/chapter02/examples/`

- [ ] 메모리 주소 개념
- [ ] 포인터 선언 (*)
- [ ] 주소 연산자 (&)
- [ ] 역참조 (*)
- [ ] 배열과 포인터 관계
- [ ] malloc / free
- [ ] 스택 vs 힙 메모리

---

### Chapter 03. 구조체
> **doc**: `chapters/chapter03/chapter03.md`
> **examples**: `chapters/chapter03/examples/`

- [ ] 구조체 정의: struct
- [ ] 멤버 접근: .
- [ ] 구조체 포인터: ->
- [ ] typedef
- [ ] 동적 구조체 할당

---

## PART 2: 자료구조

### Chapter 04. 알고리즘 분석
> **doc**: `chapters/chapter04/chapter04.md`
> **examples**: `chapters/chapter04/examples/`

- [ ] 알고리즘이란
- [ ] 시간 복잡도 개념
- [ ] 공간 복잡도 개념
- [ ] Big-O 표기법
- [ ] O(1), O(n), O(n²), O(log n), O(n log n)

---

### Chapter 05. 배열
> **doc**: `chapters/chapter05/chapter05.md`
> **examples**: `chapters/chapter05/examples/`

- [ ] 배열 메모리 구조
- [ ] 정적 배열 vs 동적 배열
- [ ] 삽입/삭제 시간 복잡도
- [ ] 2차원 배열

---

### Chapter 06. 연결 리스트
> **doc**: `chapters/chapter06/chapter06.md`
> **examples**: `chapters/chapter06/examples/`

- [ ] 연결 리스트 개념
- [ ] 단일 연결 리스트
- [ ] 이중 연결 리스트
- [ ] 배열 vs 연결 리스트

---

### Chapter 07. 스택
> **doc**: `chapters/chapter07/chapter07.md`
> **examples**: `chapters/chapter07/examples/`

- [ ] 스택 개념: LIFO
- [ ] push, pop, peek, isEmpty
- [ ] 배열 기반 스택
- [ ] 연결 리스트 기반 스택
- [ ] 활용: 괄호 검사, 후위 표기법

---

### Chapter 08. 큐
> **doc**: `chapters/chapter08/chapter08.md`
> **examples**: `chapters/chapter08/examples/`

- [ ] 큐 개념: FIFO
- [ ] enqueue, dequeue, front, isEmpty
- [ ] 선형 큐 문제점
- [ ] 원형 큐
- [ ] 덱 (Deque)

---

### Chapter 09. 정렬 - 기본
> **doc**: `chapters/chapter09/chapter09.md`
> **examples**: `chapters/chapter09/examples/`

- [ ] 버블 정렬: O(n²)
- [ ] 선택 정렬: O(n²)
- [ ] 삽입 정렬: O(n²)
- [ ] 안정 정렬 vs 불안정 정렬

---

### Chapter 10. 정렬 - 고급
> **doc**: `chapters/chapter10/chapter10.md`
> **examples**: `chapters/chapter10/examples/`

- [ ] 퀵 정렬: 평균 O(n log n)
- [ ] 병합 정렬: 항상 O(n log n)
- [ ] 분할 정복 개념
- [ ] 재귀 호출

---

### Chapter 11. 탐색
> **doc**: `chapters/chapter11/chapter11.md`
> **examples**: `chapters/chapter11/examples/`

- [ ] 선형 탐색: O(n)
- [ ] 이진 탐색: O(log n)
- [ ] 이진 탐색 조건: 정렬된 배열
- [ ] 재귀 vs 반복 구현

---

## PART 3: 트리와 그래프

### Chapter 12. 트리 기초
> **doc**: `chapters/chapter12/chapter12.md`
> **examples**: `chapters/chapter12/examples/`

- [ ] 트리 용어: 루트, 노드, 리프, 깊이, 높이
- [ ] 이진 트리
- [ ] 순회: 전위, 중위, 후위, 레벨
- [ ] 재귀로 순회 구현

---

### Chapter 13. 이진 탐색 트리
> **doc**: `chapters/chapter13/chapter13.md`
> **examples**: `chapters/chapter13/examples/`

- [ ] BST 개념
- [ ] BST 삽입
- [ ] BST 탐색
- [ ] BST 삭제: 세 가지 경우
- [ ] 시간 복잡도: 평균 O(log n)

---

### Chapter 14. 힙
> **doc**: `chapters/chapter14/chapter14.md`
> **examples**: `chapters/chapter14/examples/`

- [ ] 힙 개념: 완전 이진 트리
- [ ] 최대 힙 / 최소 힙
- [ ] 삽입: Heapify Up
- [ ] 삭제: Heapify Down
- [ ] 힙 정렬
- [ ] 우선순위 큐

---

### Chapter 15. 그래프
> **doc**: `chapters/chapter15/chapter15.md`
> **examples**: `chapters/chapter15/examples/`

- [ ] 그래프 용어: 정점, 간선, 인접
- [ ] 방향/무방향 그래프
- [ ] 인접 행렬
- [ ] 인접 리스트
- [ ] DFS: 깊이 우선 탐색
- [ ] BFS: 너비 우선 탐색

---

## PART 4: 알고리즘 심화

### Chapter 16. 해시 테이블
> **doc**: `chapters/chapter16/chapter16.md`
> **examples**: `chapters/chapter16/examples/`

- [ ] 해시 함수 개념
- [ ] 해시 충돌
- [ ] 체이닝
- [ ] 개방 주소법
- [ ] 시간 복잡도: 평균 O(1)

---

### Chapter 17. 동적 프로그래밍
> **doc**: `chapters/chapter17/chapter17.md`
> **examples**: `chapters/chapter17/examples/`

- [ ] DP 개념
- [ ] 메모이제이션: Top-Down
- [ ] 타뷸레이션: Bottom-Up
- [ ] 최적 부분 구조
- [ ] 중복 부분 문제

---

### Chapter 18. 기말 프로젝트
> **doc**: `chapters/chapter18/chapter18.md`
> **examples**: `chapters/chapter18/examples/`

- [ ] 학생 성적 관리 시스템

---

## 학습 완료 후

```
✓ 포인터/메모리 이해
✓ 자료구조 직접 구현 경험
✓ 기본 알고리즘 구현
✓ CS 전공 기초 확립
✓ C++ 학습 준비 완료
```

---

## 다음 단계

→ C++ 게임 개발 학습 계획
