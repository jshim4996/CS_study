# C Language + Data Structures + Algorithms (C 언어 + 자료구조 + 알고리즘)

> Goal: Improve understanding of computers/languages + Foundation before C++
> 목표: 컴퓨터/언어 이해도 향상 + C++ 학습 전 기초

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
**Step 5: Evaluation** - [Submit code] → Pass = [x] check

### Evaluation Criteria (평가 기준)
□ Did you understand and apply the concept? (학습한 개념을 이해하고 적용했는가)

---

## PART 1: C Language Core (C 언어 핵심)

### Chapter 01. C Basic Syntax (C 기본 문법)
> **doc**: `chapters/chapter01/chapter01.md`
> **examples**: `chapters/chapter01/examples/`

- [ ] Compiler (gcc -o) (컴파일러)
- [ ] Variables and Types: int, float, char (변수와 타입)
- [ ] Input/Output: printf, scanf (입출력)
- [ ] Operators: Arithmetic, Comparison, Logical (연산자)
- [ ] Conditionals: if, switch (조건문)
- [ ] Loops: for, while (반복문)
- [ ] Functions: Declaration, Parameters, Return (함수)

---

### Chapter 02. Pointers and Memory (포인터와 메모리)
> **doc**: `chapters/chapter02/chapter02.md`
> **examples**: `chapters/chapter02/examples/`

- [ ] Memory Address Concept (메모리 주소 개념)
- [ ] Pointer Declaration (*) (포인터 선언)
- [ ] Address Operator (&) (주소 연산자)
- [ ] Dereference (*) (역참조)
- [ ] Array and Pointer Relationship (배열과 포인터 관계)
- [ ] malloc / free
- [ ] Stack vs Heap Memory (스택 vs 힙 메모리)

---

### Chapter 03. Structures (구조체)
> **doc**: `chapters/chapter03/chapter03.md`
> **examples**: `chapters/chapter03/examples/`

- [ ] Structure Definition: struct (구조체 정의)
- [ ] Member Access: . (멤버 접근)
- [ ] Structure Pointer: -> (구조체 포인터)
- [ ] typedef
- [ ] Dynamic Structure Allocation (동적 구조체 할당)

---

## PART 2: Data Structures (자료구조)

### Chapter 04. Algorithm Analysis (알고리즘 분석)
> **doc**: `chapters/chapter04/chapter04.md`
> **examples**: `chapters/chapter04/examples/`

- [ ] What is an Algorithm (알고리즘이란)
- [ ] Time Complexity Concept (시간 복잡도 개념)
- [ ] Space Complexity Concept (공간 복잡도 개념)
- [ ] Big-O Notation (Big-O 표기법)
- [ ] O(1), O(n), O(n²), O(log n), O(n log n)

---

### Chapter 05. Arrays (배열)
> **doc**: `chapters/chapter05/chapter05.md`
> **examples**: `chapters/chapter05/examples/`

- [ ] Array Memory Structure (배열 메모리 구조)
- [ ] Static vs Dynamic Array (정적 배열 vs 동적 배열)
- [ ] Insert/Delete Time Complexity (삽입/삭제 시간 복잡도)
- [ ] 2D Array (2차원 배열)

---

### Chapter 06. Linked List (연결 리스트)
> **doc**: `chapters/chapter06/chapter06.md`
> **examples**: `chapters/chapter06/examples/`

- [ ] Linked List Concept (연결 리스트 개념)
- [ ] Singly Linked List (단일 연결 리스트)
- [ ] Doubly Linked List (이중 연결 리스트)
- [ ] Array vs Linked List (배열 vs 연결 리스트)

---

### Chapter 07. Stack (스택)
> **doc**: `chapters/chapter07/chapter07.md`
> **examples**: `chapters/chapter07/examples/`

- [ ] Stack Concept: LIFO (스택 개념)
- [ ] push, pop, peek, isEmpty
- [ ] Array-based Stack (배열 기반 스택)
- [ ] Linked List-based Stack (연결 리스트 기반 스택)
- [ ] Applications: Bracket Matching, Postfix Notation (활용)

---

### Chapter 08. Queue (큐)
> **doc**: `chapters/chapter08/chapter08.md`
> **examples**: `chapters/chapter08/examples/`

- [ ] Queue Concept: FIFO (큐 개념)
- [ ] enqueue, dequeue, front, isEmpty
- [ ] Linear Queue Problem (선형 큐 문제점)
- [ ] Circular Queue (원형 큐)
- [ ] Deque

---

### Chapter 09. Sorting - Basic (정렬 기본)
> **doc**: `chapters/chapter09/chapter09.md`
> **examples**: `chapters/chapter09/examples/`

- [ ] Bubble Sort: O(n²) (버블 정렬)
- [ ] Selection Sort: O(n²) (선택 정렬)
- [ ] Insertion Sort: O(n²) (삽입 정렬)
- [ ] Stable vs Unstable Sort (안정 정렬 vs 불안정 정렬)

---

### Chapter 10. Sorting - Advanced (정렬 고급)
> **doc**: `chapters/chapter10/chapter10.md`
> **examples**: `chapters/chapter10/examples/`

- [ ] Quick Sort: O(n log n) average (퀵 정렬)
- [ ] Merge Sort: O(n log n) always (병합 정렬)
- [ ] Divide and Conquer Concept (분할 정복 개념)
- [ ] Recursive Calls (재귀 호출)

---

### Chapter 11. Searching (탐색)
> **doc**: `chapters/chapter11/chapter11.md`
> **examples**: `chapters/chapter11/examples/`

- [ ] Linear Search: O(n) (선형 탐색)
- [ ] Binary Search: O(log n) (이진 탐색)
- [ ] Binary Search Condition: Sorted Array (정렬된 배열)
- [ ] Recursive vs Iterative Implementation (재귀 vs 반복)

---

## PART 3: Trees and Graphs (트리와 그래프)

### Chapter 12. Tree Basics (트리 기초)
> **doc**: `chapters/chapter12/chapter12.md`
> **examples**: `chapters/chapter12/examples/`

- [ ] Tree Terms: Root, Node, Leaf, Depth, Height (트리 용어)
- [ ] Binary Tree (이진 트리)
- [ ] Traversal: Preorder, Inorder, Postorder, Level (순회)
- [ ] Implement Traversal with Recursion (재귀로 순회 구현)

---

### Chapter 13. Binary Search Tree (이진 탐색 트리)
> **doc**: `chapters/chapter13/chapter13.md`
> **examples**: `chapters/chapter13/examples/`

- [ ] BST Concept (BST 개념)
- [ ] BST Insert (BST 삽입)
- [ ] BST Search (BST 탐색)
- [ ] BST Delete: Three Cases (BST 삭제)
- [ ] Time Complexity: Average O(log n)

---

### Chapter 14. Heap (힙)
> **doc**: `chapters/chapter14/chapter14.md`
> **examples**: `chapters/chapter14/examples/`

- [ ] Heap Concept: Complete Binary Tree (힙 개념)
- [ ] Max Heap / Min Heap (최대 힙 / 최소 힙)
- [ ] Insert: Heapify Up (삽입)
- [ ] Delete: Heapify Down (삭제)
- [ ] Heap Sort (힙 정렬)
- [ ] Priority Queue (우선순위 큐)

---

### Chapter 15. Graph (그래프)
> **doc**: `chapters/chapter15/chapter15.md`
> **examples**: `chapters/chapter15/examples/`

- [ ] Graph Terms: Vertex, Edge, Adjacent (그래프 용어)
- [ ] Directed/Undirected Graph (방향/무방향 그래프)
- [ ] Adjacency Matrix (인접 행렬)
- [ ] Adjacency List (인접 리스트)
- [ ] DFS: Depth-First Search (깊이 우선 탐색)
- [ ] BFS: Breadth-First Search (너비 우선 탐색)

---

## PART 4: Algorithm Advanced (알고리즘 심화)

### Chapter 16. Hash Table (해시 테이블)
> **doc**: `chapters/chapter16/chapter16.md`
> **examples**: `chapters/chapter16/examples/`

- [ ] Hash Function Concept (해시 함수 개념)
- [ ] Hash Collision (해시 충돌)
- [ ] Chaining (체이닝)
- [ ] Open Addressing (개방 주소법)
- [ ] Time Complexity: Average O(1)

---

### Chapter 17. Dynamic Programming (동적 프로그래밍)
> **doc**: `chapters/chapter17/chapter17.md`
> **examples**: `chapters/chapter17/examples/`

- [ ] DP Concept (DP 개념)
- [ ] Memoization: Top-Down (메모이제이션)
- [ ] Tabulation: Bottom-Up (타뷸레이션)
- [ ] Optimal Substructure (최적 부분 구조)
- [ ] Overlapping Subproblems (중복 부분 문제)

---

### Chapter 18. Final Project (기말 프로젝트)
> **doc**: `chapters/chapter18/chapter18.md`
> **examples**: `chapters/chapter18/examples/`

- [ ] Student Grade Management System (학생 성적 관리 시스템)

---

## After Completion (학습 완료 후)

```
✓ Pointer/Memory understanding (포인터/메모리 이해)
✓ Data structure implementation experience (자료구조 직접 구현 경험)
✓ Basic algorithm implementation (기본 알고리즘 구현)
✓ CS foundation established (CS 전공 기초 확립)
✓ Ready for C++ learning (C++ 학습 준비 완료)
```

---

## Next Step (다음 단계)

→ C++ Game Development Study Plan
