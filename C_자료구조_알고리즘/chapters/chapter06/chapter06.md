# Chapter 06. Linked List (연결 리스트)

## Table of Contents (목차)
1. [Linked List Concept (연결 리스트 개념)](#1-linked-list-concept-연결-리스트-개념)
2. [Singly Linked List (단일 연결 리스트)](#2-singly-linked-list-단일-연결-리스트)
3. [Doubly Linked List (이중 연결 리스트)](#3-doubly-linked-list-이중-연결-리스트)
4. [Array vs Linked List Comparison (배열 vs 연결 리스트 비교)](#4-array-vs-linked-list-comparison-배열-vs-연결-리스트-비교)
5. [Summary (요약)](#5-summary-요약)

---

## 1. Linked List Concept (연결 리스트 개념)

### What is a Linked List? (연결 리스트란?)

A **Linked List** is a linear data structure where elements are stored in **nodes**, and each node contains a **data field** and a **pointer (reference)** to the next node in the sequence.

**연결 리스트**는 요소들이 **노드**에 저장되고, 각 노드가 **데이터 필드**와 순서상 다음 노드를 가리키는 **포인터(참조)**를 포함하는 선형 자료 구조입니다.

```
┌─────────────────────────────────────────────────────────────────┐
│                    LINKED LIST STRUCTURE                        │
│                    (연결 리스트 구조)                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   head                                                          │
│    │                                                            │
│    ▼                                                            │
│  ┌──────┬──────┐    ┌──────┬──────┐    ┌──────┬──────┐         │
│  │ Data │ Next │───▶│ Data │ Next │───▶│ Data │ Next │───▶ NULL│
│  │  10  │   ●──┼    │  20  │   ●──┼    │  30  │   ●──┼         │
│  └──────┴──────┘    └──────┴──────┘    └──────┴──────┘         │
│     Node 1             Node 2             Node 3               │
│                                                                 │
│  Each node contains:                                            │
│  각 노드는 다음을 포함:                                          │
│  - Data field (데이터 필드)                                      │
│  - Pointer to next node (다음 노드를 가리키는 포인터)             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Key Characteristics (주요 특징)

| Feature (특징) | Description (설명) |
|----------------|-------------------|
| **Dynamic Size (동적 크기)** | Can grow or shrink during runtime (실행 중 크기 변경 가능) |
| **Non-contiguous Memory (비연속 메모리)** | Nodes can be anywhere in memory (노드가 메모리 어디에나 존재 가능) |
| **No Random Access (임의 접근 불가)** | Must traverse from head (처음부터 순회해야 함) |
| **Efficient Insertion/Deletion (효율적인 삽입/삭제)** | O(1) if position is known (위치를 알면 O(1)) |

---

## 2. Singly Linked List (단일 연결 리스트)

### Node Structure Definition (노드 구조 정의)

```c
#include <stdio.h>
#include <stdlib.h>

// Node structure definition (노드 구조체 정의)
typedef struct Node {
    int data;           // Data field (데이터 필드)
    struct Node* next;  // Pointer to next node (다음 노드 포인터)
} Node;

// Create a new node (새 노드 생성)
Node* createNode(int data) {
    Node* newNode = (Node*)malloc(sizeof(Node));
    if (newNode == NULL) {
        printf("Memory allocation failed!\n");
        exit(1);
    }
    newNode->data = data;
    newNode->next = NULL;
    return newNode;
}
```

### Basic Operations (기본 연산)

#### Insert at Head (앞에 삽입)

```
┌─────────────────────────────────────────────────────────────────┐
│                   INSERT AT HEAD (앞에 삽입)                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Before (삽입 전):                                               │
│                                                                 │
│   head                                                          │
│    │                                                            │
│    ▼                                                            │
│  ┌──────┬──────┐    ┌──────┬──────┐                             │
│  │  20  │   ●──┼───▶│  30  │ NULL │                             │
│  └──────┴──────┘    └──────┴──────┘                             │
│                                                                 │
│  Insert 10 at head:                                             │
│                                                                 │
│  After (삽입 후):                                                │
│                                                                 │
│   head                                                          │
│    │                                                            │
│    ▼                                                            │
│  ┌──────┬──────┐    ┌──────┬──────┐    ┌──────┬──────┐         │
│  │  10  │   ●──┼───▶│  20  │   ●──┼───▶│  30  │ NULL │         │
│  └──────┴──────┘    └──────┴──────┘    └──────┴──────┘         │
│     NEW NODE                                                    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

```c
// Insert at head - O(1) (앞에 삽입 - O(1))
Node* insertAtHead(Node* head, int data) {
    Node* newNode = createNode(data);
    newNode->next = head;  // New node points to old head
                           // 새 노드가 기존 head를 가리킴
    return newNode;        // New node becomes head
                           // 새 노드가 head가 됨
}
```

#### Insert at Tail (끝에 삽입)

```
┌─────────────────────────────────────────────────────────────────┐
│                   INSERT AT TAIL (끝에 삽입)                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Before (삽입 전):                                               │
│                                                                 │
│   head                                                          │
│    │                                                            │
│    ▼                                                            │
│  ┌──────┬──────┐    ┌──────┬──────┐                             │
│  │  10  │   ●──┼───▶│  20  │ NULL │                             │
│  └──────┴──────┘    └──────┴──────┘                             │
│                           ▲                                     │
│                           │                                     │
│                         tail                                    │
│                                                                 │
│  Insert 30 at tail:                                             │
│                                                                 │
│  After (삽입 후):                                                │
│                                                                 │
│   head                                                          │
│    │                                                            │
│    ▼                                                            │
│  ┌──────┬──────┐    ┌──────┬──────┐    ┌──────┬──────┐         │
│  │  10  │   ●──┼───▶│  20  │   ●──┼───▶│  30  │ NULL │         │
│  └──────┴──────┘    └──────┴──────┘    └──────┴──────┘         │
│                                              ▲                  │
│                                              │                  │
│                                          NEW NODE               │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

```c
// Insert at tail - O(n) (끝에 삽입 - O(n))
Node* insertAtTail(Node* head, int data) {
    Node* newNode = createNode(data);

    // If list is empty (리스트가 비어있으면)
    if (head == NULL) {
        return newNode;
    }

    // Traverse to the last node (마지막 노드까지 순회)
    Node* current = head;
    while (current->next != NULL) {
        current = current->next;
    }

    // Link new node at the end (끝에 새 노드 연결)
    current->next = newNode;
    return head;
}
```

#### Delete Node (노드 삭제)

```
┌─────────────────────────────────────────────────────────────────┐
│                   DELETE NODE (노드 삭제)                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Before deleting 20 (20 삭제 전):                                │
│                                                                 │
│   head                                                          │
│    │                                                            │
│    ▼                                                            │
│  ┌──────┬──────┐    ┌──────┬──────┐    ┌──────┬──────┐         │
│  │  10  │   ●──┼───▶│  20  │   ●──┼───▶│  30  │ NULL │         │
│  └──────┴──────┘    └──────┴──────┘    └──────┴──────┘         │
│                        DELETE                                   │
│                                                                 │
│  Step 1: Find node before target (대상 이전 노드 찾기)           │
│  Step 2: Redirect pointer (포인터 재지정)                        │
│  Step 3: Free deleted node (삭제된 노드 메모리 해제)             │
│                                                                 │
│  After (삭제 후):                                                │
│                                                                 │
│   head                                                          │
│    │                                                            │
│    ▼                                                            │
│  ┌──────┬──────┐    ┌──────┬──────┐                             │
│  │  10  │   ●──┼───▶│  30  │ NULL │                             │
│  └──────┴──────┘    └──────┴──────┘                             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

```c
// Delete node with specific value - O(n) (특정 값 노드 삭제 - O(n))
Node* deleteNode(Node* head, int value) {
    // If list is empty (리스트가 비어있으면)
    if (head == NULL) {
        return NULL;
    }

    // If head needs to be deleted (head를 삭제해야 하면)
    if (head->data == value) {
        Node* temp = head;
        head = head->next;
        free(temp);
        return head;
    }

    // Find the node before the target (대상 이전 노드 찾기)
    Node* current = head;
    while (current->next != NULL && current->next->data != value) {
        current = current->next;
    }

    // If found, delete it (찾으면 삭제)
    if (current->next != NULL) {
        Node* temp = current->next;
        current->next = current->next->next;
        free(temp);
    }

    return head;
}
```

#### Search and Traversal (검색 및 순회)

```c
// Search for a value - O(n) (값 검색 - O(n))
Node* search(Node* head, int value) {
    Node* current = head;
    while (current != NULL) {
        if (current->data == value) {
            return current;  // Found (찾음)
        }
        current = current->next;
    }
    return NULL;  // Not found (찾지 못함)
}

// Print all nodes (모든 노드 출력)
void printList(Node* head) {
    Node* current = head;
    printf("List: ");
    while (current != NULL) {
        printf("%d -> ", current->data);
        current = current->next;
    }
    printf("NULL\n");
}

// Get list length - O(n) (리스트 길이 - O(n))
int getLength(Node* head) {
    int count = 0;
    Node* current = head;
    while (current != NULL) {
        count++;
        current = current->next;
    }
    return count;
}
```

### Complete Singly Linked List Example (완전한 단일 연결 리스트 예제)

```c
#include <stdio.h>
#include <stdlib.h>

typedef struct Node {
    int data;
    struct Node* next;
} Node;

Node* createNode(int data) {
    Node* newNode = (Node*)malloc(sizeof(Node));
    if (!newNode) {
        printf("Memory allocation failed!\n");
        exit(1);
    }
    newNode->data = data;
    newNode->next = NULL;
    return newNode;
}

Node* insertAtHead(Node* head, int data) {
    Node* newNode = createNode(data);
    newNode->next = head;
    return newNode;
}

Node* insertAtTail(Node* head, int data) {
    Node* newNode = createNode(data);
    if (!head) return newNode;

    Node* current = head;
    while (current->next) {
        current = current->next;
    }
    current->next = newNode;
    return head;
}

Node* deleteNode(Node* head, int value) {
    if (!head) return NULL;

    if (head->data == value) {
        Node* temp = head;
        head = head->next;
        free(temp);
        return head;
    }

    Node* current = head;
    while (current->next && current->next->data != value) {
        current = current->next;
    }

    if (current->next) {
        Node* temp = current->next;
        current->next = current->next->next;
        free(temp);
    }

    return head;
}

void printList(Node* head) {
    printf("List: ");
    while (head) {
        printf("%d -> ", head->data);
        head = head->next;
    }
    printf("NULL\n");
}

void freeList(Node* head) {
    Node* temp;
    while (head) {
        temp = head;
        head = head->next;
        free(temp);
    }
}

int main() {
    Node* head = NULL;

    // Insert nodes (노드 삽입)
    head = insertAtHead(head, 30);
    head = insertAtHead(head, 20);
    head = insertAtHead(head, 10);
    printList(head);  // List: 10 -> 20 -> 30 -> NULL

    head = insertAtTail(head, 40);
    printList(head);  // List: 10 -> 20 -> 30 -> 40 -> NULL

    // Delete node (노드 삭제)
    head = deleteNode(head, 20);
    printList(head);  // List: 10 -> 30 -> 40 -> NULL

    // Free memory (메모리 해제)
    freeList(head);

    return 0;
}
```

---

## 3. Doubly Linked List (이중 연결 리스트)

### Structure (구조)

A **Doubly Linked List** has nodes with pointers to both the **next** and **previous** nodes.

**이중 연결 리스트**는 **다음** 노드와 **이전** 노드를 모두 가리키는 포인터를 가진 노드들로 구성됩니다.

```
┌─────────────────────────────────────────────────────────────────┐
│               DOUBLY LINKED LIST STRUCTURE                      │
│               (이중 연결 리스트 구조)                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   head                                                    tail  │
│    │                                                       │    │
│    ▼                                                       ▼    │
│  ┌──────┬──────┬──────┐  ┌──────┬──────┬──────┐  ┌──────┬──────┬──────┐
│  │ NULL │  10  │   ●──┼──│◀──●  │  20  │   ●──┼──│◀──●  │  30  │ NULL │
│  │ prev │ data │ next │  │ prev │ data │ next │  │ prev │ data │ next │
│  └──────┴──────┴──────┘  └──────┴──────┴──────┘  └──────┴──────┴──────┘
│         Node 1                  Node 2                  Node 3  │
│                                                                 │
│  Each node contains:                                            │
│  각 노드는 다음을 포함:                                          │
│  - prev: pointer to previous node (이전 노드 포인터)             │
│  - data: stored value (저장된 값)                                │
│  - next: pointer to next node (다음 노드 포인터)                 │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Node Structure Definition (노드 구조 정의)

```c
typedef struct DNode {
    int data;
    struct DNode* prev;  // Pointer to previous node (이전 노드 포인터)
    struct DNode* next;  // Pointer to next node (다음 노드 포인터)
} DNode;

// Create a new doubly linked list node (새 이중 연결 리스트 노드 생성)
DNode* createDNode(int data) {
    DNode* newNode = (DNode*)malloc(sizeof(DNode));
    if (!newNode) {
        printf("Memory allocation failed!\n");
        exit(1);
    }
    newNode->data = data;
    newNode->prev = NULL;
    newNode->next = NULL;
    return newNode;
}
```

### Doubly Linked List Operations (이중 연결 리스트 연산)

```
┌─────────────────────────────────────────────────────────────────┐
│           DOUBLY LINKED LIST INSERT (이중 연결 리스트 삽입)      │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Insert 15 between 10 and 20:                                   │
│  10과 20 사이에 15 삽입:                                         │
│                                                                 │
│  Before (삽입 전):                                               │
│                                                                 │
│  ┌──────┬──────┬──────┐         ┌──────┬──────┬──────┐         │
│  │ NULL │  10  │   ●──┼────────▶│◀──●  │  20  │ NULL │         │
│  └──────┴──────┴──────┘         └──────┴──────┴──────┘         │
│                                                                 │
│  After (삽입 후):                                                │
│                                                                 │
│  ┌──────┬──────┬──────┐  ┌──────┬──────┬──────┐  ┌──────┬──────┬──────┐
│  │ NULL │  10  │   ●──┼──│◀──●  │  15  │   ●──┼──│◀──●  │  20  │ NULL │
│  └──────┴──────┴──────┘  └──────┴──────┴──────┘  └──────┴──────┴──────┘
│                              NEW NODE                           │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

```c
// Insert at head for doubly linked list - O(1)
// 이중 연결 리스트 앞에 삽입 - O(1)
DNode* dInsertAtHead(DNode* head, int data) {
    DNode* newNode = createDNode(data);

    if (head != NULL) {
        newNode->next = head;
        head->prev = newNode;
    }

    return newNode;  // New node is the new head
}

// Insert at tail for doubly linked list - O(n)
// 이중 연결 리스트 끝에 삽입 - O(n)
DNode* dInsertAtTail(DNode* head, int data) {
    DNode* newNode = createDNode(data);

    if (head == NULL) {
        return newNode;
    }

    DNode* current = head;
    while (current->next != NULL) {
        current = current->next;
    }

    current->next = newNode;
    newNode->prev = current;

    return head;
}

// Insert after a specific node - O(1)
// 특정 노드 뒤에 삽입 - O(1)
void dInsertAfter(DNode* prevNode, int data) {
    if (prevNode == NULL) {
        printf("Previous node cannot be NULL\n");
        return;
    }

    DNode* newNode = createDNode(data);

    newNode->next = prevNode->next;
    newNode->prev = prevNode;
    prevNode->next = newNode;

    if (newNode->next != NULL) {
        newNode->next->prev = newNode;
    }
}

// Delete a node from doubly linked list - O(1) if node is given
// 이중 연결 리스트에서 노드 삭제 - 노드가 주어지면 O(1)
DNode* dDeleteNode(DNode* head, DNode* delNode) {
    if (head == NULL || delNode == NULL) {
        return head;
    }

    // If node to be deleted is head (삭제할 노드가 head인 경우)
    if (head == delNode) {
        head = delNode->next;
    }

    // Update next node's prev (다음 노드의 prev 업데이트)
    if (delNode->next != NULL) {
        delNode->next->prev = delNode->prev;
    }

    // Update previous node's next (이전 노드의 next 업데이트)
    if (delNode->prev != NULL) {
        delNode->prev->next = delNode->next;
    }

    free(delNode);
    return head;
}

// Print doubly linked list forward (순방향 출력)
void printForward(DNode* head) {
    printf("Forward: ");
    while (head != NULL) {
        printf("%d <-> ", head->data);
        head = head->next;
    }
    printf("NULL\n");
}

// Print doubly linked list backward (역방향 출력)
void printBackward(DNode* tail) {
    printf("Backward: ");
    while (tail != NULL) {
        printf("%d <-> ", tail->data);
        tail = tail->prev;
    }
    printf("NULL\n");
}
```

### Advantages of Doubly Linked List (이중 연결 리스트의 장점)

```
┌─────────────────────────────────────────────────────────────────┐
│       DOUBLY LINKED LIST ADVANTAGES (이중 연결 리스트 장점)      │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. Bidirectional Traversal (양방향 순회)                        │
│     - Can traverse forward and backward                         │
│     - 앞으로, 뒤로 순회 가능                                     │
│                                                                 │
│  2. Easier Deletion (더 쉬운 삭제)                               │
│     - Can delete node without traversing from head              │
│     - head부터 순회하지 않고 노드 삭제 가능                       │
│                                                                 │
│  3. Quick Insert Before (빠른 앞 삽입)                           │
│     - Can insert before a given node in O(1)                    │
│     - 주어진 노드 앞에 O(1)에 삽입 가능                          │
│                                                                 │
│  Disadvantages (단점):                                           │
│  - Extra memory for prev pointer (prev 포인터에 추가 메모리)     │
│  - More complex implementation (더 복잡한 구현)                  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 4. Array vs Linked List Comparison (배열 vs 연결 리스트 비교)

```
┌─────────────────────────────────────────────────────────────────┐
│             ARRAY vs LINKED LIST COMPARISON                     │
│             (배열 vs 연결 리스트 비교)                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ARRAY (배열):                                                   │
│  ┌─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┐             │
│  │  10 │  20 │  30 │  40 │  50 │     │     │     │             │
│  └─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┘             │
│  [0]    [1]   [2]   [3]   [4]   [5]   [6]   [7]                │
│                                                                 │
│  - Contiguous memory (연속된 메모리)                             │
│  - Fixed size (고정 크기)                                        │
│  - Direct access by index (인덱스로 직접 접근)                   │
│                                                                 │
│  ─────────────────────────────────────────────────────────────  │
│                                                                 │
│  LINKED LIST (연결 리스트):                                      │
│                                                                 │
│  ┌──────┬──────┐    ┌──────┬──────┐    ┌──────┬──────┐         │
│  │  10  │   ●──┼───▶│  20  │   ●──┼───▶│  30  │ NULL │         │
│  └──────┴──────┘    └──────┴──────┘    └──────┴──────┘         │
│  (addr: 100)        (addr: 300)        (addr: 500)              │
│                                                                 │
│  - Non-contiguous memory (비연속 메모리)                         │
│  - Dynamic size (동적 크기)                                      │
│  - Sequential access (순차 접근)                                 │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Time Complexity Comparison (시간 복잡도 비교)

| Operation (연산) | Array (배열) | Linked List (연결 리스트) |
|-----------------|--------------|--------------------------|
| **Access by Index (인덱스 접근)** | O(1) | O(n) |
| **Search (검색)** | O(n) | O(n) |
| **Insert at Beginning (맨 앞 삽입)** | O(n) | O(1) |
| **Insert at End (맨 끝 삽입)** | O(1)* | O(n)** |
| **Insert at Middle (중간 삽입)** | O(n) | O(n)*** |
| **Delete at Beginning (맨 앞 삭제)** | O(n) | O(1) |
| **Delete at End (맨 끝 삭제)** | O(1) | O(n) |
| **Delete at Middle (중간 삭제)** | O(n) | O(n)*** |

*Amortized O(1) with dynamic array / 동적 배열에서 분할 상환 O(1)
**O(1) with tail pointer / tail 포인터가 있으면 O(1)
***O(1) if position is already known / 위치를 이미 알면 O(1)

### Space Complexity Comparison (공간 복잡도 비교)

```
┌─────────────────────────────────────────────────────────────────┐
│                 SPACE COMPARISON (공간 비교)                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ARRAY (for n integers on 64-bit system):                       │
│  배열 (64비트 시스템에서 n개의 정수):                            │
│                                                                 │
│  Space = n × sizeof(int) = n × 4 bytes                          │
│  공간 = n × sizeof(int) = n × 4 바이트                          │
│                                                                 │
│  ─────────────────────────────────────────────────────────────  │
│                                                                 │
│  SINGLY LINKED LIST (단일 연결 리스트):                          │
│                                                                 │
│  Space = n × (sizeof(int) + sizeof(pointer))                    │
│        = n × (4 + 8) = n × 12 bytes                             │
│  공간 = n × (4 + 8) = n × 12 바이트                             │
│                                                                 │
│  ─────────────────────────────────────────────────────────────  │
│                                                                 │
│  DOUBLY LINKED LIST (이중 연결 리스트):                          │
│                                                                 │
│  Space = n × (sizeof(int) + 2 × sizeof(pointer))                │
│        = n × (4 + 16) = n × 20 bytes                            │
│  공간 = n × (4 + 16) = n × 20 바이트                            │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### When to Use Which? (언제 무엇을 사용할까?)

```
┌─────────────────────────────────────────────────────────────────┐
│                    USE CASE GUIDE (사용 가이드)                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  USE ARRAY WHEN (배열 사용 시기):                                │
│  ✓ Need random access (임의 접근이 필요할 때)                    │
│  ✓ Size is known in advance (크기를 미리 알 때)                 │
│  ✓ Memory efficiency is important (메모리 효율이 중요할 때)      │
│  ✓ Cache locality matters (캐시 지역성이 중요할 때)              │
│                                                                 │
│  USE LINKED LIST WHEN (연결 리스트 사용 시기):                   │
│  ✓ Frequent insertions/deletions (빈번한 삽입/삭제가 있을 때)    │
│  ✓ Size varies dynamically (크기가 동적으로 변할 때)             │
│  ✓ No need for random access (임의 접근이 필요 없을 때)          │
│  ✓ Memory fragmentation is acceptable (메모리 단편화 허용 시)    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 5. Summary (요약)

```
┌─────────────────────────────────────────────────────────────────┐
│                    CHAPTER 06 SUMMARY (요약)                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  LINKED LIST (연결 리스트)                                       │
│  ─────────────────────────────────────────────────────────────  │
│  • Linear data structure with nodes containing data and         │
│    pointers to next/previous nodes                              │
│  • 데이터와 다음/이전 노드에 대한 포인터를 포함하는 선형 자료 구조   │
│                                                                 │
│  SINGLY LINKED LIST (단일 연결 리스트)                           │
│  ─────────────────────────────────────────────────────────────  │
│  • Each node has data + next pointer                            │
│  • 각 노드는 데이터 + next 포인터를 가짐                         │
│  • Insert/Delete at head: O(1)                                  │
│  • Insert/Delete at tail: O(n)                                  │
│  • Search: O(n)                                                 │
│                                                                 │
│  DOUBLY LINKED LIST (이중 연결 리스트)                           │
│  ─────────────────────────────────────────────────────────────  │
│  • Each node has data + prev pointer + next pointer             │
│  • 각 노드는 데이터 + prev 포인터 + next 포인터를 가짐            │
│  • Bidirectional traversal possible                             │
│  • 양방향 순회 가능                                              │
│  • Delete given node: O(1)                                      │
│                                                                 │
│  KEY POINTS (핵심 포인트)                                        │
│  ─────────────────────────────────────────────────────────────  │
│  • Dynamic size - grows/shrinks at runtime                      │
│  • 동적 크기 - 런타임에 증가/감소                                │
│  • No random access - must traverse from head                   │
│  • 임의 접근 불가 - head부터 순회해야 함                         │
│  • Efficient insertion/deletion if position is known            │
│  • 위치를 알면 효율적인 삽입/삭제                                │
│  • Uses more memory than array (for pointers)                   │
│  • 배열보다 더 많은 메모리 사용 (포인터 때문)                     │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Key Takeaways (핵심 요점)

1. **Linked List** is a dynamic data structure using nodes connected by pointers
   - **연결 리스트**는 포인터로 연결된 노드를 사용하는 동적 자료 구조

2. **Singly Linked List** is simpler but only allows forward traversal
   - **단일 연결 리스트**는 더 간단하지만 순방향 순회만 가능

3. **Doubly Linked List** allows bidirectional traversal at the cost of extra memory
   - **이중 연결 리스트**는 추가 메모리 비용으로 양방향 순회 가능

4. Choose **Array** for random access, **Linked List** for frequent insertions/deletions
   - 임의 접근에는 **배열**, 빈번한 삽입/삭제에는 **연결 리스트** 선택
