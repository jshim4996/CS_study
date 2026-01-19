# Chapter 17. Real-time Communication (실시간 통신)

---

## Table of Contents (목차)

1. [Polling vs Long Polling](#1-polling-vs-long-polling)
2. [Server-Sent Events (SSE)](#2-server-sent-events-sse)
3. [WebSocket Concept and Handshake](#3-websocket-concept-and-handshake)
4. [WebSocket vs HTTP](#4-websocket-vs-http)
5. [Game Networking Basics](#5-game-networking-basics)
6. [Summary (요약)](#6-summary-요약)

---

## 1. Polling vs Long Polling

### The Real-time Challenge (실시간의 과제)

```
+-----------------------------------------------------------------------+
|                    The Problem: HTTP is Request-Response               |
+-----------------------------------------------------------------------+
|                                                                       |
|   Traditional HTTP:                                                   |
|   +---------------------------------------------------------------+   |
|   |   Client initiates ALL communication                          |   |
|   |   클라이언트가 모든 통신 시작                                 |   |
|   |                                                               |   |
|   |   Client                            Server                    |   |
|   |      |                                 |                      |   |
|   |      |  Request ------------------>    |                      |   |
|   |      |  <------------------ Response   |                      |   |
|   |      |                                 |                      |   |
|   |   Server CANNOT send data without a request!                  |   |
|   |   서버는 요청 없이 데이터를 보낼 수 없음!                     |   |
|   +---------------------------------------------------------------+   |
|                                                                       |
|   Real-time scenarios need:                                           |
|   - Chat messages (채팅 메시지)                                       |
|   - Live notifications (실시간 알림)                                  |
|   - Stock price updates (주가 업데이트)                               |
|   - Multiplayer game state (멀티플레이어 게임 상태)                   |
|                                                                       |
+-----------------------------------------------------------------------+
```

### Short Polling (짧은 폴링)

```
+-----------------------------------------------------------------------+
|                    Short Polling                                       |
+-----------------------------------------------------------------------+
|                                                                       |
|   Concept: Client repeatedly asks "Any new data?"                     |
|   개념: 클라이언트가 반복적으로 "새 데이터 있나요?" 질문              |
|                                                                       |
|   Client                                            Server            |
|      |                                                 |              |
|      |  GET /messages?after=100 ------------------>    |              |
|      |  <------------------- 200 OK (no new data)      |  t=0s       |
|      |                                                 |              |
|      |  ... wait 2 seconds ...                         |              |
|      |                                                 |              |
|      |  GET /messages?after=100 ------------------>    |              |
|      |  <------------------- 200 OK (no new data)      |  t=2s       |
|      |                                                 |              |
|      |  ... wait 2 seconds ...                         |              |
|      |                                                 |              |
|      |  GET /messages?after=100 ------------------>    |              |
|      |  <------------------- 200 OK (1 new message!)   |  t=4s       |
|      |                                                 |              |
|                                                                       |
+-----------------------------------------------------------------------+

JavaScript Example:
+-----------------------------------------------------------------------+
| function pollForMessages() {                                          |
|   setInterval(async () => {                                           |
|     const response = await fetch('/api/messages?after=' + lastId);    |
|     const data = await response.json();                               |
|     if (data.messages.length > 0) {                                   |
|       displayMessages(data.messages);                                 |
|       lastId = data.messages[data.messages.length - 1].id;            |
|     }                                                                 |
|   }, 2000);  // Poll every 2 seconds                                  |
| }                                                                     |
+-----------------------------------------------------------------------+

Pros (장점):
- Simple to implement (구현이 간단)
- Works everywhere (어디서나 작동)
- Stateless (상태 없음)

Cons (단점):
- High latency (up to polling interval) (높은 지연 시간 - 폴링 간격까지)
- Wastes bandwidth on empty responses (빈 응답에 대역폭 낭비)
- Server load from constant requests (지속적인 요청으로 서버 부하)
```

### Long Polling (롱 폴링)

```
+-----------------------------------------------------------------------+
|                    Long Polling                                        |
+-----------------------------------------------------------------------+
|                                                                       |
|   Concept: Server holds request open until data is available          |
|   개념: 데이터가 있을 때까지 서버가 요청을 열어둠                     |
|                                                                       |
|   Client                                            Server            |
|      |                                                 |              |
|      |  GET /messages --------------------------->     |              |
|      |                                                 |              |
|      |  ... server waits ...                           |  Held open   |
|      |  ... server waits ...                           |  열린 상태 유지
|      |  ... server waits ...                           |              |
|      |                                                 |              |
|      |  [New message arrives on server!]               |              |
|      |                                                 |              |
|      |  <---------------------- 200 OK (new message)   |              |
|      |                                                 |              |
|      |  GET /messages --------------------------->     |  Immediately |
|      |                                                 |  reconnect   |
|      |  ... server waits ...                           |  즉시 재연결 |
|      |                                                 |              |
|                                                                       |
+-----------------------------------------------------------------------+

JavaScript Example:
+-----------------------------------------------------------------------+
| async function longPoll() {                                           |
|   while (true) {                                                      |
|     try {                                                             |
|       const response = await fetch('/api/messages/subscribe', {       |
|         timeout: 30000  // 30 second timeout                          |
|       });                                                             |
|                                                                       |
|       if (response.ok) {                                              |
|         const data = await response.json();                           |
|         displayMessages(data.messages);                               |
|       }                                                               |
|     } catch (err) {                                                   |
|       // Timeout or error - reconnect                                 |
|       await new Promise(r => setTimeout(r, 1000));                    |
|     }                                                                 |
|   }                                                                   |
| }                                                                     |
+-----------------------------------------------------------------------+

Server Implementation (Node.js):
+-----------------------------------------------------------------------+
| app.get('/api/messages/subscribe', async (req, res) => {              |
|   const timeout = 30000;                                              |
|   const startTime = Date.now();                                       |
|                                                                       |
|   while (Date.now() - startTime < timeout) {                          |
|     const messages = await getNewMessages(req.query.lastId);          |
|                                                                       |
|     if (messages.length > 0) {                                        |
|       return res.json({ messages });                                  |
|     }                                                                 |
|                                                                       |
|     await sleep(100);  // Check every 100ms                           |
|   }                                                                   |
|                                                                       |
|   res.json({ messages: [] });  // Timeout, return empty               |
| });                                                                   |
+-----------------------------------------------------------------------+
```

### Short Polling vs Long Polling Comparison (비교)

```
+----------------------+------------------------+------------------------+
| Aspect               | Short Polling          | Long Polling           |
+----------------------+------------------------+------------------------+
| Latency              | Up to polling interval | Near real-time         |
| 지연 시간            | 폴링 간격까지          | 거의 실시간            |
+----------------------+------------------------+------------------------+
| Bandwidth            | Many empty responses   | Fewer responses        |
| 대역폭               | 많은 빈 응답           | 적은 응답              |
+----------------------+------------------------+------------------------+
| Server Connections   | Short-lived            | Long-lived             |
| 서버 연결            | 단기 연결              | 장기 연결              |
+----------------------+------------------------+------------------------+
| Implementation       | Very simple            | More complex           |
| 구현                 | 매우 간단              | 더 복잡                |
+----------------------+------------------------+------------------------+
| Server Resources     | CPU from requests      | Memory for connections |
| 서버 리소스          | 요청으로 인한 CPU      | 연결을 위한 메모리     |
+----------------------+------------------------+------------------------+
| Use Case             | Low-frequency updates  | Near real-time needed  |
| 용도                 | 저빈도 업데이트        | 거의 실시간 필요       |
+----------------------+------------------------+------------------------+
```

---

## 2. Server-Sent Events (SSE)

### What is SSE? (SSE란?)

Server-Sent Events is a **standard for server-to-client streaming** over HTTP. The server can push data to the client, but it's one-directional.

서버 전송 이벤트는 HTTP를 통한 **서버에서 클라이언트로의 스트리밍 표준**입니다. 서버가 클라이언트에 데이터를 푸시할 수 있지만 단방향입니다.

```
+-----------------------------------------------------------------------+
|                    SSE Overview                                        |
+-----------------------------------------------------------------------+
|                                                                       |
|   Client                                            Server            |
|      |                                                 |              |
|      |  GET /events (Accept: text/event-stream) -->    |              |
|      |                                                 |              |
|      |  <---- HTTP 200 OK                              |              |
|      |  <---- Content-Type: text/event-stream          |              |
|      |                                                 |              |
|      |  <---- data: {"message": "Hello"}               |  Event 1     |
|      |                                                 |              |
|      |  <---- data: {"message": "World"}               |  Event 2     |
|      |                                                 |              |
|      |  <---- data: {"price": 100.50}                  |  Event 3     |
|      |                                                 |              |
|      |  ... connection stays open ...                  |              |
|      |                                                 |              |
|                                                                       |
|   Key: Server pushes, client receives (one-way)                       |
|   핵심: 서버가 푸시, 클라이언트가 수신 (단방향)                       |
|                                                                       |
+-----------------------------------------------------------------------+
```

### SSE Protocol Format (SSE 프로토콜 형식)

```
+-----------------------------------------------------------------------+
|                    SSE Message Format                                  |
+-----------------------------------------------------------------------+
|                                                                       |
|   Basic message:                                                      |
|   +---------------------------------------------------------------+   |
|   | data: Hello, World!\n                                         |   |
|   | \n                                                            |   |
|   +---------------------------------------------------------------+   |
|                                                                       |
|   Multi-line data:                                                    |
|   +---------------------------------------------------------------+   |
|   | data: First line\n                                            |   |
|   | data: Second line\n                                           |   |
|   | \n                                                            |   |
|   +---------------------------------------------------------------+   |
|                                                                       |
|   With event type and ID:                                             |
|   +---------------------------------------------------------------+   |
|   | id: 12345\n                                                   |   |
|   | event: notification\n                                         |   |
|   | data: {"type": "like", "from": "user123"}\n                   |   |
|   | \n                                                            |   |
|   +---------------------------------------------------------------+   |
|                                                                       |
|   Retry instruction (reconnection delay):                             |
|   +---------------------------------------------------------------+   |
|   | retry: 5000\n                                                 |   |
|   | \n                                                            |   |
|   +---------------------------------------------------------------+   |
|                                                                       |
|   Fields:                                                             |
|   - data: The actual data (required)                                  |
|   - id: Event ID (for resuming after disconnect)                      |
|   - event: Event type (client can listen for specific types)          |
|   - retry: Reconnection time in ms                                    |
|                                                                       |
+-----------------------------------------------------------------------+
```

### SSE Client Implementation (SSE 클라이언트 구현)

```
+-----------------------------------------------------------------------+
|                    JavaScript SSE Client                               |
+-----------------------------------------------------------------------+
|                                                                       |
| // Create EventSource connection                                      |
| const eventSource = new EventSource('/api/events');                   |
|                                                                       |
| // Listen for all messages                                            |
| eventSource.onmessage = (event) => {                                  |
|   console.log('Received:', event.data);                               |
|   const data = JSON.parse(event.data);                                |
|   updateUI(data);                                                     |
| };                                                                    |
|                                                                       |
| // Listen for specific event types                                    |
| eventSource.addEventListener('notification', (event) => {             |
|   showNotification(JSON.parse(event.data));                           |
| });                                                                   |
|                                                                       |
| eventSource.addEventListener('stockUpdate', (event) => {              |
|   updateStockPrice(JSON.parse(event.data));                           |
| });                                                                   |
|                                                                       |
| // Handle connection errors                                           |
| eventSource.onerror = (error) => {                                    |
|   console.error('SSE error:', error);                                 |
|   // Browser auto-reconnects!                                         |
|   // 브라우저가 자동으로 재연결!                                      |
| };                                                                    |
|                                                                       |
| // Close connection when done                                         |
| // eventSource.close();                                               |
|                                                                       |
+-----------------------------------------------------------------------+
```

### SSE Server Implementation (SSE 서버 구현)

```
+-----------------------------------------------------------------------+
|                    Node.js SSE Server                                  |
+-----------------------------------------------------------------------+
|                                                                       |
| app.get('/api/events', (req, res) => {                                |
|   // Set SSE headers                                                  |
|   res.setHeader('Content-Type', 'text/event-stream');                 |
|   res.setHeader('Cache-Control', 'no-cache');                         |
|   res.setHeader('Connection', 'keep-alive');                          |
|                                                                       |
|   // Send initial connection message                                  |
|   res.write('data: {"connected": true}\n\n');                         |
|                                                                       |
|   // Send periodic updates                                            |
|   const intervalId = setInterval(() => {                              |
|     const data = { time: new Date().toISOString() };                  |
|     res.write(`data: ${JSON.stringify(data)}\n\n`);                   |
|   }, 1000);                                                           |
|                                                                       |
|   // Clean up on disconnect                                           |
|   req.on('close', () => {                                             |
|     clearInterval(intervalId);                                        |
|     console.log('Client disconnected');                               |
|   });                                                                 |
| });                                                                   |
|                                                                       |
+-----------------------------------------------------------------------+
```

### SSE with Last-Event-ID (Last-Event-ID를 사용한 SSE)

```
+-----------------------------------------------------------------------+
|                    SSE Resumption                                      |
+-----------------------------------------------------------------------+
|                                                                       |
|   When connection drops, browser reconnects with last ID:             |
|   연결이 끊어지면 브라우저가 마지막 ID로 재연결:                      |
|                                                                       |
|   Client                                            Server            |
|      |                                                 |              |
|      |  GET /events ---------------------------->      |              |
|      |  <---- id: 1, data: "Message A"                 |              |
|      |  <---- id: 2, data: "Message B"                 |              |
|      |  <---- id: 3, data: "Message C"                 |              |
|      |                                                 |              |
|      |  [Connection lost!]                             |              |
|      |                                                 |              |
|      |  GET /events ---------------------------->      |              |
|      |  Last-Event-ID: 3        ^                      |              |
|      |                          |                      |              |
|      |                  Auto-reconnect with last ID    |              |
|      |                  마지막 ID로 자동 재연결        |              |
|      |                                                 |              |
|      |  <---- id: 4, data: "Message D"                 |  Resume from |
|      |  <---- id: 5, data: "Message E"                 |  where left  |
|      |                                                 |  off         |
|                                                                       |
+-----------------------------------------------------------------------+
```

### SSE Pros and Cons (SSE 장단점)

```
+---------------------------+-------------------------------------------+
| Advantages (장점)         | Disadvantages (단점)                      |
+---------------------------+-------------------------------------------+
| Simple API (EventSource)  | One-way only (server to client)           |
| 간단한 API                | 단방향만 (서버에서 클라이언트)            |
+---------------------------+-------------------------------------------+
| Auto-reconnection         | Text-based (no binary)                    |
| 자동 재연결               | 텍스트 기반 (바이너리 불가)               |
+---------------------------+-------------------------------------------+
| Built-in event IDs        | Limited browser connections (~6)          |
| 내장 이벤트 ID            | 제한된 브라우저 연결 (~6개)               |
+---------------------------+-------------------------------------------+
| Uses standard HTTP        | No IE support (polyfill needed)           |
| 표준 HTTP 사용            | IE 미지원 (폴리필 필요)                   |
+---------------------------+-------------------------------------------+
| Works through proxies     | Cannot send custom headers                |
| 프록시를 통해 작동        | 커스텀 헤더 전송 불가                     |
+---------------------------+-------------------------------------------+
```

---

## 3. WebSocket Concept and Handshake

### What is WebSocket? (웹소켓이란?)

WebSocket provides **full-duplex, bidirectional communication** over a single TCP connection. Both client and server can send messages at any time.

웹소켓은 단일 TCP 연결을 통해 **전이중, 양방향 통신**을 제공합니다. 클라이언트와 서버 모두 언제든지 메시지를 보낼 수 있습니다.

```
+-----------------------------------------------------------------------+
|                    WebSocket Overview                                  |
+-----------------------------------------------------------------------+
|                                                                       |
|   Client                                            Server            |
|      |                                                 |              |
|      |  HTTP Upgrade Request ------------------>       |              |
|      |  <----------------- 101 Switching Protocols     |              |
|      |                                                 |              |
|      |  ============= WebSocket Connection =========== |              |
|      |                                                 |              |
|      |  Binary/Text Frame ---------------------->      |              |
|      |  <---------------------- Binary/Text Frame      |              |
|      |  Binary/Text Frame ---------------------->      |              |
|      |  <---------------------- Binary/Text Frame      |              |
|      |  <---------------------- Binary/Text Frame      |              |
|      |  Binary/Text Frame ---------------------->      |              |
|      |                                                 |              |
|      |  ============================================== |              |
|      |                                                 |              |
|                                                                       |
|   Key: Full-duplex - both sides can send anytime                      |
|   핵심: 전이중 - 양쪽 모두 언제든지 전송 가능                         |
|                                                                       |
+-----------------------------------------------------------------------+
```

### WebSocket Handshake (웹소켓 핸드셰이크)

```
+-----------------------------------------------------------------------+
|                    WebSocket Handshake Process                         |
+-----------------------------------------------------------------------+
|                                                                       |
|   Step 1: Client sends HTTP Upgrade request                           |
|   +---------------------------------------------------------------+   |
|   | GET /chat HTTP/1.1                                            |   |
|   | Host: example.com                                             |   |
|   | Upgrade: websocket                                            |   |
|   | Connection: Upgrade                                           |   |
|   | Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==                    |   |
|   | Sec-WebSocket-Version: 13                                     |   |
|   | Sec-WebSocket-Protocol: chat, superchat                       |   |
|   +---------------------------------------------------------------+   |
|                                                                       |
|   Key headers:                                                        |
|   - Upgrade: websocket - Request protocol upgrade                     |
|   - Sec-WebSocket-Key - Random base64 key for handshake               |
|   - Sec-WebSocket-Version - Must be 13                                |
|   - Sec-WebSocket-Protocol - Optional subprotocol negotiation         |
|                                                                       |
|   Step 2: Server responds with 101 Switching Protocols                |
|   +---------------------------------------------------------------+   |
|   | HTTP/1.1 101 Switching Protocols                              |   |
|   | Upgrade: websocket                                            |   |
|   | Connection: Upgrade                                           |   |
|   | Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=             |   |
|   | Sec-WebSocket-Protocol: chat                                  |   |
|   +---------------------------------------------------------------+   |
|                                                                       |
|   Sec-WebSocket-Accept calculation:                                   |
|   = Base64(SHA1(Key + "258EAFA5-E914-47DA-95CA-C5AB0DC85B11"))         |
|                                                                       |
|   Step 3: Connection upgraded, WebSocket frames begin                 |
|                                                                       |
+-----------------------------------------------------------------------+
```

### WebSocket Frame Format (웹소켓 프레임 형식)

```
+-----------------------------------------------------------------------+
|                    WebSocket Frame Structure                           |
+-----------------------------------------------------------------------+
|                                                                       |
|   0                   1                   2                   3       |
|   0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1     |
|   +-+-+-+-+-------+-+-------------+-------------------------------+   |
|   |F|R|R|R| opcode|M| Payload len |    Extended payload length    |   |
|   |I|S|S|S|  (4)  |A|     (7)     |             (16/64)           |   |
|   |N|V|V|V|       |S|             |   (if payload len==126/127)   |   |
|   | |1|2|3|       |K|             |                               |   |
|   +-+-+-+-+-------+-+-------------+-------------------------------+   |
|   |     Extended payload length continued, if payload len == 127  |   |
|   +-------------------------------+-------------------------------+   |
|   |                               |Masking-key, if MASK set to 1  |   |
|   +-------------------------------+-------------------------------+   |
|   | Masking-key (continued)       |          Payload Data         |   |
|   +--------------------------------- - - - - - - - - - - - - - - -+   |
|   :                     Payload Data continued ...                :   |
|   + - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - +   |
|   |                     Payload Data continued ...                |   |
|   +---------------------------------------------------------------+   |
|                                                                       |
+-----------------------------------------------------------------------+

Opcodes:
+------------------+------------------------------------------------+
| Opcode           | Description                                    |
+------------------+------------------------------------------------+
| 0x0              | Continuation frame (연속 프레임)               |
| 0x1              | Text frame (UTF-8) (텍스트 프레임)             |
| 0x2              | Binary frame (바이너리 프레임)                 |
| 0x8              | Close frame (종료 프레임)                      |
| 0x9              | Ping frame (핑)                                |
| 0xA              | Pong frame (퐁)                                |
+------------------+------------------------------------------------+
```

### WebSocket Client Implementation (웹소켓 클라이언트 구현)

```
+-----------------------------------------------------------------------+
|                    JavaScript WebSocket Client                         |
+-----------------------------------------------------------------------+
|                                                                       |
| // Create WebSocket connection                                        |
| const ws = new WebSocket('wss://example.com/chat');                    |
|                                                                       |
| // Connection opened                                                  |
| ws.onopen = () => {                                                   |
|   console.log('Connected!');                                          |
|   ws.send(JSON.stringify({ type: 'join', room: 'general' }));         |
| };                                                                    |
|                                                                       |
| // Receive message                                                    |
| ws.onmessage = (event) => {                                           |
|   const data = JSON.parse(event.data);                                |
|   console.log('Received:', data);                                     |
|                                                                       |
|   if (data.type === 'message') {                                      |
|     displayMessage(data);                                             |
|   }                                                                   |
| };                                                                    |
|                                                                       |
| // Handle errors                                                      |
| ws.onerror = (error) => {                                             |
|   console.error('WebSocket error:', error);                           |
| };                                                                    |
|                                                                       |
| // Connection closed                                                  |
| ws.onclose = (event) => {                                             |
|   console.log('Disconnected:', event.code, event.reason);             |
|   // Implement reconnection logic                                     |
|   setTimeout(() => reconnect(), 3000);                                |
| };                                                                    |
|                                                                       |
| // Send message                                                       |
| function sendMessage(text) {                                          |
|   if (ws.readyState === WebSocket.OPEN) {                             |
|     ws.send(JSON.stringify({ type: 'message', text }));               |
|   }                                                                   |
| }                                                                     |
|                                                                       |
+-----------------------------------------------------------------------+
```

### WebSocket Server Implementation (웹소켓 서버 구현)

```
+-----------------------------------------------------------------------+
|                    Node.js WebSocket Server (ws library)               |
+-----------------------------------------------------------------------+
|                                                                       |
| const WebSocket = require('ws');                                      |
| const wss = new WebSocket.Server({ port: 8080 });                     |
|                                                                       |
| // Store connected clients                                            |
| const clients = new Set();                                            |
|                                                                       |
| wss.on('connection', (ws, req) => {                                   |
|   console.log('New client connected');                                |
|   clients.add(ws);                                                    |
|                                                                       |
|   // Handle incoming messages                                         |
|   ws.on('message', (message) => {                                     |
|     const data = JSON.parse(message);                                 |
|     console.log('Received:', data);                                   |
|                                                                       |
|     if (data.type === 'message') {                                    |
|       // Broadcast to all clients                                     |
|       broadcast({                                                     |
|         type: 'message',                                              |
|         from: data.from,                                              |
|         text: data.text,                                              |
|         timestamp: Date.now()                                         |
|       });                                                             |
|     }                                                                 |
|   });                                                                 |
|                                                                       |
|   // Handle disconnect                                                |
|   ws.on('close', () => {                                              |
|     clients.delete(ws);                                               |
|     console.log('Client disconnected');                               |
|   });                                                                 |
|                                                                       |
|   // Send welcome message                                             |
|   ws.send(JSON.stringify({ type: 'welcome', message: 'Hello!' }));    |
| });                                                                   |
|                                                                       |
| // Broadcast to all connected clients                                 |
| function broadcast(data) {                                            |
|   const message = JSON.stringify(data);                               |
|   clients.forEach(client => {                                         |
|     if (client.readyState === WebSocket.OPEN) {                       |
|       client.send(message);                                           |
|     }                                                                 |
|   });                                                                 |
| }                                                                     |
|                                                                       |
+-----------------------------------------------------------------------+
```

### WebSocket Heartbeat (웹소켓 하트비트)

```
+-----------------------------------------------------------------------+
|                    WebSocket Keep-Alive (Ping/Pong)                    |
+-----------------------------------------------------------------------+
|                                                                       |
|   Problem: Connections may silently drop (NAT timeout, etc.)          |
|   문제: 연결이 조용히 끊어질 수 있음 (NAT 타임아웃 등)                |
|                                                                       |
|   Solution: Periodic ping/pong frames                                 |
|   해결: 주기적인 핑/퐁 프레임                                         |
|                                                                       |
|   Client                                            Server            |
|      |                                                 |              |
|      |  <-------------------------- Ping               |              |
|      |  Pong -------------------------->               |  t=0s        |
|      |                                                 |              |
|      |  ... 30 seconds ...                             |              |
|      |                                                 |              |
|      |  <-------------------------- Ping               |              |
|      |  Pong -------------------------->               |  t=30s       |
|      |                                                 |              |
|      |  ... 30 seconds ...                             |              |
|      |                                                 |              |
|      |  <-------------------------- Ping               |              |
|      |  [No pong received - connection dead!]          |  t=60s       |
|      |  [Close and reconnect]                          |              |
|                                                                       |
+-----------------------------------------------------------------------+

Server-side heartbeat:
+-----------------------------------------------------------------------+
| const HEARTBEAT_INTERVAL = 30000;                                     |
|                                                                       |
| wss.on('connection', (ws) => {                                        |
|   ws.isAlive = true;                                                  |
|                                                                       |
|   ws.on('pong', () => {                                               |
|     ws.isAlive = true;                                                |
|   });                                                                 |
| });                                                                   |
|                                                                       |
| // Check all connections periodically                                 |
| setInterval(() => {                                                   |
|   wss.clients.forEach((ws) => {                                       |
|     if (!ws.isAlive) {                                                |
|       return ws.terminate();  // Dead connection                      |
|     }                                                                 |
|     ws.isAlive = false;                                               |
|     ws.ping();                                                        |
|   });                                                                 |
| }, HEARTBEAT_INTERVAL);                                               |
+-----------------------------------------------------------------------+
```

---

## 4. WebSocket vs HTTP

### Communication Pattern Comparison (통신 패턴 비교)

```
+-----------------------------------------------------------------------+
|                    HTTP vs WebSocket Communication                     |
+-----------------------------------------------------------------------+
|                                                                       |
|   HTTP (Request-Response):                                            |
|   +---------------------------------------------------------------+   |
|   |   Client                            Server                    |   |
|   |      |                                 |                      |   |
|   |      | ---- Request 1 ------>          |                      |   |
|   |      | <----- Response 1 ----          |                      |   |
|   |      |                                 |                      |   |
|   |      | ---- Request 2 ------>          |                      |   |
|   |      | <----- Response 2 ----          |                      |   |
|   |      |                                 |                      |   |
|   |   Every communication needs a request                         |   |
|   |   모든 통신에 요청 필요                                       |   |
|   +---------------------------------------------------------------+   |
|                                                                       |
|   WebSocket (Full-Duplex):                                            |
|   +---------------------------------------------------------------+   |
|   |   Client                            Server                    |   |
|   |      |                                 |                      |   |
|   |      | ---- Message ------>            |                      |   |
|   |      | <----- Message -----            |                      |   |
|   |      | <----- Message -----            |                      |   |
|   |      | <----- Message -----            |                      |   |
|   |      | ---- Message ------>            |                      |   |
|   |      | <----- Message -----            |                      |   |
|   |      |                                 |                      |   |
|   |   Either side can send anytime                                |   |
|   |   어느 쪽이든 언제든지 전송 가능                              |   |
|   +---------------------------------------------------------------+   |
|                                                                       |
+-----------------------------------------------------------------------+
```

### Detailed Comparison (상세 비교)

```
+------------------------+------------------------+------------------------+
| Feature                | HTTP                   | WebSocket              |
+------------------------+------------------------+------------------------+
| Protocol               | Request-Response       | Full-Duplex            |
| 프로토콜               | 요청-응답              | 전이중                 |
+------------------------+------------------------+------------------------+
| Connection             | New or Keep-Alive      | Persistent             |
| 연결                   | 새로운 또는 Keep-Alive | 지속적                 |
+------------------------+------------------------+------------------------+
| Direction              | Client initiates       | Bidirectional          |
| 방향                   | 클라이언트가 시작      | 양방향                 |
+------------------------+------------------------+------------------------+
| Overhead per message   | HTTP headers (~700B)   | 2-14 bytes             |
| 메시지당 오버헤드      |                        |                        |
+------------------------+------------------------+------------------------+
| Server Push            | Not natively           | Yes                    |
| 서버 푸시              | 기본적으로 불가        | 가능                   |
+------------------------+------------------------+------------------------+
| Binary Data            | Base64 encoded         | Native support         |
| 바이너리 데이터        | Base64 인코딩          | 네이티브 지원          |
+------------------------+------------------------+------------------------+
| State                  | Stateless              | Stateful               |
| 상태                   | 무상태                 | 상태 유지              |
+------------------------+------------------------+------------------------+
| Caching                | Built-in support       | Not applicable         |
| 캐싱                   | 내장 지원              | 해당 없음              |
+------------------------+------------------------+------------------------+
| Load Balancing         | Easy                   | Requires sticky        |
| 로드 밸런싱            | 쉬움                   | 고정 세션 필요         |
+------------------------+------------------------+------------------------+
```

### When to Use What (언제 무엇을 사용할까)

```
+-----------------------------------------------------------------------+
|                    Use Case Recommendations                            |
+-----------------------------------------------------------------------+
|                                                                       |
|   Use HTTP when:                                                      |
|   HTTP를 사용할 때:                                                   |
|   +---------------------------------------------------------------+   |
|   | - Traditional request-response (API calls)                    |   |
|   | - File uploads/downloads                                      |   |
|   | - One-time data fetching                                      |   |
|   | - Need caching                                                |   |
|   | - SEO important (crawlable)                                   |   |
|   +---------------------------------------------------------------+   |
|                                                                       |
|   Use SSE when:                                                       |
|   SSE를 사용할 때:                                                    |
|   +---------------------------------------------------------------+   |
|   | - Server-to-client streaming only                             |   |
|   | - News feeds, notifications                                   |   |
|   | - Live updates (stock prices, sports scores)                  |   |
|   | - Need auto-reconnection                                      |   |
|   +---------------------------------------------------------------+   |
|                                                                       |
|   Use WebSocket when:                                                 |
|   WebSocket을 사용할 때:                                              |
|   +---------------------------------------------------------------+   |
|   | - Bidirectional real-time communication                       |   |
|   | - Chat applications                                           |   |
|   | - Multiplayer games                                           |   |
|   | - Collaborative editing                                       |   |
|   | - Low latency required                                        |   |
|   | - High-frequency messages                                     |   |
|   +---------------------------------------------------------------+   |
|                                                                       |
+-----------------------------------------------------------------------+
```

### Comparison Summary Table (비교 요약표)

```
+------------------+------------+------------+------------+------------+
| Feature          | Short Poll | Long Poll  | SSE        | WebSocket  |
+------------------+------------+------------+------------+------------+
| Direction        | Client->   | Client->   | Server->   | Both       |
| 방향             | Server     | Server     | Client     | 양방향     |
+------------------+------------+------------+------------+------------+
| Latency          | High       | Medium     | Low        | Very Low   |
| 지연 시간        | 높음       | 중간       | 낮음       | 매우 낮음  |
+------------------+------------+------------+------------+------------+
| Efficiency       | Low        | Medium     | Good       | Best       |
| 효율성           | 낮음       | 중간       | 좋음       | 최고       |
+------------------+------------+------------+------------+------------+
| Complexity       | Simple     | Medium     | Simple     | Complex    |
| 복잡도           | 간단       | 중간       | 간단       | 복잡       |
+------------------+------------+------------+------------+------------+
| Binary Support   | No         | No         | No         | Yes        |
| 바이너리 지원    | 없음       | 없음       | 없음       | 있음       |
+------------------+------------+------------+------------+------------+
| Auto-reconnect   | Manual     | Manual     | Built-in   | Manual     |
| 자동 재연결      | 수동       | 수동       | 내장       | 수동       |
+------------------+------------+------------+------------+------------+
```

---

## 5. Game Networking Basics

### Game Networking Challenges (게임 네트워킹 과제)

```
+-----------------------------------------------------------------------+
|                    Game Networking Requirements                        |
+-----------------------------------------------------------------------+
|                                                                       |
|   Challenges unique to games:                                         |
|   게임만의 고유한 과제:                                               |
|                                                                       |
|   1. Low Latency (저지연)                                             |
|      - Players expect instant response                                |
|      - 플레이어는 즉각적인 응답 기대                                  |
|      - 50ms feels laggy, 100ms+ is unplayable for fast games          |
|      - 50ms는 느리게 느껴지고, 100ms+는 빠른 게임에서 플레이 불가     |
|                                                                       |
|   2. High Frequency Updates (높은 빈도 업데이트)                      |
|      - Positions updated 20-60 times per second                       |
|      - 위치가 초당 20-60회 업데이트                                   |
|                                                                       |
|   3. State Synchronization (상태 동기화)                              |
|      - All players must see consistent world                          |
|      - 모든 플레이어가 일관된 월드를 봐야 함                          |
|                                                                       |
|   4. Cheating Prevention (치팅 방지)                                  |
|      - Never trust the client                                         |
|      - 클라이언트를 절대 신뢰하지 않음                                |
|                                                                       |
+-----------------------------------------------------------------------+
```

### Network Architectures (네트워크 아키텍처)

```
+-----------------------------------------------------------------------+
|                    Peer-to-Peer (P2P)                                  |
+-----------------------------------------------------------------------+
|                                                                       |
|        [Player A]<-------->[Player B]                                 |
|            ^                    ^                                     |
|            |                    |                                     |
|            v                    v                                     |
|        [Player C]<-------->[Player D]                                 |
|                                                                       |
|   Pros:                                                               |
|   - No server costs                                                   |
|   - Low latency between nearby players                                |
|                                                                       |
|   Cons:                                                               |
|   - Hard to prevent cheating                                          |
|   - Complex NAT traversal                                             |
|   - Scales poorly (N players = N*(N-1)/2 connections)                 |
|                                                                       |
|   Use case: Small multiplayer games, LAN games                        |
|   용도: 소규모 멀티플레이어 게임, LAN 게임                            |
|                                                                       |
+-----------------------------------------------------------------------+

+-----------------------------------------------------------------------+
|                    Client-Server                                       |
+-----------------------------------------------------------------------+
|                                                                       |
|             [Player A]                                                |
|                  \                                                    |
|                   \                                                   |
|        [Player B]-->[Server]<--[Player C]                             |
|                   /                                                   |
|                  /                                                    |
|             [Player D]                                                |
|                                                                       |
|   Pros:                                                               |
|   - Authoritative server prevents cheating                            |
|   - Simpler connection management                                     |
|   - Scales better                                                     |
|                                                                       |
|   Cons:                                                               |
|   - Server costs                                                      |
|   - Single point of failure                                           |
|   - Higher latency (client -> server -> client)                       |
|                                                                       |
|   Use case: Most online games                                         |
|   용도: 대부분의 온라인 게임                                          |
|                                                                       |
+-----------------------------------------------------------------------+
```

### Client-Side Prediction (클라이언트 측 예측)

```
+-----------------------------------------------------------------------+
|                    Client-Side Prediction                              |
+-----------------------------------------------------------------------+
|                                                                       |
|   Problem: Waiting for server response feels laggy                    |
|   문제: 서버 응답을 기다리면 느리게 느껴짐                            |
|                                                                       |
|   Without prediction:                                                 |
|   +---------------------------------------------------------------+   |
|   | Client presses "move right"                                   |   |
|   |    |                                                          |   |
|   |    | --- "move right" ---> Server                             |   |
|   |    |                        |                                 |   |
|   |    |                        | Process                         |   |
|   |    |                        v                                 |   |
|   |    | <--- "position X" --- Server                             |   |
|   |    |                                                          |   |
|   |    v (100ms later!)                                           |   |
|   | Character finally moves                                       |   |
|   +---------------------------------------------------------------+   |
|                                                                       |
|   With prediction:                                                    |
|   +---------------------------------------------------------------+   |
|   | Client presses "move right"                                   |   |
|   |    |                                                          |   |
|   |    | [Immediately move character locally]                     |   |
|   |    | [로컬에서 즉시 캐릭터 이동]                              |   |
|   |    |                                                          |   |
|   |    | --- "move right" ---> Server                             |   |
|   |    |                        |                                 |   |
|   |    | <--- "position X" ---  | (confirm or correct)            |   |
|   |    |                                                          |   |
|   | Character moved instantly!                                    |   |
|   | 캐릭터가 즉시 이동!                                           |   |
|   +---------------------------------------------------------------+   |
|                                                                       |
|   If server disagrees, client "snaps" to correct position             |
|   서버가 동의하지 않으면 클라이언트가 올바른 위치로 "스냅"            |
|                                                                       |
+-----------------------------------------------------------------------+
```

### Server Reconciliation (서버 조정)

```
+-----------------------------------------------------------------------+
|                    Server Reconciliation                               |
+-----------------------------------------------------------------------+
|                                                                       |
|   Problem: Client predicted position may differ from server           |
|   문제: 클라이언트 예측 위치가 서버와 다를 수 있음                    |
|                                                                       |
|   Client timeline:                                                    |
|   +---------------------------------------------------------------+   |
|   | Input 1 | Input 2 | Input 3 | Input 4 | Input 5               |   |
|   | (sent)  | (sent)  | (sent)  | (pending)| (pending)            |   |
|   +---------------------------------------------------------------+   |
|                                                                       |
|   Server sends: "After Input 2, position is X"                        |
|   서버 전송: "Input 2 이후 위치는 X"                                  |
|                                                                       |
|   Client reconciles:                                                  |
|   +---------------------------------------------------------------+   |
|   | 1. Set position to X                                          |   |
|   | 2. Re-apply Input 3, 4, 5 from X                              |   |
|   | 3. Continue with predicted position                           |   |
|   +---------------------------------------------------------------+   |
|                                                                       |
|   This handles packet loss and latency gracefully                     |
|   이것은 패킷 손실과 지연을 우아하게 처리함                           |
|                                                                       |
+-----------------------------------------------------------------------+
```

### Interpolation and Extrapolation (보간과 외삽)

```
+-----------------------------------------------------------------------+
|                    Entity Interpolation                                |
+-----------------------------------------------------------------------+
|                                                                       |
|   Problem: Other players' positions come in discrete updates          |
|   문제: 다른 플레이어 위치가 이산적인 업데이트로 옴                   |
|                                                                       |
|   Without interpolation:                                              |
|   +---------------------------------------------------------------+   |
|   |  Enemy position: [A].....[B].....[C].....[D]                  |   |
|   |                  Jumps between positions (teleporting)         |   |
|   |                  위치 사이를 점프 (텔레포팅)                   |   |
|   +---------------------------------------------------------------+   |
|                                                                       |
|   With interpolation:                                                 |
|   +---------------------------------------------------------------+   |
|   |  Enemy position: [A]--x--x--[B]--x--x--[C]--x--x--[D]         |   |
|   |                  Smooth movement between known positions       |   |
|   |                  알려진 위치 사이의 부드러운 이동              |   |
|   |                                                               |   |
|   |  Client displays position slightly behind actual (e.g., 100ms)|   |
|   |  클라이언트가 실제보다 약간 뒤의 위치 표시 (예: 100ms)        |   |
|   +---------------------------------------------------------------+   |
|                                                                       |
|   Extrapolation (when data is late):                                  |
|   +---------------------------------------------------------------+   |
|   |  Known: [A]--->[B]      Predicted: [B]--->[C?]                |   |
|   |                                                               |   |
|   |  If packet is late, predict where enemy WOULD be              |   |
|   |  패킷이 늦으면 적이 어디에 있을지 예측                        |   |
|   |                                                               |   |
|   |  Risk: If enemy changes direction, prediction is wrong        |   |
|   |  위험: 적이 방향을 바꾸면 예측이 틀림                         |   |
|   +---------------------------------------------------------------+   |
|                                                                       |
+-----------------------------------------------------------------------+
```

### TCP vs UDP for Games (게임에서 TCP vs UDP)

```
+-----------------------------------------------------------------------+
|                    TCP vs UDP for Games                                |
+-----------------------------------------------------------------------+
|                                                                       |
|   TCP:                                                                |
|   +---------------------------------------------------------------+   |
|   | - Guaranteed delivery (보장된 전달)                           |   |
|   | - Ordered packets (순서가 보장된 패킷)                        |   |
|   | - Congestion control (혼잡 제어)                              |   |
|   |                                                               |   |
|   | Problem: Packet loss causes ALL packets to wait!              |   |
|   | 문제: 패킷 손실 시 모든 패킷이 대기!                          |   |
|   |                                                               |   |
|   | [Pkt 1][Pkt 2][LOST][Pkt 4][Pkt 5] -> Wait for Pkt 3 retx     |   |
|   |                                                               |   |
|   | Use for: Turn-based games, chat, login/auth                   |   |
|   | 용도: 턴제 게임, 채팅, 로그인/인증                            |   |
|   +---------------------------------------------------------------+   |
|                                                                       |
|   UDP:                                                                |
|   +---------------------------------------------------------------+   |
|   | - No guaranteed delivery (전달 보장 없음)                     |   |
|   | - No ordering (순서 보장 없음)                                |   |
|   | - No congestion control (혼잡 제어 없음)                      |   |
|   |                                                               |   |
|   | Benefit: Packet loss doesn't block other packets              |   |
|   | 장점: 패킷 손실이 다른 패킷을 막지 않음                       |   |
|   |                                                               |   |
|   | [Pkt 1][Pkt 2][LOST][Pkt 4][Pkt 5] -> Continue with 4, 5      |   |
|   |                                                               |   |
|   | Use for: Real-time action games, FPS, racing                  |   |
|   | 용도: 실시간 액션 게임, FPS, 레이싱                           |   |
|   +---------------------------------------------------------------+   |
|                                                                       |
+-----------------------------------------------------------------------+

Game-Specific Protocol on UDP:
+-----------------------------------------------------------------------+
|                                                                       |
|   Most games implement their own reliability on top of UDP:           |
|   대부분의 게임이 UDP 위에 자체 신뢰성 구현:                          |
|                                                                       |
|   - Reliable ordered (for important events like kills)                |
|   - Reliable unordered (for events where order doesn't matter)        |
|   - Unreliable (for position updates - old data is useless)           |
|                                                                       |
+-----------------------------------------------------------------------+
```

---

## 6. Summary (요약)

### Key Concepts (핵심 개념)

```
+-----------------------------------------------------------------------+
|                     Chapter 17 Summary                                 |
+-----------------------------------------------------------------------+
|                                                                       |
|  1. Polling (폴링)                                                    |
|     - Short polling: Repeated requests at intervals                   |
|     - Long polling: Server holds request until data available         |
|     - Simple but inefficient for real-time                            |
|                                                                       |
|  2. Server-Sent Events (SSE)                                          |
|     - Server-to-client streaming over HTTP                            |
|     - text/event-stream content type                                  |
|     - Auto-reconnection, event IDs                                    |
|     - One-way only (server to client)                                 |
|                                                                       |
|  3. WebSocket                                                         |
|     - Full-duplex bidirectional communication                         |
|     - HTTP upgrade handshake (101 Switching Protocols)                |
|     - Low overhead (2-14 bytes per frame)                             |
|     - Supports binary and text data                                   |
|                                                                       |
|  4. WebSocket vs HTTP                                                 |
|     - HTTP: Request-response, stateless, cacheable                    |
|     - WebSocket: Full-duplex, stateful, real-time                     |
|     - Choose based on communication pattern needs                     |
|                                                                       |
|  5. Game Networking                                                   |
|     - Client-side prediction for responsiveness                       |
|     - Server reconciliation for correctness                           |
|     - Interpolation for smooth visuals                                |
|     - UDP for low latency, TCP for reliability                        |
|                                                                       |
+-----------------------------------------------------------------------+
```

### Technology Selection Guide (기술 선택 가이드)

```
+-----------------------------------------------------------------------+
|                    When to Use What                                    |
+-----------------------------------------------------------------------+
|                                                                       |
|   Need                    | Recommendation                            |
|   ------------------------|-------------------------------------------|
|   One-time data fetch     | HTTP GET                                  |
|   Form submission         | HTTP POST                                 |
|   Server push (one-way)   | SSE                                       |
|   Chat application        | WebSocket                                 |
|   Live notifications      | SSE or WebSocket                          |
|   Collaborative editing   | WebSocket                                 |
|   Real-time game          | WebSocket (or UDP for action games)       |
|   Stock ticker            | SSE or WebSocket                          |
|   File upload progress    | SSE                                       |
|                                                                       |
+-----------------------------------------------------------------------+
```

### Quick Reference (빠른 참조)

```
+------------------+--------------------------------------------------+
| Technology       | Key Characteristics                              |
+------------------+--------------------------------------------------+
| Short Polling    | Simple, high latency, wastes bandwidth           |
| 짧은 폴링        | 간단, 높은 지연, 대역폭 낭비                     |
+------------------+--------------------------------------------------+
| Long Polling     | Lower latency than short, holds connections      |
| 롱 폴링          | 짧은 폴링보다 낮은 지연, 연결 유지               |
+------------------+--------------------------------------------------+
| SSE              | Server push, auto-reconnect, text only           |
|                  | 서버 푸시, 자동 재연결, 텍스트만                 |
+------------------+--------------------------------------------------+
| WebSocket        | Full-duplex, low overhead, binary support        |
|                  | 전이중, 낮은 오버헤드, 바이너리 지원             |
+------------------+--------------------------------------------------+
| Game UDP         | Lowest latency, no HOL blocking, unreliable      |
| 게임 UDP         | 최저 지연, HOL 블로킹 없음, 신뢰성 없음          |
+------------------+--------------------------------------------------+
```

---

*End of Chapter 17*
