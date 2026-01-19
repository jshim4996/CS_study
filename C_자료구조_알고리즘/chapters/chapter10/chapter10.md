# Chapter 10. Sorting - Advanced (정렬 고급)

## Table of Contents (목차)
1. [Divide and Conquer Concept (분할 정복 개념)](#1-divide-and-conquer-concept-분할-정복-개념)
2. [Quick Sort O(n log n) (퀵 정렬)](#2-quick-sort-on-log-n-퀵-정렬)
3. [Merge Sort O(n log n) (병합 정렬)](#3-merge-sort-on-log-n-병합-정렬)
4. [Recursive Calls Analysis (재귀 호출 분석)](#4-recursive-calls-analysis-재귀-호출-분석)
5. [Summary (요약)](#5-summary-요약)

---

## 1. Divide and Conquer Concept (분할 정복 개념)

### What is Divide and Conquer? (분할 정복이란?)

**Divide and Conquer** is a problem-solving paradigm that breaks a problem into smaller subproblems, solves them independently, and combines the results.

**분할 정복**은 문제를 더 작은 하위 문제로 나누고, 독립적으로 해결한 후, 결과를 결합하는 문제 해결 패러다임입니다.

```
┌─────────────────────────────────────────────────────────────────┐
│            DIVIDE AND CONQUER PARADIGM (분할 정복 패러다임)       │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Three Steps (세 단계):                                          │
│                                                                 │
│  1. DIVIDE (분할)                                                │
│     Break the problem into smaller subproblems                  │
│     문제를 더 작은 하위 문제로 나눔                              │
│                                                                 │
│  2. CONQUER (정복)                                               │
│     Solve subproblems recursively                               │
│     하위 문제를 재귀적으로 해결                                  │
│     Base case: solve directly if small enough                   │
│     기본 조건: 충분히 작으면 직접 해결                           │
│                                                                 │
│  3. COMBINE (결합)                                               │
│     Merge solutions of subproblems                              │
│     하위 문제의 해결책을 병합                                    │
│                                                                 │
│                    ┌─────────────┐                              │
│                    │   Problem   │                              │
│                    │    (문제)    │                              │
│                    └──────┬──────┘                              │
│                           │ DIVIDE (분할)                        │
│              ┌────────────┼────────────┐                        │
│              ▼            ▼            ▼                        │
│        ┌─────────┐  ┌─────────┐  ┌─────────┐                   │
│        │ Sub 1   │  │ Sub 2   │  │ Sub 3   │                   │
│        └────┬────┘  └────┬────┘  └────┬────┘                   │
│             │ CONQUER    │            │                         │
│             ▼            ▼            ▼                         │
│        ┌─────────┐  ┌─────────┐  ┌─────────┐                   │
│        │ Sol 1   │  │ Sol 2   │  │ Sol 3   │                   │
│        └────┬────┘  └────┬────┘  └────┬────┘                   │
│             │            │            │                         │
│             └────────────┼────────────┘                         │
│                          │ COMBINE (결합)                        │
│                          ▼                                      │
│                    ┌─────────────┐                              │
│                    │  Solution   │                              │
│                    │   (해결책)   │                              │
│                    └─────────────┘                              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Why is it Efficient? (왜 효율적인가?)

```
┌─────────────────────────────────────────────────────────────────┐
│          DIVIDE AND CONQUER EFFICIENCY (분할 정복 효율성)        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Problem Size: n                                                │
│  문제 크기: n                                                    │
│                                                                 │
│  Direct Solution (직접 해결):                                    │
│  T(n) = O(n²) for some algorithms                               │
│                                                                 │
│  Divide and Conquer (분할 정복):                                 │
│                                                                 │
│       n               Level 0: 1 problem of size n              │
│      / \                                                        │
│    n/2  n/2          Level 1: 2 problems of size n/2            │
│    /\    /\                                                     │
│  n/4 n/4 n/4 n/4     Level 2: 4 problems of size n/4            │
│                                                                 │
│  ...continues until problem size = 1                            │
│  ...문제 크기가 1이 될 때까지 계속                               │
│                                                                 │
│  Number of levels: log₂(n)                                      │
│  레벨 수: log₂(n)                                               │
│                                                                 │
│  Work per level: O(n)                                           │
│  레벨당 작업량: O(n)                                            │
│                                                                 │
│  Total: O(n) × O(log n) = O(n log n)                           │
│  총합: O(n) × O(log n) = O(n log n)                            │
│                                                                 │
│  Improvement: O(n²) → O(n log n)                                │
│  개선: O(n²) → O(n log n)                                       │
│                                                                 │
│  Example (n = 1,000,000):                                       │
│  O(n²) = 1,000,000,000,000 operations                          │
│  O(n log n) ≈ 20,000,000 operations                             │
│  ⚡ 50,000x faster! (50,000배 빠름!)                            │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. Quick Sort O(n log n) (퀵 정렬)

### Concept (개념)

**Quick Sort** selects a **pivot** element and partitions the array into elements less than and greater than the pivot. It then recursively sorts the partitions.

**퀵 정렬**은 **피벗** 요소를 선택하고 배열을 피벗보다 작은 요소와 큰 요소로 분할합니다. 그런 다음 분할된 부분을 재귀적으로 정렬합니다.

```
┌─────────────────────────────────────────────────────────────────┐
│                QUICK SORT CONCEPT (퀵 정렬 개념)                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Algorithm (알고리즘):                                           │
│  1. Choose a pivot element (피벗 요소 선택)                      │
│  2. Partition: elements < pivot go left, > pivot go right       │
│     분할: 피벗보다 작은 요소는 왼쪽, 큰 요소는 오른쪽            │
│  3. Recursively sort left and right partitions                  │
│     왼쪽과 오른쪽 분할을 재귀적으로 정렬                         │
│                                                                 │
│  Visual (시각화):                                                │
│                                                                 │
│  [3, 7, 8, 5, 2, 1, 9, 5, 4]    Choose pivot: 5                 │
│                       ▲         피벗 선택: 5                     │
│                     pivot                                       │
│                                                                 │
│  Partition result (분할 결과):                                   │
│  [3, 2, 1, 4] [5] [7, 8, 9, 5]                                  │
│       ▲        ▲        ▲                                       │
│    < pivot  pivot    > pivot                                    │
│   피벗보다 작음     피벗보다 큼                                   │
│                                                                 │
│  Pivot is now in its final sorted position!                     │
│  피벗이 이제 정렬된 최종 위치에 있음!                            │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Partition Process (분할 과정)

```
┌─────────────────────────────────────────────────────────────────┐
│              PARTITION PROCESS (분할 과정)                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Array: [8, 3, 7, 4, 9, 2, 6, 5]   Pivot = 5 (last element)    │
│                               ▲                                 │
│                             pivot                               │
│                                                                 │
│  i: boundary of elements < pivot (피벗보다 작은 요소의 경계)     │
│  j: current element being examined (현재 검사 중인 요소)        │
│                                                                 │
│  Initial: i = -1                                                │
│                                                                 │
│  j=0: arr[0]=8 > pivot(5)  →  no swap                          │
│  [8, 3, 7, 4, 9, 2, 6, 5]  i=-1                                │
│   ▲                                                             │
│                                                                 │
│  j=1: arr[1]=3 < pivot(5)  →  i++, swap arr[i] and arr[j]      │
│  [3, 8, 7, 4, 9, 2, 6, 5]  i=0                                 │
│   ▲                                                             │
│                                                                 │
│  j=2: arr[2]=7 > pivot(5)  →  no swap                          │
│  [3, 8, 7, 4, 9, 2, 6, 5]  i=0                                 │
│                                                                 │
│  j=3: arr[3]=4 < pivot(5)  →  i++, swap arr[i] and arr[j]      │
│  [3, 4, 7, 8, 9, 2, 6, 5]  i=1                                 │
│      ▲                                                          │
│                                                                 │
│  j=4: arr[4]=9 > pivot(5)  →  no swap                          │
│  j=5: arr[5]=2 < pivot(5)  →  i++, swap arr[i] and arr[j]      │
│  [3, 4, 2, 8, 9, 7, 6, 5]  i=2                                 │
│         ▲                                                       │
│                                                                 │
│  j=6: arr[6]=6 > pivot(5)  →  no swap                          │
│                                                                 │
│  Final: swap arr[i+1] with pivot                                │
│  최종: arr[i+1]과 피벗 교환                                      │
│  [3, 4, 2, 5, 9, 7, 6, 8]                                       │
│            ▲                                                    │
│         pivot in place (피벗 위치 확정)                         │
│                                                                 │
│  Elements < 5: [3, 4, 2]                                        │
│  Elements > 5: [9, 7, 6, 8]                                     │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Complete Quick Sort Visualization (전체 퀵 정렬 시각화)

```
┌─────────────────────────────────────────────────────────────────┐
│           QUICK SORT VISUALIZATION (퀵 정렬 시각화)              │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Initial: [8, 3, 7, 4, 9, 2, 6, 5]                              │
│                                                                 │
│                    [8,3,7,4,9,2,6,5]                            │
│                    pivot=5                                      │
│                          │                                      │
│            ┌─────────────┼─────────────┐                        │
│            ▼             ▼             ▼                        │
│        [3,4,2]          [5]       [9,7,6,8]                     │
│        pivot=2                    pivot=8                       │
│            │                          │                         │
│      ┌─────┼─────┐              ┌─────┼─────┐                   │
│      ▼     ▼     ▼              ▼     ▼     ▼                   │
│     []    [2]  [3,4]          [7,6]  [8]   [9]                  │
│                pivot=4       pivot=6                            │
│                    │              │                             │
│              ┌─────┼─────┐   ┌────┼────┐                        │
│              ▼     ▼     ▼   ▼    ▼    ▼                        │
│             [3]   [4]   []  []   [6]  [7]                       │
│                                                                 │
│  Combine results (결과 결합):                                    │
│  [] + [2] + [3] + [4] + [] + [5] + [] + [6] + [7] + [8] + [9]  │
│  = [2, 3, 4, 5, 6, 7, 8, 9]                                     │
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

// Partition function - Lomuto partition scheme
// 분할 함수 - Lomuto 분할 방식
int partition(int arr[], int low, int high) {
    // Choose last element as pivot
    // 마지막 요소를 피벗으로 선택
    int pivot = arr[high];

    // Index of smaller element
    // 작은 요소의 인덱스
    int i = low - 1;

    for (int j = low; j < high; j++) {
        // If current element is smaller than pivot
        // 현재 요소가 피벗보다 작으면
        if (arr[j] < pivot) {
            i++;
            swap(&arr[i], &arr[j]);
        }
    }

    // Place pivot in correct position
    // 피벗을 올바른 위치에 배치
    swap(&arr[i + 1], &arr[high]);

    return i + 1;  // Return pivot index (피벗 인덱스 반환)
}

// Quick Sort recursive function (퀵 정렬 재귀 함수)
void quickSort(int arr[], int low, int high) {
    if (low < high) {
        // Partition and get pivot index
        // 분할하고 피벗 인덱스 얻기
        int pivotIndex = partition(arr, low, high);

        // Recursively sort elements before and after partition
        // 분할 전후의 요소들을 재귀적으로 정렬
        quickSort(arr, low, pivotIndex - 1);   // Left partition (왼쪽)
        quickSort(arr, pivotIndex + 1, high);  // Right partition (오른쪽)
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
    int arr[] = {8, 3, 7, 4, 9, 2, 6, 5};
    int n = sizeof(arr) / sizeof(arr[0]);

    printf("Original array: ");
    printArray(arr, n);

    quickSort(arr, 0, n - 1);

    printf("Sorted array: ");
    printArray(arr, n);

    return 0;
}
```

### Pivot Selection Strategies (피벗 선택 전략)

```
┌─────────────────────────────────────────────────────────────────┐
│           PIVOT SELECTION STRATEGIES (피벗 선택 전략)            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. Last Element (마지막 요소)                                   │
│     Simple but can lead to O(n²) for sorted arrays              │
│     간단하지만 정렬된 배열에서 O(n²) 발생 가능                   │
│                                                                 │
│  2. First Element (첫 번째 요소)                                 │
│     Same issue as last element                                  │
│     마지막 요소와 같은 문제                                      │
│                                                                 │
│  3. Random Element (무작위 요소)                                 │
│     Avoids worst case on sorted data                            │
│     정렬된 데이터에서 최악의 경우 방지                           │
│                                                                 │
│  4. Median of Three (세 값의 중앙값)                             │
│     Median of (first, middle, last)                             │
│     (첫 번째, 중간, 마지막)의 중앙값                             │
│     Best practical choice (가장 좋은 실용적 선택)               │
│                                                                 │
│  Example of Median of Three:                                    │
│  Array: [8, 3, 7, 4, 9, 2, 6, 5]                                │
│  first=8, middle=4, last=5                                      │
│  Median = 5 (good pivot!)                                       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Quick Sort Analysis (퀵 정렬 분석)

```
┌─────────────────────────────────────────────────────────────────┐
│              QUICK SORT ANALYSIS (퀵 정렬 분석)                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Time Complexity (시간 복잡도):                                  │
│  ─────────────────────────────────────────────────────────────  │
│                                                                 │
│  Best Case (최선): O(n log n)                                   │
│  - Pivot always divides array in half                           │
│  - 피벗이 항상 배열을 반으로 나눔                               │
│                                                                 │
│  Average Case (평균): O(n log n)                                │
│  - Most common case with random data                            │
│  - 무작위 데이터에서 가장 일반적인 경우                         │
│                                                                 │
│  Worst Case (최악): O(n²)                                       │
│  - Already sorted array with poor pivot choice                  │
│  - 이미 정렬된 배열에서 나쁜 피벗 선택                          │
│                                                                 │
│  Worst Case Example (최악의 경우 예시):                          │
│  [1, 2, 3, 4, 5] with last element as pivot                     │
│                                                                 │
│       [1,2,3,4,5]      pivot=5                                  │
│           │                                                     │
│     [1,2,3,4] [5]                                               │
│         │                                                       │
│     [1,2,3] [4]                                                 │
│        │                                                        │
│     [1,2] [3]                                                   │
│       │                                                         │
│     [1] [2]                                                     │
│                                                                 │
│  n levels instead of log n! (log n 대신 n개 레벨!)              │
│                                                                 │
│  Space Complexity (공간 복잡도):                                 │
│  • Average: O(log n) - recursion stack                          │
│  • Worst: O(n) - unbalanced partitions                          │
│                                                                 │
│  Characteristics (특징):                                         │
│  ✓ In-place (제자리)                                            │
│  ✓ Fast in practice (실제로 빠름)                               │
│  ✓ Cache-friendly (캐시 친화적)                                 │
│  ✗ NOT stable (안정 정렬 아님)                                  │
│  ✗ Worst case O(n²) (최악의 경우 O(n²))                        │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 3. Merge Sort O(n log n) (병합 정렬)

### Concept (개념)

**Merge Sort** divides the array in half, recursively sorts each half, and then merges the sorted halves together.

**병합 정렬**은 배열을 반으로 나누고, 각 반을 재귀적으로 정렬한 다음, 정렬된 반쪽들을 함께 병합합니다.

```
┌─────────────────────────────────────────────────────────────────┐
│               MERGE SORT CONCEPT (병합 정렬 개념)                │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Algorithm (알고리즘):                                           │
│  1. DIVIDE: Split array into two halves                         │
│     분할: 배열을 두 반으로 나눔                                  │
│  2. CONQUER: Recursively sort each half                         │
│     정복: 각 반을 재귀적으로 정렬                                │
│  3. COMBINE: Merge the sorted halves                            │
│     결합: 정렬된 반쪽들을 병합                                   │
│                                                                 │
│  Visual (시각화):                                                │
│                                                                 │
│           [38, 27, 43, 3, 9, 82, 10]                            │
│                     │                                           │
│           ┌─────────┴─────────┐                                 │
│           │                   │                                 │
│     [38, 27, 43, 3]     [9, 82, 10]                             │
│           │                   │                                 │
│     ┌─────┴─────┐       ┌─────┴─────┐                           │
│     │           │       │           │                           │
│  [38, 27]   [43, 3]   [9, 82]    [10]                           │
│     │           │       │           │                           │
│  ┌──┴──┐   ┌───┴───┐  ┌──┴──┐      │                           │
│  │     │   │       │  │     │      │                           │
│ [38]  [27] [43]   [3] [9]  [82]  [10]                          │
│  │     │   │       │  │     │      │                           │
│  └──┬──┘   └───┬───┘  └──┬──┘      │                           │
│  [27, 38]   [3, 43]   [9, 82]    [10]    ← Merge               │
│     │           │       │           │      병합                 │
│     └─────┬─────┘       └─────┬─────┘                           │
│    [3, 27, 38, 43]     [9, 10, 82]       ← Merge               │
│           │                   │                                 │
│           └─────────┬─────────┘                                 │
│        [3, 9, 10, 27, 38, 43, 82]        ← Final Merge         │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Merge Process (병합 과정)

```
┌─────────────────────────────────────────────────────────────────┐
│                 MERGE PROCESS (병합 과정)                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Merging [3, 27, 38, 43] and [9, 10, 82]                        │
│          left array           right array                       │
│                                                                 │
│  Use three pointers: i (left), j (right), k (result)            │
│  세 개의 포인터 사용: i (왼쪽), j (오른쪽), k (결과)             │
│                                                                 │
│  Step 1:  L[i]=3 < R[j]=9  →  result[k]=3, i++, k++            │
│  Left:  [3, 27, 38, 43]    Result: [3]                          │
│          ▲                                                      │
│  Right: [9, 10, 82]                                             │
│          ▲                                                      │
│                                                                 │
│  Step 2:  L[i]=27 > R[j]=9  →  result[k]=9, j++, k++           │
│  Left:  [3, 27, 38, 43]    Result: [3, 9]                       │
│             ▲                                                   │
│  Right: [9, 10, 82]                                             │
│             ▲                                                   │
│                                                                 │
│  Step 3:  L[i]=27 > R[j]=10  →  result[k]=10, j++, k++         │
│  Left:  [3, 27, 38, 43]    Result: [3, 9, 10]                   │
│             ▲                                                   │
│  Right: [9, 10, 82]                                             │
│                 ▲                                               │
│                                                                 │
│  Step 4:  L[i]=27 < R[j]=82  →  result[k]=27, i++, k++         │
│  Result: [3, 9, 10, 27]                                         │
│                                                                 │
│  Step 5:  L[i]=38 < R[j]=82  →  result[k]=38, i++, k++         │
│  Result: [3, 9, 10, 27, 38]                                     │
│                                                                 │
│  Step 6:  L[i]=43 < R[j]=82  →  result[k]=43, i++, k++         │
│  Result: [3, 9, 10, 27, 38, 43]                                 │
│                                                                 │
│  Step 7:  Left exhausted, copy remaining from right             │
│           왼쪽 소진, 오른쪽 나머지 복사                          │
│  Result: [3, 9, 10, 27, 38, 43, 82]                             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Implementation (구현)

```c
#include <stdio.h>
#include <stdlib.h>

// Merge two sorted subarrays (두 정렬된 부분 배열 병합)
void merge(int arr[], int left, int mid, int right) {
    int n1 = mid - left + 1;  // Size of left subarray
    int n2 = right - mid;      // Size of right subarray

    // Create temporary arrays (임시 배열 생성)
    int* L = (int*)malloc(n1 * sizeof(int));
    int* R = (int*)malloc(n2 * sizeof(int));

    // Copy data to temporary arrays (임시 배열에 데이터 복사)
    for (int i = 0; i < n1; i++) {
        L[i] = arr[left + i];
    }
    for (int j = 0; j < n2; j++) {
        R[j] = arr[mid + 1 + j];
    }

    // Merge the temporary arrays back (임시 배열을 다시 병합)
    int i = 0;      // Index for left subarray
    int j = 0;      // Index for right subarray
    int k = left;   // Index for merged array

    while (i < n1 && j < n2) {
        if (L[i] <= R[j]) {
            arr[k] = L[i];
            i++;
        } else {
            arr[k] = R[j];
            j++;
        }
        k++;
    }

    // Copy remaining elements of L[] (L[]의 남은 요소 복사)
    while (i < n1) {
        arr[k] = L[i];
        i++;
        k++;
    }

    // Copy remaining elements of R[] (R[]의 남은 요소 복사)
    while (j < n2) {
        arr[k] = R[j];
        j++;
        k++;
    }

    // Free temporary arrays (임시 배열 해제)
    free(L);
    free(R);
}

// Merge Sort recursive function (병합 정렬 재귀 함수)
void mergeSort(int arr[], int left, int right) {
    if (left < right) {
        // Find middle point (중간 지점 찾기)
        int mid = left + (right - left) / 2;

        // Sort first and second halves (첫 번째와 두 번째 반 정렬)
        mergeSort(arr, left, mid);
        mergeSort(arr, mid + 1, right);

        // Merge the sorted halves (정렬된 반쪽들 병합)
        merge(arr, left, mid, right);
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
    int arr[] = {38, 27, 43, 3, 9, 82, 10};
    int n = sizeof(arr) / sizeof(arr[0]);

    printf("Original array: ");
    printArray(arr, n);

    mergeSort(arr, 0, n - 1);

    printf("Sorted array: ");
    printArray(arr, n);

    return 0;
}
```

### Merge Sort Analysis (병합 정렬 분석)

```
┌─────────────────────────────────────────────────────────────────┐
│              MERGE SORT ANALYSIS (병합 정렬 분석)                │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Time Complexity (시간 복잡도):                                  │
│  ─────────────────────────────────────────────────────────────  │
│                                                                 │
│  All Cases: O(n log n) - ALWAYS! (모든 경우: O(n log n) - 항상!)│
│                                                                 │
│  Why? (왜?)                                                      │
│  • Array is always divided in half: log n levels                │
│    배열이 항상 반으로 나뉨: log n 레벨                          │
│  • Each level processes all n elements: O(n) per level          │
│    각 레벨이 모든 n개 요소 처리: 레벨당 O(n)                    │
│  • Total: O(n) × O(log n) = O(n log n)                         │
│                                                                 │
│       Level 0:        n elements                                │
│       Level 1:        n elements (n/2 + n/2)                    │
│       Level 2:        n elements (n/4 + n/4 + n/4 + n/4)        │
│       ...                                                       │
│       Level log n:    n elements (1 + 1 + ... + 1)              │
│                                                                 │
│  Space Complexity (공간 복잡도): O(n)                            │
│  • Temporary arrays for merging                                 │
│  • 병합을 위한 임시 배열                                        │
│                                                                 │
│  Characteristics (특징):                                         │
│  ✓ STABLE sort (안정 정렬)                                      │
│  ✓ Guaranteed O(n log n) (보장된 O(n log n))                    │
│  ✓ Good for linked lists (연결 리스트에 좋음)                   │
│  ✓ Parallelizable (병렬화 가능)                                 │
│  ✗ NOT in-place - requires O(n) extra space                     │
│    (제자리 아님 - O(n) 추가 공간 필요)                          │
│  ✗ Slower than Quick Sort in practice (cache)                   │
│    (실제로는 퀵 정렬보다 느림 - 캐시)                           │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 4. Recursive Calls Analysis (재귀 호출 분석)

### Recursion Tree (재귀 트리)

```
┌─────────────────────────────────────────────────────────────────┐
│              RECURSION TREE ANALYSIS (재귀 트리 분석)            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  MERGE SORT Recursion Tree for n=8:                             │
│  n=8에 대한 병합 정렬 재귀 트리:                                 │
│                                                                 │
│                      mergeSort(0,7)                             │
│                     /             \                             │
│             mergeSort(0,3)    mergeSort(4,7)                    │
│             /         \       /         \                       │
│      mergeSort    mergeSort  mergeSort  mergeSort               │
│       (0,1)        (2,3)      (4,5)      (6,7)                  │
│       /   \        /   \      /   \      /   \                  │
│     m(0,0) m(1,1) m(2,2) m(3,3) m(4,4) m(5,5) m(6,6) m(7,7)    │
│                                                                 │
│  Level 0: 1 call                                                │
│  Level 1: 2 calls                                               │
│  Level 2: 4 calls                                               │
│  Level 3: 8 calls (base case)                                   │
│                                                                 │
│  Total levels = log₂(8) + 1 = 4                                 │
│  총 레벨 = log₂(8) + 1 = 4                                      │
│                                                                 │
│  Work at each level = O(n)                                      │
│  각 레벨의 작업량 = O(n)                                        │
│                                                                 │
│  Total work = O(n) × O(log n) = O(n log n)                     │
│  총 작업량 = O(n) × O(log n) = O(n log n)                      │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Stack Depth Analysis (스택 깊이 분석)

```
┌─────────────────────────────────────────────────────────────────┐
│              STACK DEPTH ANALYSIS (스택 깊이 분석)               │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  QUICK SORT Stack Trace (퀵 정렬 스택 추적):                     │
│                                                                 │
│  Best Case (최선): Balanced partitions                          │
│  균형 잡힌 분할                                                  │
│                                                                 │
│  Stack:  │ qSort(0,7)  │                                        │
│          │ qSort(0,3)  │                                        │
│          │ qSort(0,1)  │                                        │
│          │ qSort(0,0)  │ ← Maximum depth = log n                │
│          └─────────────┘   최대 깊이 = log n                    │
│                                                                 │
│  Worst Case (최악): Unbalanced partitions                       │
│  불균형 분할                                                     │
│                                                                 │
│  Stack:  │ qSort(0,7)  │                                        │
│          │ qSort(0,6)  │                                        │
│          │ qSort(0,5)  │                                        │
│          │ qSort(0,4)  │                                        │
│          │ qSort(0,3)  │                                        │
│          │ qSort(0,2)  │                                        │
│          │ qSort(0,1)  │                                        │
│          │ qSort(0,0)  │ ← Maximum depth = n                    │
│          └─────────────┘   최대 깊이 = n                        │
│                                                                 │
│  MERGE SORT always has stack depth = O(log n)                   │
│  병합 정렬은 항상 스택 깊이 = O(log n)                          │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Recurrence Relations (점화 관계식)

```
┌─────────────────────────────────────────────────────────────────┐
│           RECURRENCE RELATIONS (점화 관계식)                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  MERGE SORT (병합 정렬):                                         │
│  ─────────────────────────────────────────────────────────────  │
│  T(n) = 2T(n/2) + O(n)                                         │
│         ▲         ▲                                             │
│    2 recursive   Merge                                          │
│    calls on n/2  operation                                      │
│    n/2에 대한 2번 병합 연산                                     │
│    재귀 호출                                                    │
│                                                                 │
│  Solution: T(n) = O(n log n)                                    │
│                                                                 │
│  QUICK SORT (퀵 정렬):                                           │
│  ─────────────────────────────────────────────────────────────  │
│  Best case:  T(n) = 2T(n/2) + O(n)  →  O(n log n)              │
│  Worst case: T(n) = T(n-1) + O(n)   →  O(n²)                   │
│                     ▲                                           │
│               One partition                                     │
│               has n-1 elements                                  │
│               한 분할이 n-1개                                   │
│               요소를 가짐                                       │
│                                                                 │
│  MASTER THEOREM (마스터 정리):                                   │
│  ─────────────────────────────────────────────────────────────  │
│  For T(n) = aT(n/b) + O(n^d):                                  │
│                                                                 │
│  If a < b^d:  T(n) = O(n^d)                                    │
│  If a = b^d:  T(n) = O(n^d log n)                              │
│  If a > b^d:  T(n) = O(n^(log_b a))                            │
│                                                                 │
│  For Merge Sort: a=2, b=2, d=1                                 │
│  2 = 2^1, so T(n) = O(n^1 log n) = O(n log n) ✓                │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 5. Summary (요약)

```
┌─────────────────────────────────────────────────────────────────┐
│                    CHAPTER 10 SUMMARY (요약)                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  DIVIDE AND CONQUER (분할 정복)                                  │
│  ─────────────────────────────────────────────────────────────  │
│  • Break problem into smaller subproblems                       │
│  • 문제를 더 작은 하위 문제로 분할                              │
│  • Solve subproblems recursively                                │
│  • 하위 문제를 재귀적으로 해결                                  │
│  • Combine solutions                                            │
│  • 해결책 결합                                                  │
│                                                                 │
│  QUICK SORT (퀵 정렬)                                            │
│  ─────────────────────────────────────────────────────────────  │
│  • Select pivot, partition array                                │
│  • 피벗 선택, 배열 분할                                         │
│  • Recursively sort partitions                                  │
│  • 분할을 재귀적으로 정렬                                       │
│  • Average: O(n log n), Worst: O(n²)                           │
│  • Space: O(log n) average                                      │
│  • Unstable, In-place                                           │
│  • 불안정, 제자리                                               │
│  • Fastest in practice (실제로 가장 빠름)                       │
│                                                                 │
│  MERGE SORT (병합 정렬)                                          │
│  ─────────────────────────────────────────────────────────────  │
│  • Divide in half, recursively sort, merge                      │
│  • 반으로 분할, 재귀적으로 정렬, 병합                           │
│  • Always O(n log n) (항상 O(n log n))                          │
│  • Space: O(n)                                                  │
│  • STABLE sort (안정 정렬)                                       │
│  • Guaranteed performance (보장된 성능)                         │
│                                                                 │
│  COMPARISON TABLE (비교표)                                       │
│  ─────────────────────────────────────────────────────────────  │
│  │           │ Best      │ Average   │ Worst     │ Space    │  │
│  │ Quick     │ O(n log n)│ O(n log n)│ O(n²)     │ O(log n) │  │
│  │ Merge     │ O(n log n)│ O(n log n)│ O(n log n)│ O(n)     │  │
│                                                                 │
│  │           │ Stable    │ In-place  │ Adaptive  │ Cache    │  │
│  │ Quick     │ No        │ Yes       │ No        │ Good     │  │
│  │ Merge     │ Yes       │ No        │ No        │ Poor     │  │
│                                                                 │
│  WHEN TO USE (사용 시기)                                         │
│  ─────────────────────────────────────────────────────────────  │
│  Quick Sort (퀵 정렬):                                           │
│  • General-purpose sorting (범용 정렬)                          │
│  • When average case matters (평균 경우가 중요할 때)            │
│  • Memory-constrained environments (메모리 제한 환경)           │
│                                                                 │
│  Merge Sort (병합 정렬):                                         │
│  • Guaranteed O(n log n) needed (O(n log n) 보장 필요)          │
│  • Stability required (안정성 필요)                             │
│  • Sorting linked lists (연결 리스트 정렬)                      │
│  • External sorting (외부 정렬)                                 │
│  • Parallel sorting (병렬 정렬)                                 │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Visual Comparison (시각적 비교)

```
┌─────────────────────────────────────────────────────────────────┐
│             QUICK SORT vs MERGE SORT (시각적 비교)               │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  QUICK SORT:                                                    │
│  ──────────                                                     │
│  [unsorted] ──partition──▶ [<pivot] [pivot] [>pivot]           │
│              (work done)          │         │                   │
│                                   ▼         ▼                   │
│                              [recurse]  [recurse]               │
│                                   │         │                   │
│                                   ▼         ▼                   │
│                               [sorted]  [sorted]                │
│  Work is done BEFORE recursion (재귀 전에 작업 수행)            │
│                                                                 │
│  MERGE SORT:                                                    │
│  ───────────                                                    │
│  [unsorted] ──divide──▶ [left half] [right half]               │
│                              │           │                      │
│                              ▼           ▼                      │
│                         [recurse]    [recurse]                  │
│                              │           │                      │
│                              ▼           ▼                      │
│                          [sorted]    [sorted]                   │
│                              │           │                      │
│                              └─────┬─────┘                      │
│                                    │ merge (work done)          │
│                                    ▼                            │
│                                [sorted]                         │
│  Work is done AFTER recursion (재귀 후에 작업 수행)             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Key Takeaways (핵심 요점)

1. **Divide and Conquer** reduces O(n²) to O(n log n)
   - **분할 정복**은 O(n²)을 O(n log n)으로 줄임

2. **Quick Sort** is fastest in practice but has O(n²) worst case
   - **퀵 정렬**은 실제로 가장 빠르지만 최악의 경우 O(n²)

3. **Merge Sort** guarantees O(n log n) but needs O(n) extra space
   - **병합 정렬**은 O(n log n)을 보장하지만 O(n) 추가 공간 필요

4. Choose based on: stability needs, space constraints, data characteristics
   - 선택 기준: 안정성 필요, 공간 제약, 데이터 특성
