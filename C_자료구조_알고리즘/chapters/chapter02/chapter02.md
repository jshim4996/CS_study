# Chapter 02. Pointers and Memory (포인터와 메모리)

> Understanding memory addresses, pointer operations, and dynamic memory management in C.

---

## Memory Address (메모리 주소)

### Concept (개념)

Every variable in C is stored at a specific location in memory, identified by a unique **memory address** (메모리 주소). The address is typically represented as a hexadecimal number.

Memory can be visualized as a sequence of bytes, each with its own address:

```
Address     | Value
------------|-------
0x1000      | 10
0x1004      | 20
0x1008      | 30
```

```c
#include <stdio.h>

int main(void) {
    int num = 42;
    double pi = 3.14159;
    char ch = 'A';

    // Print addresses using & (address-of operator)
    printf("Value of num: %d\n", num);
    printf("Address of num: %p\n", (void*)&num);

    printf("Value of pi: %f\n", pi);
    printf("Address of pi: %p\n", (void*)&pi);

    printf("Value of ch: %c\n", ch);
    printf("Address of ch: %p\n", (void*)&ch);

    // Size of variables (변수의 크기)
    printf("\nSize of int: %zu bytes\n", sizeof(int));
    printf("Size of double: %zu bytes\n", sizeof(double));
    printf("Size of char: %zu bytes\n", sizeof(char));

    return 0;
}
```

### Example
> See `examples/` folder

---

## Pointer Declaration (포인터 선언)

### Concept (개념)

A **pointer** (포인터) is a variable that stores the memory address of another variable. Pointers are declared using the `*` symbol.

**Syntax:** `type* pointer_name;`

```c
#include <stdio.h>

int main(void) {
    // Pointer declaration (포인터 선언)
    int* intPtr;        // Pointer to int
    double* doublePtr;  // Pointer to double
    char* charPtr;      // Pointer to char

    // Initialize pointers
    int num = 100;
    intPtr = &num;  // Store address of num

    printf("Value of num: %d\n", num);
    printf("Address of num: %p\n", (void*)&num);
    printf("Value of intPtr: %p\n", (void*)intPtr);
    printf("Value pointed to by intPtr: %d\n", *intPtr);

    // NULL pointer (널 포인터)
    int* nullPtr = NULL;  // Points to nothing
    if (nullPtr == NULL) {
        printf("nullPtr is NULL\n");
    }

    // Pointer size (포인터 크기) - same regardless of type
    printf("\nSize of int*: %zu bytes\n", sizeof(int*));
    printf("Size of double*: %zu bytes\n", sizeof(double*));
    printf("Size of char*: %zu bytes\n", sizeof(char*));

    return 0;
}
```

### Example
> See `examples/` folder

---

## Address Operator (주소 연산자)

### Concept (개념)

The **address operator** `&` (주소 연산자) returns the memory address of a variable. It is also called the "address-of" operator.

```c
#include <stdio.h>

int main(void) {
    int x = 10;
    int y = 20;
    int z = 30;

    // Using address operator (주소 연산자 사용)
    printf("Address of x: %p\n", (void*)&x);
    printf("Address of y: %p\n", (void*)&y);
    printf("Address of z: %p\n", (void*)&z);

    // Storing address in pointer
    int* ptr = &x;
    printf("\nptr stores address: %p\n", (void*)ptr);
    printf("Value at that address: %d\n", *ptr);

    // Changing what ptr points to
    ptr = &y;
    printf("\nptr now stores address: %p\n", (void*)ptr);
    printf("Value at that address: %d\n", *ptr);

    return 0;
}
```

### Example
> See `examples/` folder

---

## Dereference Operator (역참조 연산자)

### Concept (개념)

The **dereference operator** `*` (역참조 연산자) is used to access or modify the value stored at the address a pointer holds. It is also called the "indirection" operator.

```c
#include <stdio.h>

int main(void) {
    int value = 50;
    int* ptr = &value;

    // Reading value using dereference (역참조로 값 읽기)
    printf("value = %d\n", value);
    printf("*ptr = %d\n", *ptr);

    // Modifying value using dereference (역참조로 값 수정)
    *ptr = 100;
    printf("\nAfter *ptr = 100:\n");
    printf("value = %d\n", value);
    printf("*ptr = %d\n", *ptr);

    // Multiple levels of indirection (다중 포인터)
    int num = 5;
    int* p1 = &num;    // Pointer to int
    int** p2 = &p1;    // Pointer to pointer to int (이중 포인터)

    printf("\nMultiple indirection:\n");
    printf("num = %d\n", num);
    printf("*p1 = %d\n", *p1);
    printf("**p2 = %d\n", **p2);

    // Modify through double pointer
    **p2 = 99;
    printf("\nAfter **p2 = 99:\n");
    printf("num = %d\n", num);

    return 0;
}
```

### Example
> See `examples/` folder

---

## Array-Pointer Relationship (배열과 포인터의 관계)

### Concept (개념)

In C, arrays and pointers are closely related. The array name acts as a pointer to the first element. **Pointer arithmetic** (포인터 연산) allows navigation through array elements.

```c
#include <stdio.h>

int main(void) {
    int arr[5] = {10, 20, 30, 40, 50};

    // Array name is a pointer to first element
    printf("arr = %p\n", (void*)arr);
    printf("&arr[0] = %p\n", (void*)&arr[0]);

    // Accessing elements using pointer arithmetic
    int* ptr = arr;  // Same as: int* ptr = &arr[0];

    printf("\nAccessing elements:\n");
    for (int i = 0; i < 5; i++) {
        // All these are equivalent:
        printf("arr[%d] = %d, *(arr+%d) = %d, *(ptr+%d) = %d, ptr[%d] = %d\n",
               i, arr[i], i, *(arr + i), i, *(ptr + i), i, ptr[i]);
    }

    // Pointer arithmetic (포인터 연산)
    printf("\nPointer arithmetic:\n");
    printf("ptr = %p (points to %d)\n", (void*)ptr, *ptr);
    ptr++;  // Move to next int (adds sizeof(int) bytes)
    printf("ptr++ -> ptr = %p (points to %d)\n", (void*)ptr, *ptr);
    ptr += 2;  // Move forward by 2 integers
    printf("ptr += 2 -> ptr = %p (points to %d)\n", (void*)ptr, *ptr);

    // Difference between pointers
    int* start = arr;
    int* end = &arr[4];
    printf("\nDifference: end - start = %ld elements\n", end - start);

    return 0;
}
```

### Example
> See `examples/` folder

---

## malloc and free (동적 메모리 할당)

### Concept (개념)

**Dynamic memory allocation** (동적 메모리 할당) allows programs to request memory at runtime using functions from `<stdlib.h>`:

- `malloc(size)`: Allocates `size` bytes of memory
- `calloc(n, size)`: Allocates memory for `n` elements, initialized to zero
- `realloc(ptr, size)`: Resizes previously allocated memory
- `free(ptr)`: Deallocates memory (메모리 해제)

**Important:** Always free dynamically allocated memory to avoid **memory leaks** (메모리 누수).

```c
#include <stdio.h>
#include <stdlib.h>

int main(void) {
    // malloc - allocate memory for 5 integers
    int* arr = (int*)malloc(5 * sizeof(int));

    if (arr == NULL) {
        printf("Memory allocation failed!\n");
        return 1;
    }

    // Initialize and use the array
    for (int i = 0; i < 5; i++) {
        arr[i] = (i + 1) * 10;
    }

    printf("Array using malloc:\n");
    for (int i = 0; i < 5; i++) {
        printf("arr[%d] = %d\n", i, arr[i]);
    }

    // free - deallocate memory (메모리 해제)
    free(arr);
    arr = NULL;  // Good practice: set to NULL after free

    // calloc - allocate and zero-initialize
    int* zeros = (int*)calloc(5, sizeof(int));

    if (zeros == NULL) {
        printf("Memory allocation failed!\n");
        return 1;
    }

    printf("\nArray using calloc (initialized to 0):\n");
    for (int i = 0; i < 5; i++) {
        printf("zeros[%d] = %d\n", i, zeros[i]);
    }

    // realloc - resize the array
    zeros = (int*)realloc(zeros, 10 * sizeof(int));

    if (zeros == NULL) {
        printf("Reallocation failed!\n");
        return 1;
    }

    printf("\nArray after realloc (new elements uninitialized):\n");
    for (int i = 0; i < 10; i++) {
        printf("zeros[%d] = %d\n", i, zeros[i]);
    }

    free(zeros);
    zeros = NULL;

    return 0;
}
```

### Example
> See `examples/` folder

---

## Stack vs Heap (스택 vs 힙)

### Concept (개념)

C programs use two main memory regions:

**Stack (스택):**
- Automatic memory management
- Stores local variables and function call information
- Fixed size, fast allocation
- Memory is automatically freed when function returns
- LIFO (Last In, First Out) structure

**Heap (힙):**
- Manual memory management (malloc/free)
- Stores dynamically allocated memory
- Larger, but slower allocation
- Must be explicitly freed by programmer
- Can lead to memory leaks or fragmentation

```
Memory Layout (메모리 레이아웃):
+------------------+
|      Stack       |  <- Local variables, function calls
|        |         |     (grows downward)
|        v         |
|                  |
|        ^         |
|        |         |
|       Heap       |  <- Dynamic allocation (malloc)
|                  |     (grows upward)
+------------------+
|    Data/BSS      |  <- Global/static variables
+------------------+
|      Code        |  <- Program instructions
+------------------+
```

```c
#include <stdio.h>
#include <stdlib.h>

// Global variable (전역 변수) - stored in Data segment
int globalVar = 100;

void demonstrateStack(void) {
    // Local variable (지역 변수) - stored on Stack
    int stackVar = 50;
    printf("Stack variable: %d at address %p\n", stackVar, (void*)&stackVar);
    // stackVar is automatically freed when function returns
}

int main(void) {
    // Local variables on stack (스택의 지역 변수)
    int localA = 10;
    int localB = 20;

    printf("=== Stack Variables (스택 변수) ===\n");
    printf("localA: %d at %p\n", localA, (void*)&localA);
    printf("localB: %d at %p\n", localB, (void*)&localB);

    demonstrateStack();

    printf("\n=== Heap Variables (힙 변수) ===\n");
    // Dynamic allocation on heap (힙의 동적 할당)
    int* heapVar = (int*)malloc(sizeof(int));
    *heapVar = 30;
    printf("heapVar: %d at %p\n", *heapVar, (void*)heapVar);

    int* heapArray = (int*)malloc(3 * sizeof(int));
    heapArray[0] = 1;
    heapArray[1] = 2;
    heapArray[2] = 3;
    printf("heapArray: [%d, %d, %d] at %p\n",
           heapArray[0], heapArray[1], heapArray[2], (void*)heapArray);

    printf("\n=== Global Variable (전역 변수) ===\n");
    printf("globalVar: %d at %p\n", globalVar, (void*)&globalVar);

    // Must free heap memory (힙 메모리 해제 필수)
    free(heapVar);
    free(heapArray);

    return 0;
}
```

### Stack vs Heap Comparison (비교)

| Feature | Stack (스택) | Heap (힙) |
|---------|-------------|----------|
| Management | Automatic | Manual (malloc/free) |
| Size | Limited (usually ~1-8MB) | Large (limited by RAM) |
| Speed | Fast | Slower |
| Lifetime | Function scope | Until explicitly freed |
| Fragmentation | No | Possible |
| Growth | Downward | Upward |

### Example
> See `examples/` folder

---

## Common Pointer Pitfalls (포인터 함정)

### Concept (개념)

Common mistakes when working with pointers:

```c
#include <stdio.h>
#include <stdlib.h>

int main(void) {
    // 1. Uninitialized pointer (초기화되지 않은 포인터)
    // int* badPtr;
    // *badPtr = 10;  // DANGEROUS! Undefined behavior

    // 2. Dangling pointer (댕글링 포인터)
    int* danglingPtr = (int*)malloc(sizeof(int));
    *danglingPtr = 42;
    free(danglingPtr);
    // *danglingPtr = 10;  // DANGEROUS! Memory already freed
    danglingPtr = NULL;  // Good practice: set to NULL

    // 3. Memory leak (메모리 누수)
    // int* leakPtr = (int*)malloc(sizeof(int));
    // leakPtr = (int*)malloc(sizeof(int));  // Original memory lost!
    // Always free before reassigning

    // 4. Double free (이중 해제)
    int* ptr = (int*)malloc(sizeof(int));
    free(ptr);
    // free(ptr);  // DANGEROUS! Undefined behavior
    ptr = NULL;  // Safe: free(NULL) is harmless

    printf("Avoided common pointer pitfalls!\n");

    return 0;
}
```

### Example
> See `examples/` folder

---

## Summary (요약)

| Topic | Key Points |
|-------|------------|
| Memory Address | Unique location in memory, accessed with `&` |
| Pointer Declaration | `type* ptr;` stores address of type |
| Address Operator | `&var` returns the address of var |
| Dereference | `*ptr` accesses value at address |
| Array-Pointer | Array name = pointer to first element |
| malloc/free | Dynamic memory allocation and deallocation |
| Stack | Automatic, fast, limited, local variables |
| Heap | Manual, large, slower, dynamic allocation |

---

## Practice Exercises (연습 문제)

1. Write a function that swaps two integers using pointers.
2. Create a dynamic array that can be resized using realloc.
3. Implement a function that reverses an array using pointer arithmetic.
4. Write a program demonstrating the difference between passing by value and by reference.
5. Create a function that allocates a 2D array dynamically.
