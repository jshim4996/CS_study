# Chapter 01. Introduction to Networks (네트워크 개요)

> Understanding the fundamental concepts of computer networks and their layered architecture.

---

## Network (네트워크)

### Concept (개념)
A **network** is a collection of interconnected devices (computers, servers, routers, etc.) that can communicate and share resources with each other. Networks enable data exchange between devices using standardized rules called protocols.

**Types of Networks:**
- **LAN (Local Area Network / 근거리 통신망)**: Covers a small geographic area like a home or office
- **WAN (Wide Area Network / 광역 통신망)**: Spans large geographic areas, connecting multiple LANs
- **MAN (Metropolitan Area Network / 도시권 통신망)**: Covers a city or large campus

### Diagram / Example
```
[Computer A] ----+
                 |
[Computer B] ----+---- [Switch] ---- [Router] ---- [Internet]
                 |
[Computer C] ----+

        LAN (Local Area Network)              WAN
```

---

## Internet (인터넷)

### Concept (개념)
The **Internet** is a global network of interconnected networks that uses the TCP/IP protocol suite. It's often called the "network of networks" because it connects millions of private, public, academic, business, and government networks.

**Key Components:**
- **End Systems (Host)**: Devices that send/receive data (computers, smartphones, servers)
- **Communication Links**: Physical media (fiber optics, copper wire, wireless)
- **Packet Switches**: Routers and switches that forward data packets
- **ISP (Internet Service Provider / 인터넷 서비스 제공자)**: Organizations providing Internet access

### Diagram / Example
```
[Home Network] ---- [ISP] ----+
                              |
[Company Network] ---- [ISP] ----+---- [Internet Backbone] ---- [Data Center]
                              |
[University Network] ---- [ISP] ----+
```

---

## Protocol (프로토콜)

### Concept (개념)
A **protocol** is a set of rules that governs how data is transmitted and received over a network. Protocols define:

- **Syntax**: Format and structure of data
- **Semantics**: Meaning of each section of bits
- **Timing**: When data should be sent and how fast

**Common Protocols:**
| Protocol | Purpose |
|----------|---------|
| HTTP/HTTPS | Web browsing |
| FTP | File transfer |
| SMTP | Email sending |
| DNS | Domain name resolution |
| TCP | Reliable data transmission |
| IP | Addressing and routing |

### Diagram / Example
```
Communication Example:

[Client]                                    [Server]
   |                                           |
   |  -------- SYN (Connection Request) ---->  |
   |  <------- SYN-ACK (Acknowledgment) -----  |
   |  -------- ACK (Confirmed) ------------->  |
   |                                           |
   |  ======== Data Transfer ===============>  |
   |                                           |

This follows the TCP protocol rules for establishing a connection.
```

---

## OSI 7-Layer Model (OSI 7계층)

### Concept (개념)
The **OSI (Open Systems Interconnection)** model is a conceptual framework that standardizes network communication into seven distinct layers. Each layer has specific functions and communicates with adjacent layers.

| Layer | Name | Function (기능) | Example |
|-------|------|-----------------|---------|
| 7 | Application (응용) | User interface, network services | HTTP, FTP, SMTP |
| 6 | Presentation (표현) | Data formatting, encryption | SSL/TLS, JPEG, ASCII |
| 5 | Session (세션) | Session management | NetBIOS, RPC |
| 4 | Transport (전송) | End-to-end communication | TCP, UDP |
| 3 | Network (네트워크) | Logical addressing, routing | IP, ICMP |
| 2 | Data Link (데이터 링크) | Physical addressing, framing | Ethernet, MAC |
| 1 | Physical (물리) | Bit transmission | Cables, Hubs |

**Mnemonic**: "**A**ll **P**eople **S**eem **T**o **N**eed **D**ata **P**rocessing" (top to bottom)

### Diagram / Example
```
Application    [HTTP Request: GET /index.html]
     ↓
Presentation   [Data Encryption/Compression]
     ↓
Session        [Session Management]
     ↓
Transport      [TCP Segment: Port 80]
     ↓
Network        [IP Packet: 192.168.1.1 → 10.0.0.1]
     ↓
Data Link      [Ethernet Frame: MAC addresses]
     ↓
Physical       [Electrical signals on wire]
```

---

## TCP/IP 4-Layer Model (TCP/IP 4계층)

### Concept (개념)
The **TCP/IP model** is a more practical, implementation-focused model used in the actual Internet. It consolidates the OSI model into four layers.

| TCP/IP Layer | OSI Equivalent | Protocols |
|--------------|----------------|-----------|
| Application (응용) | Application + Presentation + Session | HTTP, FTP, DNS, SMTP |
| Transport (전송) | Transport | TCP, UDP |
| Internet (인터넷) | Network | IP, ICMP, ARP |
| Network Access (네트워크 액세스) | Data Link + Physical | Ethernet, Wi-Fi |

**Key Difference from OSI:**
- TCP/IP is protocol-specific and practically implemented
- OSI is a theoretical reference model
- TCP/IP was developed alongside the Internet

### Diagram / Example
```
OSI Model                    TCP/IP Model
---------                    ------------
Application  ─┐
Presentation  ├─────────►    Application
Session      ─┘
Transport    ─────────────►  Transport
Network      ─────────────►  Internet
Data Link    ─┐
Physical     ─┴─────────►    Network Access
```

---

## Encapsulation (캡슐화)

### Concept (개념)
**Encapsulation** is the process of wrapping data with protocol information at each layer as it moves down the protocol stack. Each layer adds its own header (and sometimes trailer) to the data from the layer above.

**Process:**
1. **Application Layer**: Creates data (e.g., HTTP request)
2. **Transport Layer**: Adds TCP/UDP header → **Segment (세그먼트)**
3. **Network Layer**: Adds IP header → **Packet (패킷)**
4. **Data Link Layer**: Adds MAC header and trailer → **Frame (프레임)**
5. **Physical Layer**: Converts to bits → **Bits (비트)**

**Decapsulation (역캡슐화)**: The reverse process at the receiving end, where headers are removed at each layer going up.

### Diagram / Example
```
Sending Side (Encapsulation)           Receiving Side (Decapsulation)

[    Data    ]                         [    Data    ]
      ↓                                      ↑
[TCP|  Data  ]  ← Segment              [TCP|  Data  ]
      ↓                                      ↑
[IP|TCP|Data ]  ← Packet               [IP|TCP|Data ]
      ↓                                      ↑
[ETH|IP|TCP|Data|FCS]  ← Frame         [ETH|IP|TCP|Data|FCS]
      ↓                                      ↑
[10110010...]  ← Bits                  [10110010...]

====== Network Transmission ======>
```

**PDU (Protocol Data Unit) Names:**
| Layer | PDU Name |
|-------|----------|
| Application | Data (데이터) / Message (메시지) |
| Transport | Segment (세그먼트) |
| Network | Packet (패킷) |
| Data Link | Frame (프레임) |
| Physical | Bit (비트) |

---

## Summary (요약)

| Concept | Key Points |
|---------|------------|
| Network | Interconnected devices sharing resources |
| Internet | Global network of networks using TCP/IP |
| Protocol | Rules for data communication |
| OSI Model | 7-layer theoretical reference model |
| TCP/IP Model | 4-layer practical implementation model |
| Encapsulation | Adding headers at each layer going down |

---

## Key Terms (주요 용어)

- **Network (네트워크)**: 통신망, 연결된 장치들의 집합
- **Protocol (프로토콜)**: 통신 규약
- **Layer (계층)**: 네트워크 기능의 추상화 단위
- **Encapsulation (캡슐화)**: 데이터에 헤더를 추가하는 과정
- **Packet (패킷)**: 네트워크 계층의 데이터 단위
- **Frame (프레임)**: 데이터 링크 계층의 데이터 단위
