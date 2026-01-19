# Chapter 02. Data Representation (데이터 표현)

> How computers encode numbers and why understanding representation matters for avoiding bugs and optimizing performance.

---

## Binary and Hexadecimal Systems (이진수와 16진수)

### Concept (개념)

Computers operate on **binary (이진수)** - base 2 number system using only 0 and 1.

**Why Binary?**
- Electronic circuits have two stable states (on/off, high/low voltage)
- Reliable signal processing with clear distinction between states
- Simple logic operations

**Number System Bases:**
| Base | Name | Korean | Digits |
|------|------|--------|--------|
| 2 | Binary | 이진수 | 0, 1 |
| 10 | Decimal | 십진수 | 0-9 |
| 16 | Hexadecimal | 16진수 | 0-9, A-F |

**Hexadecimal (16진수)** is commonly used because:
- Compact representation of binary (4 bits = 1 hex digit)
- Easy conversion to/from binary
- Common in memory addresses, color codes, debugging

**Conversion:**
```
Binary to Decimal:
1101₂ = 1×2³ + 1×2² + 0×2¹ + 1×2⁰
      = 8 + 4 + 0 + 1 = 13₁₀

Binary to Hexadecimal (group by 4):
1010 1111₂ = AF₁₆

Hexadecimal to Decimal:
2F₁₆ = 2×16¹ + 15×16⁰ = 32 + 15 = 47₁₀
```

### Diagram / Example

```
Bit Positions and Values:
┌────┬────┬────┬────┬────┬────┬────┬────┐
│ 2⁷ │ 2⁶ │ 2⁵ │ 2⁴ │ 2³ │ 2² │ 2¹ │ 2⁰ │
├────┼────┼────┼────┼────┼────┼────┼────┤
│128 │ 64 │ 32 │ 16 │ 8  │ 4  │ 2  │ 1  │
├────┼────┼────┼────┼────┼────┼────┼────┤
│ 1  │ 0  │ 1  │ 0  │ 0  │ 1  │ 1  │ 0  │  = 166₁₀
└────┴────┴────┴────┴────┴────┴────┴────┘
 └────────┘ └────────┘
     A₁₆        6₁₆    → 0xA6

Common Bit Patterns:
┌──────────────┬───────────────┬─────────────┐
│   Decimal    │    Binary     │ Hexadecimal │
├──────────────┼───────────────┼─────────────┤
│      0       │   0000 0000   │    0x00     │
│     255      │   1111 1111   │    0xFF     │
│     256      │ 1 0000 0000   │   0x100     │
│    65535     │ (16 bits all 1)│   0xFFFF   │
└──────────────┴───────────────┴─────────────┘
```

---

## Integer Representation (정수 표현)

### Concept (개념)

**Unsigned Integers (부호 없는 정수):**
- All bits represent magnitude
- Range: 0 to 2ⁿ - 1
- 8-bit: 0 to 255
- 32-bit: 0 to 4,294,967,295

**Signed Integers (부호 있는 정수):**
Three main methods exist, but modern systems use **Two's Complement (2의 보수)**:

1. **Sign-Magnitude (부호-크기):**
   - MSB (Most Significant Bit) = sign (0: positive, 1: negative)
   - Problem: Two zeros (+0 and -0), complex arithmetic

2. **One's Complement (1의 보수):**
   - Negative: flip all bits
   - Problem: Still has two zeros

3. **Two's Complement (2의 보수):** ← Standard method
   - Negative: flip all bits, then add 1
   - Single zero representation
   - Simple addition/subtraction circuits

**Two's Complement Properties:**
```
To negate a number: Flip bits + 1
Example (8-bit):
  5 = 0000 0101
 -5 = 1111 1010 + 1 = 1111 1011

Range for n bits: -2^(n-1) to 2^(n-1) - 1
  8-bit: -128 to 127
 16-bit: -32,768 to 32,767
 32-bit: -2,147,483,648 to 2,147,483,647
```

### Diagram / Example

```
Two's Complement (2의 보수) - 4-bit example:

Positive numbers:
0000 =  0
0001 =  1
0010 =  2
0011 =  3
0100 =  4
0101 =  5
0110 =  6
0111 =  7  ← Maximum positive

Negative numbers:
1000 = -8  ← Minimum (most negative)
1001 = -7
1010 = -6
1011 = -5
1100 = -4
1101 = -3
1110 = -2
1111 = -1

Number Circle (4-bit):
           0 (0000)
     -1 ←───┼───→ 1
   (1111)   │   (0001)
            │
    -8 ────────── 7
  (1000)        (0111)

Integer Overflow (정수 오버플로우):
127 + 1 in 8-bit signed:
  0111 1111  (127)
+ 0000 0001  (1)
───────────
  1000 0000  (-128) ← Overflow! Wraps to negative
```

**Performance Implications:**
```c
// Signed vs Unsigned comparison matters!
for (unsigned int i = 10; i >= 0; i--) {
    // INFINITE LOOP! When i=0, i-- becomes 4294967295
}

// Safe version:
for (int i = 10; i >= 0; i--) {
    // Works correctly
}
```

---

## Floating Point - IEEE 754 (부동소수점)

### Concept (개념)

**Floating Point (부동소수점)** represents real numbers using scientific notation in binary:

```
Number = (-1)^S × 1.M × 2^(E-bias)

Where:
- S = Sign bit (부호 비트): 0 = positive, 1 = negative
- M = Mantissa/Fraction (가수): The significant digits
- E = Exponent (지수): The power of 2
- Bias = Offset to allow negative exponents
```

**IEEE 754 Formats:**

| Format | Total Bits | Sign | Exponent | Mantissa | Bias |
|--------|------------|------|----------|----------|------|
| Single (float) | 32 | 1 | 8 | 23 | 127 |
| Double (double) | 64 | 1 | 11 | 52 | 1023 |

**Special Values:**
- **Zero**: Exponent = 0, Mantissa = 0
- **Infinity (무한대)**: Exponent = all 1s, Mantissa = 0
- **NaN (Not a Number)**: Exponent = all 1s, Mantissa ≠ 0
- **Denormalized**: Exponent = 0, Mantissa ≠ 0 (very small numbers)

### Diagram / Example

```
IEEE 754 Single Precision (32-bit float):
┌───┬──────────┬───────────────────────────┐
│ S │ Exponent │        Mantissa           │
│ 1 │    8     │           23              │
└───┴──────────┴───────────────────────────┘
 31   30----23   22--------------------0

Example: Represent -6.5 in IEEE 754 single precision

Step 1: Convert to binary
6.5₁₀ = 110.1₂

Step 2: Normalize (move decimal to get 1.xxx)
110.1 = 1.101 × 2²

Step 3: Determine components
Sign: 1 (negative)
Exponent: 2 + 127 (bias) = 129 = 1000 0001₂
Mantissa: 101 (implied 1. is not stored)

Result:
┌───┬──────────┬───────────────────────────┐
│ 1 │ 10000001 │ 10100000000000000000000   │
└───┴──────────┴───────────────────────────┘

Hexadecimal: 0xC0D00000

Precision Ranges:
┌────────────────────────────────────────────────┐
│  Float (32-bit):   ~7 decimal digits precision │
│  Double (64-bit): ~15 decimal digits precision │
└────────────────────────────────────────────────┘
```

---

## Precision Errors (정밀도 오류)

### Concept (개념)

**Why Floating Point Has Errors:**

1. **Finite representation**: Cannot represent all real numbers exactly
2. **Binary fractions**: Many decimal fractions have infinite binary representations
3. **Rounding**: Results must be rounded to fit available bits

**Common Problem Numbers:**
- 0.1₁₀ = 0.0001100110011...₂ (infinite repeating)
- 0.2₁₀ = 0.0011001100110...₂ (infinite repeating)

**Types of Errors:**
1. **Representation Error (표현 오차)**: Number cannot be exactly represented
2. **Rounding Error (반올림 오차)**: Operations produce results needing more precision
3. **Catastrophic Cancellation (상쇄 오차)**: Subtracting nearly equal numbers

**Practical Rules:**
```
NEVER: if (x == 0.1)           // Direct comparison
ALWAYS: if (abs(x - 0.1) < epsilon)  // Tolerance comparison

NEVER: for (float f = 0; f != 1.0; f += 0.1)  // Dangerous
ALWAYS: for (int i = 0; i < 10; i++)          // Use integers
```

### Diagram / Example

```
Precision Error Example:

0.1 + 0.2 in floating point:
┌─────────────────────────────────────────┐
│ 0.1 ≈ 0.100000001490116119384765625     │
│ 0.2 ≈ 0.200000002980232238769531250     │
│ Sum = 0.300000004470348358154296875     │
│                                         │
│ Expected: 0.3                           │
│ Actual:   0.30000000000000004 (approx)  │
└─────────────────────────────────────────┘

Catastrophic Cancellation Example:
┌─────────────────────────────────────────┐
│ a = 1.000000000000001                   │
│ b = 1.000000000000000                   │
│                                         │
│ a - b should be: 0.000000000000001      │
│ Computed result: 0.0 (precision lost!)  │
└─────────────────────────────────────────┘

Accumulation Error in Loops:
┌─────────────────────────────────────────┐
│ float sum = 0;                          │
│ for (int i = 0; i < 1000000; i++) {     │
│     sum += 0.1f;                        │
│ }                                       │
│ // Expected: 100000.0                   │
│ // Actual:   100958.34 (error grows!)   │
└─────────────────────────────────────────┘

Kahan Summation Algorithm (more accurate):
┌─────────────────────────────────────────┐
│ float sum = 0, c = 0;                   │
│ for (int i = 0; i < n; i++) {           │
│     float y = arr[i] - c;               │
│     float t = sum + y;                  │
│     c = (t - sum) - y;  // compensation │
│     sum = t;                            │
│ }                                       │
└─────────────────────────────────────────┘
```

**Performance vs Precision Trade-offs:**
```
Type        Precision    Speed     Use Case
──────────────────────────────────────────────────
float       ~7 digits    Faster    Graphics, games, audio
double      ~15 digits   Slower    Scientific computing
int         Exact        Fastest   Counting, indexing
fixed-point Configurable Fast      Financial calculations

Game Development Tip:
- Use float for positions, velocities (fast, good enough)
- Use int for scores, item counts (exact)
- Use double only when precision is critical
```

---

## Summary (요약)

| Representation | Korean | Range/Precision | Use Case |
|---------------|--------|-----------------|----------|
| Unsigned int | 부호없는 정수 | 0 to 2^n-1 | Array indices, sizes |
| Signed int (2's comp) | 부호있는 정수 | -2^(n-1) to 2^(n-1)-1 | General integers |
| Float (32-bit) | 단정밀도 | ~7 digits | Graphics, games |
| Double (64-bit) | 배정밀도 | ~15 digits | Scientific computing |

**Key Takeaways for Performance:**
1. Integer operations are faster than floating point
2. Float is faster than double (and uses half the memory bandwidth)
3. Avoid float comparisons with == ; use epsilon tolerance
4. Watch for overflow in integer arithmetic
5. Use appropriate data types to minimize memory and maximize cache efficiency
