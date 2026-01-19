# Chapter 13. Cryptography Basics (암호화 기초)

---

## Table of Contents (목차)

1. [Why Encryption? (암호화가 필요한 이유)](#1-why-encryption-암호화가-필요한-이유)
2. [Symmetric Key Encryption (대칭키 암호화)](#2-symmetric-key-encryption-대칭키-암호화)
3. [Asymmetric Key Encryption (비대칭키 암호화)](#3-asymmetric-key-encryption-비대칭키-암호화)
4. [Hash Functions (해시 함수)](#4-hash-functions-해시-함수)
5. [Digital Signatures (디지털 서명)](#5-digital-signatures-디지털-서명)
6. [Summary (요약)](#6-summary-요약)

---

## 1. Why Encryption? (암호화가 필요한 이유)

### The Problem: Insecure Communication (문제: 안전하지 않은 통신)

```
+-----------------------------------------------------------------------+
|                    Without Encryption                                  |
+-----------------------------------------------------------------------+
|                                                                       |
|   Alice                                            Bob                |
|   +-----+                                        +-----+              |
|   |     |  "My password is secret123"            |     |              |
|   | :-) |  =================================>    | :-) |              |
|   +-----+                                        +-----+              |
|                        ^                                              |
|                        |                                              |
|                   +----+----+                                         |
|                   | Attacker|  "I can see: My password is secret123"  |
|                   |   >:-)  |                                         |
|                   +---------+                                         |
|                                                                       |
+-----------------------------------------------------------------------+
```

### Security Goals (보안 목표)

```
+-----------------------------------------------------------------------+
|                    CIA Triad + Authentication                          |
+-----------------------------------------------------------------------+
|                                                                       |
|                    Confidentiality                                    |
|                    (기밀성)                                           |
|                         /\                                            |
|                        /  \                                           |
|                       /    \                                          |
|                      /      \                                         |
|                     /________\                                        |
|                    /          \                                       |
|     Integrity     /            \    Availability                      |
|     (무결성)     /              \   (가용성)                          |
|                                                                       |
|             + Authentication (인증)                                   |
|                                                                       |
+-----------------------------------------------------------------------+

+----------------------+------------------------------------------------+
| Goal                 | Description                                    |
+----------------------+------------------------------------------------+
| Confidentiality      | Only authorized parties can read the data      |
| 기밀성               | 권한이 있는 사람만 데이터를 읽을 수 있음       |
+----------------------+------------------------------------------------+
| Integrity            | Data has not been modified in transit          |
| 무결성               | 데이터가 전송 중 변경되지 않음                 |
+----------------------+------------------------------------------------+
| Availability         | Data is accessible when needed                 |
| 가용성               | 필요할 때 데이터에 접근 가능                   |
+----------------------+------------------------------------------------+
| Authentication       | Verify identity of sender/receiver             |
| 인증                 | 송신자/수신자의 신원 확인                      |
+----------------------+------------------------------------------------+
```

### Encryption vs Hashing vs Encoding (암호화 vs 해싱 vs 인코딩)

```
+-----------------------------------------------------------------------+
|                 Encryption vs Hashing vs Encoding                      |
+-----------------------------------------------------------------------+
|                                                                       |
|   ENCODING (인코딩)                                                   |
|   - Transform data to different format                                |
|   - NOT for security                                                  |
|   - Reversible without key                                            |
|   - Example: Base64, URL encoding                                     |
|                                                                       |
|   "Hello" --Base64--> "SGVsbG8=" --Decode--> "Hello"                  |
|                                                                       |
+-----------------------------------------------------------------------+
|                                                                       |
|   ENCRYPTION (암호화)                                                 |
|   - Transform data for confidentiality                                |
|   - Requires key to decrypt                                           |
|   - Reversible WITH correct key                                       |
|   - Example: AES, RSA                                                 |
|                                                                       |
|   "Hello" + Key --Encrypt--> "x#$%^" --Decrypt + Key--> "Hello"       |
|                                                                       |
+-----------------------------------------------------------------------+
|                                                                       |
|   HASHING (해싱)                                                      |
|   - One-way transformation                                            |
|   - Cannot be reversed                                                |
|   - Fixed output length                                               |
|   - Example: SHA-256, MD5                                             |
|                                                                       |
|   "Hello" --Hash--> "185f8db32..." (Cannot reverse!)                  |
|                                                                       |
+-----------------------------------------------------------------------+
```

---

## 2. Symmetric Key Encryption (대칭키 암호화)

### What is Symmetric Encryption? (대칭키 암호화란?)

Symmetric encryption uses the **same key** for both encryption and decryption. The sender and receiver must both know the secret key.

대칭키 암호화는 암호화와 복호화에 **동일한 키**를 사용합니다. 송신자와 수신자 모두 비밀 키를 알아야 합니다.

```
+-----------------------------------------------------------------------+
|                    Symmetric Encryption                                |
+-----------------------------------------------------------------------+
|                                                                       |
|   Alice                                               Bob             |
|   +-----+                                           +-----+           |
|   |     |                                           |     |           |
|   +-----+                                           +-----+           |
|      |                                                 |              |
|      | Plaintext: "Hello"                              |              |
|      |    +                                            |              |
|      | Key: "secret123"                                |              |
|      |                                                 |              |
|      v                                                 |              |
|   +--------+                                      +--------+          |
|   |Encrypt |                                      |Decrypt |          |
|   +--------+                                      +--------+          |
|      |                                                 ^              |
|      | Ciphertext: "x#$%^&"                            |              |
|      +------------------------>------------------------+              |
|                                                        |              |
|                                          Key: "secret123"             |
|                                          (same key!)                  |
|                                                        |              |
|                                                        v              |
|                                          Plaintext: "Hello"           |
|                                                                       |
+-----------------------------------------------------------------------+

Key Point: Same key used for both encryption and decryption
핵심: 암호화와 복호화에 동일한 키 사용
```

### AES (Advanced Encryption Standard)

```
+-----------------------------------------------------------------------+
|                         AES Overview                                   |
+-----------------------------------------------------------------------+
|                                                                       |
|   - Most widely used symmetric cipher today                           |
|   - 현재 가장 널리 사용되는 대칭키 암호                               |
|                                                                       |
|   - Replaced DES in 2001                                              |
|   - 2001년 DES를 대체                                                 |
|                                                                       |
|   - Block cipher: encrypts fixed-size blocks (128 bits)               |
|   - 블록 암호: 고정 크기 블록(128비트)을 암호화                       |
|                                                                       |
|   +----------------+--------------------------------+                  |
|   | Key Size       | Security Level                |                  |
|   +----------------+--------------------------------+                  |
|   | AES-128        | Good (2^128 possible keys)    |                  |
|   | AES-192        | Better                        |                  |
|   | AES-256        | Best (military grade)         |                  |
|   +----------------+--------------------------------+                  |
|                                                                       |
+-----------------------------------------------------------------------+
```

### AES Operation (AES 동작 방식)

```
+-----------------------------------------------------------------------+
|                      AES Encryption Process                            |
+-----------------------------------------------------------------------+
|                                                                       |
|   Plaintext (128-bit block)                                           |
|   +---+---+---+---+                                                   |
|   | S | E | C | R |                                                   |
|   +---+---+---+---+                                                   |
|   | E | T | M | E |                                                   |
|   +---+---+---+---+                                                   |
|   | S | S | A | G |                                                   |
|   +---+---+---+---+                                                   |
|   | E | ! | ! | ! |                                                   |
|   +---+---+---+---+                                                   |
|           |                                                           |
|           v                                                           |
|   +------------------+                                                |
|   | Round 1-10/12/14 |  (10 rounds for AES-128)                       |
|   +------------------+                                                |
|   |                  |                                                |
|   | 1. SubBytes      | Substitute each byte using S-box              |
|   | 2. ShiftRows     | Shift rows left by different amounts          |
|   | 3. MixColumns    | Mix columns (except last round)               |
|   | 4. AddRoundKey   | XOR with round key                            |
|   |                  |                                                |
|   +------------------+                                                |
|           |                                                           |
|           v                                                           |
|   Ciphertext (128-bit block)                                          |
|   +---+---+---+---+                                                   |
|   | a | 7 | * | # |                                                   |
|   +---+---+---+---+                                                   |
|   | @ | b | $ | 2 |                                                   |
|   +---+---+---+---+                                                   |
|   | x | ! | c | % |                                                   |
|   +---+---+---+---+                                                   |
|   | 9 | ^ | & | d |                                                   |
|   +---+---+---+---+                                                   |
|                                                                       |
+-----------------------------------------------------------------------+
```

### Block Cipher Modes (블록 암호 모드)

```
+-----------------------------------------------------------------------+
|                      ECB Mode (Electronic Codebook)                    |
+-----------------------------------------------------------------------+
|                                                                       |
|   Problem: Same plaintext block = Same ciphertext block               |
|   문제: 동일한 평문 블록 = 동일한 암호문 블록                         |
|                                                                       |
|   Plaintext:  [Block A] [Block B] [Block A] [Block C]                 |
|                   |         |         |         |                     |
|                   v         v         v         v                     |
|               +-------+ +-------+ +-------+ +-------+                 |
|               |  AES  | |  AES  | |  AES  | |  AES  |                 |
|               +-------+ +-------+ +-------+ +-------+                 |
|                   |         |         |         |                     |
|                   v         v         v         v                     |
|   Ciphertext: [Cipher X][Cipher Y][Cipher X][Cipher Z]                |
|                   ^                   ^                               |
|                   |___________________|                               |
|                   Same input = Same output (BAD!)                     |
|                                                                       |
|   Do NOT use ECB for anything other than single blocks!               |
|   단일 블록 외에는 ECB를 사용하지 마세요!                             |
|                                                                       |
+-----------------------------------------------------------------------+

+-----------------------------------------------------------------------+
|                      CBC Mode (Cipher Block Chaining)                  |
+-----------------------------------------------------------------------+
|                                                                       |
|   Solution: Chain blocks together using previous ciphertext           |
|   해결책: 이전 암호문을 사용하여 블록을 연결                          |
|                                                                       |
|        IV        Ciphertext 1   Ciphertext 2                          |
|         |             |              |                                |
|         v             v              v                                |
|   [Block 1]--XOR-->[Block 2]--XOR-->[Block 3]                         |
|         |             |              |                                |
|         v             v              v                                |
|     +-------+     +-------+      +-------+                            |
|     |  AES  |     |  AES  |      |  AES  |                            |
|     +-------+     +-------+      +-------+                            |
|         |             |              |                                |
|         v             v              v                                |
|   [Cipher 1]    [Cipher 2]     [Cipher 3]                             |
|                                                                       |
|   IV (Initialization Vector): Random starting value                   |
|   IV (초기화 벡터): 무작위 시작 값                                    |
|                                                                       |
+-----------------------------------------------------------------------+
```

### Symmetric Encryption: Pros and Cons (장단점)

```
+---------------------------+-------------------------------------------+
| Advantages (장점)         | Disadvantages (단점)                      |
+---------------------------+-------------------------------------------+
| Fast (빠름)               | Key distribution problem                  |
|                           | (키 배포 문제)                            |
+---------------------------+-------------------------------------------+
| Efficient for large data  | Key must be shared securely               |
| (대용량 데이터에 효율적)  | (키를 안전하게 공유해야 함)               |
+---------------------------+-------------------------------------------+
| Less computationally      | N users need N(N-1)/2 keys                |
| intensive                 | (N명의 사용자는 N(N-1)/2개의 키 필요)     |
| (계산 부담 적음)          |                                           |
+---------------------------+-------------------------------------------+

Key Distribution Problem:
+-----------------------------------------------------------------------+
|                                                                       |
|   How does Alice securely share the key with Bob?                     |
|   Alice가 어떻게 Bob에게 안전하게 키를 공유할 수 있을까?              |
|                                                                       |
|   If they meet in person: Inconvenient                                |
|   직접 만나면: 불편함                                                 |
|                                                                       |
|   If they send it over network: Attacker can intercept!               |
|   네트워크로 보내면: 공격자가 가로챌 수 있음!                         |
|                                                                       |
|   Solution: Use asymmetric encryption for key exchange                |
|   해결책: 키 교환에 비대칭 암호화 사용                                |
|                                                                       |
+-----------------------------------------------------------------------+
```

---

## 3. Asymmetric Key Encryption (비대칭키 암호화)

### What is Asymmetric Encryption? (비대칭키 암호화란?)

Asymmetric encryption uses a **pair of keys**: a public key and a private key. What one key encrypts, only the other can decrypt.

비대칭키 암호화는 **한 쌍의 키**를 사용합니다: 공개키와 개인키. 한 키로 암호화한 것은 다른 키로만 복호화할 수 있습니다.

```
+-----------------------------------------------------------------------+
|                    Asymmetric Encryption                               |
+-----------------------------------------------------------------------+
|                                                                       |
|   Bob's Key Pair:                                                     |
|   +------------------+    +------------------+                         |
|   | Public Key       |    | Private Key      |                         |
|   | (Everyone knows) |    | (Only Bob knows) |                         |
|   +------------------+    +------------------+                         |
|                                                                       |
|   Alice                                               Bob             |
|   +-----+                                           +-----+           |
|   |     |                                           |     |           |
|   +-----+                                           +-----+           |
|      |                                                 |              |
|      | Gets Bob's public key                           |              |
|      |                                                 |              |
|      | Plaintext: "Hello"                              |              |
|      |    +                                            |              |
|      | Bob's Public Key                                |              |
|      |                                                 |              |
|      v                                                 |              |
|   +--------+                                      +--------+          |
|   |Encrypt |                                      |Decrypt |          |
|   +--------+                                      +--------+          |
|      |                                                 ^              |
|      | Ciphertext: "x#$%^&"                            |              |
|      +------------------------>------------------------+              |
|                                                        |              |
|                                          Bob's Private Key            |
|                                          (ONLY Bob has this!)         |
|                                                        |              |
|                                                        v              |
|                                          Plaintext: "Hello"           |
|                                                                       |
+-----------------------------------------------------------------------+
```

### RSA Algorithm (RSA 알고리즘)

```
+-----------------------------------------------------------------------+
|                         RSA Overview                                   |
+-----------------------------------------------------------------------+
|                                                                       |
|   Named after creators: Rivest, Shamir, Adleman (1977)                |
|   창시자 이름: Rivest, Shamir, Adleman (1977년)                       |
|                                                                       |
|   Based on the mathematical difficulty of:                            |
|   - Factoring large prime numbers                                     |
|   - 큰 소수의 인수분해 어려움에 기반                                  |
|                                                                       |
|   Key Sizes:                                                          |
|   +----------------+--------------------------------+                  |
|   | Key Size       | Security Level                |                  |
|   +----------------+--------------------------------+                  |
|   | 1024-bit       | Deprecated (취약)             |                  |
|   | 2048-bit       | Minimum recommended (권장)   |                  |
|   | 4096-bit       | High security (고보안)        |                  |
|   +----------------+--------------------------------+                  |
|                                                                       |
+-----------------------------------------------------------------------+
```

### RSA Key Generation (Simplified) (RSA 키 생성 - 간략화)

```
+-----------------------------------------------------------------------+
|                    RSA Key Generation Steps                            |
+-----------------------------------------------------------------------+
|                                                                       |
|   Step 1: Choose two large prime numbers p and q                      |
|           두 개의 큰 소수 p와 q 선택                                  |
|           Example: p = 61, q = 53                                     |
|                                                                       |
|   Step 2: Compute n = p x q                                           |
|           n = p x q 계산                                              |
|           n = 61 x 53 = 3233                                          |
|                                                                       |
|   Step 3: Compute phi(n) = (p-1) x (q-1)                              |
|           phi(n) = (p-1) x (q-1) 계산                                 |
|           phi(n) = 60 x 52 = 3120                                     |
|                                                                       |
|   Step 4: Choose e such that 1 < e < phi(n) and gcd(e, phi(n)) = 1    |
|           e 선택: 1 < e < phi(n)이고 gcd(e, phi(n)) = 1               |
|           e = 17                                                      |
|                                                                       |
|   Step 5: Compute d such that d x e mod phi(n) = 1                    |
|           d 계산: d x e mod phi(n) = 1                                |
|           d = 2753                                                    |
|                                                                       |
|   +------------------------+------------------------+                  |
|   | Public Key             | Private Key            |                  |
|   | (e, n) = (17, 3233)    | (d, n) = (2753, 3233)  |                  |
|   +------------------------+------------------------+                  |
|                                                                       |
+-----------------------------------------------------------------------+
```

### RSA Encryption/Decryption (RSA 암호화/복호화)

```
+-----------------------------------------------------------------------+
|                    RSA Encryption/Decryption                           |
+-----------------------------------------------------------------------+
|                                                                       |
|   Encryption (암호화):                                                |
|   +---------------------------------------------------------------+   |
|   | C = M^e mod n                                                 |   |
|   | where M = plaintext message (as number)                       |   |
|   |       C = ciphertext                                          |   |
|   |       e = public exponent                                     |   |
|   |       n = modulus                                             |   |
|   +---------------------------------------------------------------+   |
|                                                                       |
|   Example: Encrypt message M = 65 (letter 'A')                        |
|   C = 65^17 mod 3233 = 2790                                           |
|                                                                       |
|   Decryption (복호화):                                                |
|   +---------------------------------------------------------------+   |
|   | M = C^d mod n                                                 |   |
|   | where C = ciphertext                                          |   |
|   |       M = decrypted plaintext                                 |   |
|   |       d = private exponent                                    |   |
|   |       n = modulus                                             |   |
|   +---------------------------------------------------------------+   |
|                                                                       |
|   Example: Decrypt ciphertext C = 2790                                |
|   M = 2790^2753 mod 3233 = 65 (back to 'A')                           |
|                                                                       |
+-----------------------------------------------------------------------+
```

### Symmetric vs Asymmetric Comparison (대칭키 vs 비대칭키 비교)

```
+----------------------+------------------------+------------------------+
| Feature              | Symmetric              | Asymmetric             |
|----------------------|------------------------|------------------------|
| Keys                 | One shared key         | Key pair (public +     |
| 키                   | 하나의 공유 키         | private) 키 쌍         |
+----------------------+------------------------+------------------------+
| Speed                | Fast                   | Slow (100-1000x)       |
| 속도                 | 빠름                   | 느림 (100-1000배)      |
+----------------------+------------------------+------------------------+
| Key Distribution     | Difficult              | Easy (public key       |
| 키 배포              | 어려움                 | can be shared openly)  |
+----------------------+------------------------+------------------------+
| Key Size             | 128-256 bits           | 2048-4096 bits         |
| 키 크기              |                        |                        |
+----------------------+------------------------+------------------------+
| Use Case             | Bulk data encryption   | Key exchange,          |
| 용도                 | 대용량 데이터 암호화   | digital signatures     |
+----------------------+------------------------+------------------------+
| Examples             | AES, ChaCha20          | RSA, ECC, DSA          |
| 예시                 |                        |                        |
+----------------------+------------------------+------------------------+
```

### Hybrid Encryption (하이브리드 암호화)

```
+-----------------------------------------------------------------------+
|                    Hybrid Encryption (TLS uses this!)                  |
+-----------------------------------------------------------------------+
|                                                                       |
|   Combine the best of both worlds:                                    |
|   두 방식의 장점을 결합:                                              |
|                                                                       |
|   1. Use asymmetric encryption to exchange symmetric key              |
|      비대칭 암호화로 대칭키 교환                                      |
|                                                                       |
|   2. Use symmetric encryption for actual data                         |
|      실제 데이터는 대칭 암호화 사용                                   |
|                                                                       |
|   Alice                                               Bob             |
|   +-----+                                           +-----+           |
|   |     |                                           |     |           |
|   +-----+                                           +-----+           |
|      |                                                 |              |
|      |  1. Generate random symmetric key (AES key)     |              |
|      |     무작위 대칭키 생성                          |              |
|      |                                                 |              |
|      |  2. Encrypt AES key with Bob's public key       |              |
|      |     Bob의 공개키로 AES 키 암호화                |              |
|      |------------------------------------------>      |              |
|      |  3. Bob decrypts with private key               |              |
|      |     Bob이 개인키로 복호화                       |              |
|      |                                                 |              |
|      |  4. Now both have the same AES key!             |              |
|      |     이제 둘 다 동일한 AES 키 보유!              |              |
|      |                                                 |              |
|      |  5. Encrypt actual data with AES (fast!)        |              |
|      |     AES로 실제 데이터 암호화 (빠름!)            |              |
|      |<===========================================     |              |
|                                                                       |
+-----------------------------------------------------------------------+
```

---

## 4. Hash Functions (해시 함수)

### What is a Hash Function? (해시 함수란?)

A hash function takes input of any size and produces a **fixed-size output** (hash/digest). It's a one-way function - you cannot reverse it to get the original input.

해시 함수는 임의 크기의 입력을 받아 **고정 크기의 출력**(해시/다이제스트)을 생성합니다. 단방향 함수로, 원본 입력을 역산할 수 없습니다.

```
+-----------------------------------------------------------------------+
|                    Hash Function Properties                            |
+-----------------------------------------------------------------------+
|                                                                       |
|   Input (any size)                                                    |
|   +-----------------------------------------------+                   |
|   | "Hello World" or entire file or anything...  |                   |
|   +-----------------------------------------------+                   |
|                      |                                                |
|                      v                                                |
|               +-------------+                                         |
|               | Hash Function|                                        |
|               +-------------+                                         |
|                      |                                                |
|                      v                                                |
|   Output (fixed size, e.g., 256 bits for SHA-256)                     |
|   +-----------------------------------------------+                   |
|   | a591a6d40bf420404a011733cfb7b190d62c65bf0bcda |                   |
|   +-----------------------------------------------+                   |
|                                                                       |
|   Properties:                                                         |
|   1. Deterministic - Same input always gives same output              |
|      결정적 - 동일 입력은 항상 동일 출력                              |
|                                                                       |
|   2. One-way - Cannot reverse hash to find input                      |
|      단방향 - 해시에서 입력을 역산 불가                               |
|                                                                       |
|   3. Collision resistant - Hard to find two inputs with same hash     |
|      충돌 저항성 - 같은 해시를 가진 두 입력 찾기 어려움               |
|                                                                       |
|   4. Avalanche effect - Small change = completely different hash      |
|      눈사태 효과 - 작은 변화 = 완전히 다른 해시                       |
|                                                                       |
+-----------------------------------------------------------------------+
```

### Avalanche Effect Example (눈사태 효과 예시)

```
+-----------------------------------------------------------------------+
|                    Avalanche Effect                                    |
+-----------------------------------------------------------------------+
|                                                                       |
|   Input 1: "Hello"                                                    |
|   SHA-256: 185f8db32271fe25f561a6fc938b2e264306ec304eda518007d1764826  |
|                                                                       |
|   Input 2: "hello" (just lowercase h)                                 |
|   SHA-256: 2cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c1fa7425e73043362938  |
|                                                                       |
|   Input 3: "Hello!" (added exclamation)                               |
|   SHA-256: 334d016f755cd6dc58c53a86e183882f8ec14f52fb05345887c8a5edd42  |
|                                                                       |
|   Notice: Completely different outputs for slightly different inputs! |
|   주목: 약간 다른 입력에 완전히 다른 출력!                            |
|                                                                       |
+-----------------------------------------------------------------------+
```

### Common Hash Algorithms (주요 해시 알고리즘)

```
+-----------------------------------------------------------------------+
|                    Hash Algorithm Comparison                           |
+-----------------------------------------------------------------------+
|                                                                       |
|   +------------+-----------+------------+-----------------------------+
|   | Algorithm  | Output    | Status     | Use Case                    |
|   +------------+-----------+------------+-----------------------------+
|   | MD5        | 128 bits  | BROKEN     | Legacy systems only         |
|   |            |           | 취약       | 레거시 시스템만             |
|   +------------+-----------+------------+-----------------------------+
|   | SHA-1      | 160 bits  | DEPRECATED | Avoid for security          |
|   |            |           | 권장안함   | 보안용 사용 금지            |
|   +------------+-----------+------------+-----------------------------+
|   | SHA-256    | 256 bits  | SECURE     | General use, TLS            |
|   | (SHA-2)    |           | 안전       | 일반 용도, TLS              |
|   +------------+-----------+------------+-----------------------------+
|   | SHA-384    | 384 bits  | SECURE     | High security               |
|   | (SHA-2)    |           | 안전       | 고보안                      |
|   +------------+-----------+------------+-----------------------------+
|   | SHA-512    | 512 bits  | SECURE     | Maximum security            |
|   | (SHA-2)    |           | 안전       | 최대 보안                   |
|   +------------+-----------+------------+-----------------------------+
|   | SHA-3      | Variable  | SECURE     | Future standard             |
|   |            |           | 안전       | 미래 표준                   |
|   +------------+-----------+------------+-----------------------------+
|                                                                       |
+-----------------------------------------------------------------------+
```

### Hash Function Use Cases (해시 함수 사용 사례)

```
+-----------------------------------------------------------------------+
|                    Use Case 1: Password Storage                        |
+-----------------------------------------------------------------------+
|                                                                       |
|   WRONG way (잘못된 방법):                                            |
|   Store password directly: password123                                |
|   비밀번호 직접 저장                                                  |
|                                                                       |
|   BETTER way (더 나은 방법):                                          |
|   Store hash: SHA256(password123) = ef92b778...                       |
|   해시 저장                                                           |
|                                                                       |
|   BEST way (최선의 방법):                                             |
|   Store salted hash: SHA256(salt + password123)                       |
|   솔트된 해시 저장                                                    |
|                                                                       |
|   +--------------------+-------------------------------+              |
|   | User               | Stored Value                  |              |
|   +--------------------+-------------------------------+              |
|   | alice              | $2b$12$salt...hashedpassword  |              |
|   | bob                | $2b$12$salt...hashedpassword  |              |
|   +--------------------+-------------------------------+              |
|                                                                       |
+-----------------------------------------------------------------------+

+-----------------------------------------------------------------------+
|                    Use Case 2: File Integrity                          |
+-----------------------------------------------------------------------+
|                                                                       |
|   Download verification (다운로드 검증):                              |
|                                                                       |
|   1. Website shows: "File.zip SHA-256: abc123..."                     |
|   2. User downloads file                                              |
|   3. User computes: SHA256(downloaded_file)                           |
|   4. Compare hashes                                                   |
|      - Match: File is authentic                                       |
|      - Mismatch: File may be corrupted or tampered                    |
|                                                                       |
+-----------------------------------------------------------------------+

+-----------------------------------------------------------------------+
|                    Use Case 3: Data Deduplication                      |
+-----------------------------------------------------------------------+
|                                                                       |
|   Cloud storage (클라우드 스토리지):                                  |
|                                                                       |
|   File1.pdf --> Hash: abc123                                          |
|   File2.pdf --> Hash: abc123  (Same hash = same content!)             |
|   File3.pdf --> Hash: def456                                          |
|                                                                       |
|   Only store unique files, save space!                                |
|   고유 파일만 저장, 공간 절약!                                        |
|                                                                       |
+-----------------------------------------------------------------------+
```

---

## 5. Digital Signatures (디지털 서명)

### What is a Digital Signature? (디지털 서명이란?)

A digital signature provides **authentication** (proves who sent the message), **integrity** (proves the message wasn't changed), and **non-repudiation** (sender cannot deny sending).

디지털 서명은 **인증**(누가 메시지를 보냈는지 증명), **무결성**(메시지가 변경되지 않았음 증명), **부인 방지**(발신자가 보낸 것을 부인할 수 없음)를 제공합니다.

```
+-----------------------------------------------------------------------+
|                    Digital Signature Overview                          |
+-----------------------------------------------------------------------+
|                                                                       |
|   Unlike encryption (which hides content),                            |
|   signatures PROVE authenticity and integrity                         |
|                                                                       |
|   암호화(내용 숨김)와 달리,                                           |
|   서명은 진위성과 무결성을 증명                                       |
|                                                                       |
|   +---------------------+--------------------------------------------+|
|   | Property            | What it means                              ||
|   +---------------------+--------------------------------------------+|
|   | Authentication      | "This is really from Alice"               ||
|   | 인증                | "이것은 정말 Alice가 보낸 것임"            ||
|   +---------------------+--------------------------------------------+|
|   | Integrity           | "The message hasn't been changed"         ||
|   | 무결성              | "메시지가 변경되지 않음"                   ||
|   +---------------------+--------------------------------------------+|
|   | Non-repudiation     | "Alice cannot deny she sent this"         ||
|   | 부인 방지           | "Alice는 보낸 것을 부인할 수 없음"         ||
|   +---------------------+--------------------------------------------+|
|                                                                       |
+-----------------------------------------------------------------------+
```

### How Digital Signatures Work (디지털 서명 동작 방식)

```
+-----------------------------------------------------------------------+
|                    Digital Signature Process                           |
+-----------------------------------------------------------------------+
|                                                                       |
|   SIGNING (서명하기) - Alice's side                                   |
|   +---------------------------------------------------------------+   |
|   |                                                               |   |
|   |   Original Message                                            |   |
|   |   +-----------------------+                                   |   |
|   |   | "Transfer $1000 to   |                                   |   |
|   |   |  Bob's account"      |                                   |   |
|   |   +-----------------------+                                   |   |
|   |              |                                                |   |
|   |              v                                                |   |
|   |       +------------+                                          |   |
|   |       | Hash (SHA) |                                          |   |
|   |       +------------+                                          |   |
|   |              |                                                |   |
|   |              v                                                |   |
|   |   Message Digest: "abc123..."                                 |   |
|   |              |                                                |   |
|   |              v                                                |   |
|   |   +-------------------+                                       |   |
|   |   | Encrypt with      |                                       |   |
|   |   | Alice's PRIVATE   |  <-- Only Alice has this!             |   |
|   |   | key               |                                       |   |
|   |   +-------------------+                                       |   |
|   |              |                                                |   |
|   |              v                                                |   |
|   |   Digital Signature: "xyz789..."                              |   |
|   |                                                               |   |
|   +---------------------------------------------------------------+   |
|                                                                       |
|   Send: [Message] + [Signature]                                       |
|                                                                       |
+-----------------------------------------------------------------------+

+-----------------------------------------------------------------------+
|   VERIFICATION (검증하기) - Bob's side                                |
+-----------------------------------------------------------------------+
|                                                                       |
|   Received: [Message] + [Signature]                                   |
|                                                                       |
|   +-------------------------+     +-------------------------+         |
|   | Step 1: Hash received   |     | Step 2: Decrypt         |         |
|   | message                 |     | signature with Alice's  |         |
|   |                         |     | PUBLIC key              |         |
|   +------------+------------+     +------------+------------+         |
|                |                               |                      |
|                v                               v                      |
|        Hash A: "abc123..."           Hash B: "abc123..."              |
|                |                               |                      |
|                +---------------+---------------+                      |
|                                |                                      |
|                                v                                      |
|                        +--------------+                               |
|                        | Compare      |                               |
|                        | Hash A = B?  |                               |
|                        +--------------+                               |
|                           |       |                                   |
|                          YES      NO                                  |
|                           |       |                                   |
|                           v       v                                   |
|                     Valid!    Invalid!                                |
|                     유효!     무효!                                   |
|                                                                       |
+-----------------------------------------------------------------------+
```

### Signature vs Encryption Direction (서명 vs 암호화 방향)

```
+-----------------------------------------------------------------------+
|                    Key Usage Comparison                                |
+-----------------------------------------------------------------------+
|                                                                       |
|   ENCRYPTION (암호화):                                                |
|   Purpose: Hide content from everyone except recipient                |
|   목적: 수신자 외에 모든 사람으로부터 내용 숨김                       |
|                                                                       |
|   Encrypt with: RECIPIENT'S PUBLIC key                                |
|   Decrypt with: RECIPIENT'S PRIVATE key                               |
|                                                                       |
|   "Anyone can encrypt, only recipient can decrypt"                    |
|   "누구나 암호화 가능, 수신자만 복호화 가능"                          |
|                                                                       |
+-----------------------------------------------------------------------+
|                                                                       |
|   DIGITAL SIGNATURE (디지털 서명):                                    |
|   Purpose: Prove who sent the message                                 |
|   목적: 누가 메시지를 보냈는지 증명                                   |
|                                                                       |
|   Sign with: SENDER'S PRIVATE key                                     |
|   Verify with: SENDER'S PUBLIC key                                    |
|                                                                       |
|   "Only sender can sign, anyone can verify"                           |
|   "송신자만 서명 가능, 누구나 검증 가능"                              |
|                                                                       |
+-----------------------------------------------------------------------+
```

### Common Signature Algorithms (주요 서명 알고리즘)

```
+----------------------+------------------------------------------------+
| Algorithm            | Description                                    |
+----------------------+------------------------------------------------+
| RSA                  | Based on RSA encryption                        |
|                      | RSA 암호화 기반                                |
+----------------------+------------------------------------------------+
| DSA                  | Digital Signature Algorithm                    |
|                      | Signing only (cannot encrypt)                  |
|                      | 서명만 가능 (암호화 불가)                      |
+----------------------+------------------------------------------------+
| ECDSA                | Elliptic Curve DSA                             |
|                      | Smaller keys, faster                           |
|                      | 더 작은 키, 더 빠름                            |
+----------------------+------------------------------------------------+
| EdDSA (Ed25519)      | Modern, fast, secure                           |
|                      | Used in SSH, TLS 1.3                           |
|                      | 현대적, 빠름, 안전                             |
+----------------------+------------------------------------------------+
```

---

## 6. Summary (요약)

### Key Concepts (핵심 개념)

```
+-----------------------------------------------------------------------+
|                     Chapter 13 Summary                                 |
+-----------------------------------------------------------------------+
|                                                                       |
|  1. Why Encryption (암호화가 필요한 이유)                             |
|     - Protect data confidentiality, integrity, authenticity           |
|     - CIA triad: Confidentiality, Integrity, Availability             |
|     - Encryption != Hashing != Encoding                               |
|                                                                       |
|  2. Symmetric Encryption (대칭키 암호화)                              |
|     - Same key for encryption and decryption                          |
|     - Fast, good for large data                                       |
|     - AES is the standard (128/192/256 bits)                          |
|     - Key distribution is a challenge                                 |
|                                                                       |
|  3. Asymmetric Encryption (비대칭키 암호화)                           |
|     - Key pair: public and private                                    |
|     - Slow, but solves key distribution                               |
|     - RSA is common (2048+ bits)                                      |
|     - Used with symmetric encryption (hybrid)                         |
|                                                                       |
|  4. Hash Functions (해시 함수)                                        |
|     - One-way, fixed output                                           |
|     - SHA-256 is recommended                                          |
|     - Uses: passwords, integrity, deduplication                       |
|     - Avalanche effect: small change = big difference                 |
|                                                                       |
|  5. Digital Signatures (디지털 서명)                                  |
|     - Sign with private key, verify with public key                   |
|     - Provides: authentication, integrity, non-repudiation            |
|     - Hash message, encrypt hash with private key                     |
|                                                                       |
+-----------------------------------------------------------------------+
```

### Cryptography Cheat Sheet (암호화 요약표)

```
+------------------+------------------+------------------+------------------+
| Need             | Use              | Example          | Key              |
+------------------+------------------+------------------+------------------+
| Hide data        | Symmetric        | AES-256          | Shared secret    |
| 데이터 숨김      | encryption       |                  | 공유 비밀       |
+------------------+------------------+------------------+------------------+
| Exchange key     | Asymmetric       | RSA-2048         | Public/Private   |
| 키 교환          | encryption       |                  | 공개/개인       |
+------------------+------------------+------------------+------------------+
| Verify integrity | Hash function    | SHA-256          | None             |
| 무결성 검증      |                  |                  | 없음             |
+------------------+------------------+------------------+------------------+
| Prove identity   | Digital          | RSA, ECDSA       | Private to sign  |
| 신원 증명        | signature        |                  | 개인키로 서명   |
+------------------+------------------+------------------+------------------+
| Store password   | Hash + salt      | bcrypt, Argon2   | Random salt      |
| 비밀번호 저장    |                  |                  | 무작위 솔트     |
+------------------+------------------+------------------+------------------+
```

### Security Best Practices (보안 모범 사례)

```
+-----------------------------------------------------------------------+
|                    Security Recommendations                            |
+-----------------------------------------------------------------------+
|                                                                       |
|   DO (해야 할 것):                                                    |
|   - Use AES-256 for symmetric encryption                              |
|   - Use RSA-2048+ or ECDSA for asymmetric                             |
|   - Use SHA-256 or SHA-3 for hashing                                  |
|   - Use bcrypt/Argon2 for passwords                                   |
|   - Use TLS 1.3 for communication                                     |
|                                                                       |
|   DON'T (하지 말아야 할 것):                                          |
|   - Never use MD5 or SHA-1 for security                               |
|   - Never use ECB mode for multi-block encryption                     |
|   - Never store passwords in plaintext                                |
|   - Never implement your own cryptography                             |
|   - Never reuse encryption keys or IVs                                |
|                                                                       |
+-----------------------------------------------------------------------+
```

---

*End of Chapter 13*
