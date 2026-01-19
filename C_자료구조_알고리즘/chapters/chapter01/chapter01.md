# Chapter 01. C Basic Syntax (C 기본 문법)

> The foundation of C programming: compilation, variables, input/output, operators, control flow, and functions.

---

## Compiler (컴파일러)

### Concept (개념)

A **compiler** (컴파일러) is a program that translates source code written in C into machine code (기계어) that the computer can execute. The compilation process consists of several stages:

1. **Preprocessing** (전처리): Handles directives like `#include` and `#define`
2. **Compilation** (컴파일): Converts C code to assembly code
3. **Assembly** (어셈블): Converts assembly to object code
4. **Linking** (링킹): Combines object files into an executable

```c
// Compilation command using GCC
// gcc -o program program.c

#include <stdio.h>  // Preprocessor directive (전처리 지시문)

int main(void) {
    printf("Hello, World!\n");
    return 0;
}
```

### Example
> See `examples/` folder

---

## Variables (변수)

### Concept (개념)

A **variable** (변수) is a named storage location in memory that holds a value. In C, variables must be declared with a specific **data type** (자료형) before use.

**Primitive Data Types (기본 자료형):**

| Type | Size (bytes) | Description |
|------|-------------|-------------|
| `char` | 1 | Character (문자) |
| `int` | 4 | Integer (정수) |
| `float` | 4 | Single-precision floating point (단정밀도 실수) |
| `double` | 8 | Double-precision floating point (배정밀도 실수) |

```c
#include <stdio.h>

int main(void) {
    // Variable declaration and initialization (변수 선언과 초기화)
    int age = 25;
    char grade = 'A';
    float height = 175.5f;
    double pi = 3.14159265358979;

    // Constants (상수) using const keyword
    const int MAX_SIZE = 100;

    printf("Age: %d\n", age);
    printf("Grade: %c\n", grade);
    printf("Height: %.1f\n", height);
    printf("Pi: %.14f\n", pi);

    return 0;
}
```

### Example
> See `examples/` folder

---

## Input/Output (입출력)

### Concept (개념)

C provides standard library functions for **input** (입력) and **output** (출력) operations through `<stdio.h>`.

**Key Functions:**
- `printf()`: Formatted output (형식화된 출력)
- `scanf()`: Formatted input (형식화된 입력)
- `getchar()`: Read a single character
- `putchar()`: Write a single character

**Format Specifiers (형식 지정자):**

| Specifier | Data Type |
|-----------|-----------|
| `%d` | int |
| `%f` | float |
| `%lf` | double |
| `%c` | char |
| `%s` | string (문자열) |

```c
#include <stdio.h>

int main(void) {
    int num;
    char name[50];

    // Output (출력)
    printf("Enter your name: ");

    // Input (입력)
    scanf("%s", name);

    printf("Enter a number: ");
    scanf("%d", &num);  // Note: & (address operator) needed for non-array types

    printf("Hello, %s! Your number is %d.\n", name, num);

    return 0;
}
```

### Example
> See `examples/` folder

---

## Operators (연산자)

### Concept (개념)

**Operators** (연산자) are symbols that perform operations on operands.

**Categories:**

1. **Arithmetic Operators** (산술 연산자): `+`, `-`, `*`, `/`, `%`
2. **Relational Operators** (관계 연산자): `==`, `!=`, `<`, `>`, `<=`, `>=`
3. **Logical Operators** (논리 연산자): `&&` (AND), `||` (OR), `!` (NOT)
4. **Assignment Operators** (대입 연산자): `=`, `+=`, `-=`, `*=`, `/=`
5. **Bitwise Operators** (비트 연산자): `&`, `|`, `^`, `~`, `<<`, `>>`

```c
#include <stdio.h>

int main(void) {
    int a = 10, b = 3;

    // Arithmetic operators (산술 연산자)
    printf("a + b = %d\n", a + b);   // 13
    printf("a - b = %d\n", a - b);   // 7
    printf("a * b = %d\n", a * b);   // 30
    printf("a / b = %d\n", a / b);   // 3 (integer division)
    printf("a %% b = %d\n", a % b);  // 1 (remainder/modulo)

    // Relational operators (관계 연산자)
    printf("a > b: %d\n", a > b);    // 1 (true)
    printf("a == b: %d\n", a == b);  // 0 (false)

    // Logical operators (논리 연산자)
    int x = 1, y = 0;
    printf("x && y: %d\n", x && y);  // 0 (false)
    printf("x || y: %d\n", x || y);  // 1 (true)
    printf("!x: %d\n", !x);          // 0 (false)

    // Increment and decrement (증감 연산자)
    int count = 5;
    printf("count++: %d\n", count++);  // 5 (post-increment)
    printf("count: %d\n", count);      // 6
    printf("++count: %d\n", ++count);  // 7 (pre-increment)

    return 0;
}
```

### Example
> See `examples/` folder

---

## Conditionals (조건문)

### Concept (개념)

**Conditional statements** (조건문) allow the program to make decisions and execute different code blocks based on conditions.

**Types:**
- `if` statement
- `if-else` statement
- `if-else if-else` ladder
- `switch` statement
- Ternary operator `? :`

```c
#include <stdio.h>

int main(void) {
    int score = 85;
    char grade;

    // if-else if-else ladder
    if (score >= 90) {
        grade = 'A';
    } else if (score >= 80) {
        grade = 'B';
    } else if (score >= 70) {
        grade = 'C';
    } else if (score >= 60) {
        grade = 'D';
    } else {
        grade = 'F';
    }

    printf("Score: %d, Grade: %c\n", score, grade);

    // Ternary operator (삼항 연산자)
    const char* result = (score >= 60) ? "Pass" : "Fail";
    printf("Result: %s\n", result);

    // Switch statement
    int day = 3;
    switch (day) {
        case 1:
            printf("Monday\n");
            break;
        case 2:
            printf("Tuesday\n");
            break;
        case 3:
            printf("Wednesday\n");
            break;
        default:
            printf("Other day\n");
            break;
    }

    return 0;
}
```

### Example
> See `examples/` folder

---

## Loops (반복문)

### Concept (개념)

**Loops** (반복문) allow repeated execution of a block of code.

**Types:**
- `for` loop: Best when the number of iterations is known
- `while` loop: Best when the condition is checked before execution
- `do-while` loop: Best when the code must execute at least once

```c
#include <stdio.h>

int main(void) {
    // for loop (for 반복문)
    printf("For loop:\n");
    for (int i = 0; i < 5; i++) {
        printf("%d ", i);
    }
    printf("\n");

    // while loop (while 반복문)
    printf("While loop:\n");
    int j = 0;
    while (j < 5) {
        printf("%d ", j);
        j++;
    }
    printf("\n");

    // do-while loop (do-while 반복문)
    printf("Do-while loop:\n");
    int k = 0;
    do {
        printf("%d ", k);
        k++;
    } while (k < 5);
    printf("\n");

    // Nested loops (중첩 반복문) - Multiplication table
    printf("\nMultiplication Table (2):\n");
    for (int n = 1; n <= 9; n++) {
        printf("2 x %d = %d\n", n, 2 * n);
    }

    // Loop control: break and continue
    printf("\nBreak and Continue:\n");
    for (int m = 0; m < 10; m++) {
        if (m == 3) continue;  // Skip 3 (건너뛰기)
        if (m == 7) break;     // Stop at 7 (중단)
        printf("%d ", m);
    }
    printf("\n");

    return 0;
}
```

### Example
> See `examples/` folder

---

## Functions (함수)

### Concept (개념)

A **function** (함수) is a reusable block of code that performs a specific task. Functions help organize code, improve readability, and reduce redundancy.

**Components:**
- **Return type** (반환 타입): The type of value the function returns
- **Function name** (함수 이름): Identifier for the function
- **Parameters** (매개변수): Input values passed to the function
- **Function body** (함수 본문): The code that executes

```c
#include <stdio.h>

// Function prototype/declaration (함수 선언)
int add(int a, int b);
void printMessage(const char* msg);
int factorial(int n);

int main(void) {
    // Function calls (함수 호출)
    int sum = add(5, 3);
    printf("5 + 3 = %d\n", sum);

    printMessage("Hello from function!");

    printf("5! = %d\n", factorial(5));

    return 0;
}

// Function definitions (함수 정의)

// Function with return value (반환값이 있는 함수)
int add(int a, int b) {
    return a + b;
}

// Function without return value (반환값이 없는 함수 - void)
void printMessage(const char* msg) {
    printf("%s\n", msg);
}

// Recursive function (재귀 함수)
int factorial(int n) {
    if (n <= 1) {
        return 1;  // Base case (기저 조건)
    }
    return n * factorial(n - 1);  // Recursive call (재귀 호출)
}
```

**Pass by Value vs Pass by Reference:**

```c
#include <stdio.h>

// Pass by value (값에 의한 전달) - copy is made
void incrementValue(int x) {
    x = x + 1;  // Only modifies the local copy
}

// Pass by reference using pointer (참조에 의한 전달)
void incrementReference(int* x) {
    *x = *x + 1;  // Modifies the original value
}

int main(void) {
    int num = 10;

    incrementValue(num);
    printf("After incrementValue: %d\n", num);  // Still 10

    incrementReference(&num);
    printf("After incrementReference: %d\n", num);  // Now 11

    return 0;
}
```

### Example
> See `examples/` folder

---

## Summary (요약)

| Topic | Key Points |
|-------|------------|
| Compiler | Translates C source code to machine code |
| Variables | Named memory locations with specific data types |
| I/O | `printf()` for output, `scanf()` for input |
| Operators | Arithmetic, relational, logical, assignment, bitwise |
| Conditionals | `if`, `if-else`, `switch`, ternary operator |
| Loops | `for`, `while`, `do-while`, `break`, `continue` |
| Functions | Reusable code blocks with parameters and return values |

---

## Practice Exercises (연습 문제)

1. Write a program that calculates the area of a circle given its radius.
2. Create a function that determines if a number is prime.
3. Implement a simple calculator using switch statement.
4. Write a program that prints the Fibonacci sequence up to n terms.
5. Create a function that swaps two integers using pointers.
