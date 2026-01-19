# Chapter 05. Transport Layer Overview (전송 계층 개요)

> Understanding the transport layer's role in providing end-to-end communication services.

---

## Role of Transport Layer (전송 계층의 역할)

### Concept (개념)
The **Transport Layer (전송 계층)** is Layer 4 in the OSI model and provides end-to-end communication services between applications running on different hosts.

**Primary Functions:**
- **Process-to-Process Communication**: Delivers data to specific applications (not just hosts)
- **Segmentation and Reassembly**: Breaks data into segments, reassembles at destination
- **Connection Management**: Establishes and terminates connections (TCP)
- **Flow Control**: Prevents sender from overwhelming receiver
- **Error Control**: Detects and handles transmission errors
- **Congestion Control**: Manages network traffic to prevent congestion

**Transport Layer vs Network Layer:**
| Network Layer (IP) | Transport Layer (TCP/UDP) |
|-------------------|---------------------------|
| Host-to-Host delivery | Process-to-Process delivery |
| Logical addressing (IP) | Port addressing |
| Routing decisions | End-to-end reliability |
| Best-effort delivery | Reliable or unreliable delivery |

### Diagram / Example
```
Transport Layer Position in Protocol Stack:

[Application Layer]   HTTP, FTP, SMTP, DNS
        │
        ▼ (Messages)
┌─────────────────────────────────────────┐
│     [Transport Layer]  TCP, UDP         │  ← This layer
│     - Segments data                     │
│     - Adds port numbers                 │
│     - Provides reliability (TCP)        │
└─────────────────────────────────────────┘
        │
        ▼ (Segments)
[Network Layer]       IP
        │
        ▼ (Packets)
[Data Link Layer]     Ethernet


End-to-End Communication:

Host A                                          Host B
[App A] Port 5000                               [App B] Port 80
    │                                               ▲
    ▼                                               │
[Transport]──────────────────────────────────>[Transport]
    │                                               ▲
    ▼                                               │
[Network]───>[Router]───>[Router]───>[Router]──>[Network]
```

---

## Port Numbers (포트 번호)

### Concept (개념)
**Port numbers** are 16-bit integers (0-65535) that identify specific applications or services on a host. They enable multiple applications to share the same network connection.

**Port Number Ranges:**
| Range | Name | Description |
|-------|------|-------------|
| 0-1023 | Well-Known Ports (잘 알려진 포트) | Reserved for standard services |
| 1024-49151 | Registered Ports (등록된 포트) | Registered with IANA for specific apps |
| 49152-65535 | Dynamic/Private Ports (동적/사설 포트) | Temporary, client-side ports |

**Socket (소켓):**
A socket is the combination of an IP address and a port number that uniquely identifies a communication endpoint.
- Socket = IP Address + Port Number
- Example: 192.168.1.100:8080

### Diagram / Example
```
Port Number Identification:

Host A (192.168.1.10)              Host B (10.0.0.50)
┌─────────────────────┐            ┌─────────────────────┐
│ [Browser]    :52431 │            │ [Web Server]   :80  │
│ [Email]      :52432 │            │ [FTP Server]   :21  │
│ [Game]       :52433 │            │ [SSH Server]   :22  │
└─────────────────────┘            └─────────────────────┘

Connection Identification (5-tuple):
┌─────────────────────────────────────────────────────────┐
│ Source IP    : 192.168.1.10                             │
│ Source Port  : 52431                                    │
│ Dest IP      : 10.0.0.50                                │
│ Dest Port    : 80                                       │
│ Protocol     : TCP                                      │
└─────────────────────────────────────────────────────────┘


Socket Example:
Client Socket: 192.168.1.10:52431
Server Socket: 10.0.0.50:80

These two sockets define a unique TCP connection.
```

---

## Multiplexing and Demultiplexing (다중화와 역다중화)

### Concept (개념)
**Multiplexing (다중화)** and **Demultiplexing (역다중화)** are fundamental transport layer functions that enable multiple applications to share a single network connection.

**Multiplexing (at sender):**
- Gathering data from multiple application processes
- Adding transport headers (including port numbers)
- Passing segments to the network layer

**Demultiplexing (at receiver):**
- Receiving segments from the network layer
- Examining port numbers in headers
- Delivering data to the correct application process

### Diagram / Example
```
Multiplexing (Sender Side):

Application Layer:
[HTTP]     [FTP]      [SMTP]
 Port       Port       Port
  80         21         25
   │          │          │
   ▼          ▼          ▼
┌─────────────────────────────────────┐
│         Transport Layer             │
│  ┌─────┐  ┌─────┐  ┌─────┐         │
│  │Seg 1│  │Seg 2│  │Seg 3│         │
│  │:80  │  │:21  │  │:25  │         │
│  └─────┘  └─────┘  └─────┘         │
│              │                      │
│              ▼                      │
│    Multiplexed into single stream   │
└─────────────────────────────────────┘
              │
              ▼
[Network Layer - Single IP connection]


Demultiplexing (Receiver Side):

[Network Layer - Receives packets]
              │
              ▼
┌─────────────────────────────────────┐
│         Transport Layer             │
│              │                      │
│    Check destination port number    │
│              │                      │
│  ┌─────┐  ┌─────┐  ┌─────┐         │
│  │:80  │  │:21  │  │:25  │         │
│  └──┬──┘  └──┬──┘  └──┬──┘         │
└─────│────────│────────│─────────────┘
      │        │        │
      ▼        ▼        ▼
   [HTTP]   [FTP]   [SMTP]


UDP Demultiplexing:
- Uses only destination port
- All segments to same port go to same socket

TCP Demultiplexing:
- Uses 4-tuple: (src IP, src port, dst IP, dst port)
- Different source = different socket (even same dst port)
```

---

## Well-Known Ports (잘 알려진 포트)

### Concept (개념)
**Well-known ports (0-1023)** are reserved for standard network services. These ports are assigned by IANA (Internet Assigned Numbers Authority).

**Common Well-Known Ports:**

| Port | Protocol | Service | Description |
|------|----------|---------|-------------|
| 20 | TCP | FTP Data | FTP data transfer |
| 21 | TCP | FTP Control | FTP command/control |
| 22 | TCP | SSH | Secure Shell |
| 23 | TCP | Telnet | Unencrypted remote login |
| 25 | TCP | SMTP | Email sending |
| 53 | TCP/UDP | DNS | Domain name resolution |
| 67/68 | UDP | DHCP | Dynamic IP assignment |
| 80 | TCP | HTTP | Web traffic |
| 110 | TCP | POP3 | Email retrieval |
| 143 | TCP | IMAP | Email access |
| 443 | TCP | HTTPS | Secure web traffic |
| 465 | TCP | SMTPS | Secure email sending |
| 993 | TCP | IMAPS | Secure email access |
| 995 | TCP | POP3S | Secure email retrieval |
| 3306 | TCP | MySQL | MySQL database |
| 5432 | TCP | PostgreSQL | PostgreSQL database |
| 6379 | TCP | Redis | Redis database |
| 27017 | TCP | MongoDB | MongoDB database |

### Diagram / Example
```
Server Listening on Well-Known Ports:

Web Server (192.168.1.100)
┌────────────────────────────────────┐
│  Listening Ports:                  │
│                                    │
│  Port 80   ← HTTP requests         │
│  Port 443  ← HTTPS requests        │
│  Port 22   ← SSH connections       │
│                                    │
└────────────────────────────────────┘

Incoming Connections:
┌──────────────────────────────────────────────────────────┐
│ Client 1:  192.168.1.10:54321  →  192.168.1.100:80      │
│ Client 2:  192.168.1.11:54322  →  192.168.1.100:80      │
│ Client 3:  192.168.1.12:54323  →  192.168.1.100:443     │
│ Admin:     192.168.1.5:54324   →  192.168.1.100:22      │
└──────────────────────────────────────────────────────────┘


Common Commands to Check Ports:
# List listening ports
netstat -tlnp          # Linux
netstat -an | grep LISTEN   # macOS
netstat -an            # Windows

# Check if port is in use
lsof -i :80            # macOS/Linux
```

---

## TCP vs UDP Overview (TCP vs UDP 개요)

### Concept (개념)
The transport layer provides two main protocols: **TCP** and **UDP**. They offer different tradeoffs between reliability and performance.

| Feature | TCP | UDP |
|---------|-----|-----|
| Connection | Connection-oriented (연결 지향) | Connectionless (비연결) |
| Reliability | Guaranteed delivery | Best-effort delivery |
| Ordering | Maintains order | No ordering guarantee |
| Error Checking | Checksum + recovery | Checksum only |
| Flow Control | Yes | No |
| Congestion Control | Yes | No |
| Speed | Slower (overhead) | Faster (minimal overhead) |
| Header Size | 20-60 bytes | 8 bytes |
| Use Cases | Web, email, file transfer | Streaming, gaming, DNS |

### Diagram / Example
```
TCP vs UDP Characteristics:

TCP (Transmission Control Protocol):
┌─────────────────────────────────────────────────────────┐
│ "The Reliable Delivery Service"                         │
│                                                         │
│ [Sender] ═══ Connection Setup ════> [Receiver]          │
│          ═══ Data with ACKs ════>                       │
│          <══ Acknowledgments ═══                        │
│          ═══ Connection Close ═══>                      │
│                                                         │
│ Features: Reliable, Ordered, Error-checked              │
│ Overhead: High (connection setup, ACKs)                 │
└─────────────────────────────────────────────────────────┘

UDP (User Datagram Protocol):
┌─────────────────────────────────────────────────────────┐
│ "The Fast Delivery Service"                             │
│                                                         │
│ [Sender] ───> Datagram 1 ───> [Receiver]               │
│          ───> Datagram 2 ───> (maybe received)          │
│          ───> Datagram 3 ───> (maybe lost)              │
│                                                         │
│ Features: Fast, Simple, No connection setup             │
│ Overhead: Low (just send and forget)                    │
└─────────────────────────────────────────────────────────┘


When to Use Each:

TCP - Use when data must arrive correctly:
✓ Web browsing (HTTP/HTTPS)
✓ Email (SMTP, IMAP, POP3)
✓ File transfer (FTP, SFTP)
✓ Database connections
✓ SSH

UDP - Use when speed matters more than reliability:
✓ Live video streaming
✓ Online gaming
✓ VoIP calls
✓ DNS queries
✓ IoT sensors
```

---

## Transport Layer Addressing (전송 계층 주소 지정)

### Concept (개념)
The transport layer uses **port numbers** combined with **IP addresses** to provide complete addressing for network communication.

**Address Types:**
- **IP Address**: Identifies the host (network layer)
- **Port Number**: Identifies the application (transport layer)
- **Socket Address**: IP + Port combination

**Connection Identification:**
A TCP connection is uniquely identified by a 5-tuple:
1. Source IP Address
2. Source Port Number
3. Destination IP Address
4. Destination Port Number
5. Protocol (TCP or UDP)

### Diagram / Example
```
Complete Transport Layer Addressing:

Communication Endpoint (Socket):
┌─────────────────────────────────┐
│ IP Address:  192.168.1.100      │
│ Port Number: 443                │
│ Socket:      192.168.1.100:443  │
└─────────────────────────────────┘

Full Connection Example:
┌───────────────────────────────────────────────────────────┐
│                                                           │
│  Client                              Server               │
│  ┌─────────────────┐                ┌─────────────────┐  │
│  │ IP: 10.0.0.5    │                │ IP: 93.184.216.34│  │
│  │ Port: 51234     │◄─────────────►│ Port: 443        │  │
│  └─────────────────┘    HTTPS       └─────────────────┘  │
│                                                           │
│  5-tuple:                                                 │
│  (10.0.0.5, 51234, 93.184.216.34, 443, TCP)              │
│                                                           │
└───────────────────────────────────────────────────────────┘


Multiple Connections to Same Server:

Server: 93.184.216.34:443 (HTTPS)

Connection 1: (10.0.0.5, 51234, 93.184.216.34, 443, TCP)
Connection 2: (10.0.0.5, 51235, 93.184.216.34, 443, TCP)  ← Same client, diff port
Connection 3: (10.0.0.6, 51234, 93.184.216.34, 443, TCP)  ← Different client

Each connection is unique even though destination is same!
```

---

## Summary (요약)

| Concept | Key Points |
|---------|------------|
| Transport Layer Role | Process-to-process communication, segmentation, reliability |
| Port Numbers | 16-bit identifiers for applications (0-65535) |
| Multiplexing | Combining multiple streams into one |
| Demultiplexing | Separating streams based on port numbers |
| Well-Known Ports | 0-1023, reserved for standard services |
| TCP vs UDP | Reliable vs fast, connection-oriented vs connectionless |

---

## Key Terms (주요 용어)

- **Transport Layer (전송 계층)**: OSI 4계층, 종단간 통신 담당
- **Port (포트)**: 애플리케이션 식별 번호
- **Socket (소켓)**: IP 주소 + 포트 번호 조합
- **Multiplexing (다중화)**: 여러 데이터 스트림을 하나로 결합
- **Demultiplexing (역다중화)**: 하나의 스트림을 여러 애플리케이션으로 분리
- **Segment (세그먼트)**: 전송 계층의 데이터 단위
- **Well-Known Port (잘 알려진 포트)**: 표준 서비스용 예약 포트 (0-1023)
- **Ephemeral Port (임시 포트)**: 클라이언트 측 동적 포트
