# Chapter 08. Queue (큐)

## Table of Contents (목차)
1. [Queue Concept - FIFO (큐 개념 - 선입선출)](#1-queue-concept---fifo-큐-개념---선입선출)
2. [Queue Operations (큐 연산)](#2-queue-operations-큐-연산)
3. [Linear Queue and its Problem (선형 큐와 문제점)](#3-linear-queue-and-its-problem-선형-큐와-문제점)
4. [Circular Queue (원형 큐)](#4-circular-queue-원형-큐)
5. [Deque - Double Ended Queue (덱 - 양방향 큐)](#5-deque---double-ended-queue-덱---양방향-큐)
6. [Summary (요약)](#6-summary-요약)

---

## 1. Queue Concept - FIFO (큐 개념 - 선입선출)

### What is a Queue? (큐란?)

A **Queue** is a linear data structure that follows the **FIFO (First In, First Out)** principle. The first element added to the queue is the first one to be removed.

**큐**는 **FIFO (First In, First Out - 선입선출)** 원리를 따르는 선형 자료 구조입니다. 큐에 처음으로 추가된 요소가 가장 먼저 제거됩니다.

```
┌─────────────────────────────────────────────────────────────────┐
│                      QUEUE CONCEPT (큐 개념)                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│      FIFO: First In, First Out (선입선출)                        │
│                                                                 │
│      Enqueue                                         Dequeue    │
│      (삽입)                                          (제거)     │
│         │                                              │        │
│         ▼                                              ▼        │
│       ┌─────┬─────┬─────┬─────┬─────┐                          │
│       │  D  │  C  │  B  │  A  │     │  ──────▶   A              │
│       └─────┴─────┴─────┴─────┴─────┘                          │
│         ▲                       ▲                               │
│        Rear                   Front                             │
│       (뒤)                    (앞)                              │
│                                                                 │
│      Real-world examples (실제 예시):                            │
│      ┌─────────────────────────────────────────────────────┐   │
│      │  🎫 Waiting line at ticket counter (매표소 대기줄)   │   │
│      │  🖨️  Print job queue (인쇄 작업 대기열)              │   │
│      │  📬 Message queue (메시지 큐)                        │   │
│      │  🚗 Cars at toll booth (톨게이트 차량)               │   │
│      │  ⌨️  Keyboard buffer (키보드 버퍼)                   │   │
│      └─────────────────────────────────────────────────────┘   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Stack vs Queue Comparison (스택 vs 큐 비교)

```
┌─────────────────────────────────────────────────────────────────┐
│                  STACK vs QUEUE COMPARISON                      │
│                  (스택 vs 큐 비교)                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  STACK (LIFO)                    QUEUE (FIFO)                   │
│  스택 (후입선출)                  큐 (선입선출)                   │
│                                                                 │
│       │     │                    ┌─────────────────────┐        │
│       │  C  │ ◀─ top            │  A  │  B  │  C  │  D │        │
│       │─────│                    └─────────────────────┘        │
│       │  B  │                      ▲                 ▲          │
│       │─────│                    Front             Rear         │
│       │  A  │                    (앞)              (뒤)         │
│       └─────┘                                                   │
│                                                                 │
│  Insert/Remove                   Insert at Rear                 │
│  at same end                     Remove at Front                │
│  (같은 쪽에서 삽입/제거)          (뒤에서 삽입, 앞에서 제거)       │
│                                                                 │
│  Last In → First Out             First In → First Out           │
│  마지막 입력 → 첫 출력            첫 입력 → 첫 출력               │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. Queue Operations (큐 연산)

### Basic Operations (기본 연산)

```
┌─────────────────────────────────────────────────────────────────┐
│                    QUEUE OPERATIONS (큐 연산)                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Operation     Description               Time      Korean       │
│  (연산)        (설명)                    (시간)    (한국어)      │
│  ─────────────────────────────────────────────────────────────  │
│                                                                 │
│  enqueue(x)   Add element x at rear      O(1)     rear에 요소   │
│                                                   x 추가        │
│                                                                 │
│  dequeue()    Remove element from        O(1)     front 요소    │
│               front and return it                 제거 및 반환  │
│                                                                 │
│  front()/     Return front element       O(1)     front 요소    │
│  peek()       without removing                    반환 (제거X)  │
│                                                                 │
│  isEmpty()    Check if queue is          O(1)     큐가 비었는지 │
│               empty                               확인          │
│                                                                 │
│  isFull()     Check if queue is          O(1)     큐가 가득     │
│               full (array only)                   찼는지 확인   │
│                                                                 │
│  size()       Return number of           O(1)     요소 개수     │
│               elements                            반환          │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Enqueue Operation (Enqueue 연산)

```
┌─────────────────────────────────────────────────────────────────┐
│                 ENQUEUE OPERATION (Enqueue 연산)                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Before enqueue(40):                                            │
│  enqueue(40) 전:                                                │
│                                                                 │
│    front                        rear                            │
│      ▼                           ▼                              │
│    ┌─────┬─────┬─────┬─────┬─────┐                             │
│    │  10 │  20 │  30 │     │     │                             │
│    └─────┴─────┴─────┴─────┴─────┘                             │
│                                                                 │
│  After enqueue(40):                                             │
│  enqueue(40) 후:                                                │
│                                                                 │
│    front                              rear (new)                │
│      ▼                                 ▼                        │
│    ┌─────┬─────┬─────┬─────┬─────┐                             │
│    │  10 │  20 │  30 │  40 │     │                             │
│    └─────┴─────┴─────┴─────┴─────┘                             │
│                         ▲                                       │
│                     NEW ELEMENT                                 │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Dequeue Operation (Dequeue 연산)

```
┌─────────────────────────────────────────────────────────────────┐
│                 DEQUEUE OPERATION (Dequeue 연산)                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Before dequeue():                                              │
│  dequeue() 전:                                                  │
│                                                                 │
│    front                        rear                            │
│      ▼                           ▼                              │
│    ┌─────┬─────┬─────┬─────┬─────┐                             │
│    │  10 │  20 │  30 │  40 │     │                             │
│    └─────┴─────┴─────┴─────┴─────┘                             │
│      ▲                                                          │
│   REMOVED (제거됨)                                               │
│                                                                 │
│  After dequeue():    Returns: 10                                │
│  dequeue() 후:       반환값: 10                                  │
│                                                                 │
│          front (new)            rear                            │
│            ▼                     ▼                              │
│    ┌─────┬─────┬─────┬─────┬─────┐                             │
│    │     │  20 │  30 │  40 │     │                             │
│    └─────┴─────┴─────┴─────┴─────┘                             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 3. Linear Queue and its Problem (선형 큐와 문제점)

### Linear Queue Implementation (선형 큐 구현)

```c
#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>

#define MAX_SIZE 5

// Linear Queue structure (선형 큐 구조)
typedef struct {
    int data[MAX_SIZE];
    int front;  // Index of front element (front 요소의 인덱스)
    int rear;   // Index of rear element (rear 요소의 인덱스)
} LinearQueue;

// Initialize queue (큐 초기화)
void initQueue(LinearQueue* q) {
    q->front = -1;
    q->rear = -1;
}

// Check if queue is empty (큐가 비었는지 확인)
bool isEmpty(LinearQueue* q) {
    return q->front == -1;
}

// Check if queue is full (큐가 가득 찼는지 확인)
bool isFull(LinearQueue* q) {
    return q->rear == MAX_SIZE - 1;
}

// Enqueue - add element at rear (Enqueue - rear에 요소 추가)
bool enqueue(LinearQueue* q, int value) {
    if (isFull(q)) {
        printf("Queue is full! Cannot enqueue %d\n", value);
        return false;
    }

    if (isEmpty(q)) {
        q->front = 0;  // First element (첫 번째 요소)
    }
    q->rear++;
    q->data[q->rear] = value;
    return true;
}

// Dequeue - remove element from front (Dequeue - front에서 요소 제거)
int dequeue(LinearQueue* q) {
    if (isEmpty(q)) {
        printf("Queue is empty! Cannot dequeue\n");
        return -1;
    }

    int value = q->data[q->front];

    if (q->front == q->rear) {
        // Last element being removed (마지막 요소 제거 중)
        q->front = -1;
        q->rear = -1;
    } else {
        q->front++;
    }

    return value;
}

// Get front element without removing (제거 없이 front 요소 반환)
int front(LinearQueue* q) {
    if (isEmpty(q)) {
        printf("Queue is empty!\n");
        return -1;
    }
    return q->data[q->front];
}

// Print queue (큐 출력)
void printQueue(LinearQueue* q) {
    if (isEmpty(q)) {
        printf("Queue is empty\n");
        return;
    }
    printf("Queue: ");
    for (int i = q->front; i <= q->rear; i++) {
        printf("%d ", q->data[i]);
    }
    printf("(front=%d, rear=%d)\n", q->front, q->rear);
}
```

### Linear Queue Problem (선형 큐의 문제점)

```
┌─────────────────────────────────────────────────────────────────┐
│              LINEAR QUEUE PROBLEM (선형 큐의 문제점)             │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Problem: Wasted space after dequeue operations                 │
│  문제점: dequeue 연산 후 공간 낭비                               │
│                                                                 │
│  Initial state (초기 상태):                                      │
│  ┌─────┬─────┬─────┬─────┬─────┐                               │
│  │  A  │  B  │  C  │  D  │  E  │  (Full / 가득 참)              │
│  └─────┴─────┴─────┴─────┴─────┘                               │
│    ▲                       ▲                                    │
│  front                   rear                                   │
│                                                                 │
│  After dequeue() x 3 (dequeue() 3번 후):                        │
│  ┌─────┬─────┬─────┬─────┬─────┐                               │
│  │ XXX │ XXX │ XXX │  D  │  E  │                               │
│  └─────┴─────┴─────┴─────┴─────┘                               │
│    ▲     ▲     ▲     ▲     ▲                                    │
│  Wasted space!      front  rear                                 │
│  낭비된 공간!                                                    │
│                                                                 │
│  Cannot enqueue even though there's space!                      │
│  공간이 있어도 enqueue 불가!                                     │
│                                                                 │
│  Solutions (해결책):                                             │
│  1. Shift all elements (모든 요소 이동) - O(n), inefficient      │
│  2. Circular Queue (원형 큐) - O(1), efficient ✓                │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 4. Circular Queue (원형 큐)

### Circular Queue Concept (원형 큐 개념)

```
┌─────────────────────────────────────────────────────────────────┐
│              CIRCULAR QUEUE CONCEPT (원형 큐 개념)               │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Logical View (논리적 구조):                                     │
│                                                                 │
│              ┌─────┐                                            │
│           ┌──│  4  │──┐                                         │
│           │  └─────┘  │                                         │
│        ┌──┴──┐     ┌──┴──┐                                      │
│        │  3  │     │  0  │ ◀── front                            │
│        └──┬──┘     └──┬──┘                                      │
│           │  ┌─────┐  │                                         │
│           └──│  2  │──┘                                         │
│              └──┬──┘                                            │
│                 │                                               │
│              ┌──┴──┐                                            │
│      rear ──▶│  1  │                                            │
│              └─────┘                                            │
│                                                                 │
│  Physical View (물리적 구조):                                    │
│  ┌─────┬─────┬─────┬─────┬─────┐                               │
│  │  A  │  B  │     │     │     │                               │
│  └─────┴─────┴─────┴─────┴─────┘                               │
│  [0]    [1]   [2]   [3]   [4]                                  │
│   ▲      ▲                                                      │
│ front   rear                                                    │
│                                                                 │
│  Key Formula (핵심 공식):                                        │
│  next_index = (current_index + 1) % MAX_SIZE                    │
│  다음_인덱스 = (현재_인덱스 + 1) % 최대_크기                     │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Circular Queue Implementation (원형 큐 구현)

```c
#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>

#define MAX_SIZE 5

// Circular Queue structure (원형 큐 구조)
typedef struct {
    int data[MAX_SIZE];
    int front;  // Index of front element (front 요소의 인덱스)
    int rear;   // Index of rear element (rear 요소의 인덱스)
    int count;  // Number of elements (요소 개수)
} CircularQueue;

// Initialize circular queue (원형 큐 초기화)
void initCQueue(CircularQueue* q) {
    q->front = 0;
    q->rear = -1;
    q->count = 0;
}

// Check if queue is empty (큐가 비었는지 확인)
bool cqIsEmpty(CircularQueue* q) {
    return q->count == 0;
}

// Check if queue is full (큐가 가득 찼는지 확인)
bool cqIsFull(CircularQueue* q) {
    return q->count == MAX_SIZE;
}

// Enqueue - add element at rear (Enqueue - rear에 요소 추가)
bool cqEnqueue(CircularQueue* q, int value) {
    if (cqIsFull(q)) {
        printf("Circular Queue Overflow! Cannot enqueue %d\n", value);
        return false;
    }

    // Move rear circularly (rear를 원형으로 이동)
    q->rear = (q->rear + 1) % MAX_SIZE;
    q->data[q->rear] = value;
    q->count++;
    return true;
}

// Dequeue - remove element from front (Dequeue - front에서 요소 제거)
int cqDequeue(CircularQueue* q) {
    if (cqIsEmpty(q)) {
        printf("Circular Queue Underflow! Cannot dequeue\n");
        return -1;
    }

    int value = q->data[q->front];
    // Move front circularly (front를 원형으로 이동)
    q->front = (q->front + 1) % MAX_SIZE;
    q->count--;
    return value;
}

// Get front element (front 요소 반환)
int cqFront(CircularQueue* q) {
    if (cqIsEmpty(q)) {
        printf("Queue is empty!\n");
        return -1;
    }
    return q->data[q->front];
}

// Get rear element (rear 요소 반환)
int cqRear(CircularQueue* q) {
    if (cqIsEmpty(q)) {
        printf("Queue is empty!\n");
        return -1;
    }
    return q->data[q->rear];
}

// Get queue size (큐 크기 반환)
int cqSize(CircularQueue* q) {
    return q->count;
}

// Print circular queue (원형 큐 출력)
void printCQueue(CircularQueue* q) {
    if (cqIsEmpty(q)) {
        printf("Queue is empty\n");
        return;
    }

    printf("Circular Queue: ");
    int i = q->front;
    for (int c = 0; c < q->count; c++) {
        printf("%d ", q->data[i]);
        i = (i + 1) % MAX_SIZE;
    }
    printf("(front=%d, rear=%d, count=%d)\n", q->front, q->rear, q->count);
}
```

### Circular Queue Operations Visualization (원형 큐 연산 시각화)

```
┌─────────────────────────────────────────────────────────────────┐
│         CIRCULAR QUEUE OPERATIONS (원형 큐 연산)                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Step 1: Initial (초기)           Step 2: enqueue(A,B,C)        │
│  front=0, rear=-1, count=0        front=0, rear=2, count=3      │
│                                                                 │
│  ┌─────┬─────┬─────┬─────┬─────┐  ┌─────┬─────┬─────┬─────┬─────┐
│  │     │     │     │     │     │  │  A  │  B  │  C  │     │     │
│  └─────┴─────┴─────┴─────┴─────┘  └─────┴─────┴─────┴─────┴─────┘
│   [0]   [1]   [2]   [3]   [4]      [0]   [1]   [2]   [3]   [4]  │
│    ▲                                ▲           ▲               │
│  front                            front       rear              │
│                                                                 │
│  Step 3: dequeue() x 2            Step 4: enqueue(D,E,F)        │
│  Returns A, B                     front=2, rear=0, count=4      │
│  front=2, rear=2, count=1                                       │
│                                                                 │
│  ┌─────┬─────┬─────┬─────┬─────┐  ┌─────┬─────┬─────┬─────┬─────┐
│  │     │     │  C  │     │     │  │  F  │     │  C  │  D  │  E  │
│  └─────┴─────┴─────┴─────┴─────┘  └─────┴─────┴─────┴─────┴─────┘
│   [0]   [1]   [2]   [3]   [4]      [0]   [1]   [2]   [3]   [4]  │
│                ▲                    ▲           ▲               │
│            front=rear             rear        front             │
│                                                                 │
│  Wrap-around occurred! (순환 발생!)                              │
│  rear went from 4 → 0             │
│  rear가 4에서 0으로 이동                                         │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Complete Circular Queue Example (완전한 원형 큐 예제)

```c
#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>

#define MAX_SIZE 5

typedef struct {
    int data[MAX_SIZE];
    int front;
    int rear;
    int count;
} CircularQueue;

void initCQueue(CircularQueue* q) {
    q->front = 0;
    q->rear = -1;
    q->count = 0;
}

bool cqIsEmpty(CircularQueue* q) { return q->count == 0; }
bool cqIsFull(CircularQueue* q) { return q->count == MAX_SIZE; }

bool cqEnqueue(CircularQueue* q, int value) {
    if (cqIsFull(q)) {
        printf("Queue Overflow!\n");
        return false;
    }
    q->rear = (q->rear + 1) % MAX_SIZE;
    q->data[q->rear] = value;
    q->count++;
    return true;
}

int cqDequeue(CircularQueue* q) {
    if (cqIsEmpty(q)) {
        printf("Queue Underflow!\n");
        return -1;
    }
    int value = q->data[q->front];
    q->front = (q->front + 1) % MAX_SIZE;
    q->count--;
    return value;
}

void printCQueue(CircularQueue* q) {
    if (cqIsEmpty(q)) {
        printf("Queue: (empty)\n");
        return;
    }
    printf("Queue: ");
    int i = q->front;
    for (int c = 0; c < q->count; c++) {
        printf("%d ", q->data[i]);
        i = (i + 1) % MAX_SIZE;
    }
    printf("| front=%d, rear=%d, count=%d\n", q->front, q->rear, q->count);
}

int main() {
    CircularQueue q;
    initCQueue(&q);

    printf("=== Circular Queue Demo ===\n");
    printf("=== 원형 큐 데모 ===\n\n");

    // Enqueue elements (요소 추가)
    cqEnqueue(&q, 10);
    cqEnqueue(&q, 20);
    cqEnqueue(&q, 30);
    printCQueue(&q);  // Queue: 10 20 30 | front=0, rear=2, count=3

    // Dequeue some elements (일부 요소 제거)
    printf("Dequeue: %d\n", cqDequeue(&q));  // 10
    printf("Dequeue: %d\n", cqDequeue(&q));  // 20
    printCQueue(&q);  // Queue: 30 | front=2, rear=2, count=1

    // Enqueue more - demonstrates wrap-around (추가 - 순환 시연)
    cqEnqueue(&q, 40);
    cqEnqueue(&q, 50);
    cqEnqueue(&q, 60);  // Wraps to index 0 (인덱스 0으로 순환)
    printCQueue(&q);  // Queue: 30 40 50 60 | front=2, rear=0, count=4

    // Try to enqueue one more (하나 더 추가 시도)
    cqEnqueue(&q, 70);  // Queue: 30 40 50 60 70 (Full)
    printCQueue(&q);

    // This will fail - queue is full (실패 - 큐가 가득 참)
    cqEnqueue(&q, 80);  // Queue Overflow!

    return 0;
}
```

---

## 5. Deque - Double Ended Queue (덱 - 양방향 큐)

### Deque Concept (덱 개념)

```
┌─────────────────────────────────────────────────────────────────┐
│                     DEQUE CONCEPT (덱 개념)                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Deque = Double Ended Queue (양방향 큐)                          │
│  Can insert/delete from both ends (양쪽 끝에서 삽입/삭제 가능)   │
│                                                                 │
│       addFront           addRear                                │
│         │                   │                                   │
│         ▼                   ▼                                   │
│       ┌─────┬─────┬─────┬─────┬─────┐                          │
│       │  A  │  B  │  C  │  D  │  E  │                          │
│       └─────┴─────┴─────┴─────┴─────┘                          │
│         ▲                       ▲                               │
│         │                       │                               │
│     removeFront            removeRear                           │
│                                                                 │
│  Operations (연산):                                              │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  addFront(x)    - Insert at front (앞에 삽입)           │   │
│  │  addRear(x)     - Insert at rear (뒤에 삽입)            │   │
│  │  removeFront()  - Remove from front (앞에서 제거)       │   │
│  │  removeRear()   - Remove from rear (뒤에서 제거)        │   │
│  │  getFront()     - Get front element (앞 요소 반환)      │   │
│  │  getRear()      - Get rear element (뒤 요소 반환)       │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  Special Cases (특별한 경우):                                    │
│  • Input-restricted Deque: Insert only at one end               │
│    (입력 제한 덱: 한쪽에서만 삽입)                               │
│  • Output-restricted Deque: Delete only at one end              │
│    (출력 제한 덱: 한쪽에서만 삭제)                               │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Deque vs Stack vs Queue (덱 vs 스택 vs 큐)

```
┌─────────────────────────────────────────────────────────────────┐
│              DATA STRUCTURE COMPARISON (자료구조 비교)           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Structure     Insert         Delete         Behavior           │
│  구조          삽입           삭제           동작               │
│  ─────────────────────────────────────────────────────────────  │
│                                                                 │
│  Stack         Top only       Top only       LIFO              │
│  스택          top만          top만          후입선출           │
│                                                                 │
│  Queue         Rear only      Front only     FIFO              │
│  큐            rear만         front만        선입선출           │
│                                                                 │
│  Deque         Both ends      Both ends      Flexible          │
│  덱            양쪽 끝        양쪽 끝        유연               │
│                                                                 │
│  Note: Deque can simulate both Stack and Queue!                 │
│  참고: 덱은 스택과 큐를 모두 시뮬레이션할 수 있음!               │
│                                                                 │
│  • Use only rear: behaves like Stack                            │
│    (rear만 사용: 스택처럼 동작)                                  │
│  • Use rear for insert, front for delete: behaves like Queue    │
│    (삽입에 rear, 삭제에 front 사용: 큐처럼 동작)                 │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Deque Implementation (덱 구현)

```c
#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>

#define MAX_SIZE 5

// Deque structure (덱 구조)
typedef struct {
    int data[MAX_SIZE];
    int front;
    int rear;
    int count;
} Deque;

// Initialize deque (덱 초기화)
void initDeque(Deque* dq) {
    dq->front = 0;
    dq->rear = MAX_SIZE - 1;
    dq->count = 0;
}

// Check if deque is empty (덱이 비었는지 확인)
bool dequeIsEmpty(Deque* dq) {
    return dq->count == 0;
}

// Check if deque is full (덱이 가득 찼는지 확인)
bool dequeIsFull(Deque* dq) {
    return dq->count == MAX_SIZE;
}

// Add element at front (앞에 요소 추가)
bool addFront(Deque* dq, int value) {
    if (dequeIsFull(dq)) {
        printf("Deque Overflow!\n");
        return false;
    }

    // Move front backwards circularly
    // front를 원형으로 뒤로 이동
    dq->front = (dq->front - 1 + MAX_SIZE) % MAX_SIZE;
    dq->data[dq->front] = value;
    dq->count++;
    return true;
}

// Add element at rear (뒤에 요소 추가)
bool addRear(Deque* dq, int value) {
    if (dequeIsFull(dq)) {
        printf("Deque Overflow!\n");
        return false;
    }

    // Move rear forward circularly
    // rear를 원형으로 앞으로 이동
    dq->rear = (dq->rear + 1) % MAX_SIZE;
    dq->data[dq->rear] = value;
    dq->count++;
    return true;
}

// Remove element from front (앞에서 요소 제거)
int removeFront(Deque* dq) {
    if (dequeIsEmpty(dq)) {
        printf("Deque Underflow!\n");
        return -1;
    }

    int value = dq->data[dq->front];
    dq->front = (dq->front + 1) % MAX_SIZE;
    dq->count--;
    return value;
}

// Remove element from rear (뒤에서 요소 제거)
int removeRear(Deque* dq) {
    if (dequeIsEmpty(dq)) {
        printf("Deque Underflow!\n");
        return -1;
    }

    int value = dq->data[dq->rear];
    dq->rear = (dq->rear - 1 + MAX_SIZE) % MAX_SIZE;
    dq->count--;
    return value;
}

// Get front element (앞 요소 반환)
int getFront(Deque* dq) {
    if (dequeIsEmpty(dq)) {
        printf("Deque is empty!\n");
        return -1;
    }
    return dq->data[dq->front];
}

// Get rear element (뒤 요소 반환)
int getRear(Deque* dq) {
    if (dequeIsEmpty(dq)) {
        printf("Deque is empty!\n");
        return -1;
    }
    return dq->data[dq->rear];
}

// Print deque (덱 출력)
void printDeque(Deque* dq) {
    if (dequeIsEmpty(dq)) {
        printf("Deque: (empty)\n");
        return;
    }

    printf("Deque (front to rear): ");
    int i = dq->front;
    for (int c = 0; c < dq->count; c++) {
        printf("%d ", dq->data[i]);
        i = (i + 1) % MAX_SIZE;
    }
    printf("\n");
}

int main() {
    Deque dq;
    initDeque(&dq);

    printf("=== Deque Demo ===\n");
    printf("=== 덱 데모 ===\n\n");

    // Add from rear (like queue) (뒤에서 추가 - 큐처럼)
    addRear(&dq, 10);
    addRear(&dq, 20);
    addRear(&dq, 30);
    printDeque(&dq);  // Deque: 10 20 30

    // Add from front (앞에서 추가)
    addFront(&dq, 5);
    printDeque(&dq);  // Deque: 5 10 20 30

    // Remove from rear (like stack) (뒤에서 제거 - 스택처럼)
    printf("Remove rear: %d\n", removeRear(&dq));  // 30
    printDeque(&dq);  // Deque: 5 10 20

    // Remove from front (like queue) (앞에서 제거 - 큐처럼)
    printf("Remove front: %d\n", removeFront(&dq));  // 5
    printDeque(&dq);  // Deque: 10 20

    return 0;
}
```

### Deque Applications (덱 응용)

```
┌─────────────────────────────────────────────────────────────────┐
│                 DEQUE APPLICATIONS (덱 응용)                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. Palindrome Checking (회문 검사)                              │
│     - Compare characters from both ends                         │
│     - 양쪽 끝에서 문자 비교                                      │
│     Example: "racecar" is a palindrome                          │
│                                                                 │
│  2. Sliding Window Maximum (슬라이딩 윈도우 최대값)              │
│     - Efficiently find max in moving window                     │
│     - 이동하는 윈도우에서 효율적으로 최대값 찾기                 │
│                                                                 │
│  3. Undo/Redo Operations (실행 취소/다시 실행)                   │
│     - Undo: remove from rear, add to redo deque                 │
│     - Redo: remove from redo deque, add to rear                 │
│                                                                 │
│  4. A-Steal Job Scheduling (작업 훔치기 스케줄링)                │
│     - Threads take work from front of own deque                 │
│     - Steal from rear of other thread's deque                   │
│     - 스레드는 자신의 덱 앞에서 작업을 가져감                    │
│     - 다른 스레드의 덱 뒤에서 작업 훔치기                        │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 6. Summary (요약)

```
┌─────────────────────────────────────────────────────────────────┐
│                    CHAPTER 08 SUMMARY (요약)                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  QUEUE CONCEPT (큐 개념)                                         │
│  ─────────────────────────────────────────────────────────────  │
│  • FIFO: First In, First Out (선입선출)                         │
│  • Insert at rear, remove from front                            │
│  • rear에서 삽입, front에서 제거                                 │
│  • Like a waiting line (대기줄과 같음)                          │
│                                                                 │
│  OPERATIONS - All O(1) (연산 - 모두 O(1))                       │
│  ─────────────────────────────────────────────────────────────  │
│  • enqueue(x): Add x at rear                                    │
│  • dequeue():  Remove from front                                │
│  • front():    View front without removing                      │
│  • isEmpty():  Check if empty                                   │
│                                                                 │
│  LINEAR QUEUE PROBLEM (선형 큐 문제)                             │
│  ─────────────────────────────────────────────────────────────  │
│  • After dequeue, front space is wasted                         │
│  • dequeue 후 앞 공간이 낭비됨                                  │
│  • Cannot reuse freed space                                     │
│  • 해제된 공간 재사용 불가                                      │
│                                                                 │
│  CIRCULAR QUEUE (원형 큐)                                        │
│  ─────────────────────────────────────────────────────────────  │
│  • Solves linear queue problem (선형 큐 문제 해결)              │
│  • Uses modulo arithmetic (모듈로 연산 사용)                    │
│  • next = (current + 1) % MAX_SIZE                              │
│  • Efficiently reuses space (공간을 효율적으로 재사용)          │
│                                                                 │
│  DEQUE - Double Ended Queue (덱 - 양방향 큐)                    │
│  ─────────────────────────────────────────────────────────────  │
│  • Insert/delete from both ends (양쪽 끝에서 삽입/삭제)         │
│  • Can simulate both stack and queue                            │
│  • 스택과 큐 모두 시뮬레이션 가능                               │
│  • addFront, addRear, removeFront, removeRear                   │
│                                                                 │
│  COMPARISON (비교)                                               │
│  ─────────────────────────────────────────────────────────────  │
│  │ Type   │ Insert     │ Delete     │ Principle │              │
│  │ Stack  │ Top        │ Top        │ LIFO      │              │
│  │ Queue  │ Rear       │ Front      │ FIFO      │              │
│  │ Deque  │ Both ends  │ Both ends  │ Flexible  │              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Quick Reference Table (빠른 참조 테이블)

| Data Structure | Insert | Delete | Key Feature |
|---------------|--------|--------|-------------|
| Stack (스택) | push O(1) | pop O(1) | LIFO |
| Queue (큐) | enqueue O(1) | dequeue O(1) | FIFO |
| Circular Queue (원형 큐) | O(1) | O(1) | No space waste |
| Deque (덱) | addFront/addRear O(1) | removeFront/removeRear O(1) | Both ends |
