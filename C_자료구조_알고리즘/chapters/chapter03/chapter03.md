# Chapter 03. Structures (구조체)

> Grouping related data together using user-defined types in C.

---

## struct (구조체)

### Concept (개념)

A **structure** (구조체) is a user-defined data type that groups related variables of different types under a single name. Each variable within a structure is called a **member** (멤버).

**Syntax:**
```c
struct structure_name {
    type1 member1;
    type2 member2;
    // ...
};
```

```c
#include <stdio.h>

// Structure definition (구조체 정의)
struct Student {
    char name[50];
    int id;
    float gpa;
    int age;
};

int main(void) {
    // Structure variable declaration (구조체 변수 선언)
    struct Student student1;

    // Structure initialization (구조체 초기화)
    struct Student student2 = {"Kim", 20230001, 3.8f, 20};

    // Designated initializers (지정 초기화) - C99
    struct Student student3 = {
        .name = "Lee",
        .id = 20230002,
        .gpa = 3.5f,
        .age = 21
    };

    printf("Student 2: %s, ID: %d, GPA: %.2f, Age: %d\n",
           student2.name, student2.id, student2.gpa, student2.age);

    printf("Student 3: %s, ID: %d, GPA: %.2f, Age: %d\n",
           student3.name, student3.id, student3.gpa, student3.age);

    return 0;
}
```

### Example
> See `examples/` folder

---

## Member Access (멤버 접근)

### Concept (개념)

Structure members are accessed using the **dot operator** `.` (점 연산자) for structure variables, or the **arrow operator** `->` (화살표 연산자) for structure pointers.

```c
#include <stdio.h>
#include <string.h>

struct Point {
    int x;
    int y;
};

struct Rectangle {
    struct Point topLeft;     // Nested structure (중첩 구조체)
    struct Point bottomRight;
    char color[20];
};

int main(void) {
    // Using dot operator (점 연산자 사용)
    struct Point p1;
    p1.x = 10;
    p1.y = 20;

    printf("Point: (%d, %d)\n", p1.x, p1.y);

    // Nested structure access (중첩 구조체 접근)
    struct Rectangle rect;
    rect.topLeft.x = 0;
    rect.topLeft.y = 0;
    rect.bottomRight.x = 100;
    rect.bottomRight.y = 50;
    strcpy(rect.color, "Red");

    printf("Rectangle:\n");
    printf("  Top-Left: (%d, %d)\n", rect.topLeft.x, rect.topLeft.y);
    printf("  Bottom-Right: (%d, %d)\n", rect.bottomRight.x, rect.bottomRight.y);
    printf("  Color: %s\n", rect.color);

    // Calculate area
    int width = rect.bottomRight.x - rect.topLeft.x;
    int height = rect.bottomRight.y - rect.topLeft.y;
    printf("  Area: %d\n", width * height);

    // Array of structures (구조체 배열)
    struct Point points[3] = {
        {0, 0},
        {10, 20},
        {30, 40}
    };

    printf("\nArray of Points:\n");
    for (int i = 0; i < 3; i++) {
        printf("  Point %d: (%d, %d)\n", i, points[i].x, points[i].y);
    }

    return 0;
}
```

### Example
> See `examples/` folder

---

## Structure Pointer (구조체 포인터)

### Concept (개념)

A **structure pointer** (구조체 포인터) stores the address of a structure. Use the **arrow operator** `->` to access members through a pointer.

`ptr->member` is equivalent to `(*ptr).member`

```c
#include <stdio.h>
#include <string.h>

struct Person {
    char name[50];
    int age;
    float height;
};

// Function that takes structure pointer (구조체 포인터를 받는 함수)
void printPerson(const struct Person* p) {
    printf("Name: %s\n", p->name);
    printf("Age: %d\n", p->age);
    printf("Height: %.1f cm\n", p->height);
}

// Function that modifies structure through pointer
void birthday(struct Person* p) {
    p->age++;
    printf("Happy Birthday! %s is now %d years old.\n", p->name, p->age);
}

int main(void) {
    struct Person person1 = {"Park", 25, 175.5f};

    // Create pointer to structure
    struct Person* ptr = &person1;

    // Access using arrow operator (화살표 연산자)
    printf("Using arrow operator:\n");
    printf("Name: %s\n", ptr->name);
    printf("Age: %d\n", ptr->age);

    // Access using dereference and dot (역참조와 점 연산자)
    printf("\nUsing (*ptr).member:\n");
    printf("Name: %s\n", (*ptr).name);
    printf("Age: %d\n", (*ptr).age);

    // Modify through pointer
    ptr->age = 26;
    strcpy(ptr->name, "Park Junior");

    printf("\nAfter modification:\n");
    printPerson(ptr);

    // Function call with structure pointer
    printf("\n");
    birthday(ptr);

    return 0;
}
```

### Example
> See `examples/` folder

---

## typedef (타입 정의)

### Concept (개념)

The `typedef` keyword creates an alias (별칭) for existing types, making code more readable and easier to maintain.

```c
#include <stdio.h>
#include <string.h>

// typedef for primitive types
typedef unsigned int uint;
typedef unsigned char byte;

// typedef for structure (구조체 typedef)
typedef struct {
    int x;
    int y;
} Point;  // Now 'Point' can be used instead of 'struct Point'

// typedef with separate struct definition
struct StudentRecord {
    char name[50];
    int id;
    float scores[5];
};
typedef struct StudentRecord Student;

// Combined definition (일반적인 방식)
typedef struct Book {
    char title[100];
    char author[50];
    int year;
    float price;
} Book;

// Function pointer typedef (함수 포인터 typedef)
typedef int (*CompareFunc)(int, int);

int compareAsc(int a, int b) { return a - b; }
int compareDesc(int a, int b) { return b - a; }

int main(void) {
    // Using typedef'd types
    uint count = 100;
    byte flag = 1;

    printf("count: %u\n", count);
    printf("flag: %d\n", flag);

    // Using Point without 'struct' keyword
    Point p1 = {10, 20};
    Point p2 = {30, 40};

    printf("\nPoint 1: (%d, %d)\n", p1.x, p1.y);
    printf("Point 2: (%d, %d)\n", p2.x, p2.y);

    // Using Book typedef
    Book book1 = {
        .title = "The C Programming Language",
        .author = "Kernighan & Ritchie",
        .year = 1978,
        .price = 45.99f
    };

    printf("\nBook: %s by %s (%d) - $%.2f\n",
           book1.title, book1.author, book1.year, book1.price);

    // Using function pointer typedef
    CompareFunc compare = compareAsc;
    printf("\ncompareAsc(5, 3) = %d\n", compare(5, 3));

    compare = compareDesc;
    printf("compareDesc(5, 3) = %d\n", compare(5, 3));

    return 0;
}
```

### Example
> See `examples/` folder

---

## Dynamic Structure (동적 구조체)

### Concept (개념)

**Dynamic structures** (동적 구조체) are allocated on the heap using `malloc()`, allowing flexible memory management and creation of data structures like linked lists.

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

typedef struct {
    char name[50];
    int age;
    char* address;  // Pointer member for dynamic string
} Person;

// Create a new Person dynamically
Person* createPerson(const char* name, int age, const char* address) {
    Person* p = (Person*)malloc(sizeof(Person));

    if (p == NULL) {
        printf("Memory allocation failed!\n");
        return NULL;
    }

    strcpy(p->name, name);
    p->age = age;

    // Allocate memory for address string
    p->address = (char*)malloc(strlen(address) + 1);
    if (p->address != NULL) {
        strcpy(p->address, address);
    }

    return p;
}

// Free Person and its dynamic members
void freePerson(Person* p) {
    if (p != NULL) {
        free(p->address);  // Free nested dynamic memory first
        free(p);
    }
}

void printPerson(const Person* p) {
    printf("Name: %s\n", p->name);
    printf("Age: %d\n", p->age);
    printf("Address: %s\n", p->address);
}

int main(void) {
    // Single dynamic structure
    printf("=== Single Dynamic Structure ===\n");
    Person* person1 = createPerson("Kim", 30, "Seoul, Korea");

    if (person1 != NULL) {
        printPerson(person1);
        freePerson(person1);
    }

    // Dynamic array of structures (구조체의 동적 배열)
    printf("\n=== Dynamic Array of Structures ===\n");
    int numStudents = 3;
    Person* students = (Person*)malloc(numStudents * sizeof(Person));

    if (students != NULL) {
        // Initialize students
        strcpy(students[0].name, "Lee");
        students[0].age = 20;
        students[0].address = strdup("Busan");

        strcpy(students[1].name, "Park");
        students[1].age = 21;
        students[1].address = strdup("Daegu");

        strcpy(students[2].name, "Choi");
        students[2].age = 22;
        students[2].address = strdup("Incheon");

        // Print all students
        for (int i = 0; i < numStudents; i++) {
            printf("\nStudent %d:\n", i + 1);
            printPerson(&students[i]);
        }

        // Free all allocated memory
        for (int i = 0; i < numStudents; i++) {
            free(students[i].address);
        }
        free(students);
    }

    // Array of pointers to structures
    printf("\n=== Array of Pointers to Structures ===\n");
    Person** people = (Person**)malloc(2 * sizeof(Person*));

    if (people != NULL) {
        people[0] = createPerson("Jung", 35, "Gwangju");
        people[1] = createPerson("Kang", 28, "Daejeon");

        for (int i = 0; i < 2; i++) {
            printf("\nPerson %d:\n", i + 1);
            printPerson(people[i]);
        }

        // Free memory
        for (int i = 0; i < 2; i++) {
            freePerson(people[i]);
        }
        free(people);
    }

    return 0;
}
```

### Example
> See `examples/` folder

---

## Structure and Functions (구조체와 함수)

### Concept (개념)

Structures can be passed to functions by value or by reference (using pointers). Passing by pointer is more efficient for large structures.

```c
#include <stdio.h>
#include <string.h>
#include <math.h>

typedef struct {
    double x;
    double y;
} Vector2D;

// Pass by value (값에 의한 전달) - makes a copy
Vector2D addVectorsByValue(Vector2D v1, Vector2D v2) {
    Vector2D result;
    result.x = v1.x + v2.x;
    result.y = v1.y + v2.y;
    return result;
}

// Pass by pointer (포인터에 의한 전달) - more efficient
void addVectorsByPointer(const Vector2D* v1, const Vector2D* v2, Vector2D* result) {
    result->x = v1->x + v2->x;
    result->y = v1->y + v2->y;
}

// Calculate magnitude
double magnitude(const Vector2D* v) {
    return sqrt(v->x * v->x + v->y * v->y);
}

// Normalize vector (modifies in place)
void normalize(Vector2D* v) {
    double mag = magnitude(v);
    if (mag > 0) {
        v->x /= mag;
        v->y /= mag;
    }
}

// Return structure from function
Vector2D createVector(double x, double y) {
    Vector2D v = {x, y};
    return v;
}

void printVector(const char* name, const Vector2D* v) {
    printf("%s: (%.2f, %.2f)\n", name, v->x, v->y);
}

int main(void) {
    Vector2D v1 = {3.0, 4.0};
    Vector2D v2 = {1.0, 2.0};

    printVector("v1", &v1);
    printVector("v2", &v2);

    // Add by value
    Vector2D sum1 = addVectorsByValue(v1, v2);
    printVector("sum (by value)", &sum1);

    // Add by pointer
    Vector2D sum2;
    addVectorsByPointer(&v1, &v2, &sum2);
    printVector("sum (by pointer)", &sum2);

    // Magnitude
    printf("\nMagnitude of v1: %.2f\n", magnitude(&v1));

    // Normalize
    Vector2D v3 = {3.0, 4.0};
    printf("\nBefore normalize: ");
    printVector("v3", &v3);

    normalize(&v3);
    printf("After normalize: ");
    printVector("v3", &v3);
    printf("Magnitude after normalize: %.2f\n", magnitude(&v3));

    // Create vector using function
    Vector2D v4 = createVector(5.0, 12.0);
    printVector("\nv4 (created by function)", &v4);

    return 0;
}
```

### Example
> See `examples/` folder

---

## Self-Referential Structures (자기 참조 구조체)

### Concept (개념)

A **self-referential structure** (자기 참조 구조체) contains a pointer to the same structure type. This is essential for creating linked data structures like linked lists and trees.

```c
#include <stdio.h>
#include <stdlib.h>

// Self-referential structure for linked list (연결 리스트용 자기 참조 구조체)
typedef struct Node {
    int data;
    struct Node* next;  // Pointer to same type
} Node;

// Self-referential structure for binary tree (이진 트리용)
typedef struct TreeNode {
    int data;
    struct TreeNode* left;
    struct TreeNode* right;
} TreeNode;

// Create a new node
Node* createNode(int data) {
    Node* newNode = (Node*)malloc(sizeof(Node));
    if (newNode != NULL) {
        newNode->data = data;
        newNode->next = NULL;
    }
    return newNode;
}

// Print linked list
void printList(Node* head) {
    printf("List: ");
    Node* current = head;
    while (current != NULL) {
        printf("%d -> ", current->data);
        current = current->next;
    }
    printf("NULL\n");
}

// Free linked list
void freeList(Node* head) {
    Node* current = head;
    while (current != NULL) {
        Node* temp = current;
        current = current->next;
        free(temp);
    }
}

int main(void) {
    // Create a simple linked list: 10 -> 20 -> 30 -> NULL
    Node* head = createNode(10);
    head->next = createNode(20);
    head->next->next = createNode(30);

    printList(head);

    // Add new node at beginning
    Node* newHead = createNode(5);
    newHead->next = head;
    head = newHead;

    printf("After adding 5 at beginning:\n");
    printList(head);

    // Free memory
    freeList(head);

    return 0;
}
```

### Example
> See `examples/` folder

---

## Summary (요약)

| Topic | Key Points |
|-------|------------|
| struct | Groups related variables of different types |
| Member Access | `.` for variables, `->` for pointers |
| Structure Pointer | `struct Type* ptr` accesses members with `->` |
| typedef | Creates type aliases for readability |
| Dynamic Structure | Allocated with malloc, must be freed |
| Self-Referential | Contains pointer to same type (for linked structures) |

---

## Practice Exercises (연습 문제)

1. Create a `Complex` number structure with add, subtract, multiply operations.
2. Implement a `Date` structure with functions to compare and calculate days between dates.
3. Create a dynamic array of `Book` structures with add, remove, and search functionality.
4. Build a simple linked list of `Student` structures with insert and delete operations.
5. Design a `Shape` structure that can represent different shapes (circle, rectangle) using union.
