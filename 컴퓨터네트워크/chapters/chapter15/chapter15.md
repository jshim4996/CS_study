# Chapter 15. HTTP Evolution (HTTP 발전)

---

## Table of Contents (목차)

1. [HTTP/1.0 vs HTTP/1.1](#1-http10-vs-http11)
2. [Keep-Alive Connection (Keep-Alive 연결)](#2-keep-alive-connection-keep-alive-연결)
3. [HTTP/2 Features (HTTP/2 기능)](#3-http2-features-http2-기능)
4. [HTTP/3 and QUIC (HTTP/3와 QUIC)](#4-http3-and-quic-http3와-quic)
5. [Summary (요약)](#5-summary-요약)

---

## 1. HTTP/1.0 vs HTTP/1.1

### HTTP Protocol History (HTTP 프로토콜 역사)

```
+-----------------------------------------------------------------------+
|                    HTTP Version Timeline                               |
+-----------------------------------------------------------------------+
|                                                                       |
|   1991        1996         1997         2015         2022             |
|     |           |            |            |            |              |
|     v           v            v            v            v              |
|   +----+     +-----+     +-------+    +------+    +------+            |
|   |0.9 |     | 1.0 |     |  1.1  |    | 2.0  |    | 3.0  |            |
|   +----+     +-----+     +-------+    +------+    +------+            |
|     |           |            |            |            |              |
|   Simple    Features     Persistence  Binary      QUIC-based          |
|   GET only  Headers      Keep-alive   Multiplexing UDP-based          |
|             Status       Host header  HPACK        0-RTT              |
|                                                                       |
+-----------------------------------------------------------------------+
```

### HTTP/1.0 Characteristics (HTTP/1.0 특징)

```
+-----------------------------------------------------------------------+
|                    HTTP/1.0 Features                                   |
+-----------------------------------------------------------------------+
|                                                                       |
|   Released: 1996 (RFC 1945)                                           |
|   발표: 1996년                                                        |
|                                                                       |
|   Key Features:                                                       |
|   - Request-response model                                            |
|   - HTTP headers introduced                                           |
|   - Status codes (200, 404, etc.)                                     |
|   - Content types (MIME types)                                        |
|   - GET, POST, HEAD methods                                           |
|                                                                       |
|   Limitations:                                                        |
|   - One request per connection                                        |
|   - 연결당 하나의 요청                                                |
|   - Connection closed after each response                             |
|   - 각 응답 후 연결 종료                                              |
|   - High latency for multiple resources                               |
|   - 여러 리소스에 대한 높은 지연 시간                                 |
|                                                                       |
+-----------------------------------------------------------------------+
```

### HTTP/1.0 Connection Problem (HTTP/1.0 연결 문제)

```
+-----------------------------------------------------------------------+
|                    HTTP/1.0: New Connection Per Request                |
+-----------------------------------------------------------------------+
|                                                                       |
|   Client                                           Server             |
|      |                                               |                |
|      |  TCP Handshake (3-way)                        |                |
|      | ============================================> |                |
|      | <============================================ |                |
|      |                                               |                |
|      |  GET /index.html                              |                |
|      | --------------------------------------------> |                |
|      |  Response: index.html                         |                |
|      | <-------------------------------------------- |                |
|      |                                               |                |
|      |  Connection Closed                            |                |
|      | <--X--X--X--X--X--X--X--X--X--X--X--X--X--X--> |                |
|      |                                               |                |
|      |  TCP Handshake (again!)                       |                |
|      | ============================================> |                |
|      | <============================================ |                |
|      |                                               |                |
|      |  GET /style.css                               |                |
|      | --------------------------------------------> |                |
|      |  Response: style.css                          |                |
|      | <-------------------------------------------- |                |
|      |                                               |                |
|      |  Connection Closed (again!)                   |                |
|      | <--X--X--X--X--X--X--X--X--X--X--X--X--X--X--> |                |
|      |                                               |                |
|                                                                       |
|   Problem: TCP handshake overhead for EVERY request!                  |
|   문제: 모든 요청마다 TCP 핸드셰이크 오버헤드!                        |
|                                                                       |
+-----------------------------------------------------------------------+
```

### HTTP/1.1 Improvements (HTTP/1.1 개선사항)

```
+-----------------------------------------------------------------------+
|                    HTTP/1.1 Key Features                               |
+-----------------------------------------------------------------------+
|                                                                       |
|   Released: 1997 (RFC 2068), Updated 1999 (RFC 2616)                  |
|   발표: 1997년, 1999년 업데이트                                       |
|                                                                       |
|   +-------------------------+----------------------------------------+|
|   | Feature                 | Description                            ||
|   +-------------------------+----------------------------------------+|
|   | Persistent Connections  | Keep connection open for multiple      ||
|   | 지속 연결               | requests (default behavior)            ||
|   |                         | 여러 요청에 연결 유지 (기본 동작)      ||
|   +-------------------------+----------------------------------------+|
|   | Pipelining              | Send multiple requests without         ||
|   | 파이프라이닝            | waiting for responses                  ||
|   |                         | 응답 대기 없이 여러 요청 전송          ||
|   +-------------------------+----------------------------------------+|
|   | Host Header             | Required header for virtual hosting    ||
|   | Host 헤더               | 가상 호스팅을 위한 필수 헤더           ||
|   +-------------------------+----------------------------------------+|
|   | Chunked Transfer        | Stream response without knowing        ||
|   | 청크 전송               | total size                             ||
|   +-------------------------+----------------------------------------+|
|   | Cache Control           | Better caching with Cache-Control      ||
|   | 캐시 제어               | header                                 ||
|   +-------------------------+----------------------------------------+|
|   | New Methods             | PUT, DELETE, OPTIONS, TRACE            ||
|   | 새로운 메서드           |                                        ||
|   +-------------------------+----------------------------------------+|
|                                                                       |
+-----------------------------------------------------------------------+
```

### HTTP/1.0 vs HTTP/1.1 Comparison (비교)

```
+------------------------+----------------------+------------------------+
| Feature                | HTTP/1.0             | HTTP/1.1               |
+------------------------+----------------------+------------------------+
| Connection             | Close after each     | Persistent (Keep-Alive)|
| 연결                   | request              | 지속 연결              |
+------------------------+----------------------+------------------------+
| Host Header            | Optional             | Required               |
| Host 헤더              | 선택                 | 필수                   |
+------------------------+----------------------+------------------------+
| Pipelining             | Not supported        | Supported              |
| 파이프라이닝           | 미지원               | 지원                   |
+------------------------+----------------------+------------------------+
| Chunked Transfer       | Not supported        | Supported              |
| 청크 전송              | 미지원               | 지원                   |
+------------------------+----------------------+------------------------+
| Virtual Hosting        | Not possible         | Possible (Host header) |
| 가상 호스팅            | 불가능               | 가능                   |
+------------------------+----------------------+------------------------+
| Cache Control          | Expires only         | Cache-Control, ETag    |
| 캐시 제어              | Expires만            |                        |
+------------------------+----------------------+------------------------+
```

---

## 2. Keep-Alive Connection (Keep-Alive 연결)

### What is Keep-Alive? (Keep-Alive란?)

Keep-Alive allows multiple HTTP requests and responses to be sent over a single TCP connection, eliminating the overhead of establishing new connections.

Keep-Alive는 단일 TCP 연결을 통해 여러 HTTP 요청과 응답을 전송할 수 있게 하여 새 연결을 설정하는 오버헤드를 제거합니다.

```
+-----------------------------------------------------------------------+
|                    Without Keep-Alive (HTTP/1.0)                       |
+-----------------------------------------------------------------------+
|                                                                       |
|   Time                                                                |
|     |                                                                 |
|     |   [TCP] [GET 1] [TCP] [GET 2] [TCP] [GET 3]                     |
|     |   +---+ +-----+ +---+ +-----+ +---+ +-----+                     |
|     |   |SYN| |req 1| |SYN| |req 2| |SYN| |req 3|                     |
|     |   |ACK| |res 1| |ACK| |res 2| |ACK| |res 3|                     |
|     |   |...| |FIN  | |...| |FIN  | |...| |FIN  |                     |
|     |   +---+ +-----+ +---+ +-----+ +---+ +-----+                     |
|     v                                                                 |
|                                                                       |
|   Total time = (TCP handshake + Request/Response) x 3                 |
|                                                                       |
+-----------------------------------------------------------------------+

+-----------------------------------------------------------------------+
|                    With Keep-Alive (HTTP/1.1)                          |
+-----------------------------------------------------------------------+
|                                                                       |
|   Time                                                                |
|     |                                                                 |
|     |   [TCP] [GET 1] [GET 2] [GET 3] [FIN]                           |
|     |   +---+ +-----+ +-----+ +-----+ +---+                           |
|     |   |SYN| |req 1| |req 2| |req 3| |FIN|                           |
|     |   |ACK| |res 1| |res 2| |res 3| |ACK|                           |
|     |   |...| +-----+ +-----+ +-----+ +---+                           |
|     |   +---+                                                         |
|     v                                                                 |
|                                                                       |
|   Total time = TCP handshake + (Request/Response x 3)                 |
|   Much faster! (훨씬 빠름!)                                           |
|                                                                       |
+-----------------------------------------------------------------------+
```

### Keep-Alive Headers (Keep-Alive 헤더)

```
+-----------------------------------------------------------------------+
|                    HTTP/1.1 Keep-Alive Headers                         |
+-----------------------------------------------------------------------+
|                                                                       |
|   Request:                                                            |
|   +---------------------------------------------------------------+   |
|   | GET /index.html HTTP/1.1                                      |   |
|   | Host: www.example.com                                         |   |
|   | Connection: keep-alive    (default in HTTP/1.1)               |   |
|   +---------------------------------------------------------------+   |
|                                                                       |
|   Response:                                                           |
|   +---------------------------------------------------------------+   |
|   | HTTP/1.1 200 OK                                               |   |
|   | Connection: keep-alive                                        |   |
|   | Keep-Alive: timeout=5, max=100                                |   |
|   |             ^           ^                                     |   |
|   |             |           |                                     |   |
|   |    Idle timeout    Max requests                               |   |
|   |    유휴 타임아웃   최대 요청 수                                |   |
|   +---------------------------------------------------------------+   |
|                                                                       |
|   To close connection:                                                |
|   +---------------------------------------------------------------+   |
|   | Connection: close                                             |   |
|   +---------------------------------------------------------------+   |
|                                                                       |
+-----------------------------------------------------------------------+
```

### HTTP/1.1 Pipelining (HTTP/1.1 파이프라이닝)

```
+-----------------------------------------------------------------------+
|                    HTTP/1.1 Pipelining                                 |
+-----------------------------------------------------------------------+
|                                                                       |
|   Concept: Send multiple requests without waiting for responses       |
|   개념: 응답을 기다리지 않고 여러 요청 전송                           |
|                                                                       |
|   Client                                            Server            |
|      |                                                |               |
|      |  GET /a.html  |                                |               |
|      |  GET /b.css   |  (all at once)                 |               |
|      |  GET /c.js    |                                |               |
|      | ----------------------------------->            |               |
|      |                                                |               |
|      |               | Response /a.html               |               |
|      |               | Response /b.css  (in order)    |               |
|      |               | Response /c.js                 |               |
|      | <-----------------------------------           |               |
|      |                                                |               |
|                                                                       |
+-----------------------------------------------------------------------+
|                                                                       |
|   HEAD-OF-LINE BLOCKING Problem (HOL 블로킹 문제):                    |
|                                                                       |
|      |  GET /big.jpg    |                             |               |
|      |  GET /small.css  |                             |               |
|      |  GET /tiny.js    |                             |               |
|      | ----------------------------------->            |               |
|      |                                                |               |
|      |               | Response /big.jpg  [SLOW!]     |               |
|      |               | ....waiting....                |               |
|      |               | ....waiting....                |               |
|      |               | Response /small.css            |               |
|      |               | Response /tiny.js              |               |
|      | <-----------------------------------           |               |
|      |                                                |               |
|   small.css and tiny.js blocked by slow big.jpg!                      |
|   slow한 big.jpg 때문에 small.css와 tiny.js가 블로킹!                 |
|                                                                       |
|   Note: Most browsers disabled pipelining due to this problem         |
|   참고: 대부분의 브라우저가 이 문제로 파이프라이닝 비활성화           |
|                                                                       |
+-----------------------------------------------------------------------+
```

### HTTP/1.1 Workarounds (HTTP/1.1 우회 방법)

```
+-----------------------------------------------------------------------+
|                    HTTP/1.1 Performance Workarounds                    |
+-----------------------------------------------------------------------+
|                                                                       |
|   1. Multiple Parallel Connections (다중 병렬 연결)                   |
|   +---------------------------------------------------------------+   |
|   |   Browser opens 6-8 connections per domain                    |   |
|   |   브라우저가 도메인당 6-8개 연결 열기                         |   |
|   |                                                               |   |
|   |   Connection 1: GET /a.html  ---->  Response                  |   |
|   |   Connection 2: GET /b.css   ---->  Response                  |   |
|   |   Connection 3: GET /c.js    ---->  Response                  |   |
|   |   Connection 4: GET /d.png   ---->  Response                  |   |
|   |   ...                                                         |   |
|   +---------------------------------------------------------------+   |
|                                                                       |
|   2. Domain Sharding (도메인 샤딩)                                    |
|   +---------------------------------------------------------------+   |
|   |   Split resources across multiple domains                     |   |
|   |   리소스를 여러 도메인에 분산                                 |   |
|   |                                                               |   |
|   |   cdn1.example.com  -->  6 connections                        |   |
|   |   cdn2.example.com  -->  6 connections                        |   |
|   |   cdn3.example.com  -->  6 connections                        |   |
|   |   Total: 18 parallel connections!                             |   |
|   +---------------------------------------------------------------+   |
|                                                                       |
|   3. Resource Bundling (리소스 번들링)                                |
|   +---------------------------------------------------------------+   |
|   |   Combine multiple files into one                             |   |
|   |   여러 파일을 하나로 결합                                     |   |
|   |                                                               |   |
|   |   a.js + b.js + c.js  -->  bundle.js                          |   |
|   |   a.css + b.css       -->  styles.css                         |   |
|   +---------------------------------------------------------------+   |
|                                                                       |
|   4. Image Sprites (이미지 스프라이트)                                |
|   +---------------------------------------------------------------+   |
|   |   Combine multiple images into one                            |   |
|   |   여러 이미지를 하나로 결합                                   |   |
|   |                                                               |   |
|   |   +-----+-----+-----+                                         |   |
|   |   |icon1|icon2|icon3|  <-- One image file                     |   |
|   |   +-----+-----+-----+                                         |   |
|   |   Use CSS background-position to show each icon               |   |
|   +---------------------------------------------------------------+   |
|                                                                       |
|   Note: These workarounds are ANTI-PATTERNS in HTTP/2!                |
|   참고: 이러한 우회 방법은 HTTP/2에서는 안티패턴!                     |
|                                                                       |
+-----------------------------------------------------------------------+
```

---

## 3. HTTP/2 Features (HTTP/2 기능)

### HTTP/2 Overview (HTTP/2 개요)

```
+-----------------------------------------------------------------------+
|                    HTTP/2 Introduction                                 |
+-----------------------------------------------------------------------+
|                                                                       |
|   Released: 2015 (RFC 7540)                                           |
|   발표: 2015년                                                        |
|                                                                       |
|   Based on Google's SPDY protocol                                     |
|   Google의 SPDY 프로토콜 기반                                         |
|                                                                       |
|   Major Changes:                                                      |
|   +-------------------------------------------------------------------+
|   | 1. Binary Protocol (not text like HTTP/1.1)                      |
|   |    바이너리 프로토콜 (HTTP/1.1처럼 텍스트가 아님)                |
|   +-------------------------------------------------------------------+
|   | 2. Multiplexing (multiple streams over one connection)           |
|   |    다중화 (하나의 연결에서 여러 스트림)                          |
|   +-------------------------------------------------------------------+
|   | 3. Header Compression (HPACK)                                    |
|   |    헤더 압축                                                     |
|   +-------------------------------------------------------------------+
|   | 4. Server Push                                                   |
|   |    서버 푸시                                                     |
|   +-------------------------------------------------------------------+
|   | 5. Stream Prioritization                                         |
|   |    스트림 우선순위                                               |
|   +-------------------------------------------------------------------+
|                                                                       |
+-----------------------------------------------------------------------+
```

### Binary Framing Layer (바이너리 프레이밍 계층)

```
+-----------------------------------------------------------------------+
|                    HTTP/1.1 vs HTTP/2 Format                           |
+-----------------------------------------------------------------------+
|                                                                       |
|   HTTP/1.1 (Text-based):                                              |
|   +---------------------------------------------------------------+   |
|   | GET /index.html HTTP/1.1\r\n                                  |   |
|   | Host: example.com\r\n                                         |   |
|   | User-Agent: Mozilla/5.0\r\n                                   |   |
|   | Accept: text/html\r\n                                         |   |
|   | \r\n                                                          |   |
|   +---------------------------------------------------------------+   |
|   Human readable, but inefficient                                     |
|   사람이 읽을 수 있지만 비효율적                                      |
|                                                                       |
+-----------------------------------------------------------------------+
|                                                                       |
|   HTTP/2 (Binary frames):                                             |
|   +---------------------------------------------------------------+   |
|   | Frame Header (9 bytes)                                        |   |
|   | +--------+--------+--------+--------+--------+                |   |
|   | | Length | Type   | Flags  | Stream Identifier |              |   |
|   | | 3 bytes| 1 byte | 1 byte | 4 bytes           |              |   |
|   | +--------+--------+--------+--------+--------+                |   |
|   | Frame Payload                                                 |   |
|   | +-----------------------------------------------------+       |   |
|   | | [Binary encoded headers or data]                    |       |   |
|   | +-----------------------------------------------------+       |   |
|   +---------------------------------------------------------------+   |
|   Efficient parsing, smaller size                                     |
|   효율적인 파싱, 작은 크기                                            |
|                                                                       |
+-----------------------------------------------------------------------+
```

### HTTP/2 Multiplexing (HTTP/2 다중화)

```
+-----------------------------------------------------------------------+
|                    HTTP/2 Multiplexing                                 |
+-----------------------------------------------------------------------+
|                                                                       |
|   One TCP connection, multiple streams interleaved                    |
|   하나의 TCP 연결, 여러 스트림이 인터리빙됨                           |
|                                                                       |
|   +---------------------------------------------------------------+   |
|   |                     Single TCP Connection                     |   |
|   +---------------------------------------------------------------+   |
|   |                                                               |   |
|   |  Stream 1 (HTML)   [====]        [==]                         |   |
|   |  Stream 3 (CSS)         [===]         [====]                  |   |
|   |  Stream 5 (JS)              [==]            [===]             |   |
|   |  Stream 7 (Image)                [=====]         [===]        |   |
|   |                                                               |   |
|   |  Time ---------------------------------------------------->   |   |
|   |                                                               |   |
|   +---------------------------------------------------------------+   |
|                                                                       |
|   Benefits:                                                           |
|   - No head-of-line blocking at HTTP level                            |
|   - HTTP 수준에서 HOL 블로킹 없음                                     |
|   - Better utilization of single connection                           |
|   - 단일 연결의 더 나은 활용                                          |
|   - No need for multiple connections                                  |
|   - 다중 연결 불필요                                                  |
|                                                                       |
+-----------------------------------------------------------------------+

+-----------------------------------------------------------------------+
|                    HTTP/1.1 vs HTTP/2 Request Flow                     |
+-----------------------------------------------------------------------+
|                                                                       |
|   HTTP/1.1 (6 connections):                                           |
|                                                                       |
|   Conn 1: [---HTML---]                                                |
|   Conn 2:     [----CSS----]                                           |
|   Conn 3:         [---JS---]                                          |
|   Conn 4:             [----IMG1----]                                  |
|   Conn 5:                 [---IMG2---]                                |
|   Conn 6:                     [--IMG3--]                              |
|                                                                       |
+-----------------------------------------------------------------------+
|                                                                       |
|   HTTP/2 (1 connection):                                              |
|                                                                       |
|   Single: [HTML][CSS][JS][IMG1][IMG2][IMG3]  (interleaved frames)     |
|                                                                       |
|   Faster overall, less overhead, better for high latency networks!    |
|   전체적으로 더 빠르고, 오버헤드 적고, 고지연 네트워크에 더 좋음!     |
|                                                                       |
+-----------------------------------------------------------------------+
```

### HPACK Header Compression (HPACK 헤더 압축)

```
+-----------------------------------------------------------------------+
|                    HPACK Header Compression                            |
+-----------------------------------------------------------------------+
|                                                                       |
|   Problem: Headers are repetitive and large                           |
|   문제: 헤더가 반복적이고 큼                                          |
|                                                                       |
|   Every request includes:                                             |
|   +---------------------------------------------------------------+   |
|   | :method: GET                                                  |   |
|   | :path: /page.html                                             |   |
|   | :authority: example.com                                       |   |
|   | user-agent: Mozilla/5.0 (Windows NT...) [~200 bytes!]         |   |
|   | accept: text/html,application/xhtml+xml... [~100 bytes]       |   |
|   | accept-language: en-US,en;q=0.9,ko;q=0.8                      |   |
|   | cookie: session=abc123xyz... [can be huge!]                   |   |
|   +---------------------------------------------------------------+   |
|                                                                       |
|   HPACK Solution:                                                     |
|   +---------------------------------------------------------------+   |
|   | 1. Static Table (61 common headers pre-defined)               |   |
|   |    정적 테이블 (61개의 일반 헤더 미리 정의)                   |   |
|   |                                                               |   |
|   |    Index | Header Name    | Header Value                      |   |
|   |    1     | :authority     | (empty)                           |   |
|   |    2     | :method        | GET                               |   |
|   |    3     | :method        | POST                              |   |
|   |    ...   | ...            | ...                               |   |
|   +---------------------------------------------------------------+   |
|   | 2. Dynamic Table (connection-specific, grows with requests)   |   |
|   |    동적 테이블 (연결별로 요청에 따라 증가)                    |   |
|   |                                                               |   |
|   |    Index | Header Name    | Header Value                      |   |
|   |    62    | :authority     | example.com                       |   |
|   |    63    | cookie         | session=abc123                    |   |
|   +---------------------------------------------------------------+   |
|   | 3. Huffman Encoding (for literal values)                      |   |
|   |    허프만 인코딩 (리터럴 값용)                                |   |
|   +---------------------------------------------------------------+   |
|                                                                       |
+-----------------------------------------------------------------------+

Example compression:
+-----------------------------------------------------------------------+
|                                                                       |
|   First request (1st):                                                |
|   :method: GET          --> Index 2 (1 byte)                          |
|   :path: /index.html    --> Literal + add to dynamic table            |
|   cookie: session=abc   --> Literal + add to dynamic table            |
|                                                                       |
|   Second request (2nd):                                               |
|   :method: GET          --> Index 2 (1 byte)                          |
|   :path: /style.css     --> Literal (new path)                        |
|   cookie: session=abc   --> Index 63 (1 byte!) <-- from dynamic table |
|                                                                       |
|   Result: ~85-90% header size reduction on subsequent requests!       |
|   결과: 이후 요청에서 헤더 크기 약 85-90% 감소!                       |
|                                                                       |
+-----------------------------------------------------------------------+
```

### HTTP/2 Server Push (HTTP/2 서버 푸시)

```
+-----------------------------------------------------------------------+
|                    HTTP/2 Server Push                                  |
+-----------------------------------------------------------------------+
|                                                                       |
|   Concept: Server proactively sends resources client will need        |
|   개념: 서버가 클라이언트가 필요할 리소스를 사전에 전송               |
|                                                                       |
|   Without Server Push:                                                |
|   +---------------------------------------------------------------+   |
|   | Client                              Server                    |   |
|   |    |                                   |                      |   |
|   |    |  GET /index.html --------------> |                      |   |
|   |    |  <-------------- index.html      |                      |   |
|   |    |                                   |                      |   |
|   |    | [Parse HTML, discover needs CSS] |                      |   |
|   |    |                                   |                      |   |
|   |    |  GET /style.css ---------------> |  <-- Extra round trip|   |
|   |    |  <-------------- style.css       |                      |   |
|   +---------------------------------------------------------------+   |
|                                                                       |
|   With Server Push:                                                   |
|   +---------------------------------------------------------------+   |
|   | Client                              Server                    |   |
|   |    |                                   |                      |   |
|   |    |  GET /index.html --------------> |                      |   |
|   |    |  <-------------- index.html      |                      |   |
|   |    |  <-------------- PUSH style.css  |  <-- Proactive push! |   |
|   |    |                                   |     사전 푸시!       |   |
|   |    | [HTML parsed, CSS already there!]|                      |   |
|   +---------------------------------------------------------------+   |
|                                                                       |
|   Server sends PUSH_PROMISE frame to announce push                    |
|   서버가 PUSH_PROMISE 프레임을 보내 푸시 알림                         |
|                                                                       |
|   Client can reject push with RST_STREAM if not needed                |
|   클라이언트는 필요 없으면 RST_STREAM으로 푸시 거절 가능              |
|                                                                       |
+-----------------------------------------------------------------------+

Note: Server Push has been removed from Chrome as of 2022 due to
limited adoption and complexity. HTTP 103 Early Hints is preferred.
참고: 서버 푸시는 제한적 채택과 복잡성으로 2022년 Chrome에서 제거됨.
HTTP 103 Early Hints가 선호됨.
```

### Stream Prioritization (스트림 우선순위)

```
+-----------------------------------------------------------------------+
|                    HTTP/2 Stream Prioritization                        |
+-----------------------------------------------------------------------+
|                                                                       |
|   Concept: Client can indicate importance of different resources      |
|   개념: 클라이언트가 다른 리소스의 중요도 표시 가능                   |
|                                                                       |
|   Priority Tree Example:                                              |
|                                                                       |
|                    +----------------+                                 |
|                    |  Stream 0      |  Root                           |
|                    |  (Connection)  |                                 |
|                    +-------+--------+                                 |
|                            |                                          |
|            +---------------+---------------+                          |
|            |                               |                          |
|     +------+------+                 +------+------+                    |
|     |  Stream 1   |                 |  Stream 3   |                    |
|     |  HTML       |                 |  CSS        |                    |
|     |  Weight: 256|                 |  Weight: 256|                    |
|     +------+------+                 +-------------+                    |
|            |                                                          |
|     +------+------+                                                   |
|     |  Stream 5   |                                                   |
|     |  JS         |                                                   |
|     |  Weight: 128|  (depends on HTML)                                |
|     +-------------+                                                   |
|                                                                       |
|   Server uses priorities to decide which data to send first           |
|   서버가 우선순위를 사용하여 어떤 데이터를 먼저 보낼지 결정           |
|                                                                       |
+-----------------------------------------------------------------------+
```

---

## 4. HTTP/3 and QUIC (HTTP/3와 QUIC)

### Why HTTP/3? (왜 HTTP/3인가?)

```
+-----------------------------------------------------------------------+
|                    HTTP/2 Limitations                                  |
+-----------------------------------------------------------------------+
|                                                                       |
|   Problem: TCP Head-of-Line Blocking                                  |
|   문제: TCP 헤드 오브 라인 블로킹                                     |
|                                                                       |
|   HTTP/2 solved HOL at HTTP layer, but TCP still has the problem!     |
|   HTTP/2가 HTTP 계층에서 HOL을 해결했지만 TCP는 여전히 문제가 있음!   |
|                                                                       |
|   +---------------------------------------------------------------+   |
|   |  TCP Stream (single ordered byte stream)                      |   |
|   |                                                               |   |
|   |  [Packet 1][Packet 2][  LOST  ][Packet 4][Packet 5]           |   |
|   |      OK       OK     Packet 3     WAIT      WAIT              |   |
|   |                         |          |          |               |   |
|   |                         v          v          v               |   |
|   |                    All packets after lost packet must wait!   |   |
|   |                    손실된 패킷 이후 모든 패킷이 대기해야 함!  |   |
|   +---------------------------------------------------------------+   |
|                                                                       |
|   Even though Packets 4 and 5 are for different HTTP/2 streams,       |
|   they are blocked waiting for Packet 3 retransmission.               |
|   패킷 4와 5가 다른 HTTP/2 스트림용이어도                             |
|   패킷 3 재전송을 기다려야 함                                         |
|                                                                       |
+-----------------------------------------------------------------------+
```

### QUIC Protocol (QUIC 프로토콜)

```
+-----------------------------------------------------------------------+
|                    QUIC Overview                                       |
+-----------------------------------------------------------------------+
|                                                                       |
|   QUIC = Quick UDP Internet Connections                               |
|   Originally developed by Google, now IETF standard (RFC 9000)        |
|   Google이 개발, 현재 IETF 표준 (RFC 9000)                            |
|                                                                       |
|   Key Features:                                                       |
|   +-------------------------------------------------------------------+
|   | 1. Built on UDP (not TCP)                                        |
|   |    UDP 기반 (TCP가 아님)                                         |
|   +-------------------------------------------------------------------+
|   | 2. Integrated TLS 1.3                                            |
|   |    TLS 1.3 통합                                                  |
|   +-------------------------------------------------------------------+
|   | 3. Stream multiplexing without HOL blocking                      |
|   |    HOL 블로킹 없는 스트림 다중화                                 |
|   +-------------------------------------------------------------------+
|   | 4. Connection migration (IP address change)                      |
|   |    연결 마이그레이션 (IP 주소 변경)                              |
|   +-------------------------------------------------------------------+
|   | 5. 0-RTT connection establishment                                |
|   |    0-RTT 연결 설정                                               |
|   +-------------------------------------------------------------------+
|                                                                       |
+-----------------------------------------------------------------------+
```

### Protocol Stack Comparison (프로토콜 스택 비교)

```
+-----------------------------------------------------------------------+
|                    HTTP/2 vs HTTP/3 Stack                              |
+-----------------------------------------------------------------------+
|                                                                       |
|      HTTP/2 Stack                    HTTP/3 Stack                     |
|                                                                       |
|   +---------------+              +---------------+                    |
|   |    HTTP/2     |              |    HTTP/3     |                    |
|   +---------------+              +---------------+                    |
|   |     TLS       |              |     QUIC      |  <-- Includes TLS  |
|   +---------------+              +---------------+                    |
|   |     TCP       |              |     UDP       |                    |
|   +---------------+              +---------------+                    |
|   |      IP       |              |      IP       |                    |
|   +---------------+              +---------------+                    |
|                                                                       |
|   HTTP/2: 3 layers (TCP + TLS + HTTP/2)                               |
|   HTTP/3: 2 layers (UDP + QUIC which includes TLS + HTTP/3)           |
|                                                                       |
+-----------------------------------------------------------------------+
```

### QUIC Solves HOL Blocking (QUIC의 HOL 블로킹 해결)

```
+-----------------------------------------------------------------------+
|                    QUIC Independent Streams                            |
+-----------------------------------------------------------------------+
|                                                                       |
|   TCP (HTTP/2): Single stream, HOL blocking                           |
|   +---------------------------------------------------------------+   |
|   |  [Stream 1 data][Stream 2 data][LOST][Stream 1][Stream 2]     |   |
|   |        OK            OK        Packet      BLOCKED!           |   |
|   |                                  3                            |   |
|   +---------------------------------------------------------------+   |
|                                                                       |
|   QUIC (HTTP/3): Independent streams, no HOL blocking                 |
|   +---------------------------------------------------------------+   |
|   |  QUIC Connection                                              |   |
|   |  +-------------------+                                        |   |
|   |  | Stream 1: [data][data][ LOST ][data] <-- Wait for retx    |   |
|   |  +-------------------+                                        |   |
|   |  +-------------------+                                        |   |
|   |  | Stream 2: [data][data][data][data]   <-- Not blocked!     |   |
|   |  +-------------------+                                        |   |
|   |  +-------------------+                                        |   |
|   |  | Stream 3: [data][data][data]         <-- Not blocked!     |   |
|   |  +-------------------+                                        |   |
|   +---------------------------------------------------------------+   |
|                                                                       |
|   Packet loss in Stream 1 doesn't block Streams 2 and 3!              |
|   스트림 1의 패킷 손실이 스트림 2, 3을 블로킹하지 않음!               |
|                                                                       |
+-----------------------------------------------------------------------+
```

### QUIC Connection Establishment (QUIC 연결 설정)

```
+-----------------------------------------------------------------------+
|                    TCP+TLS vs QUIC Handshake                           |
+-----------------------------------------------------------------------+
|                                                                       |
|   TCP + TLS 1.3 (HTTP/2):                                             |
|   +---------------------------------------------------------------+   |
|   | Client                              Server                    |   |
|   |    |  TCP SYN ----------------------> |                      |   |
|   |    |  <-------------------- SYN-ACK   |  1 RTT (TCP)          |   |
|   |    |  ACK ------------------------->  |                      |   |
|   |    |                                  |                      |   |
|   |    |  TLS ClientHello --------------> |                      |   |
|   |    |  <-------- ServerHello+Finished  |  1 RTT (TLS)          |   |
|   |    |  Finished ---------------------->|                      |   |
|   |    |                                  |                      |   |
|   |    |  [Now can send data]             |  Total: 2 RTT         |   |
|   +---------------------------------------------------------------+   |
|                                                                       |
|   QUIC (HTTP/3):                                                      |
|   +---------------------------------------------------------------+   |
|   | Client                              Server                    |   |
|   |    |  Initial (ClientHello) -------> |                      |   |
|   |    |  <----- Initial (ServerHello)   |  1 RTT                 |   |
|   |    |         + Handshake             |  (combined!)           |   |
|   |    |  Handshake + Data -------------> |                      |   |
|   |    |                                  |                      |   |
|   |    |  [Can send data in 1 RTT!]       |  Total: 1 RTT         |   |
|   +---------------------------------------------------------------+   |
|                                                                       |
|   QUIC 0-RTT Resumption:                                              |
|   +---------------------------------------------------------------+   |
|   | Client                              Server                    |   |
|   |    |  Initial + Early Data --------> |  0 RTT!                |   |
|   |    |  <----------- Server Response   |  (using cached keys)   |   |
|   +---------------------------------------------------------------+   |
|                                                                       |
+-----------------------------------------------------------------------+
```

### Connection Migration (연결 마이그레이션)

```
+-----------------------------------------------------------------------+
|                    QUIC Connection Migration                           |
+-----------------------------------------------------------------------+
|                                                                       |
|   TCP Connection Identity: (Source IP, Source Port, Dest IP, Dest Port)
|   TCP 연결 식별: (출발지 IP, 출발지 포트, 목적지 IP, 목적지 포트)     |
|                                                                       |
|   Problem: When IP changes, connection breaks!                        |
|   문제: IP가 바뀌면 연결이 끊김!                                      |
|                                                                       |
|   +---------------------------------------------------------------+   |
|   | TCP: WiFi to Mobile                                           |   |
|   |                                                               |   |
|   | [Phone on WiFi]          [Phone on Mobile]                    |   |
|   | IP: 192.168.1.50   -->   IP: 10.0.0.50                        |   |
|   |       |                        |                              |   |
|   |       | TCP Connection         X TCP Connection LOST!         |   |
|   |       | to Server              | Must reconnect               |   |
|   |       v                        v                              |   |
|   |    [Server]                 [Server]                          |   |
|   +---------------------------------------------------------------+   |
|                                                                       |
|   QUIC Solution: Connection ID (not tied to IP)                       |
|   QUIC 해결책: 연결 ID (IP에 종속되지 않음)                           |
|                                                                       |
|   +---------------------------------------------------------------+   |
|   | QUIC: WiFi to Mobile                                          |   |
|   |                                                               |   |
|   | [Phone on WiFi]          [Phone on Mobile]                    |   |
|   | IP: 192.168.1.50   -->   IP: 10.0.0.50                        |   |
|   | Conn ID: abc123          Conn ID: abc123 (same!)              |   |
|   |       |                        |                              |   |
|   |       | QUIC Connection        | QUIC Connection CONTINUES!   |   |
|   |       | to Server              | Seamless handoff             |   |
|   |       v                        v                              |   |
|   |    [Server]                 [Server]                          |   |
|   +---------------------------------------------------------------+   |
|                                                                       |
|   Great for mobile users switching between WiFi and cellular!         |
|   WiFi와 셀룰러 사이를 전환하는 모바일 사용자에게 좋음!               |
|                                                                       |
+-----------------------------------------------------------------------+
```

### HTTP Version Comparison (HTTP 버전 비교)

```
+-------------------+------------------+------------------+------------------+
| Feature           | HTTP/1.1         | HTTP/2           | HTTP/3           |
+-------------------+------------------+------------------+------------------+
| Transport         | TCP              | TCP              | QUIC (UDP)       |
| 전송 프로토콜     |                  |                  |                  |
+-------------------+------------------+------------------+------------------+
| Multiplexing      | No (needs        | Yes (streams)    | Yes (streams)    |
| 다중화            | multi conn)      |                  |                  |
+-------------------+------------------+------------------+------------------+
| HOL Blocking      | HTTP + TCP       | TCP only         | None             |
| HOL 블로킹        | HTTP + TCP       | TCP만            | 없음             |
+-------------------+------------------+------------------+------------------+
| Header Compress   | No               | HPACK            | QPACK            |
| 헤더 압축         | 없음             |                  |                  |
+-------------------+------------------+------------------+------------------+
| Server Push       | No               | Yes              | Yes              |
| 서버 푸시         | 없음             | 있음             | 있음             |
+-------------------+------------------+------------------+------------------+
| TLS               | Optional         | Practically req. | Built-in (1.3)   |
|                   | 선택             | 사실상 필수      | 내장             |
+-------------------+------------------+------------------+------------------+
| Connection setup  | TCP: 1 RTT       | TCP: 1 RTT       | 1 RTT            |
| 연결 설정         | TLS: 1-2 RTT     | TLS: 1-2 RTT     | (0 RTT resumption)|
+-------------------+------------------+------------------+------------------+
| Conn Migration    | No               | No               | Yes              |
| 연결 마이그레이션 | 없음             | 없음             | 있음             |
+-------------------+------------------+------------------+------------------+
```

---

## 5. Summary (요약)

### Key Concepts (핵심 개념)

```
+-----------------------------------------------------------------------+
|                     Chapter 15 Summary                                 |
+-----------------------------------------------------------------------+
|                                                                       |
|  1. HTTP/1.0 vs HTTP/1.1                                              |
|     - HTTP/1.0: One request per connection (inefficient)              |
|     - HTTP/1.1: Persistent connections, pipelining, Host header       |
|     - HTTP/1.1 still has HOL blocking problem                         |
|                                                                       |
|  2. Keep-Alive (Keep-Alive 연결)                                      |
|     - Reuse TCP connection for multiple requests                      |
|     - Default in HTTP/1.1                                             |
|     - Reduces connection overhead significantly                       |
|                                                                       |
|  3. HTTP/2 Features (HTTP/2 기능)                                     |
|     - Binary protocol (efficient parsing)                             |
|     - Multiplexing (multiple streams, one connection)                 |
|     - HPACK header compression (85-90% reduction)                     |
|     - Server push (proactive resource sending)                        |
|     - Stream prioritization                                           |
|                                                                       |
|  4. HTTP/3 and QUIC (HTTP/3와 QUIC)                                   |
|     - Based on QUIC protocol (UDP-based)                              |
|     - No HOL blocking (independent streams)                           |
|     - Integrated TLS 1.3                                              |
|     - 1 RTT connection (0-RTT resumption)                             |
|     - Connection migration (IP change support)                        |
|                                                                       |
+-----------------------------------------------------------------------+
```

### Evolution Summary (발전 요약)

```
+-----------------------------------------------------------------------+
|                    HTTP Evolution Timeline                             |
+-----------------------------------------------------------------------+
|                                                                       |
|   HTTP/1.0 (1996)                                                     |
|   +-------------------------------------------------------------------+
|   | Problem: New TCP connection for each request                     |
|   | 문제: 각 요청마다 새 TCP 연결                                    |
|   +-------------------------------------------------------------------+
|                      |                                                |
|                      v                                                |
|   HTTP/1.1 (1997)                                                     |
|   +-------------------------------------------------------------------+
|   | Solution: Keep-Alive, pipelining                                 |
|   | 해결: Keep-Alive, 파이프라이닝                                   |
|   | Problem: HOL blocking, text-based overhead                       |
|   | 문제: HOL 블로킹, 텍스트 기반 오버헤드                           |
|   +-------------------------------------------------------------------+
|                      |                                                |
|                      v                                                |
|   HTTP/2 (2015)                                                       |
|   +-------------------------------------------------------------------+
|   | Solution: Binary, multiplexing, header compression               |
|   | 해결: 바이너리, 다중화, 헤더 압축                                |
|   | Problem: TCP HOL blocking still exists                           |
|   | 문제: TCP HOL 블로킹 여전히 존재                                 |
|   +-------------------------------------------------------------------+
|                      |                                                |
|                      v                                                |
|   HTTP/3 (2022)                                                       |
|   +-------------------------------------------------------------------+
|   | Solution: QUIC (UDP), no HOL blocking, connection migration      |
|   | 해결: QUIC (UDP), HOL 블로킹 없음, 연결 마이그레이션             |
|   +-------------------------------------------------------------------+
|                                                                       |
+-----------------------------------------------------------------------+
```

### When to Use What (언제 무엇을 사용할까)

```
+----------------------+-----------------------------------------------+
| Scenario             | Recommendation                                |
+----------------------+-----------------------------------------------+
| Legacy systems       | HTTP/1.1 (widely supported)                   |
| 레거시 시스템        | HTTP/1.1 (널리 지원됨)                        |
+----------------------+-----------------------------------------------+
| Modern web apps      | HTTP/2 (best browser support)                 |
| 현대 웹 앱           | HTTP/2 (최고의 브라우저 지원)                 |
+----------------------+-----------------------------------------------+
| High latency/mobile  | HTTP/3 (if supported)                         |
| 고지연/모바일        | HTTP/3 (지원되는 경우)                        |
+----------------------+-----------------------------------------------+
| Lossy networks       | HTTP/3 (no HOL blocking)                      |
| 손실이 많은 네트워크 | HTTP/3 (HOL 블로킹 없음)                      |
+----------------------+-----------------------------------------------+
```

---

*End of Chapter 15*
