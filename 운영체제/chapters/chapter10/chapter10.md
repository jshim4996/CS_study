# Chapter 10. File System Interface (파일 시스템 인터페이스)

## Overview (개요)

The file system interface provides a uniform mechanism for users and applications to store and retrieve data. It abstracts the physical storage details and presents a logical view of data organization.

파일 시스템 인터페이스는 사용자와 응용 프로그램이 데이터를 저장하고 검색할 수 있는 통일된 메커니즘을 제공합니다. 물리적 저장소의 세부 사항을 추상화하고 데이터 조직의 논리적 뷰를 제시합니다.

---

## 1. File Concept (파일 개념)

### 1.1 What is a File? (파일이란?)

A **file** is a named collection of related information recorded on secondary storage. From the user's perspective, a file is the smallest allotment of logical secondary storage.

**파일**은 보조 저장장치에 기록된 관련 정보의 명명된 모음입니다. 사용자 관점에서 파일은 논리적 보조 저장장치의 가장 작은 할당 단위입니다.

```
+--------------------------------------------------+
|                  FILE ABSTRACTION                |
+--------------------------------------------------+
|                                                  |
|   User View (사용자 뷰):                          |
|   +------------------------------------------+   |
|   |  filename.txt                            |   |
|   |  - Name (이름)                           |   |
|   |  - Content (내용)                        |   |
|   |  - Attributes (속성)                     |   |
|   +------------------------------------------+   |
|                      |                           |
|                      v                           |
|   System View (시스템 뷰):                        |
|   +------------------------------------------+   |
|   |  Blocks on Disk (디스크 블록들)           |   |
|   |  [Block 0][Block 5][Block 12][Block 23]  |   |
|   +------------------------------------------+   |
|                                                  |
+--------------------------------------------------+
```

### 1.2 File Types (파일 유형)

| Type (유형) | Extension (확장자) | Description (설명) |
|-------------|-------------------|-------------------|
| Executable (실행파일) | .exe, .com, .bin | Ready-to-run machine code (실행 가능한 기계어 코드) |
| Object (목적파일) | .obj, .o | Compiled but not linked code (컴파일되었지만 링크되지 않은 코드) |
| Source Code (소스코드) | .c, .java, .py | Human-readable program code (사람이 읽을 수 있는 프로그램 코드) |
| Text (텍스트) | .txt, .doc | Plain text documents (일반 텍스트 문서) |
| Library (라이브러리) | .lib, .dll, .so | Reusable code modules (재사용 가능한 코드 모듈) |
| Archive (압축파일) | .zip, .tar, .rar | Compressed files (압축된 파일들) |
| Multimedia (멀티미디어) | .jpg, .mp3, .mp4 | Images, audio, video (이미지, 오디오, 비디오) |

---

## 2. File Attributes and Operations (파일 속성과 연산)

### 2.1 File Attributes (파일 속성)

Files have various attributes (metadata) that describe their properties:

파일에는 속성을 설명하는 다양한 속성(메타데이터)이 있습니다:

```
+-----------------------------------------------------------+
|                    FILE ATTRIBUTES                        |
|                    (파일 속성)                             |
+-----------------------------------------------------------+
|                                                           |
|  +---------------------+  +-------------------------+     |
|  |  Name (이름)        |  |  Identifier (식별자)    |     |
|  |  "report.txt"       |  |  Unique number (고유번호)|     |
|  +---------------------+  +-------------------------+     |
|                                                           |
|  +---------------------+  +-------------------------+     |
|  |  Type (유형)        |  |  Location (위치)        |     |
|  |  Text file          |  |  Pointer to disk block  |     |
|  +---------------------+  +-------------------------+     |
|                                                           |
|  +---------------------+  +-------------------------+     |
|  |  Size (크기)        |  |  Protection (보호)      |     |
|  |  4096 bytes         |  |  rwxr-xr-x              |     |
|  +---------------------+  +-------------------------+     |
|                                                           |
|  +---------------------+  +-------------------------+     |
|  |  Time/Date (시간)   |  |  Owner (소유자)         |     |
|  |  Created/Modified   |  |  User ID (사용자 ID)    |     |
|  +---------------------+  +-------------------------+     |
|                                                           |
+-----------------------------------------------------------+
```

### 2.2 File Operations (파일 연산)

The operating system provides a set of system calls for file manipulation:

운영체제는 파일 조작을 위한 시스템 호출 집합을 제공합니다:

| Operation (연산) | Description (설명) | System Call Example |
|-----------------|-------------------|---------------------|
| Create (생성) | Allocate space and create directory entry (공간 할당 및 디렉토리 항목 생성) | `create()` |
| Open (열기) | Fetch attributes into memory (속성을 메모리로 가져오기) | `open()` |
| Read (읽기) | Transfer data from file to memory (파일에서 메모리로 데이터 전송) | `read()` |
| Write (쓰기) | Transfer data from memory to file (메모리에서 파일로 데이터 전송) | `write()` |
| Reposition (위치 변경) | Move file pointer (파일 포인터 이동) | `lseek()` |
| Delete (삭제) | Remove file and free space (파일 제거 및 공간 해제) | `unlink()` |
| Truncate (절단) | Erase contents but keep attributes (내용 삭제, 속성 유지) | `truncate()` |
| Close (닫기) | Release file descriptor (파일 디스크립터 해제) | `close()` |

```
+------------------------------------------------------------------+
|                    FILE OPERATION FLOW                           |
|                    (파일 연산 흐름)                               |
+------------------------------------------------------------------+
|                                                                  |
|    Application (응용 프로그램)                                    |
|         |                                                        |
|         | open("file.txt", READ)                                 |
|         v                                                        |
|    +------------------+                                          |
|    |  File Descriptor |  <-- Returned by OS (OS가 반환)          |
|    |       (fd = 3)   |                                          |
|    +------------------+                                          |
|         |                                                        |
|         | read(fd, buffer, size)                                 |
|         v                                                        |
|    +------------------+     +------------------+                  |
|    |   User Buffer    | <-- |    Disk Block    |                 |
|    +------------------+     +------------------+                  |
|         |                                                        |
|         | close(fd)                                              |
|         v                                                        |
|    [File descriptor released (파일 디스크립터 해제)]              |
|                                                                  |
+------------------------------------------------------------------+
```

### 2.3 Open File Table (열린 파일 테이블)

The OS maintains tables to track open files:

운영체제는 열린 파일을 추적하기 위해 테이블을 유지합니다:

```
+------------------------------------------------------------------+
|               OPEN FILE TABLES STRUCTURE                         |
|               (열린 파일 테이블 구조)                             |
+------------------------------------------------------------------+
|                                                                  |
|  Process (프로세스 A)          System-wide Table                 |
|  +-------------------+         (시스템 전역 테이블)               |
|  | Per-process Table |         +----------------------+          |
|  | (프로세스별 테이블)|         | Open File Entry      |          |
|  +-------------------+         +----------------------+          |
|  | fd | Pointer      |    +--> | File pointer (위치)  |          |
|  |----|--------------|    |    | Open count (열린 수) |          |
|  | 0  | stdin        |    |    | Disk location (위치) |          |
|  | 1  | stdout       |    |    | Access rights (권한) |          |
|  | 2  | stderr       |    |    +----------------------+          |
|  | 3  | -------------|----+              |                       |
|  +-------------------+                   v                       |
|                                   +-------------+                |
|                                   |   Inode     |                |
|                                   | (on disk)   |                |
|                                   +-------------+                |
|                                                                  |
+------------------------------------------------------------------+
```

---

## 3. Directory Structure (디렉토리 구조)

A directory is a collection of nodes containing information about files. It provides a way to organize files logically.

디렉토리는 파일에 대한 정보를 포함하는 노드들의 모음입니다. 파일을 논리적으로 구성하는 방법을 제공합니다.

### 3.1 Single-Level Directory (단일 레벨 디렉토리)

All files are contained in the same directory. Simple but has naming and grouping problems.

모든 파일이 동일한 디렉토리에 포함됩니다. 단순하지만 이름 지정 및 그룹화 문제가 있습니다.

```
+------------------------------------------------------------------+
|                 SINGLE-LEVEL DIRECTORY                           |
|                 (단일 레벨 디렉토리)                              |
+------------------------------------------------------------------+
|                                                                  |
|                      ROOT DIRECTORY                              |
|                      (루트 디렉토리)                              |
|   +----------------------------------------------------------+   |
|   |  file1  |  file2  |  file3  |  file4  |  file5  | ...   |   |
|   +----------------------------------------------------------+   |
|                                                                  |
|   Advantages (장점):                                             |
|   - Simple to implement (구현이 간단함)                          |
|   - Easy to understand (이해하기 쉬움)                           |
|                                                                  |
|   Disadvantages (단점):                                          |
|   - Naming problem: All names must be unique                     |
|     (이름 문제: 모든 이름이 고유해야 함)                          |
|   - No grouping capability (그룹화 기능 없음)                     |
|   - Poor for multi-user systems (다중 사용자 시스템에 부적합)     |
|                                                                  |
+------------------------------------------------------------------+
```

### 3.2 Two-Level Directory (2단계 디렉토리)

Separate directory for each user. Provides isolation between users.

각 사용자별로 별도의 디렉토리가 있습니다. 사용자 간 격리를 제공합니다.

```
+------------------------------------------------------------------+
|                   TWO-LEVEL DIRECTORY                            |
|                   (2단계 디렉토리)                                |
+------------------------------------------------------------------+
|                                                                  |
|                   Master File Directory (MFD)                    |
|                   (마스터 파일 디렉토리)                          |
|         +--------+--------+--------+--------+                    |
|         | User1  | User2  | User3  | User4  |                    |
|         +---+----+---+----+---+----+---+----+                    |
|             |        |        |        |                         |
|             v        v        v        v                         |
|   User File Directory (UFD) (사용자 파일 디렉토리)               |
|                                                                  |
|   User1:    User2:    User3:    User4:                           |
|   +------+  +------+  +------+  +------+                         |
|   |file_a|  |file_a|  |file_x|  |file_m|                         |
|   |file_b|  |file_c|  |file_y|  |file_n|                         |
|   +------+  +------+  +------+  +------+                         |
|                                                                  |
|   Path example: /User1/file_a  (경로 예시)                       |
|                                                                  |
|   Advantages (장점):                                             |
|   - Different users can have same file names                     |
|     (다른 사용자가 같은 파일 이름 사용 가능)                      |
|   - Efficient searching within user directory                    |
|     (사용자 디렉토리 내 효율적 검색)                              |
|                                                                  |
|   Disadvantages (단점):                                          |
|   - No grouping within user's files (사용자 파일 내 그룹화 불가)  |
|   - Difficult file sharing (파일 공유 어려움)                     |
|                                                                  |
+------------------------------------------------------------------+
```

### 3.3 Tree-Structured Directory (트리 구조 디렉토리)

Hierarchical directory structure allowing arbitrary depth of nesting.

임의의 깊이의 중첩을 허용하는 계층적 디렉토리 구조입니다.

```
+------------------------------------------------------------------+
|                  TREE-STRUCTURED DIRECTORY                       |
|                  (트리 구조 디렉토리)                             |
+------------------------------------------------------------------+
|                                                                  |
|                            / (root)                              |
|                    (루트 디렉토리)                                |
|                            |                                     |
|           +----------------+----------------+                    |
|           |                |                |                    |
|          home             bin              etc                   |
|           |                |                |                    |
|     +-----+-----+      +---+---+       +----+----+               |
|     |           |      |       |       |         |               |
|   user1       user2   ls     cat    passwd   hosts               |
|     |           |                                                |
|  +--+--+     +--+--+                                              |
|  |     |     |     |                                              |
| docs  src  pics  docs                                            |
|  |     |     |     |                                              |
| a.txt b.c  x.jpg y.txt                                           |
|                                                                  |
|   Features (특징):                                               |
|   - Current directory (현재 디렉토리): ./                        |
|   - Parent directory (부모 디렉토리): ../                        |
|   - Absolute path (절대 경로): /home/user1/docs/a.txt            |
|   - Relative path (상대 경로): ./docs/a.txt                      |
|                                                                  |
|   Advantages (장점):                                             |
|   - Efficient file organization (효율적인 파일 구성)              |
|   - Logical grouping (논리적 그룹화)                              |
|   - Easy navigation (쉬운 탐색)                                  |
|                                                                  |
+------------------------------------------------------------------+
```

### 3.4 Acyclic-Graph Directory (비순환 그래프 디렉토리)

Allows sharing of files and directories through links, but no cycles.

링크를 통해 파일과 디렉토리의 공유를 허용하지만 순환은 없습니다.

```
+------------------------------------------------------------------+
|                ACYCLIC-GRAPH DIRECTORY                           |
|                (비순환 그래프 디렉토리)                           |
+------------------------------------------------------------------+
|                                                                  |
|                        / (root)                                  |
|                           |                                      |
|              +------------+------------+                         |
|              |                         |                         |
|            user1                     user2                       |
|              |                         |                         |
|        +-----+-----+             +-----+-----+                   |
|        |           |             |           |                   |
|      docs       shared -------> shared    projects               |
|        |           |             (link)      |                   |
|      a.txt      report.txt                 b.txt                 |
|                    ^                                             |
|                    |                                             |
|   Hard Link (하드 링크): Points to same inode                    |
|   - Same file, multiple directory entries                        |
|   - 같은 파일, 여러 디렉토리 항목                                 |
|                                                                  |
|   Symbolic Link (심볼릭 링크): Points to pathname                |
|   - Separate file containing path                                |
|   - 경로를 포함하는 별도 파일                                     |
|                                                                  |
|   +-----------------------------------------------------------+  |
|   |  Hard Link vs Symbolic Link Comparison                    |  |
|   |  (하드 링크 vs 심볼릭 링크 비교)                           |  |
|   +-----------------------------------------------------------+  |
|   |  Feature          | Hard Link    | Symbolic Link          |  |
|   |-------------------|--------------|------------------------|  |
|   |  Inode (아이노드) | Same         | Different              |  |
|   |  Cross filesystem | No (불가)    | Yes (가능)             |  |
|   |  Link to directory| No (불가)    | Yes (가능)             |  |
|   |  Original deleted | Still works  | Broken link            |  |
|   |  (원본 삭제 시)   | (여전히 작동)| (끊어진 링크)          |  |
|   +-----------------------------------------------------------+  |
|                                                                  |
+------------------------------------------------------------------+
```

---

## 4. File Access Methods (파일 접근 방법)

### 4.1 Sequential Access (순차 접근)

Information is processed in order, one record after another. Most common method.

정보가 순서대로 처리되며, 하나의 레코드 다음에 다른 레코드가 처리됩니다. 가장 일반적인 방법입니다.

```
+------------------------------------------------------------------+
|                    SEQUENTIAL ACCESS                             |
|                    (순차 접근)                                    |
+------------------------------------------------------------------+
|                                                                  |
|   File: +-----+-----+-----+-----+-----+-----+-----+-----+        |
|         |  0  |  1  |  2  |  3  |  4  |  5  |  6  |  7  |        |
|         +-----+-----+-----+-----+-----+-----+-----+-----+        |
|                        ^                                         |
|                        |                                         |
|                 Current Position                                 |
|                 (현재 위치)                                       |
|                                                                  |
|   Operations (연산):                                             |
|   - read next  (다음 읽기)  : Read and advance pointer           |
|   - write next (다음 쓰기)  : Write and advance pointer          |
|   - reset      (재설정)     : Go to beginning                    |
|                                                                  |
|   Example Flow (예시 흐름):                                      |
|   reset() -> read() -> read() -> read() -> ...                   |
|      |         |         |         |                             |
|      v         v         v         v                             |
|   pos=0     pos=1     pos=2     pos=3                            |
|                                                                  |
|   Use Cases (사용 사례):                                         |
|   - Text editors (텍스트 편집기)                                  |
|   - Compilers reading source code (소스코드 읽는 컴파일러)        |
|   - Log files (로그 파일)                                        |
|                                                                  |
+------------------------------------------------------------------+
```

### 4.2 Direct Access (직접 접근)

Also called relative access. File is viewed as a numbered sequence of blocks or records. Can read/write in any order.

상대 접근이라고도 합니다. 파일은 번호가 매겨진 블록 또는 레코드의 시퀀스로 간주됩니다. 어떤 순서로든 읽기/쓰기가 가능합니다.

```
+------------------------------------------------------------------+
|                      DIRECT ACCESS                               |
|                      (직접 접근)                                  |
+------------------------------------------------------------------+
|                                                                  |
|   File Blocks (파일 블록):                                       |
|   +-----+-----+-----+-----+-----+-----+-----+-----+              |
|   |  0  |  1  |  2  |  3  |  4  |  5  |  6  |  7  |              |
|   +-----+-----+-----+-----+-----+-----+-----+-----+              |
|      ^           ^     ^                 ^                       |
|      |           |     |                 |                       |
|    read(0)   read(2) read(3)          read(5)                    |
|                                                                  |
|   Operations (연산):                                             |
|   - read(n)  : Read block n directly                             |
|   - write(n) : Write to block n directly                         |
|   - seek(n)  : Position to block n                               |
|                                                                  |
|   Position Calculation (위치 계산):                              |
|   Physical Address = Base + (n * Block Size)                     |
|   물리 주소 = 기준 + (n * 블록 크기)                              |
|                                                                  |
|   Use Cases (사용 사례):                                         |
|   - Database systems (데이터베이스 시스템)                        |
|   - Airline reservation systems (항공 예약 시스템)                |
|   - Any application requiring random access                      |
|     (임의 접근이 필요한 모든 응용 프로그램)                        |
|                                                                  |
+------------------------------------------------------------------+
```

### 4.3 Indexed Access (인덱스 접근)

Uses an index to access records. The index contains pointers to various blocks.

레코드에 접근하기 위해 인덱스를 사용합니다. 인덱스에는 다양한 블록에 대한 포인터가 포함됩니다.

```
+------------------------------------------------------------------+
|                     INDEXED ACCESS                               |
|                     (인덱스 접근)                                 |
+------------------------------------------------------------------+
|                                                                  |
|     Index File (인덱스 파일)        Data File (데이터 파일)       |
|     +-----------------+             +---------------------+       |
|     | Key   | Pointer |             | Block 0             |       |
|     |-------|---------|             |---------------------|       |
|     | "A"   |    0  --|------------>| Alice, 25, Seoul    |       |
|     | "B"   |    3  --|------+      |---------------------|       |
|     | "C"   |    1  --|---+  |      | Block 1             |       |
|     | "D"   |    5  --|--+|  |      |---------------------|       |
|     +-----------------+  ||  +----->| Charlie, 30, Busan  |       |
|                          ||         |---------------------|       |
|                          ||         | Block 2             |       |
|                          ||         |---------------------|       |
|                          |+-------->| Bob, 28, Incheon    |       |
|                          |          |---------------------|       |
|                          |          | Block 3             |       |
|                          |          |---------------------|       |
|                          +--------->| ... (more data)     |       |
|                                     +---------------------+       |
|                                                                  |
|   Multi-level Index (다단계 인덱스):                             |
|   +----------+     +-----------+     +------------+              |
|   | Primary  | --> | Secondary | --> | Data Block |              |
|   | Index    |     | Index     |     |            |              |
|   +----------+     +-----------+     +------------+              |
|                                                                  |
|   Advantages (장점):                                             |
|   - Fast search with large files (대용량 파일의 빠른 검색)        |
|   - Efficient for key-based access (키 기반 접근에 효율적)        |
|                                                                  |
|   Disadvantages (단점):                                          |
|   - Extra space for index (인덱스를 위한 추가 공간)               |
|   - Index maintenance overhead (인덱스 유지 관리 오버헤드)        |
|                                                                  |
+------------------------------------------------------------------+
```

### Access Methods Comparison (접근 방법 비교)

| Method (방법) | Access Pattern (접근 패턴) | Best For (적합한 용도) | Complexity (복잡도) |
|--------------|---------------------------|----------------------|---------------------|
| Sequential (순차) | One after another (순서대로) | Text files, logs | O(n) |
| Direct (직접) | Any order (임의 순서) | Databases | O(1) |
| Indexed (인덱스) | Key-based (키 기반) | Large databases | O(log n) |

---

## 5. File Protection (파일 보호)

### 5.1 Types of Access (접근 유형)

Different operations can be controlled:

다양한 작업을 제어할 수 있습니다:

| Access Type (접근 유형) | Description (설명) |
|------------------------|-------------------|
| Read (읽기) | Read file contents (파일 내용 읽기) |
| Write (쓰기) | Modify file contents (파일 내용 수정) |
| Execute (실행) | Run file as program (프로그램으로 파일 실행) |
| Append (추가) | Add data to end of file (파일 끝에 데이터 추가) |
| Delete (삭제) | Remove file (파일 제거) |
| List (목록) | List file attributes (파일 속성 나열) |

### 5.2 Access Control List (ACL) (접근 제어 목록)

Specifies which users can access a file and what operations they can perform.

어떤 사용자가 파일에 접근할 수 있고 어떤 작업을 수행할 수 있는지 지정합니다.

```
+------------------------------------------------------------------+
|                   ACCESS CONTROL LIST (ACL)                      |
|                   (접근 제어 목록)                                |
+------------------------------------------------------------------+
|                                                                  |
|   File: important_data.txt                                       |
|                                                                  |
|   +-----------------------------------------------------------+  |
|   |  User/Group      |  Permissions (권한)                    |  |
|   |------------------|----------------------------------------|  |
|   |  Owner (소유자)  |  Read, Write, Execute (읽기,쓰기,실행) |  |
|   |  Group (그룹)    |  Read, Execute (읽기, 실행)            |  |
|   |  Others (기타)   |  Read only (읽기만)                    |  |
|   +-----------------------------------------------------------+  |
|                                                                  |
|   UNIX Permission Representation (UNIX 권한 표현):               |
|                                                                  |
|       rwx   r-x   r--                                            |
|       ---   ---   ---                                            |
|        |     |     |                                             |
|        |     |     +-- Others (기타 사용자)                       |
|        |     +-- Group (그룹)                                    |
|        +-- Owner (소유자)                                        |
|                                                                  |
|   Numeric Representation (숫자 표현):                            |
|   +------+-------+--------+                                      |
|   | Mode | Binary| Meaning|                                      |
|   |------|-------|--------|                                      |
|   |  7   |  111  | rwx    |                                      |
|   |  6   |  110  | rw-    |                                      |
|   |  5   |  101  | r-x    |                                      |
|   |  4   |  100  | r--    |                                      |
|   |  3   |  011  | -wx    |                                      |
|   |  2   |  010  | -w-    |                                      |
|   |  1   |  001  | --x    |                                      |
|   |  0   |  000  | ---    |                                      |
|   +------+-------+--------+                                      |
|                                                                  |
|   Example: chmod 755 file.txt                                    |
|   - Owner: rwx (7)                                               |
|   - Group: r-x (5)                                               |
|   - Others: r-x (5)                                              |
|                                                                  |
+------------------------------------------------------------------+
```

### 5.3 Access Control Matrix (접근 제어 행렬)

A general model for protection systems:

보호 시스템을 위한 일반적인 모델:

```
+------------------------------------------------------------------+
|                   ACCESS CONTROL MATRIX                          |
|                   (접근 제어 행렬)                                |
+------------------------------------------------------------------+
|                                                                  |
|                |  File1   |  File2   |  File3   |  Printer  |    |
|   Domain       | (파일1)  | (파일2)  | (파일3)  | (프린터)  |    |
|   (도메인)     |          |          |          |           |    |
|   -------------|----------|----------|----------|-----------|    |
|   User A       |  R, W    |    R     |    -     |   Print   |    |
|   (사용자 A)   |          |          |          |           |    |
|   -------------|----------|----------|----------|-----------|    |
|   User B       |    R     |  R, W, X |    R     |     -     |    |
|   (사용자 B)   |          |          |          |           |    |
|   -------------|----------|----------|----------|-----------|    |
|   User C       |    -     |    R     |  R, W    |   Print   |    |
|   (사용자 C)   |          |          |          |           |    |
|                                                                  |
|   Implementation Methods (구현 방법):                            |
|                                                                  |
|   1. ACL (Column-based) - 열 기반:                               |
|      File1: [(UserA, RW), (UserB, R)]                            |
|                                                                  |
|   2. Capability List (Row-based) - 행 기반:                      |
|      UserA: [(File1, RW), (File2, R), (Printer, Print)]          |
|                                                                  |
+------------------------------------------------------------------+
```

---

## Summary (요약)

### Key Concepts (핵심 개념)

```
+==================================================================+
|                    CHAPTER 10 SUMMARY                            |
|                    (10장 요약)                                    |
+==================================================================+

1. FILE CONCEPT (파일 개념)
   - Named collection of data on secondary storage
   - 보조 저장장치에 있는 데이터의 명명된 모음
   - Contains data + attributes (metadata)
   - 데이터 + 속성(메타데이터) 포함

2. FILE ATTRIBUTES (파일 속성)
   - Name, Type, Location, Size, Protection, Time
   - 이름, 유형, 위치, 크기, 보호, 시간

3. FILE OPERATIONS (파일 연산)
   - Create, Open, Read, Write, Close, Delete
   - 생성, 열기, 읽기, 쓰기, 닫기, 삭제

4. DIRECTORY STRUCTURES (디렉토리 구조)
   +-------------------+-------------------------------------------+
   | Structure (구조)  | Characteristics (특징)                    |
   |-------------------|-------------------------------------------|
   | Single-level      | Simple, naming conflicts                  |
   | (단일 레벨)       | 단순함, 이름 충돌                          |
   |-------------------|-------------------------------------------|
   | Two-level         | User isolation, no sharing                |
   | (2단계)           | 사용자 격리, 공유 불가                     |
   |-------------------|-------------------------------------------|
   | Tree              | Hierarchical, easy organization           |
   | (트리)            | 계층적, 쉬운 구성                          |
   |-------------------|-------------------------------------------|
   | Acyclic-graph     | Sharing via links, no cycles              |
   | (비순환 그래프)   | 링크를 통한 공유, 순환 없음                |
   +-------------------+-------------------------------------------+

5. ACCESS METHODS (접근 방법)
   - Sequential: Read in order (순차: 순서대로 읽기)
   - Direct: Random access by block number (직접: 블록 번호로 임의 접근)
   - Indexed: Access via index (인덱스: 인덱스를 통한 접근)

6. FILE PROTECTION (파일 보호)
   - Access types: Read, Write, Execute, etc.
   - 접근 유형: 읽기, 쓰기, 실행 등
   - ACL: Per-file permission list
   - ACL: 파일별 권한 목록
   - UNIX: Owner/Group/Others with rwx permissions
   - UNIX: rwx 권한을 가진 소유자/그룹/기타

+==================================================================+
```

### Quick Reference (빠른 참조)

| Topic (주제) | Key Point (핵심 포인트) |
|--------------|------------------------|
| File (파일) | Logical storage unit with name and attributes (이름과 속성을 가진 논리적 저장 단위) |
| Directory (디렉토리) | Organizes files hierarchically (파일을 계층적으로 구성) |
| Path (경로) | Absolute: from root, Relative: from current (절대: 루트부터, 상대: 현재부터) |
| Link (링크) | Hard: same inode, Symbolic: path pointer (하드: 같은 아이노드, 심볼릭: 경로 포인터) |
| Protection (보호) | ACL controls who can do what (ACL이 누가 무엇을 할 수 있는지 제어) |
