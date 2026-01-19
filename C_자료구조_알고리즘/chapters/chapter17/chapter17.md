# Chapter 17. Dynamic Programming (동적 프로그래밍)

> Understanding dynamic programming concepts, techniques, and classic problem solutions.

---

## DP Concept (DP 개념)

### Concept (개념)

**Dynamic Programming (DP)** (동적 프로그래밍) is an algorithmic technique that solves complex problems by breaking them down into simpler subproblems. It stores the results of subproblems to avoid redundant computations.

**Key Characteristics:**
- **Optimal Substructure** (최적 부분 구조): Optimal solution contains optimal solutions to subproblems
- **Overlapping Subproblems** (중복 부분 문제): Same subproblems are solved multiple times

**DP vs Other Techniques:**
- **Divide and Conquer**: Subproblems don't overlap (e.g., Merge Sort)
- **Greedy**: Makes locally optimal choice, no backtracking
- **Dynamic Programming**: Stores results, considers all possibilities

**Two Approaches:**
1. **Top-Down (Memoization)**: Recursion + caching
2. **Bottom-Up (Tabulation)**: Iterative, build solution from smallest subproblems

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

/*
 * Dynamic Programming Introduction
 * 동적 프로그래밍 소개
 */

// Naive recursive Fibonacci (no DP)
// Time: O(2^n) - Very slow!
int fibNaive(int n) {
    if (n <= 1) return n;
    return fibNaive(n - 1) + fibNaive(n - 2);
}

// Count function calls for demonstration
long long callCount = 0;

int fibNaiveCount(int n) {
    callCount++;
    if (n <= 1) return n;
    return fibNaiveCount(n - 1) + fibNaiveCount(n - 2);
}

int main(void) {
    printf("=== Dynamic Programming Concept (DP 개념) ===\n\n");

    printf("What is Dynamic Programming?\n");
    printf("============================\n\n");

    printf("DP = Optimal Substructure + Overlapping Subproblems\n\n");

    printf("1. Optimal Substructure (최적 부분 구조):\n");
    printf("   - Problem can be solved optimally by combining\n");
    printf("     optimal solutions to its subproblems\n");
    printf("   - Example: Shortest path from A to C through B =\n");
    printf("              Shortest(A->B) + Shortest(B->C)\n\n");

    printf("2. Overlapping Subproblems (중복 부분 문제):\n");
    printf("   - Same subproblems are solved repeatedly\n");
    printf("   - DP stores results to avoid recomputation\n\n");

    // Demonstrate the problem with naive approach
    printf("=== Fibonacci: Problem with Naive Recursion ===\n\n");

    printf("fib(n) = fib(n-1) + fib(n-2)\n\n");

    printf("Call tree for fib(5):\n");
    printf("                    fib(5)\n");
    printf("                   /      \\\n");
    printf("               fib(4)      fib(3)\n");
    printf("              /     \\      /    \\\n");
    printf("          fib(3)   fib(2) fib(2) fib(1)\n");
    printf("          /   \\    /   \\\n");
    printf("      fib(2) fib(1) fib(1) fib(0)\n");
    printf("      /   \\\n");
    printf("  fib(1) fib(0)\n\n");

    printf("Notice: fib(3) called 2 times, fib(2) called 3 times!\n");
    printf("This is the 'overlapping subproblems' property.\n\n");

    // Show exponential growth
    printf("=== Exponential Growth of Naive Approach ===\n\n");
    printf("  n  | Function Calls | Result\n");
    printf("-----|----------------|--------\n");

    for (int n = 5; n <= 30; n += 5) {
        callCount = 0;
        int result = fibNaiveCount(n);
        printf(" %2d  | %14lld | %d\n", n, callCount, result);
    }

    printf("\nTime Complexity: O(2^n) - Exponential!\n");
    printf("For n=30, we make ~1 billion calls!\n\n");

    // DP approach preview
    printf("=== DP Solution Preview ===\n\n");
    printf("With Dynamic Programming:\n");
    printf("  - Store each fib(k) after computing it once\n");
    printf("  - Time: O(n), Space: O(n) or O(1)\n");
    printf("  - fib(30) computed with only 30 operations!\n\n");

    printf("Two DP Approaches:\n");
    printf("1. Memoization (Top-Down): Start from fib(n), cache results\n");
    printf("2. Tabulation (Bottom-Up): Build up from fib(0), fib(1)...\n");

    return 0;
}
```

**DP Problem Recognition:**
```
How to identify a DP problem?

1. "Find the minimum/maximum..."
2. "Count the number of ways..."
3. "Is it possible to..."
4. "Find the longest/shortest..."

Common DP Patterns:
+-------------------+------------------------+
| Pattern           | Example Problems       |
+-------------------+------------------------+
| Linear DP         | Fibonacci, Climbing    |
| Grid DP           | Unique Paths, Min Path |
| String DP         | LCS, Edit Distance     |
| Knapsack          | 0/1 Knapsack, Subset   |
| Interval DP       | Matrix Chain, Palindrome|
+-------------------+------------------------+
```

### Example
> See `examples/` folder

---

## Memoization - Top-Down Approach (메모이제이션 - 하향식 접근)

### Concept (개념)

**Memoization** (메모이제이션) is a top-down approach where we start with the original problem and recursively break it down. We store (cache) the results of subproblems to avoid redundant calculations.

**Characteristics:**
- Uses recursion
- Lazy evaluation (computes only needed subproblems)
- Natural problem decomposition
- May have stack overhead for deep recursion

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define MAX_N 100
#define NOT_COMPUTED -1

/*
 * Memoization (Top-Down DP)
 * 메모이제이션 (하향식 DP)
 */

// Memo array for caching results
long long memo[MAX_N];

// Initialize memo array
void initMemo() {
    for (int i = 0; i < MAX_N; i++) {
        memo[i] = NOT_COMPUTED;
    }
}

// Fibonacci with Memoization
// Time: O(n), Space: O(n)
long long fibMemo(int n) {
    // Base cases
    if (n <= 1) return n;

    // Check if already computed (cache hit)
    if (memo[n] != NOT_COMPUTED) {
        printf("  Cache hit for fib(%d) = %lld\n", n, memo[n]);
        return memo[n];
    }

    // Compute and store (cache miss)
    printf("  Computing fib(%d)...\n", n);
    memo[n] = fibMemo(n - 1) + fibMemo(n - 2);

    return memo[n];
}

// Climbing Stairs with Memoization
// Ways to climb n stairs taking 1 or 2 steps at a time
long long climbStairsMemo(int n, long long* dp) {
    if (n <= 2) return n;

    if (dp[n] != NOT_COMPUTED) {
        return dp[n];
    }

    dp[n] = climbStairsMemo(n - 1, dp) + climbStairsMemo(n - 2, dp);
    return dp[n];
}

// Minimum Cost Climbing Stairs
// cost[i] is cost to step on stair i
int minCostClimbingMemo(int* cost, int n, int* dp) {
    if (n < 0) return 0;
    if (n <= 1) return cost[n];

    if (dp[n] != NOT_COMPUTED) {
        return dp[n];
    }

    int cost1 = minCostClimbingMemo(cost, n - 1, dp);
    int cost2 = minCostClimbingMemo(cost, n - 2, dp);

    dp[n] = cost[n] + (cost1 < cost2 ? cost1 : cost2);
    return dp[n];
}

// Grid Path Count with Memoization
// Count paths from (0,0) to (m-1, n-1), moving only right or down
int gridPathsMemo(int m, int n, int dp[MAX_N][MAX_N]) {
    // Base cases
    if (m == 1 || n == 1) return 1;

    // Check cache
    if (dp[m][n] != NOT_COMPUTED) {
        return dp[m][n];
    }

    // Compute: paths from top + paths from left
    dp[m][n] = gridPathsMemo(m - 1, n, dp) + gridPathsMemo(m, n - 1, dp);
    return dp[m][n];
}

int main(void) {
    printf("=== Memoization (Top-Down DP) ===\n");
    printf("=== 메모이제이션 (하향식 DP) ===\n\n");

    // Fibonacci with Memoization
    printf("=== Fibonacci with Memoization ===\n\n");

    initMemo();
    printf("Computing fib(10):\n");
    long long result = fibMemo(10);
    printf("\nfib(10) = %lld\n\n", result);

    printf("Notice: Each fib(k) computed only ONCE!\n\n");

    // Show the memo array
    printf("Memo array after computation:\n");
    printf("Index: ");
    for (int i = 0; i <= 10; i++) printf("%4d ", i);
    printf("\nValue: ");
    for (int i = 0; i <= 10; i++) printf("%4lld ", memo[i]);
    printf("\n\n");

    // Climbing Stairs
    printf("=== Climbing Stairs Problem ===\n\n");
    printf("Problem: Count ways to climb n stairs,\n");
    printf("         taking 1 or 2 steps at a time.\n\n");

    long long climbDp[MAX_N];
    for (int i = 0; i < MAX_N; i++) climbDp[i] = NOT_COMPUTED;

    printf("n  | Ways\n");
    printf("---|------\n");
    for (int n = 1; n <= 10; n++) {
        printf("%2d | %lld\n", n, climbStairsMemo(n, climbDp));
    }

    // Grid Paths
    printf("\n=== Grid Paths Problem ===\n\n");
    printf("Problem: Count paths in m x n grid from\n");
    printf("         top-left to bottom-right.\n");
    printf("         Can only move right or down.\n\n");

    int gridDp[MAX_N][MAX_N];
    for (int i = 0; i < MAX_N; i++)
        for (int j = 0; j < MAX_N; j++)
            gridDp[i][j] = NOT_COMPUTED;

    printf("Grid | Paths\n");
    printf("-----|------\n");
    printf(" 2x2 | %d\n", gridPathsMemo(2, 2, gridDp));
    printf(" 3x3 | %d\n", gridPathsMemo(3, 3, gridDp));
    printf(" 3x7 | %d\n", gridPathsMemo(3, 7, gridDp));

    // Memoization Visualization
    printf("\n=== Memoization Process Visualization ===\n\n");
    printf("fib(5) with memoization:\n\n");
    printf("Call: fib(5)           memo: [_, _, _, _, _, _]\n");
    printf("  -> fib(4)            memo: [_, _, _, _, _, _]\n");
    printf("    -> fib(3)          memo: [_, _, _, _, _, _]\n");
    printf("      -> fib(2)        memo: [_, _, _, _, _, _]\n");
    printf("        -> fib(1)=1    memo: [_, 1, _, _, _, _]\n");
    printf("        -> fib(0)=0    memo: [0, 1, _, _, _, _]\n");
    printf("      <- fib(2)=1      memo: [0, 1, 1, _, _, _]\n");
    printf("      -> fib(1)=1      (cache hit!)\n");
    printf("    <- fib(3)=2        memo: [0, 1, 1, 2, _, _]\n");
    printf("    -> fib(2)=1        (cache hit!)\n");
    printf("  <- fib(4)=3          memo: [0, 1, 1, 2, 3, _]\n");
    printf("  -> fib(3)=2          (cache hit!)\n");
    printf("<- fib(5)=5            memo: [0, 1, 1, 2, 3, 5]\n");

    return 0;
}
```

**Memoization Visualization:**
```
Top-Down Approach (Memoization):

                    fib(5)
                   /      \
               fib(4)      fib(3) <- CACHED!
              /     \
          fib(3)   fib(2) <- CACHED!
          /   \
      fib(2) fib(1)
      /   \
  fib(1) fib(0)

Execution Order: 5 -> 4 -> 3 -> 2 -> 1 -> 0 (then use cache)

Memo Array:
+-----+-----+-----+-----+-----+-----+
|  0  |  1  |  2  |  3  |  4  |  5  |  <- index
+-----+-----+-----+-----+-----+-----+
|  0  |  1  |  1  |  2  |  3  |  5  |  <- value
+-----+-----+-----+-----+-----+-----+

Only 6 computations instead of 15!
```

### Example
> See `examples/` folder

---

## Tabulation - Bottom-Up Approach (타뷸레이션 - 상향식 접근)

### Concept (개념)

**Tabulation** (타뷸레이션) is a bottom-up approach where we start from the smallest subproblems and iteratively build up to the solution. We fill a table (array) in order.

**Characteristics:**
- Uses iteration (no recursion)
- Computes all subproblems (even unneeded ones)
- No stack overhead
- Often more efficient in practice
- Easier to optimize space

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define MAX_N 100

/*
 * Tabulation (Bottom-Up DP)
 * 타뷸레이션 (상향식 DP)
 */

// Fibonacci with Tabulation
// Time: O(n), Space: O(n)
long long fibTab(int n) {
    if (n <= 1) return n;

    long long dp[MAX_N];
    dp[0] = 0;
    dp[1] = 1;

    for (int i = 2; i <= n; i++) {
        dp[i] = dp[i-1] + dp[i-2];
    }

    return dp[n];
}

// Fibonacci with Space Optimization
// Time: O(n), Space: O(1)
long long fibOptimized(int n) {
    if (n <= 1) return n;

    long long prev2 = 0;  // dp[i-2]
    long long prev1 = 1;  // dp[i-1]
    long long current;

    for (int i = 2; i <= n; i++) {
        current = prev1 + prev2;
        prev2 = prev1;
        prev1 = current;
    }

    return current;
}

// Climbing Stairs with Tabulation
long long climbStairsTab(int n) {
    if (n <= 2) return n;

    long long dp[MAX_N];
    dp[1] = 1;
    dp[2] = 2;

    for (int i = 3; i <= n; i++) {
        dp[i] = dp[i-1] + dp[i-2];
    }

    return dp[n];
}

// House Robber Problem (Maximum sum of non-adjacent elements)
int houseRobber(int* nums, int n) {
    if (n == 0) return 0;
    if (n == 1) return nums[0];

    int dp[MAX_N];
    dp[0] = nums[0];
    dp[1] = (nums[0] > nums[1]) ? nums[0] : nums[1];

    for (int i = 2; i < n; i++) {
        // Either rob current house + dp[i-2], or skip and take dp[i-1]
        int robCurrent = nums[i] + dp[i-2];
        int skipCurrent = dp[i-1];
        dp[i] = (robCurrent > skipCurrent) ? robCurrent : skipCurrent;
    }

    return dp[n-1];
}

// Coin Change - Minimum coins to make amount
// Returns -1 if impossible
int coinChange(int* coins, int numCoins, int amount) {
    int dp[amount + 1];

    // Initialize with impossible value
    for (int i = 0; i <= amount; i++) {
        dp[i] = amount + 1;  // Larger than any possible answer
    }
    dp[0] = 0;  // 0 coins needed for amount 0

    // Build up solution
    for (int i = 1; i <= amount; i++) {
        for (int j = 0; j < numCoins; j++) {
            if (coins[j] <= i) {
                int prev = dp[i - coins[j]];
                if (prev + 1 < dp[i]) {
                    dp[i] = prev + 1;
                }
            }
        }
    }

    return dp[amount] > amount ? -1 : dp[amount];
}

// Print DP table
void printDpTable(long long* dp, int n, const char* label) {
    printf("%s:\n", label);
    printf("Index: ");
    for (int i = 0; i <= n; i++) printf("%4d ", i);
    printf("\nValue: ");
    for (int i = 0; i <= n; i++) printf("%4lld ", dp[i]);
    printf("\n\n");
}

int main(void) {
    printf("=== Tabulation (Bottom-Up DP) ===\n");
    printf("=== 타뷸레이션 (상향식 DP) ===\n\n");

    // Fibonacci with Tabulation
    printf("=== Fibonacci with Tabulation ===\n\n");

    printf("Building table from bottom up:\n\n");

    long long dp[MAX_N] = {0};
    dp[0] = 0;
    dp[1] = 1;

    printf("Initial: dp[0]=0, dp[1]=1\n\n");

    for (int i = 2; i <= 10; i++) {
        dp[i] = dp[i-1] + dp[i-2];
        printf("dp[%d] = dp[%d] + dp[%d] = %lld + %lld = %lld\n",
               i, i-1, i-2, dp[i-1], dp[i-2], dp[i]);
    }

    printf("\n");
    printDpTable(dp, 10, "Fibonacci Table");

    // Space Optimization
    printf("=== Space Optimization ===\n\n");
    printf("Observation: We only need dp[i-1] and dp[i-2]\n");
    printf("Instead of O(n) array, use O(1) variables!\n\n");

    printf("fib(10) with O(1) space: %lld\n\n", fibOptimized(10));

    printf("Space comparison:\n");
    printf("| Method      | Time | Space |\n");
    printf("|-------------|------|-------|\n");
    printf("| Recursive   | O(2^n)| O(n) |\n");
    printf("| Memoization | O(n) | O(n)  |\n");
    printf("| Tabulation  | O(n) | O(n)  |\n");
    printf("| Optimized   | O(n) | O(1)  |\n\n");

    // House Robber
    printf("=== House Robber Problem ===\n\n");
    printf("Problem: Rob houses, can't rob adjacent houses.\n");
    printf("         Find maximum money.\n\n");

    int houses[] = {2, 7, 9, 3, 1};
    int n = 5;

    printf("Houses: [2, 7, 9, 3, 1]\n\n");
    printf("Building DP table:\n");
    printf("dp[0] = 2 (rob house 0)\n");
    printf("dp[1] = max(2, 7) = 7 (better to rob house 1)\n");
    printf("dp[2] = max(7, 2+9) = 11 (rob houses 0 and 2)\n");
    printf("dp[3] = max(11, 7+3) = 11 (skip house 3)\n");
    printf("dp[4] = max(11, 11+1) = 12 (rob houses 0, 2, 4)\n\n");

    printf("Maximum: %d\n\n", houseRobber(houses, n));

    // Coin Change
    printf("=== Coin Change Problem ===\n\n");
    printf("Problem: Minimum coins to make amount.\n");
    printf("         Coins: [1, 2, 5], Amount: 11\n\n");

    int coins[] = {1, 2, 5};
    int amount = 11;

    printf("Building DP table:\n");
    printf("Amount: 0  1  2  3  4  5  6  7  8  9  10 11\n");
    printf("Coins:  0  1  1  2  2  1  2  2  3  3  2  3\n\n");

    printf("Minimum coins for %d: %d\n", amount, coinChange(coins, 3, amount));
    printf("Solution: 5 + 5 + 1 = 11 (3 coins)\n\n");

    // Tabulation Process Visualization
    printf("=== Tabulation Process Visualization ===\n\n");
    printf("Building Fibonacci table step by step:\n\n");
    printf("Step 0: [0] _ _ _ _ _\n");
    printf("Step 1: [0][1] _ _ _ _\n");
    printf("Step 2: [0][1][1] _ _ _\n");
    printf("Step 3: [0][1][1][2] _ _\n");
    printf("Step 4: [0][1][1][2][3] _\n");
    printf("Step 5: [0][1][1][2][3][5]\n\n");
    printf("Bottom-up: Start small, build to larger!\n");

    return 0;
}
```

**Tabulation Visualization:**
```
Bottom-Up Approach (Tabulation):

Build table from index 0 to n:

Step 1: Initialize base cases
+-----+-----+-----+-----+-----+-----+
|  0  |  1  |  ?  |  ?  |  ?  |  ?  |
+-----+-----+-----+-----+-----+-----+
  ^     ^
  |     |
 fib(0) fib(1)

Step 2: Fill table iteratively
i=2: dp[2] = dp[1] + dp[0] = 1 + 0 = 1
+-----+-----+-----+-----+-----+-----+
|  0  |  1  |  1  |  ?  |  ?  |  ?  |
+-----+-----+-----+-----+-----+-----+

i=3: dp[3] = dp[2] + dp[1] = 1 + 1 = 2
+-----+-----+-----+-----+-----+-----+
|  0  |  1  |  1  |  2  |  ?  |  ?  |
+-----+-----+-----+-----+-----+-----+

i=4: dp[4] = dp[3] + dp[2] = 2 + 1 = 3
+-----+-----+-----+-----+-----+-----+
|  0  |  1  |  1  |  2  |  3  |  ?  |
+-----+-----+-----+-----+-----+-----+

i=5: dp[5] = dp[4] + dp[3] = 3 + 2 = 5
+-----+-----+-----+-----+-----+-----+
|  0  |  1  |  1  |  2  |  3  |  5  |
+-----+-----+-----+-----+-----+-----+
                              ^
                              |
                          Answer!
```

### Example
> See `examples/` folder

---

## Optimal Substructure (최적 부분 구조)

### Concept (개념)

**Optimal Substructure** (최적 부분 구조) means that an optimal solution to the problem contains optimal solutions to its subproblems. This is a necessary condition for applying DP.

**Properties:**
- Can combine optimal sub-solutions to get optimal solution
- Greedy choice may not work (need to consider all subproblems)
- Problem can be recursively defined

```c
#include <stdio.h>
#include <stdlib.h>
#include <limits.h>

#define MAX_N 100
#define INF INT_MAX

/*
 * Optimal Substructure Examples
 * 최적 부분 구조 예제
 */

// Shortest Path - Optimal Substructure
// If A->B->C is shortest path from A to C,
// then A->B must be shortest path from A to B
void shortestPathDemo() {
    printf("=== Shortest Path - Optimal Substructure ===\n\n");

    printf("Example:\n");
    printf("           2         3\n");
    printf("     A --------> B --------> C\n");
    printf("      \\                     /\n");
    printf("       \\        6         /\n");
    printf("        +---------------->+\n\n");

    printf("Shortest path A to C:\n");
    printf("  - A -> B -> C = 2 + 3 = 5\n");
    printf("  - A -> C = 6\n");
    printf("  - Shortest = 5 (through B)\n\n");

    printf("Optimal Substructure:\n");
    printf("  If A->B->C is optimal, then A->B must also be optimal!\n");
    printf("  shortest(A,C) = min(shortest(A,B) + shortest(B,C), direct(A,C))\n\n");
}

// Longest Increasing Subsequence (LIS)
int lis(int* arr, int n) {
    int dp[MAX_N];

    // dp[i] = length of LIS ending at arr[i]
    for (int i = 0; i < n; i++) {
        dp[i] = 1;  // Minimum LIS is the element itself
    }

    // Optimal Substructure: LIS ending at i =
    // 1 + max(LIS ending at j) where j < i and arr[j] < arr[i]
    for (int i = 1; i < n; i++) {
        for (int j = 0; j < i; j++) {
            if (arr[j] < arr[i] && dp[j] + 1 > dp[i]) {
                dp[i] = dp[j] + 1;
            }
        }
    }

    // Find maximum
    int maxLis = 0;
    for (int i = 0; i < n; i++) {
        if (dp[i] > maxLis) maxLis = dp[i];
    }

    return maxLis;
}

// Rod Cutting Problem
// Cut rod of length n to maximize profit
int rodCutting(int* prices, int n) {
    int dp[MAX_N];
    dp[0] = 0;

    // Optimal Substructure:
    // max_revenue(n) = max(prices[i] + max_revenue(n-i)) for all i
    for (int len = 1; len <= n; len++) {
        int maxVal = INT_MIN;
        for (int cut = 1; cut <= len; cut++) {
            int val = prices[cut] + dp[len - cut];
            if (val > maxVal) {
                maxVal = val;
            }
        }
        dp[len] = maxVal;
    }

    return dp[n];
}

// Matrix Chain Multiplication
// Find optimal way to multiply matrices to minimize operations
int matrixChain(int* dims, int n) {
    // n matrices, dims[i-1] x dims[i] is size of matrix i
    int dp[MAX_N][MAX_N];

    // Initialize: single matrix needs 0 multiplications
    for (int i = 1; i < n; i++) {
        dp[i][i] = 0;
    }

    // Optimal Substructure:
    // dp[i][j] = min(dp[i][k] + dp[k+1][j] + cost of multiplying results)
    // for all k from i to j-1

    // l is chain length
    for (int l = 2; l < n; l++) {
        for (int i = 1; i < n - l + 1; i++) {
            int j = i + l - 1;
            dp[i][j] = INF;

            for (int k = i; k < j; k++) {
                int cost = dp[i][k] + dp[k+1][j] +
                           dims[i-1] * dims[k] * dims[j];
                if (cost < dp[i][j]) {
                    dp[i][j] = cost;
                }
            }
        }
    }

    return dp[1][n-1];
}

int main(void) {
    printf("=== Optimal Substructure (최적 부분 구조) ===\n\n");

    printf("Definition:\n");
    printf("An optimal solution to the problem contains\n");
    printf("optimal solutions to its subproblems.\n\n");

    shortestPathDemo();

    // LIS Example
    printf("=== Longest Increasing Subsequence (LIS) ===\n\n");

    int arr[] = {10, 22, 9, 33, 21, 50, 41, 60, 80};
    int n = 9;

    printf("Array: [10, 22, 9, 33, 21, 50, 41, 60, 80]\n\n");

    printf("Optimal Substructure:\n");
    printf("  LIS ending at i = 1 + max(LIS ending at j)\n");
    printf("  where j < i and arr[j] < arr[i]\n\n");

    printf("DP Table:\n");
    printf("Index:  0   1   2   3   4   5   6   7   8\n");
    printf("Value: 10  22   9  33  21  50  41  60  80\n");
    printf("LIS:    1   2   1   3   2   4   4   5   6\n\n");

    printf("LIS Length: %d\n", lis(arr, n));
    printf("One LIS: [10, 22, 33, 50, 60, 80]\n\n");

    // Rod Cutting
    printf("=== Rod Cutting Problem ===\n\n");

    int prices[] = {0, 1, 5, 8, 9, 10, 17, 17, 20};  // 0-indexed
    int rodLength = 8;

    printf("Length:  1  2  3  4  5  6  7  8\n");
    printf("Price:   1  5  8  9 10 17 17 20\n\n");

    printf("Optimal Substructure:\n");
    printf("  max_revenue(n) = max(price[i] + max_revenue(n-i))\n\n");

    printf("Maximum revenue for rod of length %d: %d\n",
           rodLength, rodCutting(prices, rodLength));
    printf("Optimal cuts: 2 + 6 (5 + 17 = 22)\n\n");

    // Matrix Chain
    printf("=== Matrix Chain Multiplication ===\n\n");

    int dims[] = {10, 30, 5, 60};  // 3 matrices: 10x30, 30x5, 5x60
    int numMatrices = 4;

    printf("Matrices: A1(10x30), A2(30x5), A3(5x60)\n\n");

    printf("Possible orders:\n");
    printf("  ((A1 A2) A3): 10*30*5 + 10*5*60 = 1500 + 3000 = 4500\n");
    printf("  (A1 (A2 A3)): 30*5*60 + 10*30*60 = 9000 + 18000 = 27000\n\n");

    printf("Minimum operations: %d\n\n", matrixChain(dims, numMatrices));

    printf("Optimal Substructure:\n");
    printf("  dp[i][j] = min(dp[i][k] + dp[k+1][j] + dims[i-1]*dims[k]*dims[j])\n");

    return 0;
}
```

**Optimal Substructure Visualization:**
```
Rod Cutting Example (n=4):

Price table:
Length: | 1 | 2 | 3 | 4 |
Price:  | 1 | 5 | 8 | 9 |

Optimal Substructure:
max_revenue(4) = max of:
  - price[1] + max_revenue(3) = 1 + 8 = 9
  - price[2] + max_revenue(2) = 5 + 5 = 10  <-- OPTIMAL
  - price[3] + max_revenue(1) = 8 + 1 = 9
  - price[4] + max_revenue(0) = 9 + 0 = 9

        max_revenue(4) = 10
             /    \
      price[2]    max_revenue(2)
        = 5           = 5
                     /    \
              price[2]    max_revenue(0)
                = 5          = 0

Cut the rod into two pieces of length 2!
```

### Example
> See `examples/` folder

---

## Overlapping Subproblems (중복 부분 문제)

### Concept (개념)

**Overlapping Subproblems** (중복 부분 문제) means the same subproblems are solved multiple times during the computation. DP stores these results to avoid redundant work.

**Key Insight:**
- Without DP: Exponential time due to repeated work
- With DP: Polynomial time by caching results

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define MAX_N 50

/*
 * Overlapping Subproblems Demonstration
 * 중복 부분 문제 시연
 */

// Counter for subproblem calls
int callCounts[MAX_N];

// Reset counters
void resetCounters() {
    memset(callCounts, 0, sizeof(callCounts));
}

// Fibonacci without memoization (shows overlapping)
int fibNoMemo(int n) {
    callCounts[n]++;

    if (n <= 1) return n;
    return fibNoMemo(n - 1) + fibNoMemo(n - 2);
}

// Binomial Coefficient C(n, k)
// Without memoization - shows overlapping subproblems
int binomialNoMemo(int n, int k) {
    callCounts[n * 10 + k]++;  // Encode (n,k) as single index

    if (k == 0 || k == n) return 1;
    return binomialNoMemo(n - 1, k - 1) + binomialNoMemo(n - 1, k);
}

// With memoization
int memo[MAX_N][MAX_N];

int binomialMemo(int n, int k) {
    if (k == 0 || k == n) return 1;

    if (memo[n][k] != -1) return memo[n][k];

    memo[n][k] = binomialMemo(n - 1, k - 1) + binomialMemo(n - 1, k);
    return memo[n][k];
}

// Count subproblem overlaps
void analyzeOverlapping(int n) {
    printf("Subproblem Call Counts for fib(%d):\n\n", n);

    resetCounters();
    fibNoMemo(n);

    printf("Subproblem | Times Called\n");
    printf("-----------|-------------\n");

    int totalCalls = 0;
    for (int i = 0; i <= n; i++) {
        if (callCounts[i] > 0) {
            printf("  fib(%2d)  |     %d\n", i, callCounts[i]);
            totalCalls += callCounts[i];
        }
    }

    printf("-----------|-------------\n");
    printf("   Total   |     %d\n\n", totalCalls);
}

int main(void) {
    printf("=== Overlapping Subproblems (중복 부분 문제) ===\n\n");

    printf("Definition:\n");
    printf("Same subproblems are solved multiple times.\n");
    printf("DP stores results to avoid redundant computation.\n\n");

    // Fibonacci overlapping visualization
    printf("=== Fibonacci: Overlapping Subproblems ===\n\n");

    printf("Call tree for fib(6):\n\n");
    printf("                        fib(6)\n");
    printf("                       /      \\\n");
    printf("                   fib(5)      fib(4)\n");
    printf("                  /     \\      /    \\\n");
    printf("              fib(4)   fib(3) fib(3) fib(2)\n");
    printf("              /   \\    /   \\   /   \\\n");
    printf("          fib(3) fib(2) ...  ...   ...\n\n");

    printf("Overlapping subproblems (marked with *):\n");
    printf("  - fib(4) is computed 2 times\n");
    printf("  - fib(3) is computed 3 times\n");
    printf("  - fib(2) is computed 5 times\n");
    printf("  - fib(1) is computed 8 times\n");
    printf("  - fib(0) is computed 5 times\n\n");

    // Detailed analysis
    analyzeOverlapping(10);
    analyzeOverlapping(15);

    // Binomial coefficients
    printf("=== Pascal's Triangle: Overlapping Subproblems ===\n\n");

    printf("C(n,k) = C(n-1,k-1) + C(n-1,k)\n\n");

    printf("Call tree for C(5,2):\n");
    printf("                    C(5,2)\n");
    printf("                   /      \\\n");
    printf("               C(4,1)      C(4,2)\n");
    printf("              /     \\      /    \\\n");
    printf("          C(3,0)  C(3,1) C(3,1) C(3,2)\n");
    printf("                        ^ ^      \n");
    printf("                        OVERLAP! \n\n");

    // Pascal's Triangle visualization
    printf("Pascal's Triangle (shows shared subproblems):\n\n");
    printf("              1\n");
    printf("             1 1\n");
    printf("            1 2 1\n");
    printf("           1 3 3 1\n");
    printf("          1 4 6 4 1\n");
    printf("         1 5 10 10 5 1\n\n");

    printf("Each number is the sum of two numbers above it.\n");
    printf("These are the shared subproblems!\n\n");

    // With vs without memoization
    printf("=== Impact of Memoization ===\n\n");

    printf("Without DP:\n");
    printf("  C(20, 10) requires computing C(10,5) many times!\n");
    printf("  Total calls: exponential growth\n\n");

    // Initialize memo
    memset(memo, -1, sizeof(memo));

    printf("With DP (Memoization):\n");
    printf("  Each C(n,k) computed exactly once.\n");
    printf("  Total: O(n*k) subproblems\n\n");

    printf("Result C(20, 10) = %d\n\n", binomialMemo(20, 10));

    // Summary
    printf("=== Overlapping vs Non-Overlapping ===\n\n");

    printf("Overlapping (use DP):           Non-Overlapping:\n");
    printf("  - Fibonacci                     - Merge Sort\n");
    printf("  - Binomial                      - Quick Sort\n");
    printf("  - Shortest Path                 - Binary Search\n");
    printf("  - Edit Distance                 - Factorial\n\n");

    printf("How to identify overlapping:\n");
    printf("  1. Draw the recursion tree\n");
    printf("  2. Look for repeated nodes\n");
    printf("  3. If same parameters appear multiple times -> overlapping!\n");

    return 0;
}
```

**Overlapping Subproblems Visualization:**
```
Fibonacci Recursion Tree (n=6):

                        fib(6)
                       /      \
                   fib(5)      fib(4)  <--+
                  /     \      /    \     |
              fib(4)   fib(3) fib(3) fib(2)  |
               |         |      |           |
               +----OVERLAPPING---+---------+

Same subproblem computed multiple times:

Subproblem  | Without DP | With DP
------------|------------|--------
fib(0)      |    8 times |  1 time
fib(1)      |   13 times |  1 time
fib(2)      |    8 times |  1 time
fib(3)      |    5 times |  1 time
fib(4)      |    3 times |  1 time
fib(5)      |    2 times |  1 time
fib(6)      |    1 time  |  1 time
------------|------------|--------
Total       |   40 calls |  7 calls

Speedup: 40/7 = 5.7x for n=6
For n=30: ~1 billion / 30 = 33 million x speedup!
```

### Example
> See `examples/` folder

---

## Classic Examples (고전 예제)

### Fibonacci Sequence (피보나치 수열)

```c
#include <stdio.h>
#include <stdlib.h>

#define MAX_N 100

/*
 * Fibonacci - All DP Approaches
 * 피보나치 - 모든 DP 접근법
 */

// 1. Naive Recursion O(2^n)
long long fibNaive(int n) {
    if (n <= 1) return n;
    return fibNaive(n-1) + fibNaive(n-2);
}

// 2. Memoization O(n) time, O(n) space
long long memo[MAX_N];
long long fibMemo(int n) {
    if (n <= 1) return n;
    if (memo[n] != -1) return memo[n];
    memo[n] = fibMemo(n-1) + fibMemo(n-2);
    return memo[n];
}

// 3. Tabulation O(n) time, O(n) space
long long fibTab(int n) {
    if (n <= 1) return n;
    long long dp[MAX_N];
    dp[0] = 0; dp[1] = 1;
    for (int i = 2; i <= n; i++) {
        dp[i] = dp[i-1] + dp[i-2];
    }
    return dp[n];
}

// 4. Space Optimized O(n) time, O(1) space
long long fibOptimal(int n) {
    if (n <= 1) return n;
    long long prev2 = 0, prev1 = 1, curr;
    for (int i = 2; i <= n; i++) {
        curr = prev1 + prev2;
        prev2 = prev1;
        prev1 = curr;
    }
    return curr;
}

int main(void) {
    printf("=== Fibonacci Sequence (피보나치 수열) ===\n\n");

    // Initialize memoization array
    for (int i = 0; i < MAX_N; i++) memo[i] = -1;

    printf("Fibonacci sequence:\n");
    printf("n  | Value\n");
    printf("---|-------------\n");
    for (int i = 0; i <= 20; i++) {
        printf("%2d | %lld\n", i, fibOptimal(i));
    }

    printf("\n=== Complexity Comparison ===\n\n");
    printf("| Method      | Time    | Space |\n");
    printf("|-------------|---------|-------|\n");
    printf("| Naive       | O(2^n)  | O(n)  |\n");
    printf("| Memoization | O(n)    | O(n)  |\n");
    printf("| Tabulation  | O(n)    | O(n)  |\n");
    printf("| Optimized   | O(n)    | O(1)  |\n");

    return 0;
}
```

### 0/1 Knapsack Problem (0/1 배낭 문제)

```c
#include <stdio.h>
#include <stdlib.h>

#define MAX_N 100
#define MAX_W 1000

/*
 * 0/1 Knapsack Problem
 * 0/1 배낭 문제
 *
 * Given n items with weights and values,
 * find maximum value that fits in capacity W.
 * Each item can be taken at most once (0 or 1).
 */

// Item structure
typedef struct {
    int weight;
    int value;
    char name[20];
} Item;

// Recursive solution with memoization
int memo[MAX_N][MAX_W];

int knapsackMemo(Item* items, int n, int W, int i) {
    // Base cases
    if (i < 0 || W == 0) return 0;

    // Check cache
    if (memo[i][W] != -1) return memo[i][W];

    // If item is too heavy, skip it
    if (items[i].weight > W) {
        memo[i][W] = knapsackMemo(items, n, W, i - 1);
    } else {
        // Max of: skip item OR take item
        int skip = knapsackMemo(items, n, W, i - 1);
        int take = items[i].value +
                   knapsackMemo(items, n, W - items[i].weight, i - 1);
        memo[i][W] = (skip > take) ? skip : take;
    }

    return memo[i][W];
}

// Tabulation solution
int knapsackTab(Item* items, int n, int W) {
    int dp[MAX_N + 1][MAX_W + 1];

    // Initialize: dp[i][w] = max value with first i items and capacity w
    for (int i = 0; i <= n; i++) {
        for (int w = 0; w <= W; w++) {
            if (i == 0 || w == 0) {
                dp[i][w] = 0;
            } else if (items[i-1].weight > w) {
                dp[i][w] = dp[i-1][w];  // Can't take item i
            } else {
                int skip = dp[i-1][w];
                int take = items[i-1].value + dp[i-1][w - items[i-1].weight];
                dp[i][w] = (skip > take) ? skip : take;
            }
        }
    }

    // Find which items were selected
    printf("\nItems selected:\n");
    int w = W;
    for (int i = n; i > 0 && w > 0; i--) {
        if (dp[i][w] != dp[i-1][w]) {
            printf("  - %s (weight=%d, value=%d)\n",
                   items[i-1].name, items[i-1].weight, items[i-1].value);
            w -= items[i-1].weight;
        }
    }

    return dp[n][W];
}

// Space optimized (1D array)
int knapsackOptimal(Item* items, int n, int W) {
    int dp[MAX_W + 1] = {0};

    for (int i = 0; i < n; i++) {
        // Must iterate backwards to avoid using updated values
        for (int w = W; w >= items[i].weight; w--) {
            int take = items[i].value + dp[w - items[i].weight];
            if (take > dp[w]) {
                dp[w] = take;
            }
        }
    }

    return dp[W];
}

void printDPTable(Item* items, int n, int W) {
    int dp[MAX_N + 1][MAX_W + 1] = {0};

    printf("\nDP Table:\n");
    printf("    w: ");
    for (int w = 0; w <= W; w++) printf("%3d", w);
    printf("\n       ");
    for (int w = 0; w <= W; w++) printf("---");
    printf("\n");

    for (int i = 0; i <= n; i++) {
        if (i == 0) {
            printf("  0  | ");
        } else {
            printf("%s | ", items[i-1].name);
        }

        for (int w = 0; w <= W; w++) {
            if (i == 0 || w == 0) {
                dp[i][w] = 0;
            } else if (items[i-1].weight > w) {
                dp[i][w] = dp[i-1][w];
            } else {
                int skip = dp[i-1][w];
                int take = items[i-1].value + dp[i-1][w - items[i-1].weight];
                dp[i][w] = (skip > take) ? skip : take;
            }
            printf("%3d", dp[i][w]);
        }
        printf("\n");
    }
}

int main(void) {
    printf("=== 0/1 Knapsack Problem (0/1 배낭 문제) ===\n\n");

    // Example items
    Item items[] = {
        {2, 3, "A"},
        {3, 4, "B"},
        {4, 5, "C"},
        {5, 6, "D"}
    };
    int n = 4;
    int W = 8;

    printf("Problem:\n");
    printf("  - Knapsack capacity: %d\n", W);
    printf("  - Items:\n");
    printf("    | Name | Weight | Value |\n");
    printf("    |------|--------|-------|\n");
    for (int i = 0; i < n; i++) {
        printf("    |  %s   |   %d    |   %d   |\n",
               items[i].name, items[i].weight, items[i].value);
    }

    printDPTable(items, n, W);

    printf("\nMaximum value: %d\n", knapsackTab(items, n, W));

    // Recurrence relation
    printf("\n=== Recurrence Relation ===\n\n");
    printf("dp[i][w] = max value using first i items with capacity w\n\n");
    printf("Base case:\n");
    printf("  dp[0][w] = 0  (no items)\n");
    printf("  dp[i][0] = 0  (no capacity)\n\n");
    printf("Transition:\n");
    printf("  If weight[i] > w:\n");
    printf("    dp[i][w] = dp[i-1][w]  (can't take item i)\n");
    printf("  Else:\n");
    printf("    dp[i][w] = max(dp[i-1][w],           // skip item i\n");
    printf("                   value[i] + dp[i-1][w-weight[i]]) // take item i\n");

    return 0;
}
```

**Knapsack Visualization:**
```
0/1 Knapsack Problem:

Items: A(w=2,v=3), B(w=3,v=4), C(w=4,v=5), D(w=5,v=6)
Capacity: W = 8

DP Table Construction:
      w:   0   1   2   3   4   5   6   7   8
         --------------------------------
    0  |   0   0   0   0   0   0   0   0   0
    A  |   0   0   3   3   3   3   3   3   3
    B  |   0   0   3   4   4   7   7   7   7
    C  |   0   0   3   4   5   7   8   9   9
    D  |   0   0   3   4   5   7   8   9  10

Answer: dp[4][8] = 10

Selected items: B(3,4) + D(5,6) = weight 8, value 10

Decision Tree:
                     dp[4][8]=10
                    /           \
            skip D:dp[3][8]=9   take D:6+dp[3][3]=6+4=10 <--
                                          |
                                    B selected (4)
```

### Longest Common Subsequence (LCS) (최장 공통 부분 수열)

```c
#include <stdio.h>
#include <string.h>

#define MAX_LEN 100

/*
 * Longest Common Subsequence (LCS)
 * 최장 공통 부분 수열
 *
 * Find the longest subsequence common to both strings.
 * A subsequence maintains relative order but need not be contiguous.
 */

int dp[MAX_LEN][MAX_LEN];
char lcs[MAX_LEN];

int lcsLength(const char* s1, const char* s2) {
    int m = strlen(s1);
    int n = strlen(s2);

    // Build DP table
    // dp[i][j] = LCS length of s1[0..i-1] and s2[0..j-1]
    for (int i = 0; i <= m; i++) {
        for (int j = 0; j <= n; j++) {
            if (i == 0 || j == 0) {
                dp[i][j] = 0;
            } else if (s1[i-1] == s2[j-1]) {
                // Characters match: extend LCS
                dp[i][j] = dp[i-1][j-1] + 1;
            } else {
                // No match: take max of excluding one character
                dp[i][j] = (dp[i-1][j] > dp[i][j-1]) ?
                            dp[i-1][j] : dp[i][j-1];
            }
        }
    }

    return dp[m][n];
}

void findLCS(const char* s1, const char* s2, char* result) {
    int m = strlen(s1);
    int n = strlen(s2);

    // Backtrack to find actual LCS
    int i = m, j = n;
    int index = dp[m][n];
    result[index] = '\0';

    while (i > 0 && j > 0) {
        if (s1[i-1] == s2[j-1]) {
            result[--index] = s1[i-1];
            i--; j--;
        } else if (dp[i-1][j] > dp[i][j-1]) {
            i--;
        } else {
            j--;
        }
    }
}

void printDPTable(const char* s1, const char* s2) {
    int m = strlen(s1);
    int n = strlen(s2);

    printf("\nDP Table:\n");
    printf("    ");
    for (int j = 0; j <= n; j++) {
        if (j == 0) printf("  - ");
        else printf("  %c ", s2[j-1]);
    }
    printf("\n");

    for (int i = 0; i <= m; i++) {
        if (i == 0) printf("-   ");
        else printf("%c   ", s1[i-1]);

        for (int j = 0; j <= n; j++) {
            printf("%2d  ", dp[i][j]);
        }
        printf("\n");
    }
}

int main(void) {
    printf("=== Longest Common Subsequence (LCS) ===\n");
    printf("=== 최장 공통 부분 수열 ===\n\n");

    const char* s1 = "ABCDGH";
    const char* s2 = "AEDFHR";

    printf("String 1: %s\n", s1);
    printf("String 2: %s\n", s2);

    int length = lcsLength(s1, s2);

    printDPTable(s1, s2);

    char result[MAX_LEN];
    findLCS(s1, s2, result);

    printf("\nLCS Length: %d\n", length);
    printf("LCS: %s\n", result);

    // Another example
    printf("\n=== Example 2 ===\n\n");

    const char* x = "AGGTAB";
    const char* y = "GXTXAYB";

    printf("String 1: %s\n", x);
    printf("String 2: %s\n", y);

    length = lcsLength(x, y);
    printDPTable(x, y);
    findLCS(x, y, result);

    printf("\nLCS Length: %d\n", length);
    printf("LCS: %s\n", result);

    // Recurrence relation
    printf("\n=== Recurrence Relation ===\n\n");
    printf("dp[i][j] = LCS length of s1[0..i-1] and s2[0..j-1]\n\n");
    printf("Base case:\n");
    printf("  dp[0][j] = 0  (empty s1)\n");
    printf("  dp[i][0] = 0  (empty s2)\n\n");
    printf("Transition:\n");
    printf("  If s1[i-1] == s2[j-1]:\n");
    printf("    dp[i][j] = dp[i-1][j-1] + 1  (extend LCS)\n");
    printf("  Else:\n");
    printf("    dp[i][j] = max(dp[i-1][j], dp[i][j-1])  (skip one char)\n");

    return 0;
}
```

**LCS Visualization:**
```
LCS of "ABCDGH" and "AEDFHR"

DP Table:
        -   A   E   D   F   H   R
    -   0   0   0   0   0   0   0
    A   0   1   1   1   1   1   1
    B   0   1   1   1   1   1   1
    C   0   1   1   1   1   1   1
    D   0   1   1   2   2   2   2
    G   0   1   1   2   2   2   2
    H   0   1   1   2   2   3   3

LCS = "ADH" (length 3)

Backtracking path:
        -   A   E   D   F   H   R
    -   0   0   0   0   0   0   0
    A   0  [1]--1   1   1   1   1
    B   0   |   1   1   1   1   1
    C   0   1   1   1   1   1   1
    D   0   1   1  [2]--2   2   2
    G   0   1   1   |   2   2   2
    H   0   1   1   2   2  [3]--3
                            |
                           END

Match at (1,1): 'A', (4,3): 'D', (6,5): 'H'
```

---

## Summary (요약)

| Topic | Key Points |
|-------|------------|
| DP Concept | Break problem into subproblems, store results |
| Memoization | Top-down, recursion + caching, lazy evaluation |
| Tabulation | Bottom-up, iterative, fills table systematically |
| Optimal Substructure | Optimal solution contains optimal sub-solutions |
| Overlapping Subproblems | Same subproblems solved multiple times |

### DP Problem Solving Steps (DP 문제 해결 단계)

1. **Define State**: What does dp[i] or dp[i][j] represent?
2. **Base Case**: What are the trivial cases?
3. **Transition**: How to compute dp[i] from smaller subproblems?
4. **Answer**: Which state gives the final answer?
5. **Optimize**: Can we reduce space complexity?

### Common DP Patterns (일반적인 DP 패턴)

| Pattern | Examples | State |
|---------|----------|-------|
| Linear | Fibonacci, Climbing Stairs | dp[i] |
| Grid | Unique Paths, Min Path Sum | dp[i][j] |
| String | LCS, Edit Distance | dp[i][j] |
| Knapsack | 0/1 Knapsack, Subset Sum | dp[i][w] |
| Interval | Matrix Chain, Palindrome | dp[i][j] |

### Time/Space Complexity Summary (시간/공간 복잡도 요약)

| Problem | Time | Space | Optimized Space |
|---------|------|-------|-----------------|
| Fibonacci | O(n) | O(n) | O(1) |
| 0/1 Knapsack | O(n*W) | O(n*W) | O(W) |
| LCS | O(m*n) | O(m*n) | O(min(m,n)) |
| Coin Change | O(n*amount) | O(amount) | - |
| LIS | O(n^2) | O(n) | - |

---

## Practice Exercises (연습 문제)

1. Implement the Edit Distance (Levenshtein Distance) algorithm.
2. Solve the Coin Change problem: minimum coins to make amount.
3. Find the Longest Palindromic Subsequence.
4. Implement the Subset Sum problem: can subset sum to target?
5. Solve the Unique Paths problem with obstacles.
