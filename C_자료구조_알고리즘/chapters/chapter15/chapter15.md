# Chapter 15. Graph (그래프)

> Understanding graph data structures, representations, and traversal algorithms.

---

## Graph Terminology (그래프 용어)

### Concept (개념)

A **Graph** (그래프) is a non-linear data structure consisting of vertices (정점) and edges (간선) that connect pairs of vertices.

**Key Terms (주요 용어):**
- **Vertex (Node)** (정점): A fundamental unit of a graph
- **Edge** (간선): A connection between two vertices
- **Adjacent** (인접): Two vertices are adjacent if connected by an edge
- **Degree** (차수): Number of edges connected to a vertex
- **Path** (경로): A sequence of vertices connected by edges
- **Cycle** (사이클): A path that starts and ends at the same vertex
- **Connected** (연결): A graph where every vertex is reachable from any other vertex

```c
#include <stdio.h>

/*
 * Graph Terminology Demonstration
 * 그래프 용어 시연
 */

int main(void) {
    printf("=== Graph Terminology (그래프 용어) ===\n\n");

    printf("Graph G = (V, E)\n");
    printf("  V = Set of Vertices (정점 집합)\n");
    printf("  E = Set of Edges (간선 집합)\n\n");

    printf("Example Graph:\n");
    printf("  V = {A, B, C, D, E}\n");
    printf("  E = {(A,B), (A,C), (B,C), (B,D), (C,E), (D,E)}\n\n");

    printf("Terminology:\n");
    printf("  - Vertex A is adjacent to B and C\n");
    printf("    (정점 A는 B, C와 인접)\n");
    printf("  - Degree of B = 3 (connected to A, C, D)\n");
    printf("    (B의 차수 = 3)\n");
    printf("  - Path A->B->D->E exists\n");
    printf("    (경로 A->B->D->E 존재)\n");
    printf("  - Cycle: A->B->C->A\n");
    printf("    (사이클: A->B->C->A)\n");

    return 0;
}
```

**Graph Visualization:**
```
        A -------- B
       / \        /|
      /   \      / |
     /     \    /  |
    C-------+--+   |
     \           \ |
      \           \|
       +---------- D
        \         /
         \       /
          \     /
           \   /
             E

Vertices (V): {A, B, C, D, E}
Edges (E): {(A,B), (A,C), (B,C), (B,D), (C,E), (D,E)}
```

### Example
> See `examples/` folder

---

## Directed vs Undirected Graph (방향 그래프 vs 무방향 그래프)

### Concept (개념)

**Undirected Graph** (무방향 그래프):
- Edges have no direction
- If (A, B) exists, both A->B and B->A are valid
- Used for: friendship networks, road maps (two-way)

**Directed Graph (Digraph)** (방향 그래프):
- Edges have direction (arrows)
- (A, B) means A->B only, not B->A
- Used for: web links, dependencies, one-way streets

**Additional Types:**
- **Weighted Graph** (가중 그래프): Edges have associated weights/costs
- **Complete Graph** (완전 그래프): Every vertex connects to every other vertex

```c
#include <stdio.h>
#include <stdbool.h>

#define MAX_VERTICES 5

/*
 * Demonstrating Directed vs Undirected Graphs
 * 방향 그래프와 무방향 그래프 비교
 */

// Undirected Graph (무방향 그래프)
typedef struct {
    int adj[MAX_VERTICES][MAX_VERTICES];
    int numVertices;
} UndirectedGraph;

// Directed Graph (방향 그래프)
typedef struct {
    int adj[MAX_VERTICES][MAX_VERTICES];
    int numVertices;
} DirectedGraph;

// Add edge to undirected graph (symmetric)
void addUndirectedEdge(UndirectedGraph* g, int src, int dest) {
    g->adj[src][dest] = 1;
    g->adj[dest][src] = 1;  // Symmetric (대칭)
}

// Add edge to directed graph (one-way)
void addDirectedEdge(DirectedGraph* g, int src, int dest) {
    g->adj[src][dest] = 1;  // Only src -> dest
}

void printMatrix(int adj[][MAX_VERTICES], int n, const char* name) {
    printf("%s:\n", name);
    printf("    ");
    for (int i = 0; i < n; i++) printf("%d ", i);
    printf("\n    ");
    for (int i = 0; i < n; i++) printf("--");
    printf("\n");

    for (int i = 0; i < n; i++) {
        printf("%d | ", i);
        for (int j = 0; j < n; j++) {
            printf("%d ", adj[i][j]);
        }
        printf("\n");
    }
    printf("\n");
}

int main(void) {
    printf("=== Directed vs Undirected Graph ===\n\n");

    // Undirected Graph (무방향 그래프)
    UndirectedGraph ug = {{{0}}, 5};
    addUndirectedEdge(&ug, 0, 1);  // 0 -- 1
    addUndirectedEdge(&ug, 0, 2);  // 0 -- 2
    addUndirectedEdge(&ug, 1, 2);  // 1 -- 2
    addUndirectedEdge(&ug, 1, 3);  // 1 -- 3
    addUndirectedEdge(&ug, 3, 4);  // 3 -- 4

    printf("Undirected Graph (무방향 그래프):\n");
    printf("  0 --- 1 --- 3\n");
    printf("  |\\   /|     |\n");
    printf("  | \\ / |     |\n");
    printf("  |  X  |     |\n");
    printf("  | / \\ |     |\n");
    printf("  |/   \\|     |\n");
    printf("  2     +---- 4\n\n");

    printMatrix(ug.adj, 5, "Adjacency Matrix (Undirected)");

    // Directed Graph (방향 그래프)
    DirectedGraph dg = {{{0}}, 5};
    addDirectedEdge(&dg, 0, 1);  // 0 -> 1
    addDirectedEdge(&dg, 0, 2);  // 0 -> 2
    addDirectedEdge(&dg, 1, 2);  // 1 -> 2
    addDirectedEdge(&dg, 2, 3);  // 2 -> 3
    addDirectedEdge(&dg, 3, 4);  // 3 -> 4
    addDirectedEdge(&dg, 4, 1);  // 4 -> 1 (creates cycle)

    printf("Directed Graph (방향 그래프):\n");
    printf("  0 ---> 1 <---+\n");
    printf("  |      |     |\n");
    printf("  v      v     |\n");
    printf("  2 ---> 3 --> 4\n\n");

    printMatrix(dg.adj, 5, "Adjacency Matrix (Directed)");

    // Key differences
    printf("=== Key Differences (주요 차이점) ===\n");
    printf("| Feature         | Undirected | Directed  |\n");
    printf("|-----------------|------------|------------|\n");
    printf("| Matrix Symmetry | Yes        | No         |\n");
    printf("| Edge notation   | (u, v)     | <u, v>     |\n");
    printf("| Edge count      | |E|        | |E|        |\n");
    printf("| Max edges       | n(n-1)/2   | n(n-1)     |\n");

    return 0;
}
```

**Visual Comparison:**
```
Undirected Graph:                 Directed Graph:
      A ---- B                        A ----> B
      |      |                        |       |
      |      |                        v       v
      C ---- D                        C ----> D

Edge (A,B) means:                 Edge (A,B) means:
  A can reach B                     A can reach B
  B can reach A                     B cannot reach A (unless B->A exists)
```

### Example
> See `examples/` folder

---

## Adjacency Matrix Representation (인접 행렬 표현)

### Concept (개념)

An **Adjacency Matrix** (인접 행렬) is a 2D array where `adj[i][j] = 1` if there is an edge from vertex i to vertex j, and `0` otherwise.

**Characteristics:**
- Space Complexity: O(V^2)
- Edge lookup: O(1)
- Finding all neighbors: O(V)
- Best for: Dense graphs (many edges)

```c
#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>

#define MAX_VERTICES 10

/*
 * Graph Implementation using Adjacency Matrix
 * 인접 행렬을 이용한 그래프 구현
 */

typedef struct {
    int matrix[MAX_VERTICES][MAX_VERTICES];
    int numVertices;
    char* vertexNames[MAX_VERTICES];  // Optional: vertex labels
} Graph;

// Initialize graph (그래프 초기화)
void initGraph(Graph* g, int vertices) {
    g->numVertices = vertices;

    // Initialize all entries to 0
    for (int i = 0; i < vertices; i++) {
        for (int j = 0; j < vertices; j++) {
            g->matrix[i][j] = 0;
        }
        g->vertexNames[i] = NULL;
    }
}

// Add edge (간선 추가) - Undirected
void addEdge(Graph* g, int src, int dest) {
    if (src >= 0 && src < g->numVertices &&
        dest >= 0 && dest < g->numVertices) {
        g->matrix[src][dest] = 1;
        g->matrix[dest][src] = 1;  // Undirected: symmetric
    }
}

// Add weighted edge (가중 간선 추가)
void addWeightedEdge(Graph* g, int src, int dest, int weight) {
    if (src >= 0 && src < g->numVertices &&
        dest >= 0 && dest < g->numVertices) {
        g->matrix[src][dest] = weight;
        g->matrix[dest][src] = weight;
    }
}

// Remove edge (간선 제거)
void removeEdge(Graph* g, int src, int dest) {
    if (src >= 0 && src < g->numVertices &&
        dest >= 0 && dest < g->numVertices) {
        g->matrix[src][dest] = 0;
        g->matrix[dest][src] = 0;
    }
}

// Check if edge exists (간선 존재 확인)
bool hasEdge(Graph* g, int src, int dest) {
    if (src >= 0 && src < g->numVertices &&
        dest >= 0 && dest < g->numVertices) {
        return g->matrix[src][dest] != 0;
    }
    return false;
}

// Get degree of vertex (정점의 차수)
int getDegree(Graph* g, int vertex) {
    int degree = 0;
    for (int i = 0; i < g->numVertices; i++) {
        if (g->matrix[vertex][i] != 0) {
            degree++;
        }
    }
    return degree;
}

// Print adjacency matrix (인접 행렬 출력)
void printGraph(Graph* g) {
    printf("Adjacency Matrix (인접 행렬):\n");
    printf("    ");
    for (int i = 0; i < g->numVertices; i++) {
        printf("%3d", i);
    }
    printf("\n    ");
    for (int i = 0; i < g->numVertices; i++) {
        printf("---");
    }
    printf("\n");

    for (int i = 0; i < g->numVertices; i++) {
        printf("%2d |", i);
        for (int j = 0; j < g->numVertices; j++) {
            printf("%3d", g->matrix[i][j]);
        }
        printf("\n");
    }
}

// Print neighbors of a vertex (정점의 이웃 출력)
void printNeighbors(Graph* g, int vertex) {
    printf("Neighbors of vertex %d: ", vertex);
    bool hasNeighbor = false;

    for (int i = 0; i < g->numVertices; i++) {
        if (g->matrix[vertex][i] != 0) {
            printf("%d ", i);
            hasNeighbor = true;
        }
    }

    if (!hasNeighbor) {
        printf("(none)");
    }
    printf("\n");
}

int main(void) {
    Graph g;
    initGraph(&g, 6);

    printf("=== Adjacency Matrix Representation ===\n");
    printf("=== 인접 행렬 표현 ===\n\n");

    // Create graph:
    //     0 --- 1
    //    /|\    |\
    //   / | \   | \
    //  2  |  3--+--4
    //      \   \|
    //       ----5

    addEdge(&g, 0, 1);
    addEdge(&g, 0, 2);
    addEdge(&g, 0, 3);
    addEdge(&g, 0, 5);
    addEdge(&g, 1, 3);
    addEdge(&g, 1, 4);
    addEdge(&g, 3, 4);
    addEdge(&g, 3, 5);

    printf("Graph Structure:\n");
    printf("    0 --- 1\n");
    printf("   /|\\    |\\\n");
    printf("  / | \\   | \\\n");
    printf(" 2  |  3--+--4\n");
    printf("     \\   \\|\n");
    printf("      ----5\n\n");

    printGraph(&g);

    printf("\n=== Graph Operations ===\n");

    // Check edges
    printf("\nEdge (0,1) exists: %s\n", hasEdge(&g, 0, 1) ? "Yes" : "No");
    printf("Edge (2,4) exists: %s\n", hasEdge(&g, 2, 4) ? "Yes" : "No");

    // Degrees
    printf("\nVertex degrees (정점 차수):\n");
    for (int i = 0; i < g.numVertices; i++) {
        printf("  Degree of %d: %d\n", i, getDegree(&g, i));
    }

    // Neighbors
    printf("\n");
    for (int i = 0; i < g.numVertices; i++) {
        printNeighbors(&g, i);
    }

    // Complexity Analysis
    printf("\n=== Complexity Analysis (복잡도 분석) ===\n");
    printf("| Operation        | Time     | Space    |\n");
    printf("|------------------|----------|----------|\n");
    printf("| Storage          | -        | O(V^2)   |\n");
    printf("| Add Edge         | O(1)     | -        |\n");
    printf("| Remove Edge      | O(1)     | -        |\n");
    printf("| Check Edge       | O(1)     | -        |\n");
    printf("| Find Neighbors   | O(V)     | -        |\n");
    printf("| Find All Edges   | O(V^2)   | -        |\n");

    return 0;
}
```

**Memory Layout Visualization:**
```
Adjacency Matrix for 5 vertices:

          Columns (Destination)
          0   1   2   3   4
        +---+---+---+---+---+
      0 | 0 | 1 | 1 | 0 | 0 |  <- Row 0: Vertex 0's connections
        +---+---+---+---+---+
Rows  1 | 1 | 0 | 1 | 1 | 0 |  <- Row 1: Vertex 1's connections
(Src) +---+---+---+---+---+
      2 | 1 | 1 | 0 | 0 | 1 |  <- Row 2: Vertex 2's connections
        +---+---+---+---+---+
      3 | 0 | 1 | 0 | 0 | 1 |  <- Row 3: Vertex 3's connections
        +---+---+---+---+---+
      4 | 0 | 0 | 1 | 1 | 0 |  <- Row 4: Vertex 4's connections
        +---+---+---+---+---+

matrix[i][j] = 1 means edge from i to j exists
matrix[i][j] = 0 means no edge from i to j
```

### Example
> See `examples/` folder

---

## Adjacency List Representation (인접 리스트 표현)

### Concept (개념)

An **Adjacency List** (인접 리스트) represents a graph as an array of linked lists. Each vertex has a list of its adjacent vertices.

**Characteristics:**
- Space Complexity: O(V + E)
- Adding an edge: O(1)
- Finding all neighbors: O(degree)
- Checking if edge exists: O(degree)
- Best for: Sparse graphs (few edges)

```c
#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>

/*
 * Graph Implementation using Adjacency List
 * 인접 리스트를 이용한 그래프 구현
 */

// Node for adjacency list (인접 리스트 노드)
typedef struct AdjListNode {
    int dest;
    int weight;  // For weighted graphs
    struct AdjListNode* next;
} AdjListNode;

// Adjacency list (인접 리스트)
typedef struct {
    AdjListNode* head;
} AdjList;

// Graph structure (그래프 구조체)
typedef struct {
    int numVertices;
    AdjList* array;
} Graph;

// Create new adjacency list node
AdjListNode* createNode(int dest, int weight) {
    AdjListNode* newNode = (AdjListNode*)malloc(sizeof(AdjListNode));
    newNode->dest = dest;
    newNode->weight = weight;
    newNode->next = NULL;
    return newNode;
}

// Create a graph (그래프 생성)
Graph* createGraph(int vertices) {
    Graph* graph = (Graph*)malloc(sizeof(Graph));
    graph->numVertices = vertices;

    // Create array of adjacency lists
    graph->array = (AdjList*)malloc(vertices * sizeof(AdjList));

    // Initialize each list as empty
    for (int i = 0; i < vertices; i++) {
        graph->array[i].head = NULL;
    }

    return graph;
}

// Add edge (간선 추가) - Undirected
void addEdge(Graph* graph, int src, int dest, int weight) {
    // Add edge from src to dest
    AdjListNode* newNode = createNode(dest, weight);
    newNode->next = graph->array[src].head;
    graph->array[src].head = newNode;

    // Add edge from dest to src (undirected)
    newNode = createNode(src, weight);
    newNode->next = graph->array[dest].head;
    graph->array[dest].head = newNode;
}

// Add directed edge (방향 간선 추가)
void addDirectedEdge(Graph* graph, int src, int dest, int weight) {
    AdjListNode* newNode = createNode(dest, weight);
    newNode->next = graph->array[src].head;
    graph->array[src].head = newNode;
}

// Check if edge exists (간선 존재 확인)
bool hasEdge(Graph* graph, int src, int dest) {
    AdjListNode* current = graph->array[src].head;
    while (current != NULL) {
        if (current->dest == dest) {
            return true;
        }
        current = current->next;
    }
    return false;
}

// Get degree of vertex (정점의 차수)
int getDegree(Graph* graph, int vertex) {
    int degree = 0;
    AdjListNode* current = graph->array[vertex].head;
    while (current != NULL) {
        degree++;
        current = current->next;
    }
    return degree;
}

// Print the graph (그래프 출력)
void printGraph(Graph* graph) {
    printf("Adjacency List (인접 리스트):\n");
    printf("============================\n");

    for (int v = 0; v < graph->numVertices; v++) {
        printf("Vertex %d:", v);

        AdjListNode* current = graph->array[v].head;
        while (current != NULL) {
            printf(" -> [%d", current->dest);
            if (current->weight != 1) {
                printf(",w=%d", current->weight);
            }
            printf("]");
            current = current->next;
        }
        printf(" -> NULL\n");
    }
}

// Free graph memory (그래프 메모리 해제)
void freeGraph(Graph* graph) {
    for (int v = 0; v < graph->numVertices; v++) {
        AdjListNode* current = graph->array[v].head;
        while (current != NULL) {
            AdjListNode* temp = current;
            current = current->next;
            free(temp);
        }
    }
    free(graph->array);
    free(graph);
}

int main(void) {
    printf("=== Adjacency List Representation ===\n");
    printf("=== 인접 리스트 표현 ===\n\n");

    // Create graph with 6 vertices
    Graph* graph = createGraph(6);

    // Add edges (weight = 1 for unweighted)
    addEdge(graph, 0, 1, 1);
    addEdge(graph, 0, 2, 1);
    addEdge(graph, 1, 2, 1);
    addEdge(graph, 1, 3, 1);
    addEdge(graph, 2, 4, 1);
    addEdge(graph, 3, 4, 1);
    addEdge(graph, 3, 5, 1);
    addEdge(graph, 4, 5, 1);

    printf("Graph Structure:\n");
    printf("    0 --- 1\n");
    printf("    |\\   /|\n");
    printf("    | \\ / |\n");
    printf("    |  X  |\n");
    printf("    | / \\ |\n");
    printf("    |/   \\|\n");
    printf("    2     3\n");
    printf("    |\\   /|\n");
    printf("    | \\ / |\n");
    printf("    |  X  |\n");
    printf("    | / \\ |\n");
    printf("    |/   \\|\n");
    printf("    4 --- 5\n\n");

    printGraph(graph);

    // Demonstrate operations
    printf("\n=== Graph Operations ===\n");

    printf("\nEdge (0,1) exists: %s\n", hasEdge(graph, 0, 1) ? "Yes" : "No");
    printf("Edge (0,5) exists: %s\n", hasEdge(graph, 0, 5) ? "Yes" : "No");

    printf("\nVertex degrees:\n");
    for (int i = 0; i < graph->numVertices; i++) {
        printf("  Degree(%d) = %d\n", i, getDegree(graph, i));
    }

    // Create weighted graph
    printf("\n=== Weighted Graph Example ===\n\n");

    Graph* wgraph = createGraph(4);
    addEdge(wgraph, 0, 1, 4);
    addEdge(wgraph, 0, 2, 2);
    addEdge(wgraph, 1, 2, 1);
    addEdge(wgraph, 1, 3, 5);
    addEdge(wgraph, 2, 3, 3);

    printGraph(wgraph);

    // Complexity comparison
    printf("\n=== Adjacency Matrix vs List (복잡도 비교) ===\n");
    printf("| Operation      | Matrix  | List        |\n");
    printf("|----------------|---------|-------------|\n");
    printf("| Space          | O(V^2)  | O(V + E)    |\n");
    printf("| Add Edge       | O(1)    | O(1)        |\n");
    printf("| Remove Edge    | O(1)    | O(E/V)      |\n");
    printf("| Check Edge     | O(1)    | O(E/V)      |\n");
    printf("| Get Neighbors  | O(V)    | O(degree)   |\n");
    printf("|----------------|---------|-------------|\n");
    printf("| Best for       | Dense   | Sparse      |\n");

    freeGraph(graph);
    freeGraph(wgraph);

    return 0;
}
```

**Memory Layout Visualization:**
```
Adjacency List Representation:

array[]
+---+
| 0 | --> [1] --> [2] --> NULL
+---+
| 1 | --> [0] --> [2] --> [3] --> NULL
+---+
| 2 | --> [0] --> [1] --> [4] --> NULL
+---+
| 3 | --> [1] --> [4] --> [5] --> NULL
+---+
| 4 | --> [2] --> [3] --> [5] --> NULL
+---+
| 5 | --> [3] --> [4] --> NULL
+---+

Each array element points to a linked list of adjacent vertices
각 배열 요소는 인접 정점의 연결 리스트를 가리킴
```

### Example
> See `examples/` folder

---

## Depth-First Search (DFS) (깊이 우선 탐색)

### Concept (개념)

**Depth-First Search (DFS)** (깊이 우선 탐색) explores as far as possible along each branch before backtracking. It uses a stack (or recursion) to keep track of vertices to visit.

**Characteristics:**
- Explores depth before breadth
- Uses stack (explicit or call stack)
- Time Complexity: O(V + E)
- Space Complexity: O(V) for visited array

**Applications:**
- Detecting cycles
- Topological sorting
- Finding connected components
- Solving mazes
- Path finding

```c
#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>

#define MAX_VERTICES 10

/*
 * Depth-First Search (DFS) Implementation
 * 깊이 우선 탐색 구현
 */

// Graph using adjacency list
typedef struct Node {
    int vertex;
    struct Node* next;
} Node;

typedef struct {
    Node* head[MAX_VERTICES];
    int numVertices;
} Graph;

// Stack for iterative DFS
typedef struct {
    int items[MAX_VERTICES];
    int top;
} Stack;

void initStack(Stack* s) { s->top = -1; }
bool isEmpty(Stack* s) { return s->top == -1; }
void push(Stack* s, int item) { s->items[++s->top] = item; }
int pop(Stack* s) { return s->items[s->top--]; }

// Create graph
Graph* createGraph(int vertices) {
    Graph* g = (Graph*)malloc(sizeof(Graph));
    g->numVertices = vertices;
    for (int i = 0; i < vertices; i++) {
        g->head[i] = NULL;
    }
    return g;
}

// Add edge
void addEdge(Graph* g, int src, int dest) {
    // Add src -> dest
    Node* newNode = (Node*)malloc(sizeof(Node));
    newNode->vertex = dest;
    newNode->next = g->head[src];
    g->head[src] = newNode;

    // Add dest -> src (undirected)
    newNode = (Node*)malloc(sizeof(Node));
    newNode->vertex = src;
    newNode->next = g->head[dest];
    g->head[dest] = newNode;
}

// DFS Recursive (재귀적 DFS)
void dfsRecursive(Graph* g, int vertex, bool visited[]) {
    // Mark current vertex as visited
    visited[vertex] = true;
    printf("%d ", vertex);

    // Visit all adjacent vertices
    Node* current = g->head[vertex];
    while (current != NULL) {
        if (!visited[current->vertex]) {
            dfsRecursive(g, current->vertex, visited);
        }
        current = current->next;
    }
}

// DFS Iterative using Stack (반복적 DFS)
void dfsIterative(Graph* g, int start) {
    bool visited[MAX_VERTICES] = {false};
    Stack stack;
    initStack(&stack);

    push(&stack, start);

    printf("DFS Iterative starting from %d: ", start);

    while (!isEmpty(&stack)) {
        int vertex = pop(&stack);

        if (!visited[vertex]) {
            visited[vertex] = true;
            printf("%d ", vertex);

            // Push all adjacent unvisited vertices
            Node* current = g->head[vertex];
            while (current != NULL) {
                if (!visited[current->vertex]) {
                    push(&stack, current->vertex);
                }
                current = current->next;
            }
        }
    }
    printf("\n");
}

// DFS to find path between two vertices
bool dfsPath(Graph* g, int start, int end, bool visited[], int path[], int* pathLen) {
    visited[start] = true;
    path[(*pathLen)++] = start;

    if (start == end) {
        return true;
    }

    Node* current = g->head[start];
    while (current != NULL) {
        if (!visited[current->vertex]) {
            if (dfsPath(g, current->vertex, end, visited, path, pathLen)) {
                return true;
            }
        }
        current = current->next;
    }

    // Backtrack (백트래킹)
    (*pathLen)--;
    return false;
}

// DFS to detect cycle (undirected graph)
bool hasCycleDFS(Graph* g, int vertex, bool visited[], int parent) {
    visited[vertex] = true;

    Node* current = g->head[vertex];
    while (current != NULL) {
        if (!visited[current->vertex]) {
            if (hasCycleDFS(g, current->vertex, visited, vertex)) {
                return true;
            }
        } else if (current->vertex != parent) {
            // Found a back edge (사이클 발견)
            return true;
        }
        current = current->next;
    }
    return false;
}

void freeGraph(Graph* g) {
    for (int i = 0; i < g->numVertices; i++) {
        Node* current = g->head[i];
        while (current != NULL) {
            Node* temp = current;
            current = current->next;
            free(temp);
        }
    }
    free(g);
}

int main(void) {
    printf("=== Depth-First Search (DFS) ===\n");
    printf("=== 깊이 우선 탐색 ===\n\n");

    /*
     * Graph structure:
     *     0 --- 1 --- 2
     *     |     |     |
     *     3 --- 4 --- 5
     *           |
     *           6
     */

    Graph* g = createGraph(7);
    addEdge(g, 0, 1);
    addEdge(g, 0, 3);
    addEdge(g, 1, 2);
    addEdge(g, 1, 4);
    addEdge(g, 2, 5);
    addEdge(g, 3, 4);
    addEdge(g, 4, 5);
    addEdge(g, 4, 6);

    printf("Graph Structure:\n");
    printf("    0 --- 1 --- 2\n");
    printf("    |     |     |\n");
    printf("    3 --- 4 --- 5\n");
    printf("          |\n");
    printf("          6\n\n");

    // Recursive DFS
    bool visited[MAX_VERTICES] = {false};
    printf("DFS Recursive starting from 0: ");
    dfsRecursive(g, 0, visited);
    printf("\n\n");

    // Iterative DFS
    dfsIterative(g, 0);
    printf("\n");

    // Find path
    bool visitedPath[MAX_VERTICES] = {false};
    int path[MAX_VERTICES];
    int pathLen = 0;

    printf("Finding path from 0 to 6:\n");
    if (dfsPath(g, 0, 6, visitedPath, path, &pathLen)) {
        printf("Path found: ");
        for (int i = 0; i < pathLen; i++) {
            printf("%d", path[i]);
            if (i < pathLen - 1) printf(" -> ");
        }
        printf("\n\n");
    }

    // Detect cycle
    bool visitedCycle[MAX_VERTICES] = {false};
    printf("Has cycle: %s\n\n", hasCycleDFS(g, 0, visitedCycle, -1) ? "Yes" : "No");

    // DFS Visualization
    printf("=== DFS Traversal Visualization ===\n");
    printf("Starting from vertex 0:\n\n");
    printf("Step  Stack            Visited        Current\n");
    printf("----  ---------------  -------------  -------\n");
    printf("  1   [0]              {}             -\n");
    printf("  2   []               {0}            0\n");
    printf("  3   [3,1]            {0}            0->push neighbors\n");
    printf("  4   [3]              {0,1}          1\n");
    printf("  5   [3,4,2]          {0,1}          1->push neighbors\n");
    printf("  6   [3,4]            {0,1,2}        2\n");
    printf("  ... (continues)\n");

    freeGraph(g);

    return 0;
}
```

**DFS Visualization:**
```
Graph:
    0 --- 1 --- 2
    |     |     |
    3 --- 4 --- 5
          |
          6

DFS from vertex 0:

Step 1: Visit 0          Step 2: Visit 1          Step 3: Visit 2
   [0]                      [0]-[1]                  [0]-[1]-[2]
    *                        *   *                    *   *   *

Step 4: Visit 5          Step 5: Visit 4          Step 6: Visit 6
   [0]-[1]-[2]              [0]-[1]-[2]              [0]-[1]-[2]
    *   *   *                *   *   *                *   *   *
            |                    |   |                    |   |
            [5]                 [4]-[5]                  [4]-[5]
             *                   *   *                    *   *
                                                         |
                                                        [6]
                                                         *

Step 7: Backtrack to 4, then 3
   [0]-[1]-[2]
    *   *   *
    |   |   |
   [3]-[4]-[5]
    *   *   *
        |
       [6]
        *

Order: 0 -> 1 -> 2 -> 5 -> 4 -> 6 -> 3 (may vary based on adjacency order)
```

### Example
> See `examples/` folder

---

## Breadth-First Search (BFS) (너비 우선 탐색)

### Concept (개념)

**Breadth-First Search (BFS)** (너비 우선 탐색) explores all vertices at the current depth before moving to the next level. It uses a queue to keep track of vertices to visit.

**Characteristics:**
- Explores breadth before depth
- Uses queue
- Time Complexity: O(V + E)
- Space Complexity: O(V) for visited array and queue

**Applications:**
- Finding shortest path (unweighted graph)
- Level-order traversal
- Finding all nodes within one connected component
- Testing bipartiteness

```c
#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>

#define MAX_VERTICES 10
#define INF 999999

/*
 * Breadth-First Search (BFS) Implementation
 * 너비 우선 탐색 구현
 */

// Graph using adjacency list
typedef struct Node {
    int vertex;
    struct Node* next;
} Node;

typedef struct {
    Node* head[MAX_VERTICES];
    int numVertices;
} Graph;

// Queue for BFS
typedef struct {
    int items[MAX_VERTICES];
    int front, rear;
} Queue;

void initQueue(Queue* q) { q->front = q->rear = -1; }
bool isQueueEmpty(Queue* q) { return q->front == -1; }

void enqueue(Queue* q, int item) {
    if (q->rear == MAX_VERTICES - 1) return;
    if (q->front == -1) q->front = 0;
    q->items[++q->rear] = item;
}

int dequeue(Queue* q) {
    if (isQueueEmpty(q)) return -1;
    int item = q->items[q->front];
    if (q->front >= q->rear) {
        q->front = q->rear = -1;
    } else {
        q->front++;
    }
    return item;
}

// Create graph
Graph* createGraph(int vertices) {
    Graph* g = (Graph*)malloc(sizeof(Graph));
    g->numVertices = vertices;
    for (int i = 0; i < vertices; i++) {
        g->head[i] = NULL;
    }
    return g;
}

// Add edge (undirected)
void addEdge(Graph* g, int src, int dest) {
    Node* newNode = (Node*)malloc(sizeof(Node));
    newNode->vertex = dest;
    newNode->next = g->head[src];
    g->head[src] = newNode;

    newNode = (Node*)malloc(sizeof(Node));
    newNode->vertex = src;
    newNode->next = g->head[dest];
    g->head[dest] = newNode;
}

// BFS Traversal (BFS 탐색)
void bfs(Graph* g, int start) {
    bool visited[MAX_VERTICES] = {false};
    Queue queue;
    initQueue(&queue);

    visited[start] = true;
    enqueue(&queue, start);

    printf("BFS starting from %d: ", start);

    while (!isQueueEmpty(&queue)) {
        int vertex = dequeue(&queue);
        printf("%d ", vertex);

        // Visit all adjacent vertices
        Node* current = g->head[vertex];
        while (current != NULL) {
            if (!visited[current->vertex]) {
                visited[current->vertex] = true;
                enqueue(&queue, current->vertex);
            }
            current = current->next;
        }
    }
    printf("\n");
}

// BFS with level tracking (레벨별 BFS)
void bfsWithLevels(Graph* g, int start) {
    bool visited[MAX_VERTICES] = {false};
    int level[MAX_VERTICES];

    for (int i = 0; i < g->numVertices; i++) {
        level[i] = -1;
    }

    Queue queue;
    initQueue(&queue);

    visited[start] = true;
    level[start] = 0;
    enqueue(&queue, start);

    printf("BFS with levels from %d:\n", start);

    int currentLevel = 0;
    printf("Level %d: ", currentLevel);

    while (!isQueueEmpty(&queue)) {
        int vertex = dequeue(&queue);

        if (level[vertex] > currentLevel) {
            currentLevel = level[vertex];
            printf("\nLevel %d: ", currentLevel);
        }

        printf("%d ", vertex);

        Node* current = g->head[vertex];
        while (current != NULL) {
            if (!visited[current->vertex]) {
                visited[current->vertex] = true;
                level[current->vertex] = level[vertex] + 1;
                enqueue(&queue, current->vertex);
            }
            current = current->next;
        }
    }
    printf("\n");
}

// BFS Shortest Path (최단 경로)
void bfsShortestPath(Graph* g, int start, int end) {
    bool visited[MAX_VERTICES] = {false};
    int parent[MAX_VERTICES];
    int distance[MAX_VERTICES];

    for (int i = 0; i < g->numVertices; i++) {
        parent[i] = -1;
        distance[i] = INF;
    }

    Queue queue;
    initQueue(&queue);

    visited[start] = true;
    distance[start] = 0;
    enqueue(&queue, start);

    while (!isQueueEmpty(&queue)) {
        int vertex = dequeue(&queue);

        if (vertex == end) break;

        Node* current = g->head[vertex];
        while (current != NULL) {
            if (!visited[current->vertex]) {
                visited[current->vertex] = true;
                parent[current->vertex] = vertex;
                distance[current->vertex] = distance[vertex] + 1;
                enqueue(&queue, current->vertex);
            }
            current = current->next;
        }
    }

    printf("\nShortest path from %d to %d:\n", start, end);

    if (distance[end] == INF) {
        printf("No path exists!\n");
        return;
    }

    printf("Distance: %d\n", distance[end]);
    printf("Path: ");

    // Reconstruct path (경로 재구성)
    int path[MAX_VERTICES];
    int pathLen = 0;
    int current = end;

    while (current != -1) {
        path[pathLen++] = current;
        current = parent[current];
    }

    // Print path in reverse
    for (int i = pathLen - 1; i >= 0; i--) {
        printf("%d", path[i]);
        if (i > 0) printf(" -> ");
    }
    printf("\n");
}

void freeGraph(Graph* g) {
    for (int i = 0; i < g->numVertices; i++) {
        Node* current = g->head[i];
        while (current != NULL) {
            Node* temp = current;
            current = current->next;
            free(temp);
        }
    }
    free(g);
}

int main(void) {
    printf("=== Breadth-First Search (BFS) ===\n");
    printf("=== 너비 우선 탐색 ===\n\n");

    /*
     * Graph structure:
     *         0
     *        /|\
     *       1 2 3
     *      /| | |\
     *     4 5 6 7 8
     */

    Graph* g = createGraph(9);
    addEdge(g, 0, 1);
    addEdge(g, 0, 2);
    addEdge(g, 0, 3);
    addEdge(g, 1, 4);
    addEdge(g, 1, 5);
    addEdge(g, 2, 6);
    addEdge(g, 3, 7);
    addEdge(g, 3, 8);

    printf("Graph Structure:\n");
    printf("        0\n");
    printf("       /|\\\n");
    printf("      1 2 3\n");
    printf("     /| | |\\\n");
    printf("    4 5 6 7 8\n\n");

    // Basic BFS
    bfs(g, 0);
    printf("\n");

    // BFS with levels
    bfsWithLevels(g, 0);
    printf("\n");

    // Shortest path
    bfsShortestPath(g, 4, 8);

    // DFS vs BFS comparison
    printf("\n=== DFS vs BFS Comparison ===\n");
    printf("| Feature          | DFS              | BFS              |\n");
    printf("|------------------|------------------|------------------|\n");
    printf("| Data Structure   | Stack            | Queue            |\n");
    printf("| Search Order     | Depth first      | Level by level   |\n");
    printf("| Shortest Path    | Not guaranteed   | Guaranteed*      |\n");
    printf("| Memory           | O(V) height      | O(V) width       |\n");
    printf("| Implementation   | Easier (recursion)| Queue required  |\n");
    printf("* For unweighted graphs\n");

    // BFS Visualization
    printf("\n=== BFS Traversal Visualization ===\n");
    printf("Starting from vertex 0:\n\n");
    printf("Step  Queue              Visited              Current\n");
    printf("----  -----------------  -------------------  -------\n");
    printf("  1   [0]                {}                   -\n");
    printf("  2   []                 {0}                  0\n");
    printf("  3   [1,2,3]            {0}                  0->add neighbors\n");
    printf("  4   [2,3]              {0,1}                1\n");
    printf("  5   [2,3,4,5]          {0,1}                1->add neighbors\n");
    printf("  6   [3,4,5]            {0,1,2}              2\n");
    printf("  ... (continues level by level)\n");

    freeGraph(g);

    return 0;
}
```

**BFS Visualization:**
```
Graph:
        0
       /|\
      1 2 3
     /| | |\
    4 5 6 7 8

BFS from vertex 0:

Level 0:        Level 1:        Level 2:
    [0]             [0]             [0]
     *             / | \           / | \
                 [1][2][3]       [1][2][3]
                  *  *  *        /|  |  |\
                                [4][5][6][7][8]
                                 *  *  *  *  *

Order: 0 -> 1 -> 2 -> 3 -> 4 -> 5 -> 6 -> 7 -> 8

Queue States:
[0] -> dequeue 0, enqueue 1,2,3
[1,2,3] -> dequeue 1, enqueue 4,5
[2,3,4,5] -> dequeue 2, enqueue 6
[3,4,5,6] -> dequeue 3, enqueue 7,8
[4,5,6,7,8] -> dequeue remaining
```

### Example
> See `examples/` folder

---

## Summary (요약)

| Topic | Key Points |
|-------|------------|
| Graph Terms | Vertex (정점), Edge (간선), Adjacent (인접), Degree (차수), Path (경로), Cycle (사이클) |
| Directed Graph | Edges have direction, (A,B) means A->B only |
| Undirected Graph | Edges are bidirectional, (A,B) means A<->B |
| Adjacency Matrix | O(V^2) space, O(1) edge lookup, good for dense graphs |
| Adjacency List | O(V+E) space, O(degree) edge lookup, good for sparse graphs |
| DFS | Uses stack, explores depth first, good for detecting cycles |
| BFS | Uses queue, explores level by level, good for shortest path |

### Time Complexity Summary (시간 복잡도 요약)

| Operation | Adjacency Matrix | Adjacency List |
|-----------|------------------|----------------|
| Space | O(V^2) | O(V + E) |
| Add Edge | O(1) | O(1) |
| Remove Edge | O(1) | O(E) |
| Check Edge | O(1) | O(V) |
| DFS/BFS | O(V^2) | O(V + E) |
| Find All Neighbors | O(V) | O(degree) |

### When to Use (사용 시기)

| Use Case | Recommended |
|----------|-------------|
| Dense graph (many edges) | Adjacency Matrix |
| Sparse graph (few edges) | Adjacency List |
| Finding shortest path (unweighted) | BFS |
| Detecting cycles | DFS |
| Topological sorting | DFS |
| Connected components | DFS or BFS |

---

## Practice Exercises (연습 문제)

1. Implement a function to count the number of connected components in an undirected graph.
2. Write a function to check if a graph is bipartite using BFS.
3. Implement topological sort using DFS for a directed acyclic graph (DAG).
4. Create a function to find all paths between two vertices.
5. Implement Dijkstra's algorithm for finding shortest paths in a weighted graph.
