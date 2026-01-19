# Chapter 10. IP Packets and Routing (IP 패킷과 라우팅)

## Overview (개요)

This chapter covers the structure of IP packets and how routing enables data to travel across interconnected networks. Understanding IP packet format and routing mechanisms is essential for network troubleshooting and design.

이 장에서는 IP 패킷의 구조와 라우팅이 상호 연결된 네트워크를 통해 데이터가 이동하는 방법을 다룹니다. IP 패킷 형식과 라우팅 메커니즘의 이해는 네트워크 문제 해결 및 설계에 필수적입니다.

---

## 1. IP Packet Structure (IP 패킷 구조)

### 1.1 IPv4 Header Format (IPv4 헤더 형식)

```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|Version|  IHL  |Type of Service|          Total Length         |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|         Identification        |Flags|      Fragment Offset    |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|  Time to Live |    Protocol   |         Header Checksum       |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                       Source Address                          |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                    Destination Address                        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                    Options                    |    Padding    |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                             Data                              |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

### 1.2 Header Fields Description (헤더 필드 설명)

| Field (필드) | Size (크기) | Description (설명) |
|-------------|-------------|-------------------|
| Version (버전) | 4 bits | IP version (4 for IPv4) / IP 버전 (IPv4는 4) |
| IHL (헤더 길이) | 4 bits | Header length in 32-bit words / 32비트 워드 단위의 헤더 길이 |
| Type of Service (서비스 유형) | 8 bits | QoS parameters / QoS 매개변수 |
| Total Length (전체 길이) | 16 bits | Entire packet size in bytes / 바이트 단위의 전체 패킷 크기 |
| Identification (식별자) | 16 bits | Unique fragment identifier / 고유 프래그먼트 식별자 |
| Flags (플래그) | 3 bits | Fragmentation control / 단편화 제어 |
| Fragment Offset (프래그먼트 오프셋) | 13 bits | Position in original datagram / 원본 데이터그램에서의 위치 |
| TTL (수명) | 8 bits | Maximum hops / 최대 홉 수 |
| Protocol (프로토콜) | 8 bits | Upper layer protocol / 상위 계층 프로토콜 |
| Header Checksum (헤더 체크섬) | 16 bits | Error checking / 오류 검사 |
| Source Address (출발지 주소) | 32 bits | Sender's IP address / 송신자의 IP 주소 |
| Destination Address (목적지 주소) | 32 bits | Receiver's IP address / 수신자의 IP 주소 |

### 1.3 Protocol Field Values (프로토콜 필드 값)

```
+----------+------------+--------------------------------+
| Value    | Protocol   | Description (설명)             |
+----------+------------+--------------------------------+
| 1        | ICMP       | Internet Control Message       |
|          |            | (인터넷 제어 메시지)           |
+----------+------------+--------------------------------+
| 6        | TCP        | Transmission Control           |
|          |            | (전송 제어)                    |
+----------+------------+--------------------------------+
| 17       | UDP        | User Datagram                  |
|          |            | (사용자 데이터그램)            |
+----------+------------+--------------------------------+
| 89       | OSPF       | Open Shortest Path First       |
|          |            | (개방형 최단 경로 우선)        |
+----------+------------+--------------------------------+

Example packet:
(패킷 예시:)

+------------------+
| IP Header        |
| Protocol = 6     | --> TCP segment follows
+------------------+
| TCP Header       |
+------------------+
| TCP Data         |
+------------------+
```

---

## 2. TTL (Time to Live) (수명)

### 2.1 TTL Concept (TTL 개념)

TTL prevents packets from circulating indefinitely in the network due to routing loops.

TTL은 라우팅 루프로 인해 패킷이 네트워크에서 무한히 순환하는 것을 방지합니다.

```
TTL Operation (TTL 동작):

1. Sender sets initial TTL value (e.g., 64, 128, or 255)
   송신자가 초기 TTL 값을 설정 (예: 64, 128, 또는 255)

2. Each router decrements TTL by 1
   각 라우터가 TTL을 1씩 감소

3. If TTL reaches 0, packet is discarded
   TTL이 0에 도달하면 패킷은 폐기됨

4. Router sends ICMP "Time Exceeded" message to source
   라우터가 출발지에 ICMP "시간 초과" 메시지 전송

    Host A               Router 1           Router 2          Host B
       |                    |                  |                 |
       |  TTL=64 ---------> |                  |                 |
       |                    |  TTL=63 -------> |                 |
       |                    |                  |  TTL=62 ------> |
       |                    |                  |                 |

Without TTL (Routing Loop):
(TTL 없이 (라우팅 루프):)

    Router A ---------> Router B
        ^                  |
        |                  |
        +------ Router C <-+

    Packet loops forever! (패킷이 영원히 순환!)
```

### 2.2 TTL and Traceroute (TTL과 Traceroute)

```
Traceroute uses TTL to discover the path to a destination:
(Traceroute는 목적지까지의 경로를 발견하기 위해 TTL을 사용:)

Source                R1              R2              R3          Dest
   |                   |               |               |             |
   |  TTL=1 --------->X                |               |             |
   |                   |               |               |             |
   |  <---- ICMP Time Exceeded         |               |             |
   |  (Identifies R1)                  |               |             |
   |                   |               |               |             |
   |  TTL=2 ---------> | ------------->X               |             |
   |                   |               |               |             |
   |  <-------------- ICMP Time Exceeded               |             |
   |  (Identifies R2)                  |               |             |
   |                   |               |               |             |
   |  TTL=3 ---------> | ------------> | ------------->X             |
   |                   |               |               |             |
   |  <---------------------------- ICMP Time Exceeded               |
   |  (Identifies R3)                  |               |             |
   |                   |               |               |             |
   |  TTL=4 ---------> | ------------> | ------------> | ----------->|
   |                   |               |               |             |
   |  <-------------------------------------------- Reply            |
   |  (Destination reached!)           |               |             |

Example traceroute output:
(traceroute 출력 예:)

$ traceroute google.com
 1  192.168.1.1     1.234 ms
 2  10.0.0.1        5.678 ms
 3  172.16.0.1      10.123 ms
 4  216.58.200.46   15.456 ms
```

---

## 3. Fragmentation and Reassembly (단편화와 재조립)

### 3.1 Why Fragmentation? (왜 단편화가 필요한가?)

Different networks have different Maximum Transmission Unit (MTU) sizes.

서로 다른 네트워크는 서로 다른 최대 전송 단위(MTU) 크기를 가집니다.

```
MTU (Maximum Transmission Unit):
(최대 전송 단위:)

Ethernet MTU:    1500 bytes
Token Ring MTU:  4464 bytes
FDDI MTU:        4352 bytes

Problem:
(문제:)

    Network A              Network B              Network C
    MTU = 4000            MTU = 1500             MTU = 4000
        |                     |                      |
   +--------+            +--------+             +--------+
   | Packet |  ------>   |  ???   |  ------>    | Packet |
   | 3000B  |            |        |             |        |
   +--------+            +--------+             +--------+

   Packet too large for Network B!
   (패킷이 네트워크 B에 비해 너무 큼!)

Solution: Fragmentation
(해결책: 단편화)

   +--------+            +------+------+         +--------+
   | Packet |  ------>   | Frag | Frag | ----->  | Packet |
   | 3000B  |            | 1    | 2    |         | 3000B  |
   +--------+            +------+------+         +--------+
```

### 3.2 Fragmentation Fields (단편화 필드)

```
IP Header Fragmentation Fields:
(IP 헤더 단편화 필드:)

+-------------------+-------------------+
| Identification    | 16 bits           |
| (식별자)          | Same for all      |
|                   | fragments         |
+-------------------+-------------------+
| Flags             | 3 bits            |
| (플래그)          | DF, MF bits       |
+-------------------+-------------------+
| Fragment Offset   | 13 bits           |
| (프래그먼트 오프셋)| Position in       |
|                   | original packet   |
+-------------------+-------------------+

Flags:
+-----+-------------------------------------------+
| Bit | Name        | Description (설명)          |
+-----+-------------------------------------------+
|  0  | Reserved    | Must be 0 (항상 0)          |
+-----+-------------------------------------------+
|  1  | DF          | Don't Fragment              |
|     |             | (단편화 금지)               |
+-----+-------------------------------------------+
|  2  | MF          | More Fragments              |
|     |             | (추가 프래그먼트 있음)      |
+-----+-------------------------------------------+
```

### 3.3 Fragmentation Example (단편화 예시)

```
Original Packet: 4000 bytes data + 20 bytes header = 4020 bytes
(원본 패킷: 4000 바이트 데이터 + 20 바이트 헤더 = 4020 바이트)

MTU of next hop: 1500 bytes
(다음 홉의 MTU: 1500 바이트)

Maximum data per fragment: 1500 - 20 = 1480 bytes
(프래그먼트당 최대 데이터: 1500 - 20 = 1480 바이트)

Fragment Offset must be multiple of 8, so: 1480 bytes = 185 * 8
(프래그먼트 오프셋은 8의 배수여야 하므로: 1480 바이트 = 185 * 8)

+-------------+--------+--------+-----------+----------+
| Fragment    | Data   | Total  | MF Flag   | Offset   |
| (프래그먼트)| (데이터)| (전체) | (MF 플래그)| (오프셋) |
+-------------+--------+--------+-----------+----------+
| Fragment 1  | 1480   | 1500   | 1         | 0        |
+-------------+--------+--------+-----------+----------+
| Fragment 2  | 1480   | 1500   | 1         | 185      |
+-------------+--------+--------+-----------+----------+
| Fragment 3  | 1040   | 1060   | 0         | 370      |
+-------------+--------+--------+-----------+----------+

All fragments have same Identification number.
모든 프래그먼트는 동일한 식별자 번호를 가집니다.

   Original Packet
   +------------------------------------+
   |  Header  |        Data (4000B)     |
   +------------------------------------+

   After Fragmentation:

   Fragment 1                  Fragment 2                  Fragment 3
   +----------+-------+        +----------+-------+        +----------+-------+
   | Header   | 1480B |        | Header   | 1480B |        | Header   | 1040B |
   | ID=12345 |       |        | ID=12345 |       |        | ID=12345 |       |
   | MF=1     |       |        | MF=1     |       |        | MF=0     |       |
   | Off=0    |       |        | Off=185  |       |        | Off=370  |       |
   +----------+-------+        +----------+-------+        +----------+-------+
```

### 3.4 Reassembly (재조립)

```
Reassembly occurs at the final destination, not at intermediate routers.
(재조립은 중간 라우터가 아닌 최종 목적지에서 발생합니다.)

Destination Host:
(목적지 호스트:)

1. Receives fragments (프래그먼트 수신)
2. Identifies related fragments by ID (ID로 관련 프래그먼트 식별)
3. Orders by Fragment Offset (프래그먼트 오프셋으로 순서 정렬)
4. Waits for fragment with MF=0 (MF=0인 프래그먼트 대기)
5. Reassembles original packet (원본 패킷 재조립)

   Received:

   Fragment 3    Fragment 1    Fragment 2
   (Off=370)     (Off=0)       (Off=185)
      |             |              |
      v             v              v
   +---------------------------------------+
   |  Reassembly Buffer (ID = 12345)       |
   +---------------------------------------+
                     |
                     v
   +---------------------------------------+
   | Original Packet (4000 bytes data)     |
   +---------------------------------------+
```

---

## 4. Routing Concept (라우팅 개념)

### 4.1 What is Routing? (라우팅이란?)

Routing is the process of selecting paths in a network along which to send network traffic.

라우팅은 네트워크 트래픽을 전송할 경로를 네트워크에서 선택하는 과정입니다.

```
Routing Decision Process:
(라우팅 결정 과정:)

                    Packet arrives
                         |
                         v
              +---------------------+
              | Extract Destination |
              | IP Address          |
              +---------------------+
                         |
                         v
              +---------------------+
              | Look up Routing     |
              | Table               |
              +---------------------+
                         |
                         v
              +---------------------+
              | Find Best Match     |
              | (Longest Prefix)    |
              +---------------------+
                         |
                         v
              +---------------------+
              | Forward to Next Hop |
              | or Local Delivery   |
              +---------------------+
```

### 4.2 Hop-by-Hop Routing (홉바이홉 라우팅)

```
Each router makes independent forwarding decisions:
(각 라우터가 독립적인 전달 결정을 합니다:)

Source         R1              R2              R3          Dest
   |            |               |               |            |
   |            |               |               |            |
   +--- Pkt --->+               |               |            |
                |               |               |            |
        R1 decides next hop     |               |            |
                |               |               |            |
                +---- Pkt ----->+               |            |
                                |               |            |
                        R2 decides next hop     |            |
                                |               |            |
                                +---- Pkt ----->+            |
                                                |            |
                                        R3 decides next hop  |
                                                |            |
                                                +--- Pkt --->+
                                                             |
                                                         Delivered!
```

---

## 5. Routing Table (라우팅 테이블)

### 5.1 Routing Table Structure (라우팅 테이블 구조)

```
Routing Table Example:
(라우팅 테이블 예시:)

+------------------+------------------+------------------+----------+
| Destination      | Subnet Mask      | Next Hop         | Interface|
| (목적지)         | (서브넷 마스크)  | (다음 홉)        | (인터페이스)|
+------------------+------------------+------------------+----------+
| 192.168.1.0      | 255.255.255.0    | 0.0.0.0          | eth0     |
| (직접 연결)      |                  | (directly        |          |
|                  |                  |  connected)      |          |
+------------------+------------------+------------------+----------+
| 10.0.0.0         | 255.0.0.0        | 192.168.1.1      | eth0     |
+------------------+------------------+------------------+----------+
| 172.16.0.0       | 255.255.0.0      | 192.168.1.2      | eth0     |
+------------------+------------------+------------------+----------+
| 0.0.0.0          | 0.0.0.0          | 192.168.1.254    | eth0     |
| (Default Route)  |                  |                  |          |
| (기본 라우트)    |                  |                  |          |
+------------------+------------------+------------------+----------+
```

### 5.2 Longest Prefix Match (최장 접두사 매칭)

```
When multiple routes match, the most specific (longest prefix) wins.
(여러 경로가 일치할 때, 가장 구체적인 (가장 긴 접두사) 경로가 선택됩니다.)

Routing Table:
+------------------+------------------+
| Destination      | Next Hop         |
+------------------+------------------+
| 192.168.0.0/16   | Router A         |
| 192.168.1.0/24   | Router B         |
| 192.168.1.128/25 | Router C         |
+------------------+------------------+

Destination: 192.168.1.150

Matching process:
(매칭 과정:)

192.168.0.0/16   --> Matches (16 bits) --> Router A
192.168.1.0/24   --> Matches (24 bits) --> Router B
192.168.1.128/25 --> Matches (25 bits) --> Router C  <-- Winner! (최장 매칭!)

Packet forwarded to Router C
(패킷이 Router C로 전달됨)
```

### 5.3 Route Types (라우트 유형)

```
+------------------+------------------------------------------+
| Route Type       | Description (설명)                       |
| (라우트 유형)    |                                          |
+------------------+------------------------------------------+
| Directly         | Networks directly attached to router     |
| Connected        | interfaces                               |
| (직접 연결)      | 라우터 인터페이스에 직접 연결된 네트워크 |
+------------------+------------------------------------------+
| Static Route     | Manually configured by administrator     |
| (정적 라우트)    | 관리자가 수동으로 구성                   |
+------------------+------------------------------------------+
| Dynamic Route    | Learned through routing protocols        |
| (동적 라우트)    | 라우팅 프로토콜을 통해 학습              |
+------------------+------------------------------------------+
| Default Route    | Used when no other route matches         |
| (기본 라우트)    | 다른 라우트가 일치하지 않을 때 사용      |
|                  | (0.0.0.0/0)                              |
+------------------+------------------------------------------+
```

---

## 6. Static vs Dynamic Routing (정적 vs 동적 라우팅)

### 6.1 Static Routing (정적 라우팅)

```
Static routes are manually configured and do not change automatically.
(정적 라우트는 수동으로 구성되며 자동으로 변경되지 않습니다.)

Configuration Example (Linux):
(구성 예시 (Linux):)

$ ip route add 10.0.0.0/8 via 192.168.1.1
$ ip route add default via 192.168.1.254

Advantages (장점):
+ Simple to configure (구성이 간단)
+ No routing protocol overhead (라우팅 프로토콜 오버헤드 없음)
+ Predictable paths (예측 가능한 경로)
+ Secure (보안성)

Disadvantages (단점):
- No automatic failover (자동 장애 조치 없음)
- Manual updates required (수동 업데이트 필요)
- Does not scale well (확장성이 좋지 않음)

Best for (적합한 환경):
- Small networks (소규모 네트워크)
- Stub networks (스텁 네트워크)
- Specific traffic engineering (특정 트래픽 엔지니어링)
```

### 6.2 Dynamic Routing (동적 라우팅)

```
Dynamic routes are learned automatically through routing protocols.
(동적 라우트는 라우팅 프로토콜을 통해 자동으로 학습됩니다.)

                +----------+
                | Router A |
                +----+-----+
                     |
              Routing Protocol
              (OSPF, RIP, BGP)
                     |
      +----+---------+---------+----+
      |              |              |
+-----+----+   +-----+----+   +-----+----+
| Router B |   | Router C |   | Router D |
+----------+   +----------+   +----------+

Routers exchange routing information automatically.
(라우터들이 라우팅 정보를 자동으로 교환합니다.)

Advantages (장점):
+ Automatic route discovery (자동 경로 발견)
+ Adapts to network changes (네트워크 변화에 적응)
+ Automatic failover (자동 장애 조치)
+ Scales to large networks (대규모 네트워크로 확장)

Disadvantages (단점):
- More complex to configure (구성이 더 복잡)
- Uses bandwidth and CPU (대역폭과 CPU 사용)
- Potential for routing loops during convergence
  (수렴 중 라우팅 루프 가능성)
```

### 6.3 Common Routing Protocols (일반적인 라우팅 프로토콜)

```
+----------+------------------+--------------------------------+
| Protocol | Type             | Description (설명)             |
| (프로토콜)| (유형)          |                                |
+----------+------------------+--------------------------------+
| RIP      | Distance Vector  | Simple, max 15 hops            |
|          | (거리 벡터)      | 간단, 최대 15홉                |
+----------+------------------+--------------------------------+
| OSPF     | Link State       | Fast convergence, used in      |
|          | (링크 상태)      | enterprises                    |
|          |                  | 빠른 수렴, 기업에서 사용       |
+----------+------------------+--------------------------------+
| EIGRP    | Hybrid           | Cisco proprietary (was),       |
|          | (하이브리드)     | efficient                      |
|          |                  | 시스코 독점(이전), 효율적      |
+----------+------------------+--------------------------------+
| BGP      | Path Vector      | Internet routing between       |
|          | (경로 벡터)      | autonomous systems             |
|          |                  | 자율 시스템 간 인터넷 라우팅   |
+----------+------------------+--------------------------------+

Interior vs Exterior:
(내부 vs 외부:)

+----------------+        +----------------+
|   AS 100       |        |   AS 200       |
|  +---------+   |        |   +---------+  |
|  | OSPF    |   |  BGP   |   |  OSPF   |  |
|  | (IGP)   |<--+------->+-->|  (IGP)  |  |
|  +---------+   | (EGP)  |   +---------+  |
+----------------+        +----------------+

IGP: Interior Gateway Protocol (내부 게이트웨이 프로토콜)
EGP: Exterior Gateway Protocol (외부 게이트웨이 프로토콜)
AS: Autonomous System (자율 시스템)
```

### 6.4 Static vs Dynamic Comparison (정적 vs 동적 비교)

```
+----------------------+------------------+------------------+
| Feature              | Static           | Dynamic          |
| (특징)               | (정적)           | (동적)           |
+----------------------+------------------+------------------+
| Configuration        | Manual           | Automatic        |
| (구성)               | (수동)           | (자동)           |
+----------------------+------------------+------------------+
| Scalability          | Low              | High             |
| (확장성)             | (낮음)           | (높음)           |
+----------------------+------------------+------------------+
| Adaptability         | None             | High             |
| (적응성)             | (없음)           | (높음)           |
+----------------------+------------------+------------------+
| Resource Usage       | Minimal          | Higher           |
| (자원 사용)          | (최소)           | (높음)           |
+----------------------+------------------+------------------+
| Network Size         | Small            | Medium to Large  |
| (네트워크 크기)      | (소규모)         | (중대규모)       |
+----------------------+------------------+------------------+
| Failure Recovery     | Manual           | Automatic        |
| (장애 복구)          | (수동)           | (자동)           |
+----------------------+------------------+------------------+
```

---

## Summary (요약)

### Key Points (핵심 포인트)

1. **IP Packet Structure (IP 패킷 구조)**
   - 20-60 byte header with version, addresses, TTL, protocol, etc.
   - Key fields enable routing and delivery
   - 20-60 바이트 헤더에 버전, 주소, TTL, 프로토콜 등 포함
   - 핵심 필드가 라우팅과 전달을 가능하게 함

2. **TTL (Time to Live) (수명)**
   - Prevents infinite packet loops
   - Decremented by 1 at each router
   - Basis for traceroute operation
   - 무한 패킷 루프 방지
   - 각 라우터에서 1씩 감소
   - traceroute 동작의 기반

3. **Fragmentation (단편화)**
   - Splits large packets for smaller MTU networks
   - Uses Identification, Flags, and Fragment Offset fields
   - Reassembly occurs at destination only
   - 더 작은 MTU 네트워크를 위해 큰 패킷을 분할
   - 식별자, 플래그, 프래그먼트 오프셋 필드 사용
   - 재조립은 목적지에서만 발생

4. **Routing (라우팅)**
   - Process of selecting paths for network traffic
   - Hop-by-hop decision making
   - Longest prefix match for route selection
   - 네트워크 트래픽의 경로 선택 과정
   - 홉바이홉 결정
   - 경로 선택을 위한 최장 접두사 매칭

5. **Routing Table (라우팅 테이블)**
   - Contains destination, next hop, and interface information
   - Includes directly connected, static, dynamic, and default routes
   - 목적지, 다음 홉, 인터페이스 정보 포함
   - 직접 연결, 정적, 동적, 기본 라우트 포함

6. **Static vs Dynamic Routing (정적 vs 동적 라우팅)**
   - Static: Manual, simple, no automatic adaptation
   - Dynamic: Automatic, scalable, adapts to changes
   - 정적: 수동, 간단, 자동 적응 없음
   - 동적: 자동, 확장 가능, 변화에 적응

```
+--------------------+----------------------------------------+
| Concept            | Key Point                              |
| (개념)             | (핵심 포인트)                          |
+--------------------+----------------------------------------+
| IP Header          | 20+ bytes, routing & delivery info     |
| (IP 헤더)          | 20+ 바이트, 라우팅 및 전달 정보        |
+--------------------+----------------------------------------+
| TTL                | Loop prevention, traceroute            |
| (TTL)              | 루프 방지, traceroute                  |
+--------------------+----------------------------------------+
| Fragmentation      | Large packets -> smaller pieces        |
| (단편화)           | 큰 패킷 -> 작은 조각                   |
+--------------------+----------------------------------------+
| Routing Table      | Destination -> Next Hop mapping        |
| (라우팅 테이블)    | 목적지 -> 다음 홉 매핑                 |
+--------------------+----------------------------------------+
| Longest Prefix     | Most specific route wins               |
| (최장 접두사)      | 가장 구체적인 경로가 선택됨            |
+--------------------+----------------------------------------+
```
