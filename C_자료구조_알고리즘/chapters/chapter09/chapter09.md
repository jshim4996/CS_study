# Chapter 09. Sorting - Basic (정렬 기본)

## Table of Contents (목차)
1. [Sorting Overview (정렬 개요)](#1-sorting-overview-정렬-개요)
2. [Bubble Sort O(n²) (버블 정렬)](#2-bubble-sort-on²-버블-정렬)
3. [Selection Sort O(n²) (선택 정렬)](#3-selection-sort-on²-선택-정렬)
4. [Insertion Sort O(n²) (삽입 정렬)](#4-insertion-sort-on²-삽입-정렬)
5. [Stable vs Unstable Sort (안정 정렬 vs 불안정 정렬)](#5-stable-vs-unstable-sort-안정-정렬-vs-불안정-정렬)
6. [Summary (요약)](#6-summary-요약)

---

## 1. Sorting Overview (정렬 개요)

### What is Sorting? (정렬이란?)

**Sorting** is the process of arranging elements in a specific order (ascending or descending) based on some criteria.

**정렬**은 특정 기준에 따라 요소들을 특정 순서(오름차순 또는 내림차순)로 배열하는 과정입니다.

```
┌─────────────────────────────────────────────────────────────────┐
│                   SORTING CONCEPT (정렬 개념)                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Unsorted Array (정렬되지 않은 배열):                            │
│  ┌─────┬─────┬─────┬─────┬─────┬─────┐                         │
│  │  64 │  34 │  25 │  12 │  22 │  11 │                         │
│  └─────┴─────┴─────┴─────┴─────┴─────┘                         │
│                                                                 │
│                      ↓ Sorting (정렬)                           │
│                                                                 │
│  Sorted Array (Ascending) (정렬된 배열 - 오름차순):              │
│  ┌─────┬─────┬─────┬─────┬─────┬─────┐                         │
│  │  11 │  12 │  22 │  25 │  34 │  64 │                         │
│  └─────┴─────┴─────┴─────┴─────┴─────┘                         │
│                                                                 │
│  Why Sorting is Important (정렬이 중요한 이유):                  │
│  ─────────────────────────────────────────────────────────────  │
│  • Binary search requires sorted data (이진 검색은 정렬된 데이터 필요)
│  • Database operations (데이터베이스 연산)                       │
│  • Finding duplicates (중복 찾기)                                │
│  • Data visualization (데이터 시각화)                            │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Classification of Sorting Algorithms (정렬 알고리즘 분류)

```
┌─────────────────────────────────────────────────────────────────┐
│            SORTING ALGORITHM CLASSIFICATION                     │
│            (정렬 알고리즘 분류)                                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  By Time Complexity (시간 복잡도별):                             │
│  ┌───────────────┬──────────────────────────────────────┐      │
│  │ O(n²)         │ Bubble, Selection, Insertion         │      │
│  │               │ Simple but slow for large data       │      │
│  │               │ 단순하지만 큰 데이터에 느림           │      │
│  ├───────────────┼──────────────────────────────────────┤      │
│  │ O(n log n)    │ Quick, Merge, Heap                   │      │
│  │               │ Efficient for large data             │      │
│  │               │ 큰 데이터에 효율적                    │      │
│  ├───────────────┼──────────────────────────────────────┤      │
│  │ O(n)          │ Counting, Radix, Bucket              │      │
│  │               │ Special cases only                   │      │
│  │               │ 특별한 경우에만                       │      │
│  └───────────────┴──────────────────────────────────────┘      │
│                                                                 │
│  By Space Complexity (공간 복잡도별):                            │
│  ┌───────────────┬──────────────────────────────────────┐      │
│  │ In-place O(1) │ Bubble, Selection, Insertion, Quick  │      │
│  │               │ 제자리 정렬 - 추가 공간 최소          │      │
│  ├───────────────┼──────────────────────────────────────┤      │
│  │ Not in-place  │ Merge, Counting, Radix               │      │
│  │ O(n)          │ 추가 공간 필요                        │      │
│  └───────────────┴──────────────────────────────────────┘      │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. Bubble Sort O(n²) (버블 정렬)

### Concept (개념)

**Bubble Sort** repeatedly compares adjacent elements and swaps them if they are in the wrong order. Larger elements "bubble up" to the end of the array.

**버블 정렬**은 인접한 요소들을 반복적으로 비교하고 잘못된 순서이면 교환합니다. 큰 요소들이 배열 끝으로 "떠오릅니다".

```
┌─────────────────────────────────────────────────────────────────┐
│               BUBBLE SORT CONCEPT (버블 정렬 개념)               │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Algorithm (알고리즘):                                           │
│  1. Compare adjacent elements (인접 요소 비교)                   │
│  2. Swap if first > second (첫 번째 > 두 번째면 교환)           │
│  3. Repeat for all pairs (모든 쌍에 대해 반복)                  │
│  4. Largest element bubbles to end (가장 큰 요소가 끝으로 이동) │
│  5. Repeat n-1 times (n-1번 반복)                               │
│                                                                 │
│  One Pass Example (한 번 순회 예시):                             │
│                                                                 │
│  [64] [34] [25] [12]      64 > 34? Yes, swap                    │
│    ↓   ↓                                                        │
│  [34] [64] [25] [12]      64 > 25? Yes, swap                    │
│         ↓   ↓                                                   │
│  [34] [25] [64] [12]      64 > 12? Yes, swap                    │
│              ↓   ↓                                              │
│  [34] [25] [12] [64]      64 is now in correct position!        │
│                    ▲      64가 이제 올바른 위치에!               │
│                 Sorted                                          │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Complete Pass Visualization (전체 순회 시각화)

```
┌─────────────────────────────────────────────────────────────────┐
│              BUBBLE SORT VISUALIZATION (버블 정렬 시각화)        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Initial: [64, 34, 25, 12, 22, 11]                              │
│                                                                 │
│  Pass 1 (순회 1):                                                │
│  [64, 34, 25, 12, 22, 11] → Compare 64, 34 → Swap              │
│  [34, 64, 25, 12, 22, 11] → Compare 64, 25 → Swap              │
│  [34, 25, 64, 12, 22, 11] → Compare 64, 12 → Swap              │
│  [34, 25, 12, 64, 22, 11] → Compare 64, 22 → Swap              │
│  [34, 25, 12, 22, 64, 11] → Compare 64, 11 → Swap              │
│  [34, 25, 12, 22, 11, 64] ← 64 in place (64 위치 확정)          │
│                        ▲                                        │
│                     Sorted                                      │
│                                                                 │
│  Pass 2 (순회 2):                                                │
│  [34, 25, 12, 22, 11, 64] → ... →                              │
│  [25, 12, 22, 11, 34, 64] ← 34 in place                        │
│                    ▲   ▲                                        │
│                   Sorted                                        │
│                                                                 │
│  Pass 3 (순회 3):                                                │
│  [25, 12, 22, 11, 34, 64] → ... →                              │
│  [12, 22, 11, 25, 34, 64] ← 25 in place                        │
│                                                                 │
│  Pass 4 (순회 4):                                                │
│  [12, 22, 11, 25, 34, 64] → ... →                              │
│  [12, 11, 22, 25, 34, 64] ← 22 in place                        │
│                                                                 │
│  Pass 5 (순회 5):                                                │
│  [12, 11, 22, 25, 34, 64] → Compare 12, 11 → Swap              │
│  [11, 12, 22, 25, 34, 64] ← SORTED! (정렬 완료!)                │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Implementation (구현)

```c
#include <stdio.h>

// Swap two elements (두 요소 교환)
void swap(int* a, int* b) {
    int temp = *a;
    *a = *b;
    *b = temp;
}

// Basic Bubble Sort (기본 버블 정렬)
// Time: O(n²), Space: O(1)
void bubbleSort(int arr[], int n) {
    // Outer loop: number of passes (외부 루프: 순회 횟수)
    for (int i = 0; i < n - 1; i++) {
        // Inner loop: compare adjacent elements
        // 내부 루프: 인접 요소 비교
        // Note: n-1-i because last i elements are already sorted
        // 참고: 마지막 i개 요소는 이미 정렬됨
        for (int j = 0; j < n - 1 - i; j++) {
            if (arr[j] > arr[j + 1]) {
                swap(&arr[j], &arr[j + 1]);
            }
        }
    }
}

// Optimized Bubble Sort with early termination
// 조기 종료가 있는 최적화된 버블 정렬
void bubbleSortOptimized(int arr[], int n) {
    for (int i = 0; i < n - 1; i++) {
        int swapped = 0;  // Flag to detect if swap occurred
                          // 교환 발생 여부 플래그

        for (int j = 0; j < n - 1 - i; j++) {
            if (arr[j] > arr[j + 1]) {
                swap(&arr[j], &arr[j + 1]);
                swapped = 1;
            }
        }

        // If no swaps in this pass, array is sorted
        // 이 순회에서 교환이 없으면 배열이 정렬됨
        if (swapped == 0) {
            break;
        }
    }
}

// Print array (배열 출력)
void printArray(int arr[], int n) {
    for (int i = 0; i < n; i++) {
        printf("%d ", arr[i]);
    }
    printf("\n");
}

int main() {
    int arr[] = {64, 34, 25, 12, 22, 11};
    int n = sizeof(arr) / sizeof(arr[0]);

    printf("Original array: ");
    printArray(arr, n);

    bubbleSortOptimized(arr, n);

    printf("Sorted array: ");
    printArray(arr, n);

    return 0;
}
```

### Bubble Sort Analysis (버블 정렬 분석)

```
┌─────────────────────────────────────────────────────────────────┐
│              BUBBLE SORT ANALYSIS (버블 정렬 분석)               │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Time Complexity (시간 복잡도):                                  │
│  ─────────────────────────────────────────────────────────────  │
│  • Worst Case (최악): O(n²) - Reverse sorted array              │
│                        역순으로 정렬된 배열                      │
│  • Average Case (평균): O(n²)                                   │
│  • Best Case (최선): O(n) - Already sorted (with optimization)  │
│                      이미 정렬됨 (최적화 시)                     │
│                                                                 │
│  Comparisons (비교 횟수):                                        │
│  (n-1) + (n-2) + ... + 1 = n(n-1)/2 ≈ O(n²)                    │
│                                                                 │
│  Space Complexity (공간 복잡도): O(1)                            │
│  In-place sorting (제자리 정렬)                                  │
│                                                                 │
│  Characteristics (특징):                                         │
│  ✓ Stable sort (안정 정렬)                                      │
│  ✓ In-place (제자리)                                            │
│  ✓ Simple to implement (구현 간단)                              │
│  ✗ Very slow for large datasets (큰 데이터에 매우 느림)         │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 3. Selection Sort O(n²) (선택 정렬)

### Concept (개념)

**Selection Sort** finds the minimum element in the unsorted portion and swaps it with the first unsorted element.

**선택 정렬**은 정렬되지 않은 부분에서 최소 요소를 찾아 정렬되지 않은 첫 번째 요소와 교환합니다.

```
┌─────────────────────────────────────────────────────────────────┐
│            SELECTION SORT CONCEPT (선택 정렬 개념)               │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Algorithm (알고리즘):                                           │
│  1. Find minimum in unsorted portion (정렬되지 않은 부분에서 최소값 찾기)
│  2. Swap with first unsorted element (첫 번째 정렬되지 않은 요소와 교환)
│  3. Move boundary of sorted portion (정렬된 부분의 경계 이동)    │
│  4. Repeat until sorted (정렬될 때까지 반복)                    │
│                                                                 │
│  Visual (시각화):                                                │
│                                                                 │
│  [64, 34, 25, 12, 22, 11]                                       │
│   ▲                   ▲                                         │
│   │                   └── Minimum found (최소값 발견)           │
│   └── Current position (현재 위치)                              │
│                                                                 │
│  Swap 64 and 11:                                                │
│  [11, 34, 25, 12, 22, 64]                                       │
│   ▲                                                             │
│   └── Sorted (정렬됨)                                           │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Complete Pass Visualization (전체 순회 시각화)

```
┌─────────────────────────────────────────────────────────────────┐
│           SELECTION SORT VISUALIZATION (선택 정렬 시각화)        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Initial: [64, 34, 25, 12, 22, 11]                              │
│                                                                 │
│  Pass 1: Find min in [64, 34, 25, 12, 22, 11] → 11              │
│          Swap 64 and 11                                         │
│          [11 | 34, 25, 12, 22, 64]                              │
│           ▲                                                     │
│         Sorted                                                  │
│                                                                 │
│  Pass 2: Find min in [34, 25, 12, 22, 64] → 12                  │
│          Swap 34 and 12                                         │
│          [11, 12 | 25, 34, 22, 64]                              │
│           ▲   ▲                                                 │
│          Sorted                                                 │
│                                                                 │
│  Pass 3: Find min in [25, 34, 22, 64] → 22                      │
│          Swap 25 and 22                                         │
│          [11, 12, 22 | 34, 25, 64]                              │
│           ▲   ▲   ▲                                             │
│             Sorted                                              │
│                                                                 │
│  Pass 4: Find min in [34, 25, 64] → 25                          │
│          Swap 34 and 25                                         │
│          [11, 12, 22, 25 | 34, 64]                              │
│                                                                 │
│  Pass 5: Find min in [34, 64] → 34                              │
│          No swap needed (already in place)                      │
│          [11, 12, 22, 25, 34 | 64]                              │
│                                                                 │
│  Final:  [11, 12, 22, 25, 34, 64] ← SORTED!                     │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Implementation (구현)

```c
#include <stdio.h>

void swap(int* a, int* b) {
    int temp = *a;
    *a = *b;
    *b = temp;
}

// Selection Sort (선택 정렬)
// Time: O(n²), Space: O(1)
void selectionSort(int arr[], int n) {
    for (int i = 0; i < n - 1; i++) {
        // Find minimum element in unsorted portion
        // 정렬되지 않은 부분에서 최소 요소 찾기
        int minIdx = i;

        for (int j = i + 1; j < n; j++) {
            if (arr[j] < arr[minIdx]) {
                minIdx = j;
            }
        }

        // Swap minimum with first unsorted element
        // 최소값을 정렬되지 않은 첫 번째 요소와 교환
        if (minIdx != i) {
            swap(&arr[i], &arr[minIdx]);
        }
    }
}

void printArray(int arr[], int n) {
    for (int i = 0; i < n; i++) {
        printf("%d ", arr[i]);
    }
    printf("\n");
}

int main() {
    int arr[] = {64, 34, 25, 12, 22, 11};
    int n = sizeof(arr) / sizeof(arr[0]);

    printf("Original array: ");
    printArray(arr, n);

    selectionSort(arr, n);

    printf("Sorted array: ");
    printArray(arr, n);

    return 0;
}
```

### Selection Sort Analysis (선택 정렬 분석)

```
┌─────────────────────────────────────────────────────────────────┐
│           SELECTION SORT ANALYSIS (선택 정렬 분석)               │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Time Complexity (시간 복잡도):                                  │
│  ─────────────────────────────────────────────────────────────  │
│  • Worst Case (최악): O(n²)                                     │
│  • Average Case (평균): O(n²)                                   │
│  • Best Case (최선): O(n²) - No early termination               │
│                      조기 종료 없음                              │
│                                                                 │
│  Comparisons are always: (n-1) + (n-2) + ... + 1 = n(n-1)/2    │
│  비교는 항상: n(n-1)/2                                          │
│                                                                 │
│  Swaps (교환 횟수):                                              │
│  • Maximum: n-1 (much better than Bubble Sort)                  │
│  • 최대: n-1 (버블 정렬보다 훨씬 적음)                          │
│                                                                 │
│  Space Complexity (공간 복잡도): O(1)                            │
│                                                                 │
│  Characteristics (특징):                                         │
│  ✓ In-place (제자리)                                            │
│  ✓ Simple to implement (구현 간단)                              │
│  ✓ Minimum swaps (최소 교환)                                    │
│  ✗ NOT stable (안정 정렬 아님)                                  │
│  ✗ Always O(n²) even for sorted arrays                          │
│    (정렬된 배열에서도 항상 O(n²))                               │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 4. Insertion Sort O(n²) (삽입 정렬)

### Concept (개념)

**Insertion Sort** builds the sorted array one element at a time by inserting each element into its correct position in the already sorted portion.

**삽입 정렬**은 이미 정렬된 부분에 각 요소를 올바른 위치에 삽입하여 한 번에 하나의 요소씩 정렬된 배열을 구축합니다.

```
┌─────────────────────────────────────────────────────────────────┐
│            INSERTION SORT CONCEPT (삽입 정렬 개념)               │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Like sorting playing cards in your hand!                       │
│  손에 있는 카드를 정렬하는 것과 같음!                            │
│                                                                 │
│  Algorithm (알고리즘):                                           │
│  1. Start from second element (두 번째 요소부터 시작)            │
│  2. Compare with elements in sorted portion (정렬된 부분과 비교) │
│  3. Shift larger elements right (큰 요소들을 오른쪽으로 이동)    │
│  4. Insert element in correct position (올바른 위치에 삽입)      │
│  5. Repeat for all elements (모든 요소에 대해 반복)             │
│                                                                 │
│  Visual (시각화):                                                │
│                                                                 │
│  Sorted    | Unsorted                                           │
│  [11, 25, 34 | 12, 22, 64]                                       │
│              ▲                                                  │
│              └── Insert 12 into sorted portion                  │
│                  12를 정렬된 부분에 삽입                         │
│                                                                 │
│  Shift: 34 → right, 25 → right                                  │
│  [11, 25, 34 | 12, 22, 64]                                       │
│       ↘   ↘                                                     │
│  [11, _, 25, 34 | 22, 64]                                        │
│      ▲                                                          │
│  Insert 12 here (12를 여기에 삽입)                               │
│  [11, 12, 25, 34 | 22, 64]                                        │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Complete Pass Visualization (전체 순회 시각화)

```
┌─────────────────────────────────────────────────────────────────┐
│           INSERTION SORT VISUALIZATION (삽입 정렬 시각화)        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Initial: [64, 34, 25, 12, 22, 11]                              │
│            ▲                                                    │
│         Sorted (first element is always sorted)                 │
│                                                                 │
│  Pass 1: Insert 34 into [64]                                    │
│          34 < 64, shift 64 right, insert 34                     │
│          [34, 64 | 25, 12, 22, 11]                              │
│                                                                 │
│  Pass 2: Insert 25 into [34, 64]                                │
│          25 < 64, shift 64; 25 < 34, shift 34; insert 25        │
│          [25, 34, 64 | 12, 22, 11]                              │
│                                                                 │
│  Pass 3: Insert 12 into [25, 34, 64]                            │
│          12 < 64, 12 < 34, 12 < 25, insert at beginning         │
│          [12, 25, 34, 64 | 22, 11]                              │
│                                                                 │
│  Pass 4: Insert 22 into [12, 25, 34, 64]                        │
│          22 < 64, 22 < 34, 22 < 25, 22 > 12                     │
│          [12, 22, 25, 34, 64 | 11]                              │
│                                                                 │
│  Pass 5: Insert 11 into [12, 22, 25, 34, 64]                    │
│          11 < all elements, insert at beginning                 │
│          [11, 12, 22, 25, 34, 64] ← SORTED!                     │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Implementation (구현)

```c
#include <stdio.h>

// Insertion Sort (삽입 정렬)
// Time: O(n²), Space: O(1)
void insertionSort(int arr[], int n) {
    for (int i = 1; i < n; i++) {
        // Key is the element to be inserted
        // key는 삽입될 요소
        int key = arr[i];
        int j = i - 1;

        // Move elements greater than key one position right
        // key보다 큰 요소들을 오른쪽으로 한 위치 이동
        while (j >= 0 && arr[j] > key) {
            arr[j + 1] = arr[j];
            j--;
        }

        // Insert key at correct position
        // key를 올바른 위치에 삽입
        arr[j + 1] = key;
    }
}

// Insertion Sort with step-by-step output (단계별 출력 포함)
void insertionSortVerbose(int arr[], int n) {
    printf("Initial: ");
    for (int k = 0; k < n; k++) printf("%d ", arr[k]);
    printf("\n\n");

    for (int i = 1; i < n; i++) {
        int key = arr[i];
        int j = i - 1;

        printf("Inserting %d:\n", key);

        while (j >= 0 && arr[j] > key) {
            arr[j + 1] = arr[j];
            j--;
        }
        arr[j + 1] = key;

        printf("Result: ");
        for (int k = 0; k < n; k++) {
            printf("%d ", arr[k]);
        }
        printf("\n\n");
    }
}

void printArray(int arr[], int n) {
    for (int i = 0; i < n; i++) {
        printf("%d ", arr[i]);
    }
    printf("\n");
}

int main() {
    int arr[] = {64, 34, 25, 12, 22, 11};
    int n = sizeof(arr) / sizeof(arr[0]);

    printf("=== Insertion Sort Demo ===\n\n");

    insertionSortVerbose(arr, n);

    printf("Final sorted array: ");
    printArray(arr, n);

    return 0;
}
```

### Insertion Sort Analysis (삽입 정렬 분석)

```
┌─────────────────────────────────────────────────────────────────┐
│           INSERTION SORT ANALYSIS (삽입 정렬 분석)               │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Time Complexity (시간 복잡도):                                  │
│  ─────────────────────────────────────────────────────────────  │
│  • Worst Case (최악): O(n²) - Reverse sorted                    │
│                        역순으로 정렬된 경우                      │
│  • Average Case (평균): O(n²)                                   │
│  • Best Case (최선): O(n) - Already sorted                      │
│                      이미 정렬된 경우                            │
│                                                                 │
│  Why O(n) best case? (왜 최선이 O(n)인가?)                      │
│  • Each element only compared once                              │
│  • 각 요소가 한 번만 비교됨                                     │
│  • No shifts needed                                             │
│  • 이동이 필요 없음                                             │
│                                                                 │
│  Space Complexity (공간 복잡도): O(1)                            │
│                                                                 │
│  Characteristics (특징):                                         │
│  ✓ Stable sort (안정 정렬)                                      │
│  ✓ In-place (제자리)                                            │
│  ✓ Adaptive - efficient for nearly sorted data                  │
│    (적응적 - 거의 정렬된 데이터에 효율적)                       │
│  ✓ Good for small datasets (작은 데이터셋에 좋음)               │
│  ✓ Online algorithm - can sort as data arrives                  │
│    (온라인 알고리즘 - 데이터가 도착할 때 정렬 가능)             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 5. Stable vs Unstable Sort (안정 정렬 vs 불안정 정렬)

### Stability Definition (안정성 정의)

```
┌─────────────────────────────────────────────────────────────────┐
│              SORTING STABILITY (정렬 안정성)                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Definition (정의):                                              │
│  A sorting algorithm is STABLE if it preserves the relative     │
│  order of elements with equal keys.                             │
│  정렬 알고리즘은 같은 키를 가진 요소들의 상대적 순서를           │
│  유지하면 안정(STABLE)합니다.                                    │
│                                                                 │
│  Example (예시):                                                 │
│  Original: [(A,2), (B,1), (C,2), (D,3), (E,1)]                  │
│             ↑      ↑      ↑                                     │
│           Note: A and C have same key (2)                       │
│                 B and E have same key (1)                       │
│                                                                 │
│  STABLE Sort Result (안정 정렬 결과):                            │
│  [(B,1), (E,1), (A,2), (C,2), (D,3)]                            │
│     ↑      ↑      ↑      ↑                                      │
│   B before E    A before C   (Original order preserved!)        │
│                              (원래 순서 유지!)                   │
│                                                                 │
│  UNSTABLE Sort Result (불안정 정렬 결과):                        │
│  [(E,1), (B,1), (C,2), (A,2), (D,3)]                            │
│     ↑      ↑      ↑      ↑                                      │
│   E before B    C before A   (Original order NOT preserved!)    │
│                              (원래 순서 유지 안됨!)              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Why Stability Matters (안정성이 중요한 이유)

```
┌─────────────────────────────────────────────────────────────────┐
│           WHY STABILITY MATTERS (안정성이 중요한 이유)           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Scenario: Sort students by grade, then by name                 │
│  시나리오: 학생들을 성적순, 그 다음 이름순으로 정렬              │
│                                                                 │
│  Original (원본):                                                │
│  ┌─────────┬───────┐                                           │
│  │ Name    │ Grade │                                           │
│  ├─────────┼───────┤                                           │
│  │ Alice   │   A   │                                           │
│  │ Bob     │   B   │                                           │
│  │ Charlie │   A   │                                           │
│  │ David   │   B   │                                           │
│  └─────────┴───────┘                                           │
│                                                                 │
│  After sorting by name (이름순 정렬 후):                         │
│  Alice(A), Bob(B), Charlie(A), David(B)                        │
│                                                                 │
│  STABLE sort by grade (안정 정렬로 성적순):                      │
│  Alice(A), Charlie(A), Bob(B), David(B)                        │
│  ↑ Alphabetical order preserved within same grade!              │
│    같은 성적 내에서 알파벳 순서 유지!                           │
│                                                                 │
│  UNSTABLE sort by grade (불안정 정렬로 성적순):                  │
│  Charlie(A), Alice(A), David(B), Bob(B)                        │
│  ↑ Alphabetical order may NOT be preserved!                     │
│    알파벳 순서가 유지되지 않을 수 있음!                         │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Stability Classification (안정성 분류)

```
┌─────────────────────────────────────────────────────────────────┐
│        SORTING ALGORITHMS STABILITY (정렬 알고리즘 안정성)       │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  STABLE Algorithms (안정 알고리즘):                              │
│  ┌────────────────────────────────────────────────────────┐    │
│  │ • Bubble Sort (버블 정렬)                              │    │
│  │ • Insertion Sort (삽입 정렬)                           │    │
│  │ • Merge Sort (병합 정렬)                               │    │
│  │ • Counting Sort (계수 정렬)                            │    │
│  │ • Radix Sort (기수 정렬)                               │    │
│  │ • Tim Sort (팀 정렬)                                   │    │
│  └────────────────────────────────────────────────────────┘    │
│                                                                 │
│  UNSTABLE Algorithms (불안정 알고리즘):                          │
│  ┌────────────────────────────────────────────────────────┐    │
│  │ • Selection Sort (선택 정렬)                           │    │
│  │ • Quick Sort (퀵 정렬)                                 │    │
│  │ • Heap Sort (힙 정렬)                                  │    │
│  │ • Shell Sort (셸 정렬)                                 │    │
│  └────────────────────────────────────────────────────────┘    │
│                                                                 │
│  Note: Unstable algorithms can be made stable with              │
│        extra memory, but this increases space complexity.       │
│  참고: 불안정 알고리즘은 추가 메모리로 안정하게 만들 수 있지만    │
│        공간 복잡도가 증가합니다.                                 │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Why Selection Sort is Unstable (선택 정렬이 불안정한 이유)

```
┌─────────────────────────────────────────────────────────────────┐
│       SELECTION SORT INSTABILITY (선택 정렬의 불안정성)          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Example showing instability:                                   │
│  불안정성을 보여주는 예시:                                       │
│                                                                 │
│  Original: [4a, 3, 4b, 2, 1]                                    │
│            ↑       ↑                                            │
│          Same value 4, 'a' comes before 'b' originally          │
│          같은 값 4, 원래 'a'가 'b' 앞에 옴                       │
│                                                                 │
│  Pass 1: Find min (1), swap with 4a                             │
│  [1, 3, 4b, 2, 4a]                                              │
│                 ↑                                               │
│          4a moved after 4b! (4a가 4b 뒤로 이동!)                │
│                                                                 │
│  After complete sort: [1, 2, 3, 4b, 4a]                         │
│                              ↑    ↑                             │
│                    4b is now before 4a! (4b가 4a 앞에!)         │
│                    Original order NOT preserved!                │
│                    원래 순서 유지되지 않음!                      │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 6. Summary (요약)

```
┌─────────────────────────────────────────────────────────────────┐
│                    CHAPTER 09 SUMMARY (요약)                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  BUBBLE SORT (버블 정렬)                                         │
│  ─────────────────────────────────────────────────────────────  │
│  • Compare and swap adjacent elements (인접 요소 비교 및 교환)  │
│  • Largest element bubbles to end each pass                     │
│  • 각 순회에서 가장 큰 요소가 끝으로 이동                       │
│  • Time: O(n²), Best: O(n) with optimization                    │
│  • Stable: Yes (안정: 예)                                       │
│                                                                 │
│  SELECTION SORT (선택 정렬)                                      │
│  ─────────────────────────────────────────────────────────────  │
│  • Find minimum, swap with first unsorted                       │
│  • 최소값 찾아 정렬되지 않은 첫 요소와 교환                     │
│  • Minimum swaps (최소 교환 횟수)                               │
│  • Time: O(n²) always                                           │
│  • Stable: No (안정: 아니오)                                    │
│                                                                 │
│  INSERTION SORT (삽입 정렬)                                      │
│  ─────────────────────────────────────────────────────────────  │
│  • Insert each element into sorted portion                      │
│  • 각 요소를 정렬된 부분에 삽입                                 │
│  • Like sorting cards (카드 정렬과 같음)                        │
│  • Time: O(n²), Best: O(n) for nearly sorted                    │
│  • Stable: Yes (안정: 예)                                       │
│  • Best for small or nearly sorted data                         │
│  • 작거나 거의 정렬된 데이터에 최적                             │
│                                                                 │
│  COMPARISON TABLE (비교표)                                       │
│  ─────────────────────────────────────────────────────────────  │
│  │ Algorithm  │ Best    │ Average │ Worst   │ Space │ Stable │ │
│  │ Bubble     │ O(n)    │ O(n²)   │ O(n²)   │ O(1)  │ Yes    │ │
│  │ Selection  │ O(n²)   │ O(n²)   │ O(n²)   │ O(1)  │ No     │ │
│  │ Insertion  │ O(n)    │ O(n²)   │ O(n²)   │ O(1)  │ Yes    │ │
│                                                                 │
│  WHEN TO USE (사용 시기)                                         │
│  ─────────────────────────────────────────────────────────────  │
│  • Small datasets (< 50 elements): Any O(n²) algorithm          │
│    작은 데이터셋 (< 50개 요소): 아무 O(n²) 알고리즘             │
│  • Nearly sorted data: Insertion Sort                           │
│    거의 정렬된 데이터: 삽입 정렬                                │
│  • Memory constrained: Selection Sort (fewest swaps)            │
│    메모리 제한: 선택 정렬 (가장 적은 교환)                      │
│  • Stability required: Bubble or Insertion Sort                 │
│    안정성 필요: 버블 또는 삽입 정렬                             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Key Takeaways (핵심 요점)

1. **O(n²) algorithms** are simple but inefficient for large data
   - **O(n²) 알고리즘**은 간단하지만 큰 데이터에 비효율적

2. **Bubble Sort** - easiest to understand, worst performance
   - **버블 정렬** - 이해하기 가장 쉬움, 성능 가장 나쁨

3. **Selection Sort** - minimum swaps, but always O(n²)
   - **선택 정렬** - 최소 교환, 하지만 항상 O(n²)

4. **Insertion Sort** - best for small/nearly sorted data, adaptive
   - **삽입 정렬** - 작은/거의 정렬된 데이터에 최적, 적응적

5. **Stability** matters when sorting on multiple keys
   - **안정성**은 여러 키로 정렬할 때 중요
