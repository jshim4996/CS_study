# Chapter 16. Hash Table (해시 테이블)

> Understanding hash functions, collision handling, and efficient key-value storage.

---

## Hash Function Concept (해시 함수 개념)

### Concept (개념)

A **Hash Function** (해시 함수) is a function that converts input data of arbitrary size into a fixed-size value (hash code). A **Hash Table** (해시 테이블) uses hash functions to map keys to array indices for efficient data storage and retrieval.

**Key Properties of Good Hash Functions:**
- **Deterministic** (결정적): Same input always produces same output
- **Uniform Distribution** (균등 분포): Minimizes collisions
- **Efficient** (효율적): Fast to compute
- **Avalanche Effect** (눈사태 효과): Small input change = big output change

**Common Hash Functions:**
1. Division Method (나눗셈법): `h(k) = k mod m`
2. Multiplication Method (곱셈법): `h(k) = floor(m * (k*A mod 1))`
3. Mid-Square Method (중간제곱법): Square key, take middle digits
4. Folding Method (접기법): Divide key into parts, combine

```c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>

#define TABLE_SIZE 10

/*
 * Hash Function Demonstrations
 * 해시 함수 시연
 */

// Division Method (나눗셈법)
// Simple and commonly used
int hashDivision(int key, int tableSize) {
    return key % tableSize;
}

// Multiplication Method (곱셈법)
// A is typically (sqrt(5) - 1) / 2 = 0.6180339887
int hashMultiplication(int key, int tableSize) {
    double A = 0.6180339887;
    double fractional = key * A - (int)(key * A);
    return (int)(tableSize * fractional);
}

// String Hash - djb2 algorithm
// One of the best string hash functions
unsigned long hashDJB2(const char* str) {
    unsigned long hash = 5381;
    int c;

    while ((c = *str++)) {
        hash = ((hash << 5) + hash) + c;  // hash * 33 + c
    }

    return hash;
}

// String Hash - Simple Sum
unsigned long hashSimpleSum(const char* str) {
    unsigned long hash = 0;
    while (*str) {
        hash += *str++;
    }
    return hash;
}

// String Hash - Polynomial Rolling Hash
unsigned long hashPolynomial(const char* str) {
    unsigned long hash = 0;
    unsigned long p = 31;  // Prime number
    unsigned long p_pow = 1;

    while (*str) {
        hash += (*str - 'a' + 1) * p_pow;
        p_pow *= p;
        str++;
    }

    return hash;
}

// Demonstrate hash distribution
void demonstrateDistribution(int tableSize) {
    int counts[TABLE_SIZE] = {0};

    printf("Distribution Test (1000 random keys):\n");
    printf("Table size: %d\n\n", tableSize);

    // Hash 1000 random keys
    for (int i = 0; i < 1000; i++) {
        int key = rand() % 10000;
        int index = hashDivision(key, tableSize);
        counts[index]++;
    }

    // Print distribution
    printf("Index  Count  Distribution\n");
    printf("-----  -----  ------------\n");

    for (int i = 0; i < tableSize; i++) {
        printf("  %d    %4d   ", i, counts[i]);
        for (int j = 0; j < counts[i] / 10; j++) {
            printf("*");
        }
        printf("\n");
    }
}

int main(void) {
    printf("=== Hash Function Concept (해시 함수 개념) ===\n\n");

    // Integer hashing
    printf("=== Integer Hash Functions ===\n\n");

    int keys[] = {12, 25, 37, 48, 52, 67, 89, 100};
    int numKeys = sizeof(keys) / sizeof(keys[0]);

    printf("Key    Division(%%10)  Multiplication\n");
    printf("----   -------------  --------------\n");

    for (int i = 0; i < numKeys; i++) {
        printf("%3d         %d              %d\n",
               keys[i],
               hashDivision(keys[i], TABLE_SIZE),
               hashMultiplication(keys[i], TABLE_SIZE));
    }

    // String hashing
    printf("\n=== String Hash Functions ===\n\n");

    const char* strings[] = {"apple", "banana", "cherry", "date", "elderberry"};
    int numStrings = sizeof(strings) / sizeof(strings[0]);

    printf("String       SimpleSum  DJB2 (%%100)  Polynomial (%%100)\n");
    printf("----------   ---------  -----------  -----------------\n");

    for (int i = 0; i < numStrings; i++) {
        printf("%-12s %6lu      %6lu        %6lu\n",
               strings[i],
               hashSimpleSum(strings[i]) % 100,
               hashDJB2(strings[i]) % 100,
               hashPolynomial(strings[i]) % 100);
    }

    // Show collision issue with simple sum
    printf("\n=== Collision Example (충돌 예시) ===\n");
    printf("'abc' simple hash: %lu\n", hashSimpleSum("abc"));
    printf("'bca' simple hash: %lu\n", hashSimpleSum("bca"));
    printf("'cab' simple hash: %lu\n", hashSimpleSum("cab"));
    printf("(All anagrams produce same hash with simple sum!)\n");

    printf("\n'abc' djb2 hash: %lu\n", hashDJB2("abc"));
    printf("'bca' djb2 hash: %lu\n", hashDJB2("bca"));
    printf("'cab' djb2 hash: %lu\n", hashDJB2("cab"));
    printf("(DJB2 produces different hashes for anagrams)\n");

    // Distribution test
    printf("\n=== Hash Distribution Test ===\n\n");
    demonstrateDistribution(TABLE_SIZE);

    // Visualization
    printf("\n=== Hash Table Visualization ===\n");
    printf("\n");
    printf("Key 'apple' ---> hash('apple') ---> 7 ---> Table[7]\n");
    printf("\n");
    printf("Hash Table (size=10):\n");
    printf("+-------+-------+-------+-------+-------+-------+-------+-------+-------+-------+\n");
    printf("|   0   |   1   |   2   |   3   |   4   |   5   |   6   |   7   |   8   |   9   |\n");
    printf("+-------+-------+-------+-------+-------+-------+-------+-------+-------+-------+\n");
    printf("|       |       | date  |       |cherry |       |banana | apple |       |       |\n");
    printf("+-------+-------+-------+-------+-------+-------+-------+-------+-------+-------+\n");

    return 0;
}
```

**Hash Function Visualization:**
```
Input Key                Hash Function              Index
-----------              -------------              -----

"apple"    ------>   h(key) = djb2("apple")  ---->   7
                          |
                          v
                    +---------------+
                    | Hash Function |
                    | h(k) = k % m  |
                    +---------------+
                          |
                          v
              +---+---+---+---+---+---+---+---+---+---+
Table Index:  | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 |
              +---+---+---+---+---+---+---+---+---+---+
                                              ^
                                              |
                                          "apple"
```

### Example
> See `examples/` folder

---

## Hash Collision (해시 충돌)

### Concept (개념)

A **Hash Collision** (해시 충돌) occurs when two different keys produce the same hash value (index). Since the hash space is finite but the key space is potentially infinite, collisions are inevitable.

**Why Collisions Occur:**
- Finite table size
- Infinite possible keys
- Pigeonhole principle (비둘기집 원리)

**Birthday Paradox:**
- With 23 people, there's a 50% chance two share a birthday
- Similarly, collisions occur earlier than expected in hash tables

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <stdbool.h>

#define TABLE_SIZE 10

/*
 * Hash Collision Demonstration
 * 해시 충돌 시연
 */

// Simple hash function
int simpleHash(int key) {
    return key % TABLE_SIZE;
}

// Demonstrate collisions
void demonstrateCollisions() {
    printf("=== Hash Collision Example ===\n\n");

    int keys[] = {12, 22, 32, 42, 52, 15, 25};
    int n = sizeof(keys) / sizeof(keys[0]);

    printf("Table Size: %d\n", TABLE_SIZE);
    printf("Hash Function: h(k) = k %% %d\n\n", TABLE_SIZE);

    printf("Key    Hash Value\n");
    printf("----   ----------\n");

    for (int i = 0; i < n; i++) {
        int hash = simpleHash(keys[i]);
        printf("%3d       %d", keys[i], hash);

        // Check for collision
        for (int j = 0; j < i; j++) {
            if (simpleHash(keys[j]) == hash) {
                printf("  <-- Collision with %d!", keys[j]);
                break;
            }
        }
        printf("\n");
    }

    printf("\nVisualization:\n");
    printf("\n");
    printf("Keys: 12, 22, 32 all hash to index 2!\n");
    printf("\n");
    printf("Index 2:\n");
    printf("   +-----+\n");
    printf("   |  ?  |  <-- What value should we store?\n");
    printf("   +-----+\n");
    printf("     ^  ^  ^\n");
    printf("     |  |  |\n");
    printf("    12 22 32  (All want this slot!)\n");
}

// Calculate collision probability
void collisionProbability() {
    printf("\n=== Collision Probability ===\n\n");

    printf("Load Factor (alpha) = n/m\n");
    printf("where n = number of elements, m = table size\n\n");

    printf("Load Factor   Expected Collisions   Performance\n");
    printf("-----------   -------------------   -----------\n");
    printf("   0.25            Low               Excellent\n");
    printf("   0.50            Medium            Good\n");
    printf("   0.75            High              Fair\n");
    printf("   1.00            Very High         Poor\n");
    printf("   > 1.0           Guaranteed        Very Poor\n\n");

    printf("Birthday Paradox Effect:\n");
    printf("  - With table size m=365, after ~23 insertions,\n");
    printf("    there's a 50%% chance of collision!\n");
}

int main(void) {
    printf("=== Hash Collision (해시 충돌) ===\n\n");

    demonstrateCollisions();
    collisionProbability();

    printf("\n=== Collision Resolution Strategies ===\n");
    printf("\n1. Chaining (체이닝)\n");
    printf("   - Each slot contains a linked list\n");
    printf("   - Multiple keys with same hash stored in list\n");
    printf("\n2. Open Addressing (개방 주소법)\n");
    printf("   - Linear Probing: Try next slot\n");
    printf("   - Quadratic Probing: Try i^2 slots away\n");
    printf("   - Double Hashing: Use second hash function\n");

    return 0;
}
```

**Collision Visualization:**
```
Without Collision Handling:

Key 12, 22, 32 all hash to index 2

Index:  0    1    2    3    4    5    6    7    8    9
      +----+----+----+----+----+----+----+----+----+----+
      |    |    | ?? |    |    |    |    |    |    |    |
      +----+----+----+----+----+----+----+----+----+----+
                  ^
                  |
            12? 22? 32?
           (COLLISION!)

Problem: Which value do we store?
         We need collision resolution!
```

### Example
> See `examples/` folder

---

## Chaining Method (체이닝 기법)

### Concept (개념)

**Chaining** (체이닝) resolves collisions by storing all elements that hash to the same index in a linked list. Each bucket (배열 슬롯) contains a pointer to a linked list of entries.

**Characteristics:**
- Simple to implement
- Can store more than m elements (m = table size)
- Performance degrades with longer chains
- Extra memory for pointers

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <stdbool.h>

#define TABLE_SIZE 10

/*
 * Hash Table with Chaining
 * 체이닝을 이용한 해시 테이블
 */

// Node for chaining (linked list node)
typedef struct Node {
    int key;
    int value;
    struct Node* next;
} Node;

// Hash table structure
typedef struct {
    Node* buckets[TABLE_SIZE];
    int size;
} HashTable;

// Create new node
Node* createNode(int key, int value) {
    Node* node = (Node*)malloc(sizeof(Node));
    node->key = key;
    node->value = value;
    node->next = NULL;
    return node;
}

// Initialize hash table
HashTable* createHashTable() {
    HashTable* table = (HashTable*)malloc(sizeof(HashTable));
    table->size = 0;

    for (int i = 0; i < TABLE_SIZE; i++) {
        table->buckets[i] = NULL;
    }

    return table;
}

// Hash function
int hash(int key) {
    return ((key % TABLE_SIZE) + TABLE_SIZE) % TABLE_SIZE;  // Handle negative keys
}

// Insert key-value pair (삽입)
void insert(HashTable* table, int key, int value) {
    int index = hash(key);
    Node* current = table->buckets[index];

    // Check if key already exists (update value)
    while (current != NULL) {
        if (current->key == key) {
            current->value = value;  // Update existing
            return;
        }
        current = current->next;
    }

    // Insert new node at the beginning of the chain
    Node* newNode = createNode(key, value);
    newNode->next = table->buckets[index];
    table->buckets[index] = newNode;
    table->size++;

    printf("Inserted (%d, %d) at index %d\n", key, value, index);
}

// Search for a key (탐색)
int* search(HashTable* table, int key) {
    int index = hash(key);
    Node* current = table->buckets[index];

    while (current != NULL) {
        if (current->key == key) {
            return &(current->value);
        }
        current = current->next;
    }

    return NULL;  // Not found
}

// Delete a key (삭제)
bool delete(HashTable* table, int key) {
    int index = hash(key);
    Node* current = table->buckets[index];
    Node* prev = NULL;

    while (current != NULL) {
        if (current->key == key) {
            if (prev == NULL) {
                table->buckets[index] = current->next;
            } else {
                prev->next = current->next;
            }
            free(current);
            table->size--;
            return true;
        }
        prev = current;
        current = current->next;
    }

    return false;  // Key not found
}

// Get chain length at index
int getChainLength(HashTable* table, int index) {
    int length = 0;
    Node* current = table->buckets[index];
    while (current != NULL) {
        length++;
        current = current->next;
    }
    return length;
}

// Print hash table
void printTable(HashTable* table) {
    printf("\n=== Hash Table Contents ===\n");

    for (int i = 0; i < TABLE_SIZE; i++) {
        printf("Bucket[%d]: ", i);

        Node* current = table->buckets[i];
        if (current == NULL) {
            printf("(empty)");
        }

        while (current != NULL) {
            printf("(%d:%d)", current->key, current->value);
            if (current->next != NULL) {
                printf(" -> ");
            }
            current = current->next;
        }
        printf("\n");
    }

    printf("\nTotal elements: %d\n", table->size);
    printf("Load factor: %.2f\n", (float)table->size / TABLE_SIZE);
}

// Free hash table
void freeTable(HashTable* table) {
    for (int i = 0; i < TABLE_SIZE; i++) {
        Node* current = table->buckets[i];
        while (current != NULL) {
            Node* temp = current;
            current = current->next;
            free(temp);
        }
    }
    free(table);
}

int main(void) {
    printf("=== Chaining Method (체이닝 기법) ===\n\n");

    HashTable* table = createHashTable();

    // Insert elements
    printf("=== Inserting Elements ===\n");
    insert(table, 12, 120);
    insert(table, 22, 220);  // Collision with 12
    insert(table, 32, 320);  // Collision with 12, 22
    insert(table, 15, 150);
    insert(table, 25, 250);  // Collision with 15
    insert(table, 7, 70);
    insert(table, 27, 270);  // Collision with 7

    printTable(table);

    // Search
    printf("\n=== Searching ===\n");
    int* result;

    result = search(table, 22);
    printf("Search 22: %s", result ? "Found" : "Not found");
    if (result) printf(" (value = %d)", *result);
    printf("\n");

    result = search(table, 99);
    printf("Search 99: %s\n", result ? "Found" : "Not found");

    // Delete
    printf("\n=== Deleting ===\n");
    printf("Delete 22: %s\n", delete(table, 22) ? "Success" : "Failed");
    printf("Delete 99: %s\n", delete(table, 99) ? "Success" : "Failed");

    printTable(table);

    // Chain length analysis
    printf("\n=== Chain Length Analysis ===\n");
    printf("Index  Chain Length\n");
    printf("-----  ------------\n");
    for (int i = 0; i < TABLE_SIZE; i++) {
        int len = getChainLength(table, i);
        printf("  %d         %d    ", i, len);
        for (int j = 0; j < len; j++) printf("*");
        printf("\n");
    }

    freeTable(table);

    return 0;
}
```

**Chaining Visualization:**
```
Hash Table with Chaining (size = 10):

Index    Bucket (Linked List)
-----    --------------------
  0      NULL
  1      NULL
  2      (12:120) -> (22:220) -> (32:320) -> NULL
  3      NULL
  4      NULL
  5      (15:150) -> (25:250) -> NULL
  6      NULL
  7      (7:70) -> (27:270) -> NULL
  8      NULL
  9      NULL

Memory Layout:
+-------+
|   0   | --> NULL
+-------+
|   1   | --> NULL
+-------+     +--------+     +--------+     +--------+
|   2   | --> | 12:120 | --> | 22:220 | --> | 32:320 | --> NULL
+-------+     +--------+     +--------+     +--------+
|   3   | --> NULL
+-------+
|   4   | --> NULL
+-------+     +--------+     +--------+
|   5   | --> | 15:150 | --> | 25:250 | --> NULL
+-------+     +--------+     +--------+
|   6   | --> NULL
+-------+     +--------+     +--------+
|   7   | --> |  7:70  | --> | 27:270 | --> NULL
+-------+     +--------+     +--------+
|   8   | --> NULL
+-------+
|   9   | --> NULL
+-------+
```

### Example
> See `examples/` folder

---

## Open Addressing Method (개방 주소법)

### Concept (개념)

**Open Addressing** (개방 주소법) resolves collisions by finding another empty slot within the hash table itself. All elements are stored directly in the table array.

**Probing Methods:**
1. **Linear Probing** (선형 탐사): `h(k, i) = (h(k) + i) mod m`
2. **Quadratic Probing** (이차 탐사): `h(k, i) = (h(k) + c1*i + c2*i^2) mod m`
3. **Double Hashing** (이중 해싱): `h(k, i) = (h1(k) + i*h2(k)) mod m`

**Characteristics:**
- No extra memory for pointers
- Better cache performance
- Table can get full (load factor <= 1)
- Clustering can occur (특히 선형 탐사)

```c
#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>

#define TABLE_SIZE 11  // Prime number recommended
#define EMPTY -1
#define DELETED -2

/*
 * Hash Table with Open Addressing
 * 개방 주소법을 이용한 해시 테이블
 */

typedef struct {
    int key;
    int value;
} Entry;

typedef struct {
    Entry* entries;
    int size;
    int count;
} HashTable;

// Create hash table
HashTable* createHashTable() {
    HashTable* table = (HashTable*)malloc(sizeof(HashTable));
    table->size = TABLE_SIZE;
    table->count = 0;
    table->entries = (Entry*)malloc(TABLE_SIZE * sizeof(Entry));

    for (int i = 0; i < TABLE_SIZE; i++) {
        table->entries[i].key = EMPTY;
        table->entries[i].value = 0;
    }

    return table;
}

// Primary hash function
int hash1(int key) {
    return ((key % TABLE_SIZE) + TABLE_SIZE) % TABLE_SIZE;
}

// Secondary hash function (for double hashing)
int hash2(int key) {
    return 7 - (key % 7);  // Should never return 0
}

// Linear Probing (선형 탐사)
int linearProbe(int key, int i) {
    return (hash1(key) + i) % TABLE_SIZE;
}

// Quadratic Probing (이차 탐사)
int quadraticProbe(int key, int i) {
    return (hash1(key) + i * i) % TABLE_SIZE;
}

// Double Hashing (이중 해싱)
int doubleHash(int key, int i) {
    return (hash1(key) + i * hash2(key)) % TABLE_SIZE;
}

// Insert with Linear Probing
bool insertLinear(HashTable* table, int key, int value) {
    if (table->count >= table->size) {
        printf("Table is full!\n");
        return false;
    }

    int i = 0;
    int index;

    do {
        index = linearProbe(key, i);

        if (table->entries[index].key == EMPTY ||
            table->entries[index].key == DELETED) {
            table->entries[index].key = key;
            table->entries[index].value = value;
            table->count++;
            printf("Inserted (%d, %d) at index %d (probes: %d)\n",
                   key, value, index, i + 1);
            return true;
        }

        if (table->entries[index].key == key) {
            table->entries[index].value = value;  // Update
            return true;
        }

        i++;
    } while (i < table->size);

    return false;
}

// Search with Linear Probing
int* searchLinear(HashTable* table, int key) {
    int i = 0;
    int index;

    do {
        index = linearProbe(key, i);

        if (table->entries[index].key == EMPTY) {
            return NULL;  // Key doesn't exist
        }

        if (table->entries[index].key == key) {
            return &(table->entries[index].value);
        }

        i++;
    } while (i < table->size);

    return NULL;
}

// Delete with Linear Probing (use DELETED marker)
bool deleteLinear(HashTable* table, int key) {
    int i = 0;
    int index;

    do {
        index = linearProbe(key, i);

        if (table->entries[index].key == EMPTY) {
            return false;
        }

        if (table->entries[index].key == key) {
            table->entries[index].key = DELETED;
            table->count--;
            return true;
        }

        i++;
    } while (i < table->size);

    return false;
}

// Print table
void printTable(HashTable* table) {
    printf("\nHash Table (Linear Probing):\n");
    printf("Index   Key    Value   Status\n");
    printf("-----   ----   -----   ------\n");

    for (int i = 0; i < table->size; i++) {
        printf("  %2d    ", i);
        if (table->entries[i].key == EMPTY) {
            printf("  -      -     EMPTY\n");
        } else if (table->entries[i].key == DELETED) {
            printf("  -      -     DELETED\n");
        } else {
            printf("%3d    %3d    OCCUPIED\n",
                   table->entries[i].key,
                   table->entries[i].value);
        }
    }

    printf("\nCount: %d, Size: %d, Load Factor: %.2f\n",
           table->count, table->size,
           (float)table->count / table->size);
}

// Free table
void freeTable(HashTable* table) {
    free(table->entries);
    free(table);
}

// Demonstrate clustering problem
void demonstrateClustering() {
    printf("\n=== Clustering Problem (클러스터링 문제) ===\n\n");

    printf("Linear Probing can cause 'Primary Clustering':\n\n");

    printf("Insert 12, 22, 32, 42, 52 (all hash to index 1):\n\n");

    printf("After insertions:\n");
    printf("Index: 0   1   2   3   4   5   6   7   8   9  10\n");
    printf("      +---+---+---+---+---+---+---+---+---+---+---+\n");
    printf("      |   | 12| 22| 32| 42| 52|   |   |   |   |   |\n");
    printf("      +---+---+---+---+---+---+---+---+---+---+---+\n");
    printf("            ^^^^^^^^^^^^^^^^^^^^^\n");
    printf("            CLUSTER FORMED!\n\n");

    printf("Problem: New keys hashing to 1-5 must probe through entire cluster\n");
    printf("Solution: Use Quadratic Probing or Double Hashing\n");
}

int main(void) {
    printf("=== Open Addressing Method (개방 주소법) ===\n\n");

    // Linear Probing Demo
    printf("=== Linear Probing Demo ===\n\n");

    HashTable* table = createHashTable();

    insertLinear(table, 12, 120);
    insertLinear(table, 22, 220);  // Collision
    insertLinear(table, 32, 320);  // Collision
    insertLinear(table, 15, 150);
    insertLinear(table, 26, 260);  // Collision
    insertLinear(table, 7, 70);

    printTable(table);

    // Search
    printf("\n=== Searching ===\n");
    int* result;

    result = searchLinear(table, 22);
    printf("Search 22: %s", result ? "Found" : "Not found");
    if (result) printf(" (value = %d)", *result);
    printf("\n");

    result = searchLinear(table, 99);
    printf("Search 99: %s\n", result ? "Found" : "Not found");

    // Delete
    printf("\n=== Deleting ===\n");
    printf("Delete 22: %s\n", deleteLinear(table, 22) ? "Success" : "Failed");

    printTable(table);

    // Show clustering
    demonstrateClustering();

    // Probing comparison
    printf("\n=== Probing Methods Comparison ===\n");
    printf("| Method          | Formula                    | Clustering   |\n");
    printf("|-----------------|----------------------------|-------------|\n");
    printf("| Linear          | h(k) + i                   | Primary      |\n");
    printf("| Quadratic       | h(k) + c1*i + c2*i^2       | Secondary    |\n");
    printf("| Double Hashing  | h1(k) + i*h2(k)            | Minimal      |\n");

    printf("\nProbing sequence for key=22 (h1=0, h2=5):\n");
    printf("Linear:    0, 1, 2, 3, 4, 5, ...\n");
    printf("Quadratic: 0, 1, 4, 9, 5, 3, ...\n");
    printf("Double:    0, 5, 10, 4, 9, 3, ...\n");

    freeTable(table);

    return 0;
}
```

**Open Addressing Visualization:**
```
Linear Probing:

Insert 12 (hash=1): Put at index 1
Insert 22 (hash=0): Put at index 0
Insert 32 (hash=10): Put at index 10
Insert 1 (hash=1): Index 1 taken, try 2, put at 2

Index: 0   1   2   3   4   5   6   7   8   9  10
      +---+---+---+---+---+---+---+---+---+---+---+
      | 22| 12|  1|   |   |   |   |   |   |   | 32|
      +---+---+---+---+---+---+---+---+---+---+---+
        ^   ^   ^                               ^
        |   |   |                               |
        |   |   +-- key 1 probed here          |
        |   +------ key 12                      |
        +---------- key 22                      |
                                    key 32 -----+

Quadratic Probing (for collision at index 0):
  Probe sequence: 0, 1, 4, 9, 5, 3, 1, ...
                  (0+0^2, 0+1^2, 0+2^2, 0+3^2, ...)

Double Hashing (h1=0, h2=3):
  Probe sequence: 0, 3, 6, 9, 1, 4, 7, 10, 2, 5, 8
                  (0+0*3, 0+1*3, 0+2*3, ...)
```

### Example
> See `examples/` folder

---

## Time Complexity Analysis (시간 복잡도 분석)

### Concept (개념)

Hash table performance depends on the **load factor** (적재율) and collision resolution strategy.

**Load Factor (alpha):** `alpha = n/m`
- n = number of elements
- m = table size

**Average Case:** O(1) for search, insert, delete
**Worst Case:** O(n) when all keys collide

```c
#include <stdio.h>
#include <stdlib.h>
#include <time.h>

#define TABLE_SIZE 1000
#define NUM_OPERATIONS 10000

/*
 * Hash Table Performance Analysis
 * 해시 테이블 성능 분석
 */

// Simple chaining hash table for testing
typedef struct Node {
    int key;
    struct Node* next;
} Node;

Node* table[TABLE_SIZE];
int comparisons = 0;

int hash(int key) {
    return ((key % TABLE_SIZE) + TABLE_SIZE) % TABLE_SIZE;
}

void insert(int key) {
    int index = hash(key);
    Node* newNode = (Node*)malloc(sizeof(Node));
    newNode->key = key;
    newNode->next = table[index];
    table[index] = newNode;
}

bool search(int key) {
    int index = hash(key);
    Node* current = table[index];

    while (current != NULL) {
        comparisons++;
        if (current->key == key) return true;
        current = current->next;
    }
    return false;
}

void clearTable() {
    for (int i = 0; i < TABLE_SIZE; i++) {
        Node* current = table[i];
        while (current != NULL) {
            Node* temp = current;
            current = current->next;
            free(temp);
        }
        table[i] = NULL;
    }
}

void analyzePerformance(int numElements) {
    clearTable();
    comparisons = 0;

    // Insert elements
    for (int i = 0; i < numElements; i++) {
        insert(rand() % 100000);
    }

    // Perform searches
    int numSearches = 1000;
    for (int i = 0; i < numSearches; i++) {
        search(rand() % 100000);
    }

    float loadFactor = (float)numElements / TABLE_SIZE;
    float avgComparisons = (float)comparisons / numSearches;

    printf("| %5d |   %.3f   |     %.2f      |\n",
           numElements, loadFactor, avgComparisons);
}

int main(void) {
    printf("=== Time Complexity Analysis (시간 복잡도 분석) ===\n\n");

    srand(time(NULL));

    // Theoretical Analysis
    printf("=== Theoretical Complexity ===\n\n");

    printf("| Operation | Average | Worst Case | Notes                  |\n");
    printf("|-----------|---------|------------|------------------------|\n");
    printf("| Search    | O(1)    | O(n)       | Depends on load factor |\n");
    printf("| Insert    | O(1)    | O(n)       | Amortized with resizing|\n");
    printf("| Delete    | O(1)    | O(n)       | Need to find first     |\n");

    printf("\n=== Load Factor Impact (적재율 영향) ===\n\n");

    printf("Load Factor (alpha) = n/m (elements/table_size)\n\n");

    printf("Chaining:\n");
    printf("  - Average search: O(1 + alpha)\n");
    printf("  - If alpha = 1, average chain length = 1\n");
    printf("  - Can exceed 1 (alpha > 1 is possible)\n\n");

    printf("Open Addressing:\n");
    printf("  - Average search: O(1/(1-alpha)) for linear probing\n");
    printf("  - Must have alpha < 1\n");
    printf("  - Typically keep alpha <= 0.7\n\n");

    // Empirical Analysis
    printf("=== Empirical Analysis (실험적 분석) ===\n\n");

    printf("Table Size: %d\n\n", TABLE_SIZE);
    printf("|   n   | Load Factor | Avg Comparisons |\n");
    printf("|-------|-------------|----------------|\n");

    int testSizes[] = {100, 250, 500, 750, 1000, 1500, 2000, 3000};
    int numTests = sizeof(testSizes) / sizeof(testSizes[0]);

    for (int i = 0; i < numTests; i++) {
        analyzePerformance(testSizes[i]);
    }

    // Performance Graph
    printf("\n=== Performance vs Load Factor ===\n\n");
    printf("Average Search Time:\n\n");
    printf("Time\n");
    printf("  ^                                      *\n");
    printf("  |                                   *\n");
    printf("  |                                *\n");
    printf("  |                             *\n");
    printf("  |                          *\n");
    printf("  |                      *\n");
    printf("  |                 *\n");
    printf("  |            *\n");
    printf("  |      *\n");
    printf("  |  *\n");
    printf("  +-----------------------------------------> Load Factor\n");
    printf("     0.1  0.2  0.3  0.4  0.5  0.6  0.7  0.8  0.9  1.0\n\n");
    printf("  As load factor increases, search time increases!\n");

    // Comparison Table
    printf("\n=== Collision Resolution Comparison ===\n\n");
    printf("| Metric           | Chaining       | Open Addressing |\n");
    printf("|------------------|----------------|----------------|\n");
    printf("| Space            | O(n + m)       | O(m)            |\n");
    printf("| Max Load Factor  | Unlimited      | < 1             |\n");
    printf("| Cache Performance| Poor           | Good            |\n");
    printf("| Deletion         | Easy           | Requires marker |\n");
    printf("| Implementation   | More complex   | Simpler         |\n");

    // Recommendations
    printf("\n=== Recommendations (권장 사항) ===\n\n");
    printf("1. Keep load factor below 0.75 (resize if needed)\n");
    printf("2. Use prime number for table size\n");
    printf("3. Choose good hash function for your data type\n");
    printf("4. Use chaining when deletion is frequent\n");
    printf("5. Use open addressing when memory is limited\n");

    clearTable();

    return 0;
}
```

**Performance Visualization:**
```
Hash Table Performance vs Load Factor:

Search Time (comparisons)
    ^
  8 |                                            *
  7 |                                        *
  6 |                                    *
  5 |                                *
  4 |                            *
  3 |                        *
  2 |                    *
  1 |     * * * * * * *
    +---------------------------------------------------> Load Factor
       0.1 0.2 0.3 0.4 0.5 0.6 0.7 0.8 0.9 1.0

Legend:
  * O(1) average case maintained until ~0.7 load factor
  * Performance degrades rapidly above 0.7
  * At load factor 1.0, chaining still O(1+alpha), open addressing fails


Comparison: Hash Table vs Other Data Structures

            | Search  | Insert  | Delete  | Space
------------|---------|---------|---------|-------
Hash Table  | O(1)*   | O(1)*   | O(1)*   | O(n)
Array       | O(n)    | O(n)    | O(n)    | O(n)
Sorted Array| O(log n)| O(n)    | O(n)    | O(n)
BST         | O(log n)| O(log n)| O(log n)| O(n)
Linked List | O(n)    | O(1)    | O(n)    | O(n)

* Average case with good hash function
```

### Example
> See `examples/` folder

---

## Complete Hash Table Implementation (완전한 해시 테이블 구현)

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <stdbool.h>

#define INITIAL_SIZE 16
#define LOAD_FACTOR_THRESHOLD 0.75

/*
 * Complete Hash Table Implementation with Dynamic Resizing
 * 동적 크기 조정이 가능한 완전한 해시 테이블 구현
 */

typedef struct Entry {
    char* key;
    int value;
    struct Entry* next;
} Entry;

typedef struct {
    Entry** buckets;
    int size;       // Current capacity
    int count;      // Number of elements
} HashMap;

// DJB2 hash function for strings
unsigned long hashString(const char* str) {
    unsigned long hash = 5381;
    int c;
    while ((c = *str++)) {
        hash = ((hash << 5) + hash) + c;
    }
    return hash;
}

// Create new entry
Entry* createEntry(const char* key, int value) {
    Entry* entry = (Entry*)malloc(sizeof(Entry));
    entry->key = strdup(key);
    entry->value = value;
    entry->next = NULL;
    return entry;
}

// Create hash map
HashMap* createHashMap() {
    HashMap* map = (HashMap*)malloc(sizeof(HashMap));
    map->size = INITIAL_SIZE;
    map->count = 0;
    map->buckets = (Entry**)calloc(INITIAL_SIZE, sizeof(Entry*));
    return map;
}

// Get index for key
int getIndex(HashMap* map, const char* key) {
    return hashString(key) % map->size;
}

// Resize hash map (rehashing)
void resize(HashMap* map) {
    int oldSize = map->size;
    Entry** oldBuckets = map->buckets;

    // Double the size
    map->size *= 2;
    map->buckets = (Entry**)calloc(map->size, sizeof(Entry*));
    map->count = 0;

    printf("Resizing: %d -> %d\n", oldSize, map->size);

    // Rehash all entries
    for (int i = 0; i < oldSize; i++) {
        Entry* current = oldBuckets[i];
        while (current != NULL) {
            Entry* next = current->next;

            // Reinsert into new bucket
            int newIndex = getIndex(map, current->key);
            current->next = map->buckets[newIndex];
            map->buckets[newIndex] = current;
            map->count++;

            current = next;
        }
    }

    free(oldBuckets);
}

// Put key-value pair
void put(HashMap* map, const char* key, int value) {
    // Check if resize needed
    if ((float)map->count / map->size >= LOAD_FACTOR_THRESHOLD) {
        resize(map);
    }

    int index = getIndex(map, key);
    Entry* current = map->buckets[index];

    // Check for existing key
    while (current != NULL) {
        if (strcmp(current->key, key) == 0) {
            current->value = value;  // Update
            return;
        }
        current = current->next;
    }

    // Insert new entry
    Entry* entry = createEntry(key, value);
    entry->next = map->buckets[index];
    map->buckets[index] = entry;
    map->count++;
}

// Get value for key
int* get(HashMap* map, const char* key) {
    int index = getIndex(map, key);
    Entry* current = map->buckets[index];

    while (current != NULL) {
        if (strcmp(current->key, key) == 0) {
            return &(current->value);
        }
        current = current->next;
    }

    return NULL;
}

// Check if key exists
bool containsKey(HashMap* map, const char* key) {
    return get(map, key) != NULL;
}

// Remove key
bool removeKey(HashMap* map, const char* key) {
    int index = getIndex(map, key);
    Entry* current = map->buckets[index];
    Entry* prev = NULL;

    while (current != NULL) {
        if (strcmp(current->key, key) == 0) {
            if (prev == NULL) {
                map->buckets[index] = current->next;
            } else {
                prev->next = current->next;
            }
            free(current->key);
            free(current);
            map->count--;
            return true;
        }
        prev = current;
        current = current->next;
    }

    return false;
}

// Print hash map
void printHashMap(HashMap* map) {
    printf("\n=== HashMap Contents ===\n");
    printf("Size: %d, Count: %d, Load: %.2f\n\n",
           map->size, map->count, (float)map->count / map->size);

    for (int i = 0; i < map->size; i++) {
        if (map->buckets[i] != NULL) {
            printf("[%2d]: ", i);
            Entry* current = map->buckets[i];
            while (current != NULL) {
                printf("('%s':%d)", current->key, current->value);
                if (current->next) printf(" -> ");
                current = current->next;
            }
            printf("\n");
        }
    }
}

// Free hash map
void freeHashMap(HashMap* map) {
    for (int i = 0; i < map->size; i++) {
        Entry* current = map->buckets[i];
        while (current != NULL) {
            Entry* next = current->next;
            free(current->key);
            free(current);
            current = next;
        }
    }
    free(map->buckets);
    free(map);
}

int main(void) {
    printf("=== Complete HashMap Implementation ===\n\n");

    HashMap* map = createHashMap();

    // Insert various key-value pairs
    printf("=== Inserting Elements ===\n");
    put(map, "apple", 100);
    put(map, "banana", 200);
    put(map, "cherry", 300);
    put(map, "date", 400);
    put(map, "elderberry", 500);
    put(map, "fig", 600);
    put(map, "grape", 700);
    put(map, "honeydew", 800);
    put(map, "kiwi", 900);
    put(map, "lemon", 1000);
    put(map, "mango", 1100);
    put(map, "nectarine", 1200);  // Should trigger resize
    put(map, "orange", 1300);

    printHashMap(map);

    // Get operations
    printf("\n=== Get Operations ===\n");
    int* value;

    value = get(map, "cherry");
    printf("get('cherry'): %d\n", value ? *value : -1);

    value = get(map, "unknown");
    printf("get('unknown'): %s\n", value ? "found" : "not found");

    // Contains
    printf("\n=== Contains Check ===\n");
    printf("containsKey('apple'): %s\n", containsKey(map, "apple") ? "true" : "false");
    printf("containsKey('pear'): %s\n", containsKey(map, "pear") ? "true" : "false");

    // Update
    printf("\n=== Update Operation ===\n");
    printf("Updating 'apple' to 999\n");
    put(map, "apple", 999);
    value = get(map, "apple");
    printf("get('apple'): %d\n", *value);

    // Remove
    printf("\n=== Remove Operation ===\n");
    printf("remove('banana'): %s\n", removeKey(map, "banana") ? "success" : "failed");
    printf("containsKey('banana'): %s\n", containsKey(map, "banana") ? "true" : "false");

    printHashMap(map);

    freeHashMap(map);

    return 0;
}
```

---

## Summary (요약)

| Topic | Key Points |
|-------|------------|
| Hash Function | Converts keys to array indices, should be deterministic and uniform |
| Hash Collision | When different keys produce same hash, inevitable due to pigeonhole principle |
| Chaining | Uses linked lists at each bucket, unlimited load factor |
| Open Addressing | Finds next empty slot, requires load factor < 1 |
| Time Complexity | Average O(1), Worst O(n), depends on load factor |

### Comparison Summary (비교 요약)

| Feature | Chaining | Open Addressing |
|---------|----------|-----------------|
| Implementation | Linked lists | Array probing |
| Load Factor | Can exceed 1 | Must be < 1 |
| Memory | Extra for pointers | Compact |
| Cache | Poor | Better |
| Deletion | Easy | Requires marker |
| Clustering | No | Yes (linear) |

### Hash Function Selection (해시 함수 선택)

| Data Type | Recommended Hash Function |
|-----------|---------------------------|
| Integer | Division method: k % m |
| String | DJB2 or polynomial rolling |
| Object | Combine member hashes |
| Float | Convert to bits, then hash |

---

## Practice Exercises (연습 문제)

1. Implement a hash table that stores string keys and string values.
2. Create a hash table with quadratic probing collision resolution.
3. Implement automatic resizing when load factor exceeds 0.75.
4. Write a function to find the most frequent element using a hash table.
5. Implement a hash set (only keys, no values) with union and intersection operations.
