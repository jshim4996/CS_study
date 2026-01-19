# Chapter 02. Web and HTTP (웹과 HTTP)

> Understanding the foundation of web communication through HTTP protocol and related concepts.

---

## Client-Server Model (클라이언트-서버 모델)

### Concept (개념)
The **Client-Server model** is a distributed application architecture where tasks are divided between providers of a resource or service (servers) and service requesters (clients).

**Characteristics:**
- **Client (클라이언트)**: Initiates requests, waits for responses (e.g., web browser)
- **Server (서버)**: Always-on host, waits for requests, provides services (e.g., web server)
- **Request-Response Pattern**: Client sends request, server processes, server sends response

**Alternatives:**
- **Peer-to-Peer (P2P)**: Both parties act as client and server (e.g., BitTorrent)
- **Hybrid**: Combination of both models

### Diagram / Example
```
Client-Server Model:

[Browser]  ───── Request ─────>  [Web Server]
(Client)   <──── Response ─────  (Server)
                                      |
                                      v
                               [Database Server]

Example:
1. User types www.example.com in browser
2. Browser (client) sends HTTP request to web server
3. Web server processes request, queries database if needed
4. Web server sends HTTP response with HTML content
5. Browser renders the webpage
```

---

## HTTP Concepts (HTTP 개념)

### Concept (개념)
**HTTP (HyperText Transfer Protocol)** is an application-layer protocol for transmitting hypermedia documents. It is the foundation of data communication on the World Wide Web.

**Key Characteristics:**
- **Stateless (무상태)**: Each request is independent; server doesn't remember previous requests
- **Text-based**: Human-readable protocol messages
- **Request-Response**: Client initiates, server responds
- **Port**: Default port 80 (HTTP), 443 (HTTPS)

**HTTP Versions:**
| Version | Year | Key Features |
|---------|------|--------------|
| HTTP/0.9 | 1991 | GET only, no headers |
| HTTP/1.0 | 1996 | Headers, status codes, POST/HEAD |
| HTTP/1.1 | 1997 | Persistent connections, pipelining |
| HTTP/2 | 2015 | Binary, multiplexing, server push |
| HTTP/3 | 2022 | QUIC (UDP-based), improved performance |

### Diagram / Example
```
HTTP Communication Flow:

[Client]                              [Server]
   |                                     |
   | ---- TCP Connection (3-way) ---->   |
   |                                     |
   | ---- HTTP Request --------------->  |
   |      GET /index.html HTTP/1.1       |
   |      Host: www.example.com          |
   |                                     |
   | <--- HTTP Response ---------------  |
   |      HTTP/1.1 200 OK                |
   |      Content-Type: text/html        |
   |      <html>...</html>               |
   |                                     |
```

---

## HTTP Message Structure (HTTP 메시지 구조)

### Concept (개념)
HTTP messages are how data is exchanged between client and server. There are two types:

**Request Message (요청 메시지):**
```
[Request Line]     Method SP URI SP Version CRLF
[Headers]          Header-Name: Header-Value CRLF
[Empty Line]       CRLF
[Body]             Optional message body
```

**Response Message (응답 메시지):**
```
[Status Line]      Version SP Status-Code SP Reason-Phrase CRLF
[Headers]          Header-Name: Header-Value CRLF
[Empty Line]       CRLF
[Body]             Optional message body
```

### Diagram / Example
```
HTTP Request Example:
┌─────────────────────────────────────────┐
│ GET /api/users HTTP/1.1                 │ ← Request Line (요청 라인)
├─────────────────────────────────────────┤
│ Host: api.example.com                   │
│ Accept: application/json                │ ← Headers (헤더)
│ Authorization: Bearer token123          │
├─────────────────────────────────────────┤
│                                         │ ← Empty Line (빈 줄)
├─────────────────────────────────────────┤
│ (no body for GET request)               │ ← Body (본문)
└─────────────────────────────────────────┘

HTTP Response Example:
┌─────────────────────────────────────────┐
│ HTTP/1.1 200 OK                         │ ← Status Line (상태 라인)
├─────────────────────────────────────────┤
│ Content-Type: application/json          │
│ Content-Length: 85                      │ ← Headers (헤더)
│ Date: Mon, 19 Jan 2026 12:00:00 GMT     │
├─────────────────────────────────────────┤
│                                         │ ← Empty Line (빈 줄)
├─────────────────────────────────────────┤
│ {"users": [{"id": 1, "name": "John"}]}  │ ← Body (본문)
└─────────────────────────────────────────┘
```

---

## HTTP Methods (HTTP 메서드)

### Concept (개념)
HTTP methods (also called verbs) indicate the desired action to be performed on a resource.

| Method | Purpose | Idempotent | Safe | Body |
|--------|---------|------------|------|------|
| **GET** | Retrieve resource | Yes | Yes | No |
| **POST** | Create resource | No | No | Yes |
| **PUT** | Replace resource | Yes | No | Yes |
| **PATCH** | Partial update | No | No | Yes |
| **DELETE** | Remove resource | Yes | No | Optional |
| **HEAD** | GET without body | Yes | Yes | No |
| **OPTIONS** | Get allowed methods | Yes | Yes | No |

**Key Terms:**
- **Safe (안전)**: Method doesn't modify resources
- **Idempotent (멱등)**: Multiple identical requests have same effect as single request

### Diagram / Example
```
CRUD Operations Mapping:

Operation    HTTP Method    Example
---------    -----------    -------
Create       POST           POST /users
                           Body: {"name": "John", "email": "john@example.com"}

Read         GET            GET /users/1
                           (Retrieve user with ID 1)

Update       PUT/PATCH      PUT /users/1
                           Body: {"name": "John", "email": "new@example.com"}

Delete       DELETE         DELETE /users/1
                           (Remove user with ID 1)
```

---

## HTTP Status Codes (HTTP 상태 코드)

### Concept (개념)
Status codes are 3-digit numbers in the response indicating the result of the request.

| Range | Category | Description |
|-------|----------|-------------|
| **1xx** | Informational (정보) | Request received, processing |
| **2xx** | Success (성공) | Request successfully processed |
| **3xx** | Redirection (리다이렉션) | Further action needed |
| **4xx** | Client Error (클라이언트 오류) | Error in request |
| **5xx** | Server Error (서버 오류) | Server failed to process |

**Common Status Codes:**
| Code | Name | Description |
|------|------|-------------|
| 200 | OK | Request succeeded |
| 201 | Created | Resource created |
| 204 | No Content | Success, no body |
| 301 | Moved Permanently | Permanent redirect |
| 302 | Found | Temporary redirect |
| 304 | Not Modified | Use cached version |
| 400 | Bad Request | Invalid request syntax |
| 401 | Unauthorized | Authentication required |
| 403 | Forbidden | Access denied |
| 404 | Not Found | Resource not found |
| 500 | Internal Server Error | Server error |
| 502 | Bad Gateway | Invalid upstream response |
| 503 | Service Unavailable | Server overloaded |

### Diagram / Example
```
Status Code Decision Flow:

Request Received
      │
      ▼
┌─────────────┐     No      ┌─────────────┐
│ Valid       │ ──────────► │ 4xx Error   │
│ Request?    │             │ (400, 404)  │
└─────────────┘             └─────────────┘
      │ Yes
      ▼
┌─────────────┐     No      ┌─────────────┐
│ Authorized? │ ──────────► │ 401 / 403   │
└─────────────┘             └─────────────┘
      │ Yes
      ▼
┌─────────────┐     No      ┌─────────────┐
│ Server OK?  │ ──────────► │ 5xx Error   │
└─────────────┘             │ (500, 503)  │
      │ Yes                 └─────────────┘
      ▼
┌─────────────┐
│ 2xx Success │
│ (200, 201)  │
└─────────────┘
```

---

## HTTP Headers (HTTP 헤더)

### Concept (개념)
HTTP headers carry metadata about the request or response. They are key-value pairs separated by colons.

**Categories:**
- **General Headers**: Apply to both request and response
- **Request Headers**: Information about request or client
- **Response Headers**: Information about response or server
- **Entity Headers**: Information about the body

**Common Headers:**
| Header | Type | Purpose |
|--------|------|---------|
| `Host` | Request | Target domain name |
| `User-Agent` | Request | Client software info |
| `Accept` | Request | Acceptable content types |
| `Content-Type` | Entity | Media type of body |
| `Content-Length` | Entity | Size of body in bytes |
| `Authorization` | Request | Authentication credentials |
| `Cache-Control` | General | Caching directives |
| `Set-Cookie` | Response | Set a cookie |
| `Cookie` | Request | Send cookies |

### Diagram / Example
```
Request Headers Example:
GET /api/data HTTP/1.1
Host: api.example.com                    ← Required in HTTP/1.1
User-Agent: Mozilla/5.0                  ← Browser identification
Accept: application/json, text/html      ← Preferred response formats
Accept-Language: en-US,en;q=0.9,ko;q=0.8 ← Preferred languages
Accept-Encoding: gzip, deflate, br       ← Supported compression
Connection: keep-alive                   ← Persistent connection
Authorization: Bearer eyJhbGc...         ← Authentication token

Response Headers Example:
HTTP/1.1 200 OK
Date: Mon, 19 Jan 2026 12:00:00 GMT      ← Response timestamp
Server: nginx/1.18.0                     ← Server software
Content-Type: application/json           ← Body format
Content-Length: 256                      ← Body size
Cache-Control: max-age=3600              ← Cache for 1 hour
Set-Cookie: session=abc123; HttpOnly     ← Set cookie
```

---

## Cookies and Sessions (쿠키와 세션)

### Concept (개념)
Since HTTP is stateless, cookies and sessions are used to maintain state across requests.

**Cookies (쿠키):**
- Small text files stored in the browser
- Sent automatically with each request to the same domain
- Used for: authentication, personalization, tracking

**Sessions (세션):**
- Server-side storage of user state
- Client receives session ID (usually in cookie)
- More secure than cookies (data stored on server)

**Cookie Attributes:**
| Attribute | Purpose |
|-----------|---------|
| `Expires/Max-Age` | Cookie lifetime |
| `Domain` | Which domains receive the cookie |
| `Path` | URL path the cookie applies to |
| `Secure` | Only send over HTTPS |
| `HttpOnly` | Not accessible via JavaScript |
| `SameSite` | CSRF protection |

### Diagram / Example
```
Cookie-based Session Flow:

[Browser]                                    [Server]
    |                                           |
    | ─── POST /login ────────────────────────> |
    |     username=john&password=secret         |
    |                                           |
    |     Server creates session:               |
    |     sessions["abc123"] = {user: "john"}   |
    |                                           |
    | <── 200 OK ──────────────────────────────  |
    |     Set-Cookie: sessionId=abc123;         |
    |                 HttpOnly; Secure          |
    |                                           |
    | [Browser stores cookie]                   |
    |                                           |
    | ─── GET /dashboard ─────────────────────> |
    |     Cookie: sessionId=abc123              |
    |                                           |
    |     Server looks up session["abc123"]     |
    |     Finds user: "john"                    |
    |                                           |
    | <── 200 OK (John's dashboard) ───────────  |
```

---

## REST API (REST API)

### Concept (개념)
**REST (Representational State Transfer)** is an architectural style for designing networked applications. RESTful APIs use HTTP methods to perform CRUD operations on resources.

**REST Principles:**
1. **Stateless**: Each request contains all necessary information
2. **Client-Server**: Separation of concerns
3. **Uniform Interface**: Consistent resource identification
4. **Layered System**: Client can't tell if connected directly to server
5. **Cacheable**: Responses must define cacheability

**RESTful URL Design:**
| Bad | Good |
|-----|------|
| `/getUsers` | `GET /users` |
| `/createUser` | `POST /users` |
| `/deleteUser?id=1` | `DELETE /users/1` |
| `/user/update/1` | `PUT /users/1` |

### Diagram / Example
```
RESTful API Design Example:

Resource: Users

┌────────────────────────────────────────────────────────────┐
│ Method    │ Endpoint         │ Description                 │
├───────────┼──────────────────┼─────────────────────────────┤
│ GET       │ /users           │ Get all users               │
│ GET       │ /users/{id}      │ Get specific user           │
│ POST      │ /users           │ Create new user             │
│ PUT       │ /users/{id}      │ Update entire user          │
│ PATCH     │ /users/{id}      │ Partial update              │
│ DELETE    │ /users/{id}      │ Delete user                 │
└────────────────────────────────────────────────────────────┘

Nested Resources:
GET    /users/1/posts        → Get all posts by user 1
POST   /users/1/posts        → Create post for user 1
GET    /users/1/posts/5      → Get post 5 by user 1

Query Parameters for Filtering/Pagination:
GET /users?role=admin&status=active
GET /users?page=2&limit=20&sort=name
```

---

## Summary (요약)

| Concept | Key Points |
|---------|------------|
| Client-Server | Request-response model, separation of roles |
| HTTP | Stateless, text-based, request-response protocol |
| Message Structure | Request/Status line + Headers + Body |
| Methods | GET, POST, PUT, PATCH, DELETE for CRUD |
| Status Codes | 1xx-5xx indicating request outcome |
| Headers | Metadata about request/response |
| Cookies/Sessions | State management in stateless HTTP |
| REST API | Resource-based URL design with HTTP methods |

---

## Key Terms (주요 용어)

- **HTTP (HyperText Transfer Protocol)**: 웹 통신 프로토콜
- **Stateless (무상태)**: 이전 요청을 기억하지 않음
- **Request (요청)**: 클라이언트가 서버에 보내는 메시지
- **Response (응답)**: 서버가 클라이언트에 보내는 메시지
- **Header (헤더)**: 메시지의 메타데이터
- **Cookie (쿠키)**: 브라우저에 저장되는 작은 데이터
- **Session (세션)**: 서버에 저장되는 사용자 상태
- **REST**: 자원 기반 API 설계 원칙
