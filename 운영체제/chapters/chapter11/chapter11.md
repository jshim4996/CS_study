# Chapter 11. File System Implementation (파일 시스템 구현)

## Overview (개요)

This chapter covers how file systems are implemented on disk. We examine the structures used to organize files, allocate disk space, and manage free space efficiently.

이 장에서는 파일 시스템이 디스크에 어떻게 구현되는지 다룹니다. 파일을 구성하고, 디스크 공간을 할당하고, 여유 공간을 효율적으로 관리하는 데 사용되는 구조를 살펴봅니다.

---

## 1. File System Structure (파일 시스템 구조)

### 1.1 Layered File System (계층적 파일 시스템)

The file system is organized in layers, each providing abstraction to the layer above.

파일 시스템은 계층으로 구성되어 있으며, 각 계층은 상위 계층에 추상화를 제공합니다.

```
+------------------------------------------------------------------+
|                  FILE SYSTEM LAYERS                              |
|                  (파일 시스템 계층)                               |
+------------------------------------------------------------------+
|                                                                  |
|   +----------------------------------------------------------+   |
|   |            Application Programs (응용 프로그램)           |   |
|   +----------------------------------------------------------+   |
|                              |                                   |
|                              v                                   |
|   +----------------------------------------------------------+   |
|   |         Logical File System (논리적 파일 시스템)          |   |
|   |    - Manages metadata (메타데이터 관리)                   |   |
|   |    - Directory structure (디렉토리 구조)                  |   |
|   |    - Protection/security (보호/보안)                      |   |
|   +----------------------------------------------------------+   |
|                              |                                   |
|                              v                                   |
|   +----------------------------------------------------------+   |
|   |     File-organization Module (파일 구성 모듈)             |   |
|   |    - Logical to physical block mapping                    |   |
|   |    - 논리적 블록을 물리적 블록으로 매핑                    |   |
|   |    - Free space management (여유 공간 관리)               |   |
|   +----------------------------------------------------------+   |
|                              |                                   |
|                              v                                   |
|   +----------------------------------------------------------+   |
|   |          Basic File System (기본 파일 시스템)             |   |
|   |    - Issues read/write commands to device driver          |   |
|   |    - 장치 드라이버에 읽기/쓰기 명령 발행                   |   |
|   |    - Manages buffers and caches                           |   |
|   |    - 버퍼와 캐시 관리                                     |   |
|   +----------------------------------------------------------+   |
|                              |                                   |
|                              v                                   |
|   +----------------------------------------------------------+   |
|   |            I/O Control (입출력 제어)                      |   |
|   |    - Device drivers (장치 드라이버)                       |   |
|   |    - Interrupt handlers (인터럽트 핸들러)                 |   |
|   +----------------------------------------------------------+   |
|                              |                                   |
|                              v                                   |
|   +----------------------------------------------------------+   |
|   |                 Devices (장치)                            |   |
|   |    - Physical disk/SSD (물리적 디스크/SSD)                |   |
|   +----------------------------------------------------------+   |
|                                                                  |
+------------------------------------------------------------------+
```

### 1.2 On-Disk File System Structures (디스크 상의 파일 시스템 구조)

```
+------------------------------------------------------------------+
|               TYPICAL DISK LAYOUT                                |
|               (일반적인 디스크 레이아웃)                          |
+------------------------------------------------------------------+
|                                                                  |
|   +------+--------+--------+--------+--------------------------+ |
|   | Boot | Super  | Free   | Inode  |      Data Blocks         | |
|   |Block | Block  | Space  | Table  |      (데이터 블록)        | |
|   |      |        | Info   |        |                          | |
|   +------+--------+--------+--------+--------------------------+ |
|      |       |        |        |                                 |
|      |       |        |        +-- File metadata (파일 메타데이터)|
|      |       |        +-- Bitmap or linked list (비트맵/연결 리스트)|
|      |       +-- FS info: size, block count, etc.                |
|      |          (FS 정보: 크기, 블록 수 등)                       |
|      +-- Bootstrap code (부트스트랩 코드)                         |
|                                                                  |
+------------------------------------------------------------------+
|                                                                  |
|   INODE STRUCTURE (아이노드 구조):                               |
|                                                                  |
|   +---------------------------+                                  |
|   |  File Attributes (속성)   |                                  |
|   |  - Mode/permissions       |                                  |
|   |  - Owner/Group ID         |                                  |
|   |  - Size (크기)            |                                  |
|   |  - Timestamps (시간정보)  |                                  |
|   +---------------------------+                                  |
|   |  Direct Pointers (직접)   |  --> [Block 0]                   |
|   |  [0] [1] [2] ... [11]     |  --> [Block 1] ... [Block 11]    |
|   +---------------------------+                                  |
|   |  Single Indirect (단일)   |  --> [Pointer Block]             |
|   +---------------------------+       |                          |
|   |  Double Indirect (이중)   |       +--> [Block 12]            |
|   +---------------------------+       +--> [Block 13]            |
|   |  Triple Indirect (삼중)   |       +--> ...                   |
|   +---------------------------+                                  |
|                                                                  |
+------------------------------------------------------------------+
```

---

## 2. Disk Allocation Methods (디스크 할당 방법)

Three major methods exist to allocate disk space to files:

파일에 디스크 공간을 할당하는 세 가지 주요 방법이 있습니다:

### 2.1 Contiguous Allocation (연속 할당)

Each file occupies a set of contiguous blocks on the disk.

각 파일은 디스크에서 연속적인 블록 집합을 차지합니다.

```
+------------------------------------------------------------------+
|                  CONTIGUOUS ALLOCATION                           |
|                  (연속 할당)                                      |
+------------------------------------------------------------------+
|                                                                  |
|   Directory Entry (디렉토리 항목):                               |
|   +----------+--------------+--------+                           |
|   | Filename | Start Block  | Length |                           |
|   +----------+--------------+--------+                           |
|   | file_a   |      0       |   3    |                           |
|   | file_b   |      6       |   4    |                           |
|   | file_c   |     14       |   5    |                           |
|   +----------+--------------+--------+                           |
|                                                                  |
|   Disk Blocks (디스크 블록):                                     |
|   +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+   |
|   | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 |10 |11 |12 |13 |14 |  |
|   +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+   |
|   |<-file_a->|       |<-- file_b -->|               |<-file_c   |
|                                                                  |
|   Access to block i of file:                                     |
|   Physical Block = Start Block + i                               |
|   파일의 블록 i 접근: 물리적 블록 = 시작 블록 + i                 |
|                                                                  |
+------------------------------------------------------------------+
|                                                                  |
|   Advantages (장점):                                             |
|   + Simple implementation (간단한 구현)                          |
|   + Excellent sequential access (우수한 순차 접근)               |
|   + Fast direct access (빠른 직접 접근)                          |
|   + Minimal seek time (최소 탐색 시간)                           |
|                                                                  |
|   Disadvantages (단점):                                          |
|   - External fragmentation (외부 단편화)                         |
|   - File size must be known at creation (생성 시 파일 크기 필요) |
|   - Difficult to grow files (파일 확장 어려움)                   |
|                                                                  |
+------------------------------------------------------------------+
|                                                                  |
|   EXTERNAL FRAGMENTATION PROBLEM (외부 단편화 문제):             |
|                                                                  |
|   After deletions (삭제 후):                                     |
|   +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+   |
|   | A | A | A |   |   | B | B | B | B |   |   |   | C | C | C |   |
|   +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+   |
|               |___|                   |_______|                  |
|                2 free                  3 free                    |
|                                                                  |
|   New file needs 4 blocks: CANNOT FIT! (4블록 필요: 불가!)       |
|   Solution: Compaction (해결책: 압축) - expensive operation      |
|                                                                  |
+------------------------------------------------------------------+
```

### 2.2 Linked Allocation (연결 할당)

Each file is a linked list of disk blocks. Blocks may be scattered anywhere on disk.

각 파일은 디스크 블록의 연결 리스트입니다. 블록은 디스크 어디에나 흩어져 있을 수 있습니다.

```
+------------------------------------------------------------------+
|                    LINKED ALLOCATION                             |
|                    (연결 할당)                                    |
+------------------------------------------------------------------+
|                                                                  |
|   Directory Entry (디렉토리 항목):                               |
|   +----------+-------------+------------+                        |
|   | Filename | Start Block | End Block  |                        |
|   +----------+-------------+------------+                        |
|   | file_a   |      2      |     15     |                        |
|   +----------+-------------+------------+                        |
|                                                                  |
|   Block Structure (블록 구조):                                   |
|   +------------------+----------+                                |
|   |      Data        |  Next    |                                |
|   |    (데이터)      | (다음)   |                                |
|   +------------------+----------+                                |
|                                                                  |
|   File "file_a" across disk (디스크에 걸친 "file_a"):            |
|                                                                  |
|   Directory                                                      |
|      |                                                           |
|      v                                                           |
|   +-------+     +-------+     +-------+     +--------+           |
|   |Block 2|     |Block 7|     |Block 9|     |Block 15|           |
|   +-------+     +-------+     +-------+     +--------+           |
|   | Data  |     | Data  |     | Data  |     | Data   |           |
|   +-------+     +-------+     +-------+     +--------+           |
|   |   7 --|---->|   9 --|---->|  15 --|---->|  -1    |           |
|   +-------+     +-------+     +-------+     +--------+           |
|                                              (End: -1)           |
|                                                                  |
+------------------------------------------------------------------+
|                                                                  |
|   Advantages (장점):                                             |
|   + No external fragmentation (외부 단편화 없음)                 |
|   + Files can grow easily (파일 확장 용이)                       |
|   + No need to know file size in advance                         |
|     (미리 파일 크기 알 필요 없음)                                 |
|                                                                  |
|   Disadvantages (단점):                                          |
|   - Sequential access only efficient (순차 접근만 효율적)        |
|   - Direct access very slow (직접 접근 매우 느림)                |
|   - Pointer space overhead (포인터 공간 오버헤드)                |
|   - Reliability: if one link breaks, data lost                   |
|     (신뢰성: 링크 하나 손상되면 데이터 손실)                      |
|                                                                  |
+------------------------------------------------------------------+
|                                                                  |
|   FILE ALLOCATION TABLE (FAT) - Improvement (개선):              |
|                                                                  |
|   FAT (stored at beginning of volume):                           |
|   FAT (볼륨 시작에 저장):                                        |
|                                                                  |
|   Block #   |  FAT Entry                                         |
|   ----------|------------                                        |
|      0      |    -1                                              |
|      1      |    -1                                              |
|      2      |     7    <-- Start of file_a                       |
|      3      |    -1                                              |
|      4      |    -1                                              |
|      5      |    -1                                              |
|      6      |    -1                                              |
|      7      |     9    <-- Points to next block                  |
|      8      |    -1                                              |
|      9      |    15                                              |
|      ...    |    ...                                             |
|      15     |    EOF   <-- End of file (-1)                      |
|                                                                  |
|   + FAT can be cached in memory (FAT를 메모리에 캐시 가능)       |
|   + Faster random access (빠른 임의 접근)                        |
|                                                                  |
+------------------------------------------------------------------+
```

### 2.3 Indexed Allocation (인덱스 할당)

Each file has its own index block, which contains pointers to all its data blocks.

각 파일에는 모든 데이터 블록에 대한 포인터를 포함하는 자체 인덱스 블록이 있습니다.

```
+------------------------------------------------------------------+
|                   INDEXED ALLOCATION                             |
|                   (인덱스 할당)                                   |
+------------------------------------------------------------------+
|                                                                  |
|   Directory Entry (디렉토리 항목):                               |
|   +----------+--------------+                                    |
|   | Filename | Index Block  |                                    |
|   +----------+--------------+                                    |
|   | file_a   |      5       |                                    |
|   +----------+--------------+                                    |
|                                                                  |
|                                                                  |
|                    Index Block 5                                 |
|                    (인덱스 블록 5)                                |
|                    +----------+                                  |
|                    |    2     |----> [Data Block 2]              |
|                    |    7     |----> [Data Block 7]              |
|                    |    9     |----> [Data Block 9]              |
|                    |   15     |----> [Data Block 15]             |
|                    |   23     |----> [Data Block 23]             |
|                    |   -1     | (null - unused)                  |
|                    |   ...    |                                  |
|                    +----------+                                  |
|                                                                  |
|   Direct Access to block i:                                      |
|   1. Read index block (인덱스 블록 읽기)                         |
|   2. Get pointer at position i (위치 i의 포인터 가져오기)        |
|   3. Read data block (데이터 블록 읽기)                          |
|                                                                  |
+------------------------------------------------------------------+
|                                                                  |
|   Advantages (장점):                                             |
|   + Supports direct access (직접 접근 지원)                      |
|   + No external fragmentation (외부 단편화 없음)                 |
|   + Files can grow dynamically (파일 동적 확장 가능)             |
|                                                                  |
|   Disadvantages (단점):                                          |
|   - Index block overhead (인덱스 블록 오버헤드)                  |
|   - Maximum file size limited by index block size                |
|     (인덱스 블록 크기에 의해 최대 파일 크기 제한)                 |
|                                                                  |
+------------------------------------------------------------------+
|                                                                  |
|   HANDLING LARGE FILES (대용량 파일 처리):                       |
|                                                                  |
|   1. Linked Scheme (연결 방식):                                  |
|      Index blocks linked together                                |
|      인덱스 블록들이 연결됨                                       |
|                                                                  |
|   2. Multi-level Index (다단계 인덱스):                          |
|                                                                  |
|      Level 1              Level 2           Data                 |
|      +--------+           +--------+        +------+             |
|      |   * ---|---------->|   2    |------->|Block2|             |
|      |   * ---|--+        |   7    |------->|Block7|             |
|      |  ...   |  |        |  ...   |        |  ... |             |
|      +--------+  |        +--------+        +------+             |
|                  |                                               |
|                  |        +--------+        +------+             |
|                  +------->|   23   |------->|Blk23 |             |
|                           |   45   |------->|Blk45 |             |
|                           |  ...   |        |  ... |             |
|                           +--------+        +------+             |
|                                                                  |
|   3. Combined Scheme (결합 방식) - Used in UNIX inode:           |
|      (UNIX 아이노드에서 사용)                                     |
|                                                                  |
|      +-------------------+                                       |
|      | Direct blocks     | --> Small files (작은 파일)           |
|      | (12 pointers)     |     (최대 48KB for 4KB blocks)        |
|      +-------------------+                                       |
|      | Single indirect   | --> Medium files (중간 파일)          |
|      | (1 pointer)       |     (추가 4MB)                        |
|      +-------------------+                                       |
|      | Double indirect   | --> Large files (큰 파일)             |
|      | (1 pointer)       |     (추가 4GB)                        |
|      +-------------------+                                       |
|      | Triple indirect   | --> Very large files (매우 큰 파일)   |
|      | (1 pointer)       |     (추가 4TB)                        |
|      +-------------------+                                       |
|                                                                  |
+------------------------------------------------------------------+
```

### Allocation Methods Comparison (할당 방법 비교)

| Feature (특징) | Contiguous (연속) | Linked (연결) | Indexed (인덱스) |
|---------------|-------------------|---------------|------------------|
| Sequential Access (순차 접근) | Excellent (우수) | Good (양호) | Good (양호) |
| Direct Access (직접 접근) | Excellent (우수) | Poor (나쁨) | Good (양호) |
| Space Efficiency (공간 효율) | Poor - fragmentation (나쁨 - 단편화) | Good (양호) | Moderate - index overhead (보통 - 인덱스 오버헤드) |
| File Growth (파일 확장) | Difficult (어려움) | Easy (쉬움) | Easy (쉬움) |
| Reliability (신뢰성) | High (높음) | Low - chain break (낮음 - 체인 끊김) | Moderate (보통) |

---

## 3. Free Space Management (여유 공간 관리)

The file system must track which disk blocks are free and which are allocated.

파일 시스템은 어떤 디스크 블록이 비어 있고 어떤 블록이 할당되었는지 추적해야 합니다.

### 3.1 Bit Vector (Bit Map) (비트 벡터/비트맵)

Each block is represented by 1 bit: 0 = free, 1 = allocated (or vice versa).

각 블록은 1비트로 표현됩니다: 0 = 비어있음, 1 = 할당됨 (또는 반대).

```
+------------------------------------------------------------------+
|                    BIT VECTOR METHOD                             |
|                    (비트 벡터 방식)                               |
+------------------------------------------------------------------+
|                                                                  |
|   Disk Blocks (디스크 블록):                                     |
|   +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+   |
|   | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 |10 |11 |12 |13 |14 |  |
|   +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+   |
|   | U | U | F | U | F | F | U | U | F | F | U | F | U | F | F |   |
|   +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+   |
|   (U = Used/사용중, F = Free/비어있음)                            |
|                                                                  |
|   Bit Vector (비트 벡터):                                        |
|   Block:  0 1 2 3 4 5 6 7 8 9 10 11 12 13 14                     |
|   Bit:    1 1 0 1 0 0 1 1 0 0  1  0  1  0  0                     |
|                                                                  |
|   Stored as: 11010011 00101010 (bytes)                           |
|              (0xD3)   (0x2A)                                      |
|                                                                  |
|   Finding Free Block (빈 블록 찾기):                             |
|   - Scan bitmap for first 0 bit (첫 번째 0비트 스캔)             |
|   - Hardware support: find-first-zero instruction                |
|     (하드웨어 지원: 첫 번째 0 찾기 명령어)                        |
|                                                                  |
|   Formula (공식):                                                |
|   Block number = (number of bits per word) * (word index) + bit  |
|   블록 번호 = (워드당 비트 수) * (워드 인덱스) + 비트 오프셋      |
|                                                                  |
+------------------------------------------------------------------+
|                                                                  |
|   Advantages (장점):                                             |
|   + Simple and efficient (단순하고 효율적)                       |
|   + Easy to find contiguous free blocks                          |
|     (연속적인 빈 블록 찾기 쉬움)                                  |
|   + Hardware acceleration available (하드웨어 가속 가능)         |
|                                                                  |
|   Disadvantages (단점):                                          |
|   - Must be kept in memory for efficiency                        |
|     (효율성을 위해 메모리에 유지해야 함)                          |
|   - Large disks = large bitmap (큰 디스크 = 큰 비트맵)           |
|                                                                  |
|   Space Calculation (공간 계산):                                 |
|   - 1TB disk with 4KB blocks = 256M blocks                       |
|   - Bitmap size = 256M bits = 32MB                               |
|   - 1TB 디스크, 4KB 블록 = 256M 블록                             |
|   - 비트맵 크기 = 256M 비트 = 32MB                               |
|                                                                  |
+------------------------------------------------------------------+
```

### 3.2 Linked List (연결 리스트)

Link all free disk blocks together, keeping a pointer to the first free block.

모든 빈 디스크 블록을 함께 연결하고 첫 번째 빈 블록에 대한 포인터를 유지합니다.

```
+------------------------------------------------------------------+
|                   LINKED LIST METHOD                             |
|                   (연결 리스트 방식)                              |
+------------------------------------------------------------------+
|                                                                  |
|   Free Space Head                                                |
|   (여유 공간 헤드)                                               |
|        |                                                         |
|        v                                                         |
|   +--------+     +--------+     +--------+     +--------+        |
|   |Block 2 |     |Block 4 |     |Block 5 |     |Block 8 |        |
|   +--------+     +--------+     +--------+     +--------+        |
|   | next:4 |---->| next:5 |---->| next:8 |---->| next:9 |---->...|
|   +--------+     +--------+     +--------+     +--------+        |
|                                                                  |
|   Allocation Process (할당 과정):                                |
|   1. Take first block from list (리스트에서 첫 번째 블록 가져옴) |
|   2. Update head to next free block                              |
|      (헤드를 다음 빈 블록으로 업데이트)                           |
|                                                                  |
|   Deallocation Process (해제 과정):                              |
|   1. Add freed block to head of list                             |
|      (해제된 블록을 리스트 헤드에 추가)                           |
|   2. Point it to old head (이전 헤드를 가리키도록 설정)           |
|                                                                  |
+------------------------------------------------------------------+
|                                                                  |
|   Advantages (장점):                                             |
|   + No space wasted (공간 낭비 없음)                             |
|   + Easy allocation (쉬운 할당)                                  |
|                                                                  |
|   Disadvantages (단점):                                          |
|   - Cannot easily get contiguous blocks                          |
|     (연속 블록 얻기 어려움)                                       |
|   - Traversing list is slow (리스트 순회 느림)                   |
|   - Inefficient for finding n free blocks                        |
|     (n개의 빈 블록 찾기 비효율적)                                 |
|                                                                  |
+------------------------------------------------------------------+
|                                                                  |
|   IMPROVEMENT: Grouping (개선: 그룹화)                           |
|                                                                  |
|   First free block stores addresses of n free blocks:            |
|   첫 번째 빈 블록이 n개의 빈 블록 주소를 저장:                    |
|                                                                  |
|   +------------------+                                           |
|   | Free Block 2     |                                           |
|   |------------------|                                           |
|   | Block 4 address  |                                           |
|   | Block 5 address  |                                           |
|   | Block 8 address  |                                           |
|   | Block 9 address  |                                           |
|   | ...              |                                           |
|   | Ptr to next group|---> [Another group of free blocks]        |
|   +------------------+                                           |
|                                                                  |
+------------------------------------------------------------------+
|                                                                  |
|   IMPROVEMENT: Counting (개선: 카운팅)                           |
|                                                                  |
|   Store (address, count) pairs for contiguous free blocks:       |
|   연속적인 빈 블록에 대해 (주소, 개수) 쌍 저장:                   |
|                                                                  |
|   +----------------+----------------+                            |
|   | Start Address  | Count (개수)  |                             |
|   |----------------|----------------|                            |
|   |       2        |       1        |  (Block 2)                 |
|   |       4        |       2        |  (Blocks 4-5)              |
|   |       8        |       3        |  (Blocks 8-10)             |
|   |      15        |       5        |  (Blocks 15-19)            |
|   +----------------+----------------+                            |
|                                                                  |
|   More efficient for contiguous allocation!                      |
|   연속 할당에 더 효율적!                                          |
|                                                                  |
+------------------------------------------------------------------+
```

### Free Space Management Comparison (여유 공간 관리 비교)

| Method (방법) | Space Overhead (공간 오버헤드) | Contiguous Finding (연속 찾기) | Performance (성능) |
|--------------|-------------------------------|-------------------------------|-------------------|
| Bit Vector (비트 벡터) | 1 bit per block (블록당 1비트) | Easy (쉬움) | Fast with HW support (HW 지원시 빠름) |
| Linked List (연결 리스트) | None (pointer in block) (없음) | Difficult (어려움) | Slow traversal (느린 순회) |
| Grouping (그룹화) | None | Moderate (보통) | Better than simple linked |
| Counting (카운팅) | None | Easy (쉬움) | Good for contiguous |

---

## 4. Disk Scheduling (디스크 스케줄링)

Disk scheduling algorithms determine the order in which disk I/O requests are processed.

디스크 스케줄링 알고리즘은 디스크 I/O 요청이 처리되는 순서를 결정합니다.

### 4.1 Disk Structure (디스크 구조)

```
+------------------------------------------------------------------+
|                     DISK STRUCTURE                               |
|                     (디스크 구조)                                 |
+------------------------------------------------------------------+
|                                                                  |
|   Top View (상면도):                                             |
|                                                                  |
|              Track 0 (트랙 0)                                    |
|                  |                                               |
|                  v                                               |
|            +-----------+                                         |
|          / |  /-----\  | \                                       |
|        /   | /       \ |   \                                     |
|       |    |    ---    |    |  <-- Spindle (스핀들)              |
|        \   | \       / |   /                                     |
|          \ |  \-----/  | /                                       |
|            +-----------+                                         |
|                  ^                                               |
|                  |                                               |
|              Track N (트랙 N)                                    |
|                                                                  |
|   Terms (용어):                                                  |
|   - Platter (플래터): Circular disk surface                      |
|   - Track (트랙): Concentric circle on platter                   |
|   - Sector (섹터): Section of a track                            |
|   - Cylinder (실린더): Set of tracks at same position            |
|   - Seek time (탐색 시간): Time to move head to track            |
|   - Rotational latency (회전 지연): Time for sector to rotate    |
|   - Transfer time (전송 시간): Time to read/write data           |
|                                                                  |
|   Access Time = Seek Time + Rotational Latency + Transfer Time   |
|   접근 시간 = 탐색 시간 + 회전 지연 + 전송 시간                    |
|                                                                  |
+------------------------------------------------------------------+
```

### 4.2 FCFS (First-Come, First-Served) (선입선출)

Requests are processed in the order they arrive.

요청은 도착 순서대로 처리됩니다.

```
+------------------------------------------------------------------+
|                    FCFS SCHEDULING                               |
|                    (FCFS 스케줄링)                                |
+------------------------------------------------------------------+
|                                                                  |
|   Request Queue (요청 큐): 98, 183, 37, 122, 14, 124, 65, 67     |
|   Initial Head Position (초기 헤드 위치): 53                     |
|                                                                  |
|   Movement (이동):                                               |
|   53 -> 98 -> 183 -> 37 -> 122 -> 14 -> 124 -> 65 -> 67          |
|                                                                  |
|   Cylinder (실린더)                                              |
|    0                                              199            |
|    |--+---+---+---+---+---+---+---+---+---+---+----|              |
|       |   |                                                      |
|      14  37    53       65 67    98    122 124      183          |
|       |   |    |         |  |     |      |  |        |           |
|       |   |    H-------->|  |     |      |  |        |           |
|       |   |              |  |<----|      |  |        |           |
|       |   |<-------------|  |            |  |        |           |
|       |   |              |  |----------->|  |        |           |
|       |<--|              |  |            |  |        |           |
|       |                  |  |            |  |------->|           |
|       |                  |<-|            |  |        |           |
|       |                  |  |            |  |        |           |
|                                                                  |
|   Total Head Movement (총 헤드 이동):                            |
|   |53-98| + |98-183| + |183-37| + |37-122| + |122-14| +          |
|   |14-124| + |124-65| + |65-67|                                  |
|   = 45 + 85 + 146 + 85 + 108 + 110 + 59 + 2 = 640 cylinders      |
|                                                                  |
|   Advantages (장점): Simple, fair                                |
|   Disadvantages (단점): High seek time, not optimal              |
|                                                                  |
+------------------------------------------------------------------+
```

### 4.3 SSTF (Shortest Seek Time First) (최단 탐색 시간 우선)

Select the request closest to the current head position.

현재 헤드 위치에 가장 가까운 요청을 선택합니다.

```
+------------------------------------------------------------------+
|                    SSTF SCHEDULING                               |
|                    (SSTF 스케줄링)                                |
+------------------------------------------------------------------+
|                                                                  |
|   Request Queue (요청 큐): 98, 183, 37, 122, 14, 124, 65, 67     |
|   Initial Head Position (초기 헤드 위치): 53                     |
|                                                                  |
|   Selection Process (선택 과정):                                 |
|   From 53: closest is 65 (distance 12)                           |
|   From 65: closest is 67 (distance 2)                            |
|   From 67: closest is 37 (distance 30)                           |
|   From 37: closest is 14 (distance 23)                           |
|   From 14: closest is 98 (distance 84)                           |
|   From 98: closest is 122 (distance 24)                          |
|   From 122: closest is 124 (distance 2)                          |
|   From 124: closest is 183 (distance 59)                         |
|                                                                  |
|   Movement (이동):                                               |
|   53 -> 65 -> 67 -> 37 -> 14 -> 98 -> 122 -> 124 -> 183          |
|                                                                  |
|   Cylinder (실린더)                                              |
|    0                                              199            |
|    |--+---+---+---+---+---+---+---+---+---+---+----|              |
|      14  37    53       65 67    98    122 124      183          |
|       |   |    |         |  |     |      |  |        |           |
|       |   |    H-------->|  |     |      |  |        |           |
|       |   |              |->|     |      |  |        |           |
|       |   |<-------------|  |     |      |  |        |           |
|       |<--|              |  |     |      |  |        |           |
|       |---------------------------->|    |  |        |           |
|       |                  |  |     |----->|  |        |           |
|       |                  |  |     |      |->|        |           |
|       |                  |  |     |      |  |------->|           |
|                                                                  |
|   Total Head Movement (총 헤드 이동):                            |
|   12 + 2 + 30 + 23 + 84 + 24 + 2 + 59 = 236 cylinders            |
|                                                                  |
|   Advantages (장점): Better than FCFS, lower seek time           |
|   Disadvantages (단점): Starvation possible (기아 가능)          |
|                                                                  |
+------------------------------------------------------------------+
```

### 4.4 SCAN (Elevator Algorithm) (엘리베이터 알고리즘)

Head moves in one direction, servicing requests, then reverses direction.

헤드가 한 방향으로 이동하면서 요청을 처리한 다음 방향을 바꿉니다.

```
+------------------------------------------------------------------+
|                    SCAN SCHEDULING                               |
|                    (SCAN 스케줄링)                                |
+------------------------------------------------------------------+
|                                                                  |
|   Request Queue (요청 큐): 98, 183, 37, 122, 14, 124, 65, 67     |
|   Initial Head Position (초기 헤드 위치): 53                     |
|   Initial Direction (초기 방향): Toward 0 (0 방향)               |
|                                                                  |
|   Movement (이동):                                               |
|   53 -> 37 -> 14 -> 0 -> 65 -> 67 -> 98 -> 122 -> 124 -> 183     |
|                                                                  |
|   Cylinder (실린더)                                              |
|    0                                              199            |
|    |--+---+---+---+---+---+---+---+---+---+---+----|              |
|    0 14  37    53       65 67    98    122 124      183          |
|    |  |   |    |         |  |     |      |  |        |           |
|    |  |   |<---H         |  |     |      |  |        |           |
|    |  |<--|              |  |     |      |  |        |           |
|    |<-|                  |  |     |      |  |        |           |
|    |-------------------->|  |     |      |  |        |           |
|    |                     |->|     |      |  |        |           |
|    |                     |  |---->|      |  |        |           |
|    |                     |  |     |----->|  |        |           |
|    |                     |  |     |      |->|        |           |
|    |                     |  |     |      |  |------->|           |
|                                                                  |
|   Total Head Movement (총 헤드 이동):                            |
|   (53-0) + (183-0) = 53 + 183 = 236 cylinders                    |
|                                                                  |
|   Behavior (동작):                                               |
|   - Like an elevator (엘리베이터처럼)                            |
|   - Goes to end before reversing (끝까지 간 후 방향 전환)        |
|   - No starvation (기아 없음)                                    |
|                                                                  |
+------------------------------------------------------------------+
```

### 4.5 C-SCAN (Circular SCAN) (순환 SCAN)

Head moves in one direction only, then jumps back to the beginning.

헤드가 한 방향으로만 이동한 다음 시작점으로 점프합니다.

```
+------------------------------------------------------------------+
|                   C-SCAN SCHEDULING                              |
|                   (C-SCAN 스케줄링)                               |
+------------------------------------------------------------------+
|                                                                  |
|   Request Queue (요청 큐): 98, 183, 37, 122, 14, 124, 65, 67     |
|   Initial Head Position (초기 헤드 위치): 53                     |
|   Direction (방향): Toward 199 (199 방향)                        |
|                                                                  |
|   Movement (이동):                                               |
|   53 -> 65 -> 67 -> 98 -> 122 -> 124 -> 183 -> 199 -> 0          |
|   -> 14 -> 37                                                    |
|                                                                  |
|   Cylinder (실린더)                                              |
|    0                                              199            |
|    |--+---+---+---+---+---+---+---+---+---+---+----|              |
|    0 14  37    53       65 67    98    122 124      183  199     |
|    |  |   |    |         |  |     |      |  |        |    |      |
|    |  |   |    H-------->|  |     |      |  |        |    |      |
|    |  |   |              |->|     |      |  |        |    |      |
|    |  |   |              |  |---->|      |  |        |    |      |
|    |  |   |              |  |     |----->|  |        |    |      |
|    |  |   |              |  |     |      |->|        |    |      |
|    |  |   |              |  |     |      |  |------->|    |      |
|    |  |   |              |  |     |      |  |        |--->|      |
|    |<-|---|--------------|--|-----|------|--|--------|----+      |
|    |->|   |              |  |     |      |  |        |    (jump) |
|    |  |-->|              |  |     |      |  |        |           |
|                                                                  |
|   Total Head Movement (총 헤드 이동):                            |
|   (199-53) + (199-0) + (37-0) = 146 + 199 + 37 = 382 cylinders   |
|   (but more uniform wait time! 하지만 더 균일한 대기 시간!)       |
|                                                                  |
|   Advantages (장점):                                             |
|   + More uniform wait time than SCAN                             |
|     (SCAN보다 더 균일한 대기 시간)                               |
|   + No starvation (기아 없음)                                    |
|                                                                  |
+------------------------------------------------------------------+
```

### Disk Scheduling Comparison (디스크 스케줄링 비교)

| Algorithm (알고리즘) | Total Movement (총 이동) | Starvation (기아) | Fairness (공정성) |
|---------------------|-------------------------|-------------------|-------------------|
| FCFS | 640 | No (없음) | High (높음) |
| SSTF | 236 | Yes (있음) | Low (낮음) |
| SCAN | 236 | No (없음) | Medium (중간) |
| C-SCAN | 382 | No (없음) | High (높음) |

```
+------------------------------------------------------------------+
|              WHEN TO USE EACH ALGORITHM                          |
|              (각 알고리즘 사용 시기)                              |
+------------------------------------------------------------------+
|                                                                  |
|   FCFS: Light load, simple systems                               |
|         가벼운 부하, 단순한 시스템                                |
|                                                                  |
|   SSTF: When minimizing seek time is priority                    |
|         탐색 시간 최소화가 우선일 때                              |
|         (but beware of starvation! 하지만 기아 주의!)            |
|                                                                  |
|   SCAN: General-purpose, good balance                            |
|         일반적인 용도, 좋은 균형                                  |
|                                                                  |
|   C-SCAN: When uniform wait time is important                    |
|           균일한 대기 시간이 중요할 때                            |
|           (e.g., heavy load systems)                             |
|                                                                  |
+------------------------------------------------------------------+
```

---

## Summary (요약)

```
+==================================================================+
|                    CHAPTER 11 SUMMARY                            |
|                    (11장 요약)                                    |
+==================================================================+

1. FILE SYSTEM STRUCTURE (파일 시스템 구조)
   - Layered design: Application -> Logical FS -> File Org ->
     Basic FS -> I/O Control -> Devices
   - 계층적 설계: 응용 -> 논리적 FS -> 파일 구성 ->
     기본 FS -> I/O 제어 -> 장치

2. DISK ALLOCATION METHODS (디스크 할당 방법)
   +---------------+------------------+------------------+
   | Method        | Pros (장점)      | Cons (단점)      |
   |---------------|------------------|------------------|
   | Contiguous    | Fast access      | Fragmentation    |
   | (연속)        | (빠른 접근)      | (단편화)         |
   |---------------|------------------|------------------|
   | Linked        | No fragmentation | Slow direct acc  |
   | (연결)        | (단편화 없음)    | (느린 직접 접근) |
   |---------------|------------------|------------------|
   | Indexed       | Direct access    | Index overhead   |
   | (인덱스)      | (직접 접근)      | (인덱스 오버헤드)|
   +---------------+------------------+------------------+

3. FREE SPACE MANAGEMENT (여유 공간 관리)
   - Bit Vector: 1 bit per block, fast with hardware
     비트 벡터: 블록당 1비트, 하드웨어로 빠름
   - Linked List: No overhead, but slow traversal
     연결 리스트: 오버헤드 없음, 하지만 느린 순회
   - Grouping & Counting: Improvements to linked list
     그룹화 & 카운팅: 연결 리스트의 개선

4. DISK SCHEDULING (디스크 스케줄링)
   - FCFS: Simple, fair, but high seek time
     FCFS: 단순, 공정, 하지만 높은 탐색 시간
   - SSTF: Shortest seek time, but starvation possible
     SSTF: 최단 탐색 시간, 하지만 기아 가능
   - SCAN: Elevator algorithm, no starvation
     SCAN: 엘리베이터 알고리즘, 기아 없음
   - C-SCAN: Uniform wait time, circular movement
     C-SCAN: 균일한 대기 시간, 순환 이동

+==================================================================+
```

### Quick Reference (빠른 참조)

| Topic (주제) | Key Formula/Concept (핵심 공식/개념) |
|--------------|-------------------------------------|
| Disk Access Time (디스크 접근 시간) | Seek + Rotation + Transfer (탐색 + 회전 + 전송) |
| UNIX inode | 12 direct + single/double/triple indirect |
| FAT | File Allocation Table - linked allocation improvement |
| Bitmap size (비트맵 크기) | Total blocks / 8 bytes |
| SCAN Direction (SCAN 방향) | Service all requests in current direction (현재 방향의 모든 요청 처리) |
