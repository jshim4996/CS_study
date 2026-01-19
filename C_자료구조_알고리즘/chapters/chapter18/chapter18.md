# Chapter 18. Final Project - Student Grade Management System (기말 프로젝트 - 학생 성적 관리 시스템)

> Comprehensive project combining structures, linked lists, file handling, and menu-driven programming.

---

## Project Overview (프로젝트 개요)

### Concept (개념)

This final project integrates the data structures and algorithms learned throughout the course into a practical **Student Grade Management System** (학생 성적 관리 시스템). The system will manage student records with full CRUD (Create, Read, Update, Delete) functionality, persistent file storage, and a user-friendly menu interface.

**Learning Objectives:**
- Apply struct and pointer concepts
- Implement linked list operations
- Handle file I/O for data persistence
- Design modular, maintainable code
- Create interactive menu systems

```c
#include <stdio.h>

/*
 * Project Introduction
 * 프로젝트 소개
 */

int main(void) {
    printf("===========================================\n");
    printf("   Student Grade Management System\n");
    printf("   학생 성적 관리 시스템\n");
    printf("===========================================\n\n");

    printf("Project Components:\n");
    printf("==================\n\n");

    printf("1. Data Structure (자료구조)\n");
    printf("   - Student struct with grades\n");
    printf("   - Linked list for dynamic storage\n\n");

    printf("2. Core Operations (핵심 연산)\n");
    printf("   - Create: Add new students\n");
    printf("   - Read: View and search records\n");
    printf("   - Update: Modify student info\n");
    printf("   - Delete: Remove students\n\n");

    printf("3. File Handling (파일 처리)\n");
    printf("   - Save data to file\n");
    printf("   - Load data from file\n");
    printf("   - Auto-save on exit\n\n");

    printf("4. User Interface (사용자 인터페이스)\n");
    printf("   - Menu-driven system\n");
    printf("   - Input validation\n");
    printf("   - Clear output formatting\n\n");

    printf("5. Additional Features (추가 기능)\n");
    printf("   - Sort by name/ID/grade\n");
    printf("   - Statistics calculation\n");
    printf("   - Grade report generation\n");

    return 0;
}
```

**System Architecture:**
```
+------------------------------------------+
|     Student Grade Management System      |
+------------------------------------------+
|                                          |
|  +------------+    +----------------+    |
|  |   Menu     |    |  File Handler  |    |
|  |  System    |    |  (I/O)         |    |
|  +-----+------+    +-------+--------+    |
|        |                   |             |
|        v                   v             |
|  +------------+    +----------------+    |
|  | Operations |<-->| Linked List    |    |
|  | (CRUD)     |    | (Students)     |    |
|  +------------+    +----------------+    |
|                                          |
+------------------------------------------+

Data Flow:
User Input --> Menu --> Operations --> Linked List --> File
```

### Example
> See `examples/` folder

---

## System Requirements (시스템 요구사항)

### Functional Requirements (기능 요구사항)

**1. Student Record Management**
- Store student information (ID, name, grades)
- Support multiple subjects per student
- Calculate averages and letter grades

**2. CRUD Operations**
- **Create**: Add new student records
- **Read**: Display individual or all records
- **Update**: Modify existing records
- **Delete**: Remove student records

**3. Search and Sort**
- Search by student ID
- Search by student name
- Sort by various criteria (ID, name, average)

**4. File Operations**
- Save all records to file
- Load records from file
- Support for both text and binary formats

**5. Statistics**
- Class average
- Highest/lowest scores
- Grade distribution

```c
#include <stdio.h>

/*
 * System Requirements Specification
 * 시스템 요구사항 명세
 */

int main(void) {
    printf("=== System Requirements (시스템 요구사항) ===\n\n");

    printf("1. Data Requirements (데이터 요구사항)\n");
    printf("   +----------------------------------+\n");
    printf("   | Student Record                   |\n");
    printf("   +----------------------------------+\n");
    printf("   | - Student ID (unique, 8 digits)  |\n");
    printf("   | - Name (max 50 characters)       |\n");
    printf("   | - Subjects (3-5 subjects)        |\n");
    printf("   |   * Korean (국어)                |\n");
    printf("   |   * English (영어)               |\n");
    printf("   |   * Math (수학)                  |\n");
    printf("   |   * Science (과학)               |\n");
    printf("   |   * History (역사)               |\n");
    printf("   | - Average score (calculated)     |\n");
    printf("   | - Letter grade (A/B/C/D/F)       |\n");
    printf("   +----------------------------------+\n\n");

    printf("2. CRUD Operations (CRUD 연산)\n");
    printf("   +----------+---------------------------+\n");
    printf("   | Create   | Add new student           |\n");
    printf("   | Read     | View one/all students     |\n");
    printf("   | Update   | Modify student info       |\n");
    printf("   | Delete   | Remove student            |\n");
    printf("   +----------+---------------------------+\n\n");

    printf("3. Search Criteria (검색 조건)\n");
    printf("   - By ID: Exact match\n");
    printf("   - By Name: Partial match supported\n");
    printf("   - By Grade Range: Filter by score\n\n");

    printf("4. Sort Options (정렬 옵션)\n");
    printf("   - By ID (ascending/descending)\n");
    printf("   - By Name (alphabetical)\n");
    printf("   - By Average (highest/lowest first)\n\n");

    printf("5. File Format (파일 형식)\n");
    printf("   Text file: students.txt\n");
    printf("   Format: ID,Name,Korean,English,Math,Science,History\n");
    printf("   Example: 20240001,Kim Min,85,90,78,92,88\n\n");

    printf("6. Statistics (통계)\n");
    printf("   - Class average per subject\n");
    printf("   - Overall class average\n");
    printf("   - Grade distribution (A/B/C/D/F count)\n");
    printf("   - Top/Bottom performers\n\n");

    printf("7. Grade Criteria (성적 기준)\n");
    printf("   +-------+-------------+\n");
    printf("   | Grade | Score Range |\n");
    printf("   +-------+-------------+\n");
    printf("   |   A   |  90 - 100   |\n");
    printf("   |   B   |  80 - 89    |\n");
    printf("   |   C   |  70 - 79    |\n");
    printf("   |   D   |  60 - 69    |\n");
    printf("   |   F   |   0 - 59    |\n");
    printf("   +-------+-------------+\n");

    return 0;
}
```

### Example
> See `examples/` folder

---

## Data Structure Design (자료구조 설계)

### Concept (개념)

The system uses a **linked list** to store student records, allowing for dynamic memory management and efficient insertions/deletions.

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

/*
 * Data Structure Definitions
 * 자료구조 정의
 */

#define MAX_NAME_LEN 50
#define NUM_SUBJECTS 5

// Subject enumeration (과목 열거형)
typedef enum {
    KOREAN = 0,
    ENGLISH,
    MATH,
    SCIENCE,
    HISTORY
} Subject;

// Subject names (과목 이름)
const char* subjectNames[] = {
    "Korean", "English", "Math", "Science", "History"
};

// Student structure (학생 구조체)
typedef struct Student {
    int id;                         // Student ID (학번)
    char name[MAX_NAME_LEN];        // Student name (이름)
    int scores[NUM_SUBJECTS];       // Subject scores (과목 점수)
    float average;                  // Average score (평균)
    char grade;                     // Letter grade (등급)
    struct Student* next;           // Pointer to next student
} Student;

// Student list (linked list head) (학생 리스트)
typedef struct {
    Student* head;
    int count;
} StudentList;

// Initialize student list (리스트 초기화)
void initList(StudentList* list) {
    list->head = NULL;
    list->count = 0;
}

// Calculate average and grade (평균 및 등급 계산)
void calculateGrade(Student* student) {
    int total = 0;
    for (int i = 0; i < NUM_SUBJECTS; i++) {
        total += student->scores[i];
    }
    student->average = (float)total / NUM_SUBJECTS;

    // Assign letter grade (등급 부여)
    if (student->average >= 90) student->grade = 'A';
    else if (student->average >= 80) student->grade = 'B';
    else if (student->average >= 70) student->grade = 'C';
    else if (student->average >= 60) student->grade = 'D';
    else student->grade = 'F';
}

// Create new student (새 학생 생성)
Student* createStudent(int id, const char* name, int scores[]) {
    Student* student = (Student*)malloc(sizeof(Student));
    if (student == NULL) {
        printf("Memory allocation failed!\n");
        return NULL;
    }

    student->id = id;
    strncpy(student->name, name, MAX_NAME_LEN - 1);
    student->name[MAX_NAME_LEN - 1] = '\0';

    for (int i = 0; i < NUM_SUBJECTS; i++) {
        student->scores[i] = scores[i];
    }

    calculateGrade(student);
    student->next = NULL;

    return student;
}

// Print student information (학생 정보 출력)
void printStudent(Student* student) {
    printf("+----------------------------------------+\n");
    printf("| Student ID: %08d                  |\n", student->id);
    printf("| Name: %-32s |\n", student->name);
    printf("+----------------------------------------+\n");
    printf("| Subject    | Score                     |\n");
    printf("+------------+---------------------------+\n");

    for (int i = 0; i < NUM_SUBJECTS; i++) {
        printf("| %-10s | %3d                       |\n",
               subjectNames[i], student->scores[i]);
    }

    printf("+------------+---------------------------+\n");
    printf("| Average    | %6.2f                    |\n", student->average);
    printf("| Grade      |   %c                       |\n", student->grade);
    printf("+----------------------------------------+\n");
}

int main(void) {
    printf("=== Data Structure Design (자료구조 설계) ===\n\n");

    // Create sample student
    int scores[] = {85, 90, 78, 92, 88};
    Student* student = createStudent(20240001, "Kim Min-jun", scores);

    if (student != NULL) {
        printStudent(student);
        free(student);
    }

    // Memory layout visualization
    printf("\n=== Memory Layout (메모리 레이아웃) ===\n\n");

    printf("Student Node:\n");
    printf("+------------------+\n");
    printf("| id: 20240001     |\n");
    printf("+------------------+\n");
    printf("| name: Kim Min... |\n");
    printf("+------------------+\n");
    printf("| scores[5]:       |\n");
    printf("|   [85,90,78,92,88]|\n");
    printf("+------------------+\n");
    printf("| average: 86.6    |\n");
    printf("+------------------+\n");
    printf("| grade: 'B'       |\n");
    printf("+------------------+\n");
    printf("| next: ---------> | (to next student or NULL)\n");
    printf("+------------------+\n\n");

    printf("Linked List Structure:\n");
    printf("HEAD\n");
    printf("  |      +--------+     +--------+     +--------+\n");
    printf("  +----> |Student1| --> |Student2| --> |Student3| --> NULL\n");
    printf("         +--------+     +--------+     +--------+\n");

    return 0;
}
```

**Structure Visualization:**
```
Student Structure (sizeof depends on system):

+------------------------+
|     Student Node       |
+------------------------+
| int id          (4B)   |
+------------------------+
| char name[50]   (50B)  |
+------------------------+
| int scores[5]   (20B)  |
|  [0]: Korean           |
|  [1]: English          |
|  [2]: Math             |
|  [3]: Science          |
|  [4]: History          |
+------------------------+
| float average   (4B)   |
+------------------------+
| char grade      (1B)   |
+------------------------+
| Student* next   (8B)   |
+------------------------+

Linked List:

  list.head
      |
      v
  +--------+     +--------+     +--------+
  |ID: 0001|---->|ID: 0002|---->|ID: 0003|----> NULL
  |Kim...  |     |Lee...  |     |Park... |
  |avg: 85 |     |avg: 92 |     |avg: 78 |
  +--------+     +--------+     +--------+
```

### Example
> See `examples/` folder

---

## CRUD Operations Implementation (CRUD 연산 구현)

### Concept (개념)

**CRUD** stands for Create, Read, Update, Delete - the four basic operations for data management.

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <stdbool.h>

#define MAX_NAME_LEN 50
#define NUM_SUBJECTS 5

/*
 * CRUD Operations Implementation
 * CRUD 연산 구현
 */

// Subject names
const char* subjectNames[] = {"Korean", "English", "Math", "Science", "History"};

// Student structure
typedef struct Student {
    int id;
    char name[MAX_NAME_LEN];
    int scores[NUM_SUBJECTS];
    float average;
    char grade;
    struct Student* next;
} Student;

// Student list
typedef struct {
    Student* head;
    int count;
} StudentList;

// Forward declarations
void calculateGrade(Student* s);
void printStudent(Student* s);

// ============================================
// CREATE: Add new student (학생 추가)
// ============================================

// Insert at end of list
bool addStudent(StudentList* list, int id, const char* name, int scores[]) {
    // Check for duplicate ID
    Student* current = list->head;
    while (current != NULL) {
        if (current->id == id) {
            printf("Error: Student ID %d already exists!\n", id);
            return false;
        }
        current = current->next;
    }

    // Create new student
    Student* newStudent = (Student*)malloc(sizeof(Student));
    if (newStudent == NULL) {
        printf("Memory allocation failed!\n");
        return false;
    }

    newStudent->id = id;
    strncpy(newStudent->name, name, MAX_NAME_LEN - 1);
    newStudent->name[MAX_NAME_LEN - 1] = '\0';

    for (int i = 0; i < NUM_SUBJECTS; i++) {
        newStudent->scores[i] = scores[i];
    }

    calculateGrade(newStudent);
    newStudent->next = NULL;

    // Insert into list
    if (list->head == NULL) {
        list->head = newStudent;
    } else {
        Student* last = list->head;
        while (last->next != NULL) {
            last = last->next;
        }
        last->next = newStudent;
    }

    list->count++;
    printf("Student added successfully! (ID: %d)\n", id);
    return true;
}

// ============================================
// READ: View students (학생 조회)
// ============================================

// Find student by ID
Student* findById(StudentList* list, int id) {
    Student* current = list->head;
    while (current != NULL) {
        if (current->id == id) {
            return current;
        }
        current = current->next;
    }
    return NULL;
}

// Find students by name (partial match)
void findByName(StudentList* list, const char* name) {
    Student* current = list->head;
    int found = 0;

    printf("\nSearch results for '%s':\n", name);
    printf("================================\n");

    while (current != NULL) {
        if (strstr(current->name, name) != NULL) {
            printf("ID: %08d, Name: %s, Average: %.2f, Grade: %c\n",
                   current->id, current->name, current->average, current->grade);
            found++;
        }
        current = current->next;
    }

    if (found == 0) {
        printf("No students found.\n");
    } else {
        printf("================================\n");
        printf("Total: %d student(s) found.\n", found);
    }
}

// Display all students
void displayAll(StudentList* list) {
    if (list->head == NULL) {
        printf("No students in the system.\n");
        return;
    }

    printf("\n===== All Students =====\n");
    printf("%-10s %-20s %-8s %-8s %-8s %-8s %-8s %-8s %-6s\n",
           "ID", "Name", "Korean", "English", "Math", "Science", "History", "Average", "Grade");
    printf("------------------------------------------------------------------------------\n");

    Student* current = list->head;
    while (current != NULL) {
        printf("%-10d %-20s ", current->id, current->name);
        for (int i = 0; i < NUM_SUBJECTS; i++) {
            printf("%-8d ", current->scores[i]);
        }
        printf("%-8.2f %-6c\n", current->average, current->grade);
        current = current->next;
    }

    printf("------------------------------------------------------------------------------\n");
    printf("Total: %d student(s)\n", list->count);
}

// ============================================
// UPDATE: Modify student (학생 수정)
// ============================================

bool updateStudent(StudentList* list, int id, const char* newName, int newScores[]) {
    Student* student = findById(list, id);

    if (student == NULL) {
        printf("Error: Student ID %d not found!\n", id);
        return false;
    }

    // Update name if provided
    if (newName != NULL && strlen(newName) > 0) {
        strncpy(student->name, newName, MAX_NAME_LEN - 1);
        student->name[MAX_NAME_LEN - 1] = '\0';
    }

    // Update scores if provided
    if (newScores != NULL) {
        for (int i = 0; i < NUM_SUBJECTS; i++) {
            if (newScores[i] >= 0) {  // -1 means no change
                student->scores[i] = newScores[i];
            }
        }
        calculateGrade(student);  // Recalculate average and grade
    }

    printf("Student ID %d updated successfully!\n", id);
    return true;
}

// ============================================
// DELETE: Remove student (학생 삭제)
// ============================================

bool deleteStudent(StudentList* list, int id) {
    if (list->head == NULL) {
        printf("Error: List is empty!\n");
        return false;
    }

    Student* current = list->head;
    Student* prev = NULL;

    // Search for student
    while (current != NULL && current->id != id) {
        prev = current;
        current = current->next;
    }

    if (current == NULL) {
        printf("Error: Student ID %d not found!\n", id);
        return false;
    }

    // Remove from list
    if (prev == NULL) {
        // Removing head
        list->head = current->next;
    } else {
        prev->next = current->next;
    }

    free(current);
    list->count--;

    printf("Student ID %d deleted successfully!\n", id);
    return true;
}

// Helper functions
void calculateGrade(Student* s) {
    int total = 0;
    for (int i = 0; i < NUM_SUBJECTS; i++) {
        total += s->scores[i];
    }
    s->average = (float)total / NUM_SUBJECTS;

    if (s->average >= 90) s->grade = 'A';
    else if (s->average >= 80) s->grade = 'B';
    else if (s->average >= 70) s->grade = 'C';
    else if (s->average >= 60) s->grade = 'D';
    else s->grade = 'F';
}

void initList(StudentList* list) {
    list->head = NULL;
    list->count = 0;
}

void freeList(StudentList* list) {
    Student* current = list->head;
    while (current != NULL) {
        Student* next = current->next;
        free(current);
        current = next;
    }
    list->head = NULL;
    list->count = 0;
}

int main(void) {
    printf("=== CRUD Operations Demo ===\n\n");

    StudentList list;
    initList(&list);

    // CREATE: Add students
    printf("--- CREATE: Adding students ---\n");
    int scores1[] = {85, 90, 78, 92, 88};
    int scores2[] = {92, 88, 95, 89, 91};
    int scores3[] = {75, 72, 68, 80, 77};

    addStudent(&list, 20240001, "Kim Min-jun", scores1);
    addStudent(&list, 20240002, "Lee Soo-yeon", scores2);
    addStudent(&list, 20240003, "Park Ji-hoon", scores3);

    // Try duplicate
    addStudent(&list, 20240001, "Duplicate", scores1);

    // READ: Display all
    printf("\n--- READ: Display all students ---");
    displayAll(&list);

    // READ: Find by ID
    printf("\n--- READ: Find by ID ---\n");
    Student* found = findById(&list, 20240002);
    if (found) {
        printf("Found: %s (Average: %.2f)\n", found->name, found->average);
    }

    // READ: Find by name
    printf("\n--- READ: Find by name ---");
    findByName(&list, "Kim");

    // UPDATE: Modify student
    printf("\n--- UPDATE: Modify student ---\n");
    int newScores[] = {95, 92, 88, 90, 93};
    updateStudent(&list, 20240003, "Park Ji-hoon (Updated)", newScores);

    // Display after update
    displayAll(&list);

    // DELETE: Remove student
    printf("\n--- DELETE: Remove student ---\n");
    deleteStudent(&list, 20240002);

    // Try to delete non-existent
    deleteStudent(&list, 99999999);

    // Display after delete
    displayAll(&list);

    // Cleanup
    freeList(&list);

    return 0;
}
```

**CRUD Operation Flow:**
```
CREATE (Add Student):
+--------+    +--------+    +--------+
| HEAD   |--->| Node 1 |--->| NULL   |
+--------+    +--------+    +--------+
                             |
                     addStudent()
                             |
                             v
+--------+    +--------+    +--------+
| HEAD   |--->| Node 1 |--->| Node 2 |--->NULL
+--------+    +--------+    +--------+

READ (Find Student):
+--------+    +--------+    +--------+
| HEAD   |--->| ID:001 |--->| ID:002 |--->NULL
+--------+    +--------+    +--------+
                 ^
                 |
          findById(001)

UPDATE (Modify Student):
Before: scores = [75, 72, 68, 80, 77]
After:  scores = [95, 92, 88, 90, 93]

DELETE (Remove Student):
+--------+    +--------+    +--------+
| HEAD   |--->| Node 1 |--->| Node 2 |--->NULL
+--------+    +--------+    +--------+
                  |              |
                  +------+-------+
                         |
               deleteStudent(Node1)
                         |
                         v
+--------+    +--------+
| HEAD   |--->| Node 2 |--->NULL
+--------+    +--------+
```

### Example
> See `examples/` folder

---

## File I/O Implementation (파일 입출력 구현)

### Concept (개념)

File I/O enables data persistence - saving student records to files and loading them back when the program starts.

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <stdbool.h>

#define MAX_NAME_LEN 50
#define NUM_SUBJECTS 5
#define DATA_FILE "students.txt"

/*
 * File I/O Implementation
 * 파일 입출력 구현
 */

// Subject names
const char* subjectNames[] = {"Korean", "English", "Math", "Science", "History"};

// Student structure
typedef struct Student {
    int id;
    char name[MAX_NAME_LEN];
    int scores[NUM_SUBJECTS];
    float average;
    char grade;
    struct Student* next;
} Student;

// Student list
typedef struct {
    Student* head;
    int count;
} StudentList;

// Forward declarations
void calculateGrade(Student* s);
void initList(StudentList* list);
bool addStudentToList(StudentList* list, Student* student);

// ============================================
// SAVE: Write data to file (파일에 저장)
// ============================================

bool saveToFile(StudentList* list, const char* filename) {
    FILE* file = fopen(filename, "w");
    if (file == NULL) {
        printf("Error: Cannot open file '%s' for writing!\n", filename);
        return false;
    }

    // Write header comment
    fprintf(file, "# Student Grade Management System Data File\n");
    fprintf(file, "# Format: ID,Name,Korean,English,Math,Science,History\n");
    fprintf(file, "# Total records: %d\n", list->count);

    // Write each student
    Student* current = list->head;
    while (current != NULL) {
        fprintf(file, "%d,%s,%d,%d,%d,%d,%d\n",
                current->id,
                current->name,
                current->scores[0],
                current->scores[1],
                current->scores[2],
                current->scores[3],
                current->scores[4]);
        current = current->next;
    }

    fclose(file);
    printf("Data saved to '%s' (%d records)\n", filename, list->count);
    return true;
}

// ============================================
// LOAD: Read data from file (파일에서 로드)
// ============================================

bool loadFromFile(StudentList* list, const char* filename) {
    FILE* file = fopen(filename, "r");
    if (file == NULL) {
        printf("Note: File '%s' not found. Starting with empty database.\n", filename);
        return false;
    }

    char line[256];
    int loadedCount = 0;

    while (fgets(line, sizeof(line), file) != NULL) {
        // Skip comments and empty lines
        if (line[0] == '#' || line[0] == '\n') {
            continue;
        }

        // Parse CSV line
        Student* student = (Student*)malloc(sizeof(Student));
        if (student == NULL) {
            printf("Memory allocation failed during load!\n");
            fclose(file);
            return false;
        }

        // Parse: ID,Name,Korean,English,Math,Science,History
        int parsed = sscanf(line, "%d,%[^,],%d,%d,%d,%d,%d",
                           &student->id,
                           student->name,
                           &student->scores[0],
                           &student->scores[1],
                           &student->scores[2],
                           &student->scores[3],
                           &student->scores[4]);

        if (parsed == 7) {
            calculateGrade(student);
            student->next = NULL;

            // Add to list
            if (addStudentToList(list, student)) {
                loadedCount++;
            } else {
                free(student);
            }
        } else {
            printf("Warning: Skipping malformed line: %s", line);
            free(student);
        }
    }

    fclose(file);
    printf("Loaded %d records from '%s'\n", loadedCount, filename);
    return true;
}

// ============================================
// BINARY FILE: Alternative format (바이너리 파일)
// ============================================

bool saveToBinaryFile(StudentList* list, const char* filename) {
    FILE* file = fopen(filename, "wb");
    if (file == NULL) {
        printf("Error: Cannot open binary file for writing!\n");
        return false;
    }

    // Write count first
    fwrite(&list->count, sizeof(int), 1, file);

    // Write each student (without the next pointer)
    Student* current = list->head;
    while (current != NULL) {
        fwrite(&current->id, sizeof(int), 1, file);
        fwrite(current->name, sizeof(char), MAX_NAME_LEN, file);
        fwrite(current->scores, sizeof(int), NUM_SUBJECTS, file);
        current = current->next;
    }

    fclose(file);
    printf("Data saved to binary file '%s'\n", filename);
    return true;
}

bool loadFromBinaryFile(StudentList* list, const char* filename) {
    FILE* file = fopen(filename, "rb");
    if (file == NULL) {
        return false;
    }

    int count;
    fread(&count, sizeof(int), 1, file);

    for (int i = 0; i < count; i++) {
        Student* student = (Student*)malloc(sizeof(Student));
        if (student == NULL) {
            fclose(file);
            return false;
        }

        fread(&student->id, sizeof(int), 1, file);
        fread(student->name, sizeof(char), MAX_NAME_LEN, file);
        fread(student->scores, sizeof(int), NUM_SUBJECTS, file);

        calculateGrade(student);
        student->next = NULL;
        addStudentToList(list, student);
    }

    fclose(file);
    printf("Loaded %d records from binary file '%s'\n", count, filename);
    return true;
}

// ============================================
// EXPORT: Generate report (보고서 생성)
// ============================================

bool exportReport(StudentList* list, const char* filename) {
    FILE* file = fopen(filename, "w");
    if (file == NULL) {
        return false;
    }

    fprintf(file, "=====================================\n");
    fprintf(file, "    STUDENT GRADE REPORT\n");
    fprintf(file, "=====================================\n\n");

    // Statistics
    float totalAvg = 0;
    int gradeCount[5] = {0};  // A, B, C, D, F

    Student* current = list->head;
    while (current != NULL) {
        totalAvg += current->average;

        switch (current->grade) {
            case 'A': gradeCount[0]++; break;
            case 'B': gradeCount[1]++; break;
            case 'C': gradeCount[2]++; break;
            case 'D': gradeCount[3]++; break;
            case 'F': gradeCount[4]++; break;
        }
        current = current->next;
    }

    if (list->count > 0) {
        totalAvg /= list->count;
    }

    fprintf(file, "Total Students: %d\n", list->count);
    fprintf(file, "Class Average: %.2f\n\n", totalAvg);

    fprintf(file, "Grade Distribution:\n");
    fprintf(file, "  A: %d students\n", gradeCount[0]);
    fprintf(file, "  B: %d students\n", gradeCount[1]);
    fprintf(file, "  C: %d students\n", gradeCount[2]);
    fprintf(file, "  D: %d students\n", gradeCount[3]);
    fprintf(file, "  F: %d students\n\n", gradeCount[4]);

    fprintf(file, "=====================================\n");
    fprintf(file, "    INDIVIDUAL RECORDS\n");
    fprintf(file, "=====================================\n\n");

    current = list->head;
    while (current != NULL) {
        fprintf(file, "ID: %08d\n", current->id);
        fprintf(file, "Name: %s\n", current->name);
        fprintf(file, "Scores: ");
        for (int i = 0; i < NUM_SUBJECTS; i++) {
            fprintf(file, "%s=%d ", subjectNames[i], current->scores[i]);
        }
        fprintf(file, "\nAverage: %.2f, Grade: %c\n", current->average, current->grade);
        fprintf(file, "-------------------------------------\n");
        current = current->next;
    }

    fclose(file);
    printf("Report exported to '%s'\n", filename);
    return true;
}

// Helper functions
void calculateGrade(Student* s) {
    int total = 0;
    for (int i = 0; i < NUM_SUBJECTS; i++) {
        total += s->scores[i];
    }
    s->average = (float)total / NUM_SUBJECTS;

    if (s->average >= 90) s->grade = 'A';
    else if (s->average >= 80) s->grade = 'B';
    else if (s->average >= 70) s->grade = 'C';
    else if (s->average >= 60) s->grade = 'D';
    else s->grade = 'F';
}

void initList(StudentList* list) {
    list->head = NULL;
    list->count = 0;
}

bool addStudentToList(StudentList* list, Student* student) {
    if (list->head == NULL) {
        list->head = student;
    } else {
        Student* last = list->head;
        while (last->next != NULL) {
            last = last->next;
        }
        last->next = student;
    }
    list->count++;
    return true;
}

void freeList(StudentList* list) {
    Student* current = list->head;
    while (current != NULL) {
        Student* next = current->next;
        free(current);
        current = next;
    }
    list->head = NULL;
    list->count = 0;
}

int main(void) {
    printf("=== File I/O Demo ===\n\n");

    StudentList list;
    initList(&list);

    // Create sample students
    Student* s1 = (Student*)malloc(sizeof(Student));
    s1->id = 20240001;
    strcpy(s1->name, "Kim Min-jun");
    int sc1[] = {85, 90, 78, 92, 88};
    memcpy(s1->scores, sc1, sizeof(sc1));
    calculateGrade(s1);
    s1->next = NULL;
    addStudentToList(&list, s1);

    Student* s2 = (Student*)malloc(sizeof(Student));
    s2->id = 20240002;
    strcpy(s2->name, "Lee Soo-yeon");
    int sc2[] = {92, 88, 95, 89, 91};
    memcpy(s2->scores, sc2, sizeof(sc2));
    calculateGrade(s2);
    s2->next = NULL;
    addStudentToList(&list, s2);

    // Save to text file
    printf("--- Saving to text file ---\n");
    saveToFile(&list, DATA_FILE);

    // Save to binary file
    printf("\n--- Saving to binary file ---\n");
    saveToBinaryFile(&list, "students.bin");

    // Export report
    printf("\n--- Exporting report ---\n");
    exportReport(&list, "report.txt");

    // Clear and reload
    printf("\n--- Clearing and reloading ---\n");
    freeList(&list);
    initList(&list);

    printf("After clear: %d students\n", list.count);
    loadFromFile(&list, DATA_FILE);
    printf("After load: %d students\n", list.count);

    // Display file content
    printf("\n--- File Content (students.txt) ---\n");
    printf("# Student Grade Management System Data File\n");
    printf("# Format: ID,Name,Korean,English,Math,Science,History\n");
    printf("# Total records: 2\n");
    printf("20240001,Kim Min-jun,85,90,78,92,88\n");
    printf("20240002,Lee Soo-yeon,92,88,95,89,91\n");

    freeList(&list);

    return 0;
}
```

**File Format Visualization:**
```
Text File (students.txt):
+--------------------------------------------------+
| # Student Grade Management System Data File       |
| # Format: ID,Name,Korean,English,Math,Science,History |
| # Total records: 3                                |
| 20240001,Kim Min-jun,85,90,78,92,88               |
| 20240002,Lee Soo-yeon,92,88,95,89,91              |
| 20240003,Park Ji-hoon,75,72,68,80,77              |
+--------------------------------------------------+

Binary File (students.bin):
+--------------------------------------------------+
| [4 bytes: count=3]                               |
| [Student 1: id, name[50], scores[5]]             |
| [Student 2: id, name[50], scores[5]]             |
| [Student 3: id, name[50], scores[5]]             |
+--------------------------------------------------+

Advantages:
  Text:   Human readable, easy to edit
  Binary: Smaller size, faster I/O, exact data
```

### Example
> See `examples/` folder

---

## Menu System Implementation (메뉴 시스템 구현)

### Concept (개념)

A menu-driven interface provides user-friendly interaction with the system through numbered options.

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <stdbool.h>

#define MAX_NAME_LEN 50
#define NUM_SUBJECTS 5
#define DATA_FILE "students.txt"

/*
 * Complete Menu System Implementation
 * 완전한 메뉴 시스템 구현
 */

const char* subjectNames[] = {"Korean", "English", "Math", "Science", "History"};

typedef struct Student {
    int id;
    char name[MAX_NAME_LEN];
    int scores[NUM_SUBJECTS];
    float average;
    char grade;
    struct Student* next;
} Student;

typedef struct {
    Student* head;
    int count;
} StudentList;

// Function declarations
void initList(StudentList* list);
void freeList(StudentList* list);
void calculateGrade(Student* s);
bool addStudent(StudentList* list, int id, const char* name, int scores[]);
Student* findById(StudentList* list, int id);
void findByName(StudentList* list, const char* name);
void displayAll(StudentList* list);
bool updateStudent(StudentList* list, int id, const char* name, int scores[]);
bool deleteStudent(StudentList* list, int id);
bool saveToFile(StudentList* list, const char* filename);
bool loadFromFile(StudentList* list, const char* filename);
void displayStatistics(StudentList* list);

// ============================================
// MENU DISPLAY (메뉴 표시)
// ============================================

void clearScreen() {
    // Cross-platform clear (simplified)
    printf("\n\n");
}

void displayMainMenu() {
    printf("\n");
    printf("========================================\n");
    printf("   Student Grade Management System\n");
    printf("   학생 성적 관리 시스템\n");
    printf("========================================\n");
    printf("  1. Add Student        (학생 추가)\n");
    printf("  2. Display All        (전체 조회)\n");
    printf("  3. Search by ID       (학번 검색)\n");
    printf("  4. Search by Name     (이름 검색)\n");
    printf("  5. Update Student     (학생 수정)\n");
    printf("  6. Delete Student     (학생 삭제)\n");
    printf("  7. Statistics         (통계)\n");
    printf("  8. Save to File       (파일 저장)\n");
    printf("  9. Load from File     (파일 로드)\n");
    printf("  0. Exit               (종료)\n");
    printf("========================================\n");
    printf("  Select option (0-9): ");
}

// ============================================
// INPUT FUNCTIONS (입력 함수)
// ============================================

void clearInputBuffer() {
    int c;
    while ((c = getchar()) != '\n' && c != EOF);
}

int getIntInput(const char* prompt, int min, int max) {
    int value;
    while (true) {
        printf("%s", prompt);
        if (scanf("%d", &value) == 1 && value >= min && value <= max) {
            clearInputBuffer();
            return value;
        }
        clearInputBuffer();
        printf("Invalid input! Please enter a number between %d and %d.\n", min, max);
    }
}

void getStringInput(const char* prompt, char* buffer, int maxLen) {
    printf("%s", prompt);
    if (fgets(buffer, maxLen, stdin) != NULL) {
        // Remove trailing newline
        size_t len = strlen(buffer);
        if (len > 0 && buffer[len-1] == '\n') {
            buffer[len-1] = '\0';
        }
    }
}

// ============================================
// MENU HANDLERS (메뉴 핸들러)
// ============================================

void handleAddStudent(StudentList* list) {
    printf("\n--- Add New Student ---\n");

    int id = getIntInput("Enter Student ID (8 digits): ", 10000000, 99999999);

    char name[MAX_NAME_LEN];
    getStringInput("Enter Student Name: ", name, MAX_NAME_LEN);

    int scores[NUM_SUBJECTS];
    printf("Enter scores (0-100):\n");
    for (int i = 0; i < NUM_SUBJECTS; i++) {
        char prompt[50];
        sprintf(prompt, "  %s: ", subjectNames[i]);
        scores[i] = getIntInput(prompt, 0, 100);
    }

    if (addStudent(list, id, name, scores)) {
        printf("\nStudent added successfully!\n");
    }
}

void handleDisplayAll(StudentList* list) {
    displayAll(list);
}

void handleSearchById(StudentList* list) {
    printf("\n--- Search by ID ---\n");
    int id = getIntInput("Enter Student ID: ", 10000000, 99999999);

    Student* found = findById(list, id);
    if (found) {
        printf("\nStudent Found:\n");
        printf("ID: %08d\n", found->id);
        printf("Name: %s\n", found->name);
        printf("Scores: ");
        for (int i = 0; i < NUM_SUBJECTS; i++) {
            printf("%s=%d ", subjectNames[i], found->scores[i]);
        }
        printf("\nAverage: %.2f, Grade: %c\n", found->average, found->grade);
    } else {
        printf("Student ID %d not found.\n", id);
    }
}

void handleSearchByName(StudentList* list) {
    printf("\n--- Search by Name ---\n");
    char name[MAX_NAME_LEN];
    getStringInput("Enter name to search: ", name, MAX_NAME_LEN);
    findByName(list, name);
}

void handleUpdateStudent(StudentList* list) {
    printf("\n--- Update Student ---\n");
    int id = getIntInput("Enter Student ID to update: ", 10000000, 99999999);

    Student* found = findById(list, id);
    if (!found) {
        printf("Student ID %d not found.\n", id);
        return;
    }

    printf("\nCurrent Info:\n");
    printf("Name: %s\n", found->name);
    printf("Scores: ");
    for (int i = 0; i < NUM_SUBJECTS; i++) {
        printf("%s=%d ", subjectNames[i], found->scores[i]);
    }
    printf("\n");

    printf("\nEnter new info (or press Enter to keep current):\n");

    char name[MAX_NAME_LEN];
    getStringInput("New name (or Enter to skip): ", name, MAX_NAME_LEN);

    int scores[NUM_SUBJECTS];
    printf("Update scores? (1=Yes, 0=No): ");
    int updateScores;
    scanf("%d", &updateScores);
    clearInputBuffer();

    if (updateScores) {
        for (int i = 0; i < NUM_SUBJECTS; i++) {
            char prompt[50];
            sprintf(prompt, "  %s (current=%d): ", subjectNames[i], found->scores[i]);
            scores[i] = getIntInput(prompt, 0, 100);
        }
    } else {
        for (int i = 0; i < NUM_SUBJECTS; i++) {
            scores[i] = -1;  // No change
        }
    }

    updateStudent(list, id, strlen(name) > 0 ? name : NULL, updateScores ? scores : NULL);
}

void handleDeleteStudent(StudentList* list) {
    printf("\n--- Delete Student ---\n");
    int id = getIntInput("Enter Student ID to delete: ", 10000000, 99999999);

    printf("Are you sure you want to delete student %d? (1=Yes, 0=No): ", id);
    int confirm;
    scanf("%d", &confirm);
    clearInputBuffer();

    if (confirm == 1) {
        deleteStudent(list, id);
    } else {
        printf("Deletion cancelled.\n");
    }
}

void handleStatistics(StudentList* list) {
    displayStatistics(list);
}

void handleSaveToFile(StudentList* list) {
    printf("\n--- Save to File ---\n");
    if (saveToFile(list, DATA_FILE)) {
        printf("Data saved successfully!\n");
    }
}

void handleLoadFromFile(StudentList* list) {
    printf("\n--- Load from File ---\n");
    printf("Warning: This will add to current data. Continue? (1=Yes, 0=No): ");
    int confirm;
    scanf("%d", &confirm);
    clearInputBuffer();

    if (confirm == 1) {
        loadFromFile(list, DATA_FILE);
    }
}

// ============================================
// MAIN PROGRAM LOOP (메인 프로그램 루프)
// ============================================

int main(void) {
    StudentList list;
    initList(&list);

    // Auto-load on startup
    printf("Loading data...\n");
    loadFromFile(&list, DATA_FILE);

    bool running = true;

    while (running) {
        displayMainMenu();

        int choice;
        if (scanf("%d", &choice) != 1) {
            clearInputBuffer();
            printf("Invalid input!\n");
            continue;
        }
        clearInputBuffer();

        switch (choice) {
            case 1: handleAddStudent(&list); break;
            case 2: handleDisplayAll(&list); break;
            case 3: handleSearchById(&list); break;
            case 4: handleSearchByName(&list); break;
            case 5: handleUpdateStudent(&list); break;
            case 6: handleDeleteStudent(&list); break;
            case 7: handleStatistics(&list); break;
            case 8: handleSaveToFile(&list); break;
            case 9: handleLoadFromFile(&list); break;
            case 0:
                printf("\nSave before exit? (1=Yes, 0=No): ");
                int save;
                scanf("%d", &save);
                if (save == 1) {
                    saveToFile(&list, DATA_FILE);
                }
                printf("\nGoodbye!\n");
                running = false;
                break;
            default:
                printf("Invalid option! Please select 0-9.\n");
        }
    }

    freeList(&list);
    return 0;
}

// ============================================
// IMPLEMENTATION OF FUNCTIONS
// (Previous implementations from CRUD and File I/O sections)
// ============================================

void initList(StudentList* list) {
    list->head = NULL;
    list->count = 0;
}

void freeList(StudentList* list) {
    Student* current = list->head;
    while (current != NULL) {
        Student* next = current->next;
        free(current);
        current = next;
    }
    list->head = NULL;
    list->count = 0;
}

void calculateGrade(Student* s) {
    int total = 0;
    for (int i = 0; i < NUM_SUBJECTS; i++) {
        total += s->scores[i];
    }
    s->average = (float)total / NUM_SUBJECTS;

    if (s->average >= 90) s->grade = 'A';
    else if (s->average >= 80) s->grade = 'B';
    else if (s->average >= 70) s->grade = 'C';
    else if (s->average >= 60) s->grade = 'D';
    else s->grade = 'F';
}

bool addStudent(StudentList* list, int id, const char* name, int scores[]) {
    Student* current = list->head;
    while (current != NULL) {
        if (current->id == id) {
            printf("Error: Student ID %d already exists!\n", id);
            return false;
        }
        current = current->next;
    }

    Student* newStudent = (Student*)malloc(sizeof(Student));
    if (newStudent == NULL) return false;

    newStudent->id = id;
    strncpy(newStudent->name, name, MAX_NAME_LEN - 1);
    for (int i = 0; i < NUM_SUBJECTS; i++) {
        newStudent->scores[i] = scores[i];
    }
    calculateGrade(newStudent);
    newStudent->next = NULL;

    if (list->head == NULL) {
        list->head = newStudent;
    } else {
        Student* last = list->head;
        while (last->next != NULL) last = last->next;
        last->next = newStudent;
    }
    list->count++;
    return true;
}

Student* findById(StudentList* list, int id) {
    Student* current = list->head;
    while (current != NULL) {
        if (current->id == id) return current;
        current = current->next;
    }
    return NULL;
}

void findByName(StudentList* list, const char* name) {
    Student* current = list->head;
    int found = 0;
    printf("\nSearch results for '%s':\n", name);
    while (current != NULL) {
        if (strstr(current->name, name) != NULL) {
            printf("ID: %08d, Name: %s, Average: %.2f, Grade: %c\n",
                   current->id, current->name, current->average, current->grade);
            found++;
        }
        current = current->next;
    }
    printf("Found %d student(s).\n", found);
}

void displayAll(StudentList* list) {
    if (list->head == NULL) {
        printf("\nNo students in the system.\n");
        return;
    }
    printf("\n%-10s %-20s %-8s %-8s %-8s %-8s %-8s %-8s %-6s\n",
           "ID", "Name", "Korean", "English", "Math", "Science", "History", "Avg", "Grade");
    printf("--------------------------------------------------------------------------------\n");
    Student* current = list->head;
    while (current != NULL) {
        printf("%-10d %-20s ", current->id, current->name);
        for (int i = 0; i < NUM_SUBJECTS; i++) printf("%-8d ", current->scores[i]);
        printf("%-8.2f %-6c\n", current->average, current->grade);
        current = current->next;
    }
    printf("--------------------------------------------------------------------------------\n");
    printf("Total: %d student(s)\n", list->count);
}

bool updateStudent(StudentList* list, int id, const char* name, int scores[]) {
    Student* s = findById(list, id);
    if (s == NULL) return false;
    if (name != NULL) strncpy(s->name, name, MAX_NAME_LEN - 1);
    if (scores != NULL) {
        for (int i = 0; i < NUM_SUBJECTS; i++) {
            if (scores[i] >= 0) s->scores[i] = scores[i];
        }
        calculateGrade(s);
    }
    printf("Student updated successfully!\n");
    return true;
}

bool deleteStudent(StudentList* list, int id) {
    if (list->head == NULL) return false;
    Student* current = list->head;
    Student* prev = NULL;
    while (current != NULL && current->id != id) {
        prev = current;
        current = current->next;
    }
    if (current == NULL) return false;
    if (prev == NULL) list->head = current->next;
    else prev->next = current->next;
    free(current);
    list->count--;
    printf("Student deleted successfully!\n");
    return true;
}

bool saveToFile(StudentList* list, const char* filename) {
    FILE* file = fopen(filename, "w");
    if (file == NULL) return false;
    fprintf(file, "# Student Data File\n");
    Student* current = list->head;
    while (current != NULL) {
        fprintf(file, "%d,%s,%d,%d,%d,%d,%d\n", current->id, current->name,
                current->scores[0], current->scores[1], current->scores[2],
                current->scores[3], current->scores[4]);
        current = current->next;
    }
    fclose(file);
    return true;
}

bool loadFromFile(StudentList* list, const char* filename) {
    FILE* file = fopen(filename, "r");
    if (file == NULL) return false;
    char line[256];
    while (fgets(line, sizeof(line), file) != NULL) {
        if (line[0] == '#' || line[0] == '\n') continue;
        int id, scores[5];
        char name[MAX_NAME_LEN];
        if (sscanf(line, "%d,%[^,],%d,%d,%d,%d,%d", &id, name,
                   &scores[0], &scores[1], &scores[2], &scores[3], &scores[4]) == 7) {
            addStudent(list, id, name, scores);
        }
    }
    fclose(file);
    return true;
}

void displayStatistics(StudentList* list) {
    if (list->count == 0) {
        printf("\nNo data for statistics.\n");
        return;
    }

    float subjectTotals[NUM_SUBJECTS] = {0};
    float totalAvg = 0;
    int gradeCount[5] = {0};
    Student* highest = list->head;
    Student* lowest = list->head;

    Student* current = list->head;
    while (current != NULL) {
        for (int i = 0; i < NUM_SUBJECTS; i++) {
            subjectTotals[i] += current->scores[i];
        }
        totalAvg += current->average;

        if (current->average > highest->average) highest = current;
        if (current->average < lowest->average) lowest = current;

        switch (current->grade) {
            case 'A': gradeCount[0]++; break;
            case 'B': gradeCount[1]++; break;
            case 'C': gradeCount[2]++; break;
            case 'D': gradeCount[3]++; break;
            case 'F': gradeCount[4]++; break;
        }
        current = current->next;
    }

    printf("\n========== STATISTICS ==========\n");
    printf("Total Students: %d\n\n", list->count);

    printf("Subject Averages:\n");
    for (int i = 0; i < NUM_SUBJECTS; i++) {
        printf("  %s: %.2f\n", subjectNames[i], subjectTotals[i] / list->count);
    }

    printf("\nClass Average: %.2f\n", totalAvg / list->count);

    printf("\nGrade Distribution:\n");
    char grades[] = {'A', 'B', 'C', 'D', 'F'};
    for (int i = 0; i < 5; i++) {
        printf("  %c: %d (%.1f%%)\n", grades[i], gradeCount[i],
               (float)gradeCount[i] / list->count * 100);
    }

    printf("\nTop Student: %s (%.2f)\n", highest->name, highest->average);
    printf("Bottom Student: %s (%.2f)\n", lowest->name, lowest->average);
    printf("================================\n");
}
```

**Menu Flow Visualization:**
```
Program Start
     |
     v
+--------------------+
| Load from File     |
| (auto-load)        |
+--------------------+
     |
     v
+--------------------+
|   Main Menu Loop   |<---------+
+--------------------+          |
     |                          |
     v                          |
+---------+    +------------+   |
| Display |    | Get User   |   |
| Menu    |--->| Choice     |   |
+---------+    +------------+   |
                    |           |
          +---------+---------+ |
          |    |    |    |    | |
          v    v    v    v    v |
        +--+  +--+  +--+  +--+  |
        |1 |  |2 |  |...|  |0 | |
        +--+  +--+  +--+  +--+  |
          |    |    |      |    |
          |    |    |      v    |
          |    |    | Save & Exit
          |    |    |           |
          +----+----+-----------+
```

### Example
> See `examples/` folder

---

## Suggested Implementation Approach (권장 구현 접근법)

### Concept (개념)

Follow this step-by-step approach to build the system incrementally.

```c
#include <stdio.h>

/*
 * Implementation Roadmap
 * 구현 로드맵
 */

int main(void) {
    printf("=== Suggested Implementation Approach ===\n");
    printf("=== 권장 구현 접근법 ===\n\n");

    printf("PHASE 1: Foundation (기초)\n");
    printf("===========================\n");
    printf("1.1 Define Student structure\n");
    printf("1.2 Implement linked list basic operations\n");
    printf("    - initList()\n");
    printf("    - freeList()\n");
    printf("1.3 Implement calculateGrade()\n");
    printf("1.4 Test with hardcoded data\n\n");

    printf("PHASE 2: Core CRUD (핵심 CRUD)\n");
    printf("==============================\n");
    printf("2.1 Implement addStudent()\n");
    printf("2.2 Implement displayAll()\n");
    printf("2.3 Implement findById()\n");
    printf("2.4 Implement updateStudent()\n");
    printf("2.5 Implement deleteStudent()\n");
    printf("2.6 Test each function individually\n\n");

    printf("PHASE 3: User Interface (사용자 인터페이스)\n");
    printf("==========================================\n");
    printf("3.1 Create menu display function\n");
    printf("3.2 Implement input validation\n");
    printf("3.3 Create handler functions for each menu option\n");
    printf("3.4 Build main program loop\n");
    printf("3.5 Test complete workflow\n\n");

    printf("PHASE 4: File I/O (파일 입출력)\n");
    printf("==============================\n");
    printf("4.1 Implement saveToFile()\n");
    printf("4.2 Implement loadFromFile()\n");
    printf("4.3 Add auto-load on startup\n");
    printf("4.4 Add save confirmation on exit\n");
    printf("4.5 Test persistence\n\n");

    printf("PHASE 5: Advanced Features (고급 기능)\n");
    printf("======================================\n");
    printf("5.1 Implement search by name\n");
    printf("5.2 Add sorting functionality\n");
    printf("5.3 Implement statistics\n");
    printf("5.4 Add report generation\n");
    printf("5.5 Error handling improvements\n\n");

    printf("PHASE 6: Testing & Polish (테스트 및 완성)\n");
    printf("=========================================\n");
    printf("6.1 Edge case testing\n");
    printf("6.2 Memory leak checking\n");
    printf("6.3 User experience improvements\n");
    printf("6.4 Code documentation\n");
    printf("6.5 Final testing\n\n");

    printf("=== Code Organization (코드 구성) ===\n\n");
    printf("Recommended File Structure:\n");
    printf("+-----------------------------------+\n");
    printf("| student_system/                   |\n");
    printf("|   |-- main.c        (entry point) |\n");
    printf("|   |-- student.h     (structures)  |\n");
    printf("|   |-- student.c     (CRUD ops)    |\n");
    printf("|   |-- file_io.c     (file ops)    |\n");
    printf("|   |-- menu.c        (UI)          |\n");
    printf("|   |-- utils.c       (helpers)     |\n");
    printf("|   +-- students.txt  (data file)   |\n");
    printf("+-----------------------------------+\n\n");

    printf("=== Testing Checklist ===\n\n");
    printf("[ ] Add student with valid data\n");
    printf("[ ] Add student with duplicate ID (should fail)\n");
    printf("[ ] Add student with boundary scores (0, 100)\n");
    printf("[ ] Display empty list\n");
    printf("[ ] Display list with multiple students\n");
    printf("[ ] Search existing ID\n");
    printf("[ ] Search non-existing ID\n");
    printf("[ ] Search by partial name\n");
    printf("[ ] Update student name only\n");
    printf("[ ] Update student scores only\n");
    printf("[ ] Update both name and scores\n");
    printf("[ ] Delete existing student\n");
    printf("[ ] Delete non-existing student\n");
    printf("[ ] Save to file\n");
    printf("[ ] Load from file\n");
    printf("[ ] Load from non-existing file\n");
    printf("[ ] Statistics with data\n");
    printf("[ ] Statistics empty list\n");
    printf("[ ] Exit without saving\n");
    printf("[ ] Exit with saving\n");

    return 0;
}
```

**Project Structure:**
```
student_system/
|
+-- main.c              # Program entry point, main loop
|   |
|   +-- Includes all headers
|   +-- Main function
|   +-- Program loop
|
+-- student.h           # Header file with declarations
|   |
|   +-- Structure definitions
|   +-- Function prototypes
|   +-- Constants and macros
|
+-- student.c           # Student CRUD operations
|   |
|   +-- createStudent()
|   +-- addStudent()
|   +-- findById()
|   +-- findByName()
|   +-- updateStudent()
|   +-- deleteStudent()
|   +-- calculateGrade()
|
+-- list.c              # Linked list operations
|   |
|   +-- initList()
|   +-- freeList()
|   +-- sortList()
|   +-- getCount()
|
+-- file_io.c           # File operations
|   |
|   +-- saveToFile()
|   +-- loadFromFile()
|   +-- exportReport()
|
+-- menu.c              # User interface
|   |
|   +-- displayMenu()
|   +-- handleXxx() functions
|   +-- Input validation
|
+-- utils.c             # Utility functions
|   |
|   +-- clearScreen()
|   +-- getIntInput()
|   +-- getStringInput()
|
+-- students.txt        # Data file (generated)
+-- Makefile            # Build configuration
```

### Example
> See `examples/` folder

---

## Summary (요약)

| Component | Description |
|-----------|-------------|
| Data Structure | Linked list of Student structures |
| CRUD | Create, Read, Update, Delete operations |
| File I/O | Text and binary file support |
| Menu System | Interactive user interface |
| Statistics | Class averages, grade distribution |

### Skills Applied (적용된 기술)

| Topic | Application |
|-------|-------------|
| Structures | Student record with multiple fields |
| Pointers | Linked list node connections |
| Dynamic Memory | malloc/free for student nodes |
| File I/O | fopen, fprintf, fscanf, fclose |
| Strings | Name handling, parsing |
| Arrays | Score storage |
| Functions | Modular code organization |
| Control Flow | Menu loop, switch statements |

### Grading Criteria (채점 기준)

| Criteria | Points |
|----------|--------|
| Correct structure design | 10 |
| CRUD operations working | 30 |
| File I/O functioning | 20 |
| Menu system usability | 15 |
| Statistics accuracy | 10 |
| Code quality and comments | 10 |
| Error handling | 5 |
| **Total** | **100** |

---

## Practice Exercises (연습 문제)

1. **Basic**: Implement the complete system following the suggested phases.

2. **Intermediate**: Add sorting functionality (by name, ID, or average).

3. **Advanced**: Implement undo/redo functionality for CRUD operations.

4. **Challenge**: Add support for multiple classes/sections.

5. **Extension**: Create a graphical histogram for grade distribution.

---

## Appendix: Complete Working Code (부록: 완전한 작동 코드)

The complete implementation combining all sections is available in the `examples/` folder as:
- `student_system_complete.c` - Single file version
- `student_system/` - Multi-file version

To compile and run:
```bash
# Single file version
gcc -o student_system student_system_complete.c
./student_system

# Multi-file version
cd student_system/
make
./student_system
```
