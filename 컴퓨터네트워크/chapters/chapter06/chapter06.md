# Chapter 06. UDP (User Datagram Protocol)

> Understanding the lightweight, connectionless transport protocol for fast data transmission.

---

## UDP Characteristics (UDP 특징)

### Concept (개념)
**UDP (User Datagram Protocol)** is a simple, connectionless transport layer protocol that provides minimal services beyond IP. It offers fast, but unreliable, data transmission.

**Key Characteristics:**
- **Connectionless (비연결형)**: No connection setup required
- **Unreliable (비신뢰성)**: No guarantee of delivery, order, or duplicate protection
- **No Congestion Control**: Sends at any rate desired
- **Lightweight**: Minimal header overhead (8 bytes)
- **Message-Oriented**: Preserves message boundaries
- **Stateless**: No connection state to maintain

**What UDP Does NOT Provide:**
- ✗ Guaranteed delivery
- ✗ Order preservation
- ✗ Duplicate detection
- ✗ Flow control
- ✗ Congestion control
- ✗ Connection management

### Diagram / Example
```
UDP Communication Model:

[Sender]                                    [Receiver]
    │                                           │
    │ ─── Datagram 1 ─────────────────────────> │  ✓ Received
    │ ─── Datagram 2 ──────────X (lost)         │  ✗ Lost
    │ ─── Datagram 3 ─────────────────────────> │  ✓ Received
    │ ─── Datagram 4 ─────────────────────────> │  ✓ Received
    │                                           │
    │  (No acknowledgments, no retransmission)  │


UDP vs TCP Analogy:

TCP = Registered Mail (등기 우편)
├── Confirmation of delivery
├── Tracking available
├── Guaranteed arrival
└── More expensive/slower

UDP = Postcard (엽서)
├── Just send it
├── No confirmation
├── Might get lost
└── Fast and cheap
```

---

## UDP Header Structure (UDP 헤더 구조)

### Concept (개념)
The UDP header is simple and fixed at **8 bytes**, containing only four fields.

**Header Fields:**
| Field | Size | Description |
|-------|------|-------------|
| Source Port (출발지 포트) | 16 bits | Sender's port number |
| Destination Port (목적지 포트) | 16 bits | Receiver's port number |
| Length (길이) | 16 bits | Total datagram length (header + data) |
| Checksum (체크섬) | 16 bits | Error detection (optional in IPv4) |

**Length Field:**
- Minimum value: 8 bytes (header only, no data)
- Maximum value: 65,535 bytes (limited by 16-bit field)
- Practical maximum: ~65,507 bytes (65,535 - 8 UDP - 20 IP)

### Diagram / Example
```
UDP Header Format (8 bytes total):

 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
├─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┴─┤
│          Source Port          │       Destination Port        │
├───────────────────────────────┴───────────────────────────────┤
│            Length             │           Checksum            │
├───────────────────────────────┴───────────────────────────────┤
│                                                               │
│                          Data (Payload)                       │
│                                                               │
└───────────────────────────────────────────────────────────────┘


Example UDP Datagram (DNS Query):

┌─────────────────────────────────────────────────────────────┐
│ Source Port:      52431                                     │
│ Destination Port: 53 (DNS)                                  │
│ Length:           45 bytes                                  │
│ Checksum:         0x4a3b                                    │
├─────────────────────────────────────────────────────────────┤
│ Data: [DNS query for www.example.com]                       │
└─────────────────────────────────────────────────────────────┘


Comparison: UDP vs TCP Header Size

UDP Header: 8 bytes
┌──────────┬──────────┬──────────┬──────────┐
│ Src Port │ Dst Port │  Length  │ Checksum │
│ (2 bytes)│ (2 bytes)│ (2 bytes)│ (2 bytes)│
└──────────┴──────────┴──────────┴──────────┘

TCP Header: 20-60 bytes (minimum 20)
┌──────────┬──────────┬──────────┬──────────┬──────────┐
│ Src Port │ Dst Port │ Seq Num  │ Ack Num  │ Flags... │
│ (2)      │ (2)      │ (4)      │ (4)      │ (8+)     │
└──────────┴──────────┴──────────┴──────────┴──────────┘
```

---

## UDP Checksum (UDP 체크섬)

### Concept (개념)
The **checksum** provides error detection for the UDP datagram. It detects bit errors in the transmitted segment.

**Checksum Calculation:**
1. Create a pseudo-header (IP source, destination, protocol, length)
2. Add UDP header and data
3. Divide into 16-bit words
4. Sum all words using 1's complement arithmetic
5. Take 1's complement of the sum

**Important Notes:**
- Optional in IPv4 (0 means no checksum)
- Mandatory in IPv6 (since IPv6 header has no checksum)
- Detects errors but doesn't correct them
- If checksum fails, datagram is silently discarded

### Diagram / Example
```
UDP Checksum Calculation:

Pseudo-Header (for checksum only, not transmitted):
┌───────────────────────────────────────────────────────┐
│ Source IP Address           (32 bits)                 │
│ Destination IP Address      (32 bits)                 │
│ Zero │ Protocol (17) │ UDP Length    (32 bits)        │
└───────────────────────────────────────────────────────┘

                    +

UDP Header + Data:
┌───────────────────────────────────────────────────────┐
│ Source Port │ Dest Port │ Length │ Checksum=0 │ Data  │
└───────────────────────────────────────────────────────┘

                    =

Sum all 16-bit words → Take 1's complement → Checksum


Example:
Pseudo-header + UDP header + Data
= 0x4500 + 0x003C + 0x1234 + ... (all 16-bit words)
= Sum: 0x2B5C1
= Wrap carry: 0xB5C1 + 0x2 = 0xB5C3
= 1's complement: 0x4A3C ← This is the checksum


Verification at Receiver:
Sum(pseudo-header + UDP header + data + checksum)
Should equal: 0xFFFF (all 1s)
If not → Error detected → Discard datagram
```

---

## UDP Use Cases (UDP 사용 사례)

### Concept (개념)
UDP is preferred when **speed** matters more than **reliability**, or when the application handles reliability itself.

**Ideal Scenarios for UDP:**
1. **Real-time applications**: Latency-sensitive, can tolerate some loss
2. **Simple request-response**: Single packet transactions
3. **Broadcast/Multicast**: One-to-many communication
4. **Applications with own reliability**: Application-layer error handling

**Common UDP Applications:**
| Application | Port | Reason for UDP |
|-------------|------|----------------|
| DNS | 53 | Simple query-response |
| DHCP | 67/68 | Broadcast-based |
| TFTP | 69 | Simple file transfer |
| SNMP | 161/162 | Network monitoring |
| NTP | 123 | Time synchronization |
| RIP | 520 | Routing protocol |
| VoIP/SIP | 5060 | Real-time voice |
| Video Streaming | varies | Real-time media |
| Online Gaming | varies | Low latency needed |

### Diagram / Example
```
Why UDP for Different Applications:

1. DNS (Domain Name System):
┌──────────────────────────────────────────────────────────┐
│ • Single request, single response                        │
│ • Small packets (typically < 512 bytes)                  │
│ • Application can retry if no response                   │
│ • TCP connection overhead not worth it                   │
└──────────────────────────────────────────────────────────┘

2. Video Streaming:
┌──────────────────────────────────────────────────────────┐
│ • Real-time: late packet = useless packet                │
│ • Can tolerate some packet loss (minor glitch)           │
│ • Retransmission would cause delays/buffering            │
│ • Constant flow more important than perfect delivery     │
└──────────────────────────────────────────────────────────┘

3. Online Gaming:
┌──────────────────────────────────────────────────────────┐
│ • Low latency critical (< 100ms)                         │
│ • Game state updates sent frequently                     │
│ • Missing one update is OK, next one coming soon         │
│ • TCP head-of-line blocking would cause lag              │
└──────────────────────────────────────────────────────────┘

4. VoIP (Voice over IP):
┌──────────────────────────────────────────────────────────┐
│ • Real-time conversation                                 │
│ • 20ms packet intervals typical                          │
│ • Missing packet = brief silence (acceptable)            │
│ • Retransmitted packet = delayed audio (unacceptable)    │
└──────────────────────────────────────────────────────────┘
```

---

## UDP vs TCP (UDP와 TCP 비교)

### Concept (개념)
Choosing between UDP and TCP depends on application requirements for reliability, speed, and overhead.

**Detailed Comparison:**

| Aspect | UDP | TCP |
|--------|-----|-----|
| Connection | Connectionless | Connection-oriented |
| Reliability | Unreliable | Reliable (ACKs, retransmission) |
| Ordering | No guarantee | Guaranteed order |
| Speed | Faster | Slower |
| Header Size | 8 bytes | 20-60 bytes |
| Flow Control | None | Sliding window |
| Congestion Control | None | Yes |
| Broadcast/Multicast | Supported | Not supported |
| State | Stateless | Stateful |
| Use Case | Real-time, simple queries | File transfer, web |

### Diagram / Example
```
TCP vs UDP Decision Flowchart:

                    ┌─────────────────────────────┐
                    │ Need guaranteed delivery?   │
                    └─────────────────────────────┘
                           │           │
                          Yes          No
                           │           │
                           ▼           ▼
                    ┌─────────┐   ┌─────────────────────┐
                    │   TCP   │   │ Need low latency?   │
                    └─────────┘   └─────────────────────┘
                                        │           │
                                       Yes          No
                                        │           │
                                        ▼           ▼
                                  ┌─────────┐  ┌─────────┐
                                  │   UDP   │  │   TCP   │
                                  └─────────┘  └─────────┘


Performance Comparison:

Scenario: Transfer 1KB data

TCP:
[SYN] ────────────────>
<──────────────── [SYN-ACK]
[ACK] ────────────────>           ~1.5 RTT for connection
[DATA] ───────────────>
<──────────────── [ACK]
[FIN] ────────────────>           + data transfer
<──────────────── [FIN-ACK]       + connection close
[ACK] ────────────────>

UDP:
[DATA] ───────────────>           Just 1 packet!
(done)


Overhead Comparison:

For 100 bytes of data:

TCP:
  IP Header (20) + TCP Header (20) + Data (100) = 140 bytes
  Overhead: 40 bytes (29%)

UDP:
  IP Header (20) + UDP Header (8) + Data (100) = 128 bytes
  Overhead: 28 bytes (22%)
```

---

## UDP Socket Programming (UDP 소켓 프로그래밍)

### Concept (개념)
UDP socket programming is simpler than TCP because there's no connection establishment or maintenance.

**UDP Socket Operations:**
1. Create socket
2. Bind to port (server) or not (client)
3. Send/Receive datagrams
4. Close socket

**Key Differences from TCP:**
- No `listen()` or `accept()` calls
- No connection establishment
- Each `sendto()` can go to different destination
- Each `recvfrom()` returns sender's address

### Diagram / Example
```
UDP Client-Server Communication:

Server:                              Client:
┌─────────────────────┐              ┌─────────────────────┐
│ socket()            │              │ socket()            │
│     ↓               │              │     ↓               │
│ bind(port)          │              │ (optional bind)     │
│     ↓               │              │     ↓               │
│ recvfrom() ←────────│──────────────│ sendto()            │
│     ↓               │              │     ↓               │
│ sendto() ───────────│──────────────│→ recvfrom()         │
│     ↓               │              │     ↓               │
│ close()             │              │ close()             │
└─────────────────────┘              └─────────────────────┘


Python UDP Example:

# UDP Server
import socket

server = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
server.bind(('0.0.0.0', 9999))

while True:
    data, addr = server.recvfrom(1024)
    print(f"Received from {addr}: {data}")
    server.sendto(b"ACK", addr)


# UDP Client
import socket

client = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
client.sendto(b"Hello", ('localhost', 9999))
response, addr = client.recvfrom(1024)
print(f"Response: {response}")
client.close()
```

---

## UDP-based Protocols (UDP 기반 프로토콜)

### Concept (개념)
Many modern protocols are built on top of UDP to get its speed benefits while adding reliability at the application layer.

**QUIC (Quick UDP Internet Connections):**
- Developed by Google, now used for HTTP/3
- Provides reliability over UDP
- Built-in encryption (TLS 1.3)
- Multiplexed streams without head-of-line blocking
- Faster connection establishment (0-RTT)

**RTP (Real-time Transport Protocol):**
- Used for streaming media
- Adds sequence numbers and timestamps to UDP
- Works with RTCP for quality feedback

### Diagram / Example
```
QUIC vs TCP+TLS:

Traditional TCP + TLS:
[SYN] ──────────────────>
<────────────────── [SYN-ACK]
[ACK] ──────────────────>      1 RTT (TCP handshake)
[ClientHello] ──────────>
<────────── [ServerHello]      1 RTT (TLS handshake)
[Finished] ─────────────>
<──────────── [Finished]       0.5 RTT
[HTTP Request] ─────────>
                               Total: ~2.5 RTT before data

QUIC over UDP:
[Initial + ClientHello] ────────>
<──────── [Initial + ServerHello + Handshake]
[Handshake + HTTP Request] ─────>
                               Total: ~1 RTT before data

QUIC 0-RTT (resumed connection):
[0-RTT Data + HTTP Request] ────>
                               Total: 0 RTT!


RTP Packet Structure:
┌───────────────────────────────────────────────────────┐
│ V │P│X│ CC │M│    PT    │   Sequence Number          │
├───────────────────────────────────────────────────────┤
│                     Timestamp                         │
├───────────────────────────────────────────────────────┤
│                SSRC (Synchronization Source)          │
├───────────────────────────────────────────────────────┤
│                     Payload (Audio/Video)             │
└───────────────────────────────────────────────────────┘

- Sequence Number: Detect packet loss and reordering
- Timestamp: Synchronize playback
- But still no retransmission (it's still UDP underneath)
```

---

## Summary (요약)

| Concept | Key Points |
|---------|------------|
| UDP Characteristics | Connectionless, unreliable, fast, low overhead |
| Header Structure | 8 bytes: source port, dest port, length, checksum |
| Checksum | Error detection using pseudo-header |
| Use Cases | DNS, streaming, gaming, VoIP |
| UDP vs TCP | Speed vs reliability tradeoff |
| Modern Usage | Foundation for QUIC (HTTP/3), RTP |

---

## Key Terms (주요 용어)

- **UDP (User Datagram Protocol)**: 사용자 데이터그램 프로토콜
- **Datagram (데이터그램)**: UDP의 데이터 전송 단위
- **Connectionless (비연결형)**: 연결 설정 없이 데이터 전송
- **Unreliable (비신뢰성)**: 전달 보장 없음
- **Checksum (체크섬)**: 오류 검출용 값
- **Best-effort (최선 노력)**: 가능한 한 전달하지만 보장 없음
- **Stateless (무상태)**: 연결 상태 유지하지 않음
- **QUIC**: UDP 기반의 현대적 전송 프로토콜
