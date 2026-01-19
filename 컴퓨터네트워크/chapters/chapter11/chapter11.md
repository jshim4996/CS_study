# Chapter 11. NAT and DHCP

## Overview (개요)

NAT (Network Address Translation) and DHCP (Dynamic Host Configuration Protocol) are essential technologies for managing IP addresses in modern networks. NAT enables multiple devices to share a single public IP address, while DHCP automates IP address assignment.

NAT(네트워크 주소 변환)와 DHCP(동적 호스트 구성 프로토콜)는 현대 네트워크에서 IP 주소를 관리하기 위한 필수 기술입니다. NAT는 여러 장치가 단일 공인 IP 주소를 공유할 수 있게 하고, DHCP는 IP 주소 할당을 자동화합니다.

---

## 1. NAT Concept and Operation (NAT 개념과 동작)

### 1.1 What is NAT? (NAT란?)

NAT translates private IP addresses to public IP addresses and vice versa, enabling devices with private addresses to communicate on the Internet.

NAT는 사설 IP 주소를 공인 IP 주소로, 또는 그 반대로 변환하여 사설 주소를 가진 장치가 인터넷에서 통신할 수 있게 합니다.

```
Without NAT:
(NAT 없이:)

    Private Network                    Internet
    +-------------+                        X
    | 192.168.1.5 |-------> Cannot communicate directly
    +-------------+         (직접 통신 불가)

    Private addresses are not routable on the Internet.
    (사설 주소는 인터넷에서 라우팅되지 않습니다.)

With NAT:
(NAT 사용:)

    Private Network          NAT Router              Internet
    +-------------+        +-----------+        +-----------+
    | 192.168.1.5 |------->| Translate |------->| Web Server|
    +-------------+        | to Public |        | 8.8.8.8   |
                           | 203.0.113.5|        +-----------+
                           +-----------+
```

### 1.2 NAT Operation (NAT 동작)

```
Outbound Traffic (Outgoing):
(아웃바운드 트래픽 (발신):)

1. Host sends packet with private source IP
   호스트가 사설 출발지 IP로 패킷 전송

2. NAT router changes source IP to public IP
   NAT 라우터가 출발지 IP를 공인 IP로 변경

3. NAT router stores mapping in NAT table
   NAT 라우터가 NAT 테이블에 매핑 저장

4. Packet sent to Internet with public IP
   공인 IP로 패킷이 인터넷으로 전송

    Internal Host          NAT Router              External Server
    192.168.1.10              |                     93.184.216.34
         |                    |                          |
         |  Src: 192.168.1.10 |                          |
         |  Dst: 93.184.216.34|                          |
         +------------------->|                          |
                              |                          |
                   [Translate Source]                    |
                              |                          |
                              |  Src: 203.0.113.5        |
                              |  Dst: 93.184.216.34      |
                              +------------------------->|
                              |                          |

Inbound Traffic (Response):
(인바운드 트래픽 (응답):)

                              |  Src: 93.184.216.34      |
                              |  Dst: 203.0.113.5        |
                              |<-------------------------+
                              |                          |
                   [Translate Destination]               |
                              |                          |
         |  Src: 93.184.216.34|                          |
         |  Dst: 192.168.1.10 |                          |
         |<-------------------+                          |
```

### 1.3 NAT Table (NAT 테이블)

```
NAT Table (Basic):
(NAT 테이블 (기본):)

+------------------+------------------+
| Internal IP      | External IP      |
| (내부 IP)        | (외부 IP)        |
+------------------+------------------+
| 192.168.1.10     | 203.0.113.5      |
| 192.168.1.11     | 203.0.113.6      |
| 192.168.1.12     | 203.0.113.7      |
+------------------+------------------+

This is "Basic NAT" or "One-to-One NAT"
(이것은 "기본 NAT" 또는 "일대일 NAT"입니다)

Problem: Requires one public IP per internal host!
(문제: 내부 호스트당 하나의 공인 IP가 필요!)
```

---

## 2. NAPT (Port-based NAT) (포트 기반 NAT)

### 2.1 NAPT Concept (NAPT 개념)

NAPT (Network Address Port Translation), also called PAT or NAT overload, uses port numbers to allow multiple internal hosts to share a single public IP address.

NAPT(네트워크 주소 포트 변환)는 PAT 또는 NAT 오버로드라고도 하며, 포트 번호를 사용하여 여러 내부 호스트가 단일 공인 IP 주소를 공유할 수 있게 합니다.

```
NAPT Operation:
(NAPT 동작:)

    Internal Network              NAT Router                 Internet

    192.168.1.10:5000   \
                          \     203.0.113.5:10000      /
    192.168.1.11:5000   ----> [  NAT Translation  ] ---->  Web Server
                          /     203.0.113.5:10001      \   93.184.216.34:80
    192.168.1.12:6000   /      203.0.113.5:10002

    Multiple internal hosts --> Single public IP (different ports)
    (여러 내부 호스트 --> 단일 공인 IP (다른 포트))
```

### 2.2 NAPT Table (NAPT 테이블)

```
NAPT Table:
(NAPT 테이블:)

+---------------------+---------------------+----------------------+
| Internal Address    | External Address    | Destination          |
| (내부 주소)         | (외부 주소)         | (목적지)             |
+---------------------+---------------------+----------------------+
| 192.168.1.10:5000   | 203.0.113.5:10000   | 93.184.216.34:80     |
| 192.168.1.11:5000   | 203.0.113.5:10001   | 93.184.216.34:80     |
| 192.168.1.12:6000   | 203.0.113.5:10002   | 8.8.8.8:53           |
| 192.168.1.10:5001   | 203.0.113.5:10003   | 172.217.0.46:443     |
+---------------------+---------------------+----------------------+

Translation Process:
(변환 과정:)

Outbound:
1. Internal: 192.168.1.10:5000 -> External dest
2. NAT assigns port: 203.0.113.5:10000
3. Stores mapping in table

Inbound:
1. Response arrives at 203.0.113.5:10000
2. NAT looks up table: 10000 -> 192.168.1.10:5000
3. Forwards to internal host

Port number space: 65,535 ports available
(포트 번호 공간: 65,535개의 포트 사용 가능)
```

### 2.3 NAPT Example Flow (NAPT 예시 흐름)

```
Step-by-Step Example:
(단계별 예시:)

Host A (192.168.1.10) wants to access http://example.com (93.184.216.34)
(호스트 A가 http://example.com에 접속하려 함)

Step 1: Host A creates packet
(1단계: 호스트 A가 패킷 생성)

+-----------------------------------------------+
| IP Header                                     |
| Src IP: 192.168.1.10  Dst IP: 93.184.216.34   |
+-----------------------------------------------+
| TCP Header                                    |
| Src Port: 50000       Dst Port: 80            |
+-----------------------------------------------+

Step 2: NAT Router receives and translates
(2단계: NAT 라우터가 수신하고 변환)

+-----------------------------------------------+
| IP Header                                     |
| Src IP: 203.0.113.5   Dst IP: 93.184.216.34   |
+-----------------------------------------------+
| TCP Header                                    |
| Src Port: 10000       Dst Port: 80            |
+-----------------------------------------------+

NAT Table Entry:
192.168.1.10:50000 <-> 203.0.113.5:10000 <-> 93.184.216.34:80

Step 3: Server responds
(3단계: 서버가 응답)

+-----------------------------------------------+
| IP Header                                     |
| Src IP: 93.184.216.34 Dst IP: 203.0.113.5     |
+-----------------------------------------------+
| TCP Header                                    |
| Src Port: 80          Dst Port: 10000         |
+-----------------------------------------------+

Step 4: NAT Router translates response
(4단계: NAT 라우터가 응답을 변환)

+-----------------------------------------------+
| IP Header                                     |
| Src IP: 93.184.216.34 Dst IP: 192.168.1.10    |
+-----------------------------------------------+
| TCP Header                                    |
| Src Port: 80          Dst Port: 50000         |
+-----------------------------------------------+
```

---

## 3. NAT Pros and Cons (NAT 장단점)

### 3.1 Advantages of NAT (NAT의 장점)

```
+-------------------------------------------------------------------+
| Advantage (장점)        | Description (설명)                      |
+-------------------------------------------------------------------+
| IP Address              | Many devices can share one public IP    |
| Conservation            | 많은 장치가 하나의 공인 IP를 공유 가능  |
| (IP 주소 절약)          |                                         |
+-------------------------------------------------------------------+
| Security                | Internal network structure is hidden    |
| (보안)                  | 내부 네트워크 구조가 숨겨짐             |
|                         | Acts as basic firewall                  |
|                         | 기본 방화벽 역할                        |
+-------------------------------------------------------------------+
| Flexibility             | Internal addressing can be changed      |
| (유연성)                | without affecting external connectivity |
|                         | 외부 연결에 영향 없이 내부 주소 변경 가능|
+-------------------------------------------------------------------+
| ISP Independence        | Can change ISP without re-addressing    |
| (ISP 독립성)            | internal network                        |
|                         | 내부 네트워크 재주소 지정 없이 ISP 변경 가능|
+-------------------------------------------------------------------+

Security Benefit Illustration:
(보안 이점 그림:)

    Internet                    NAT Router                 Internal
        |                           |                          |
        |   Cannot initiate         |                          |
        |   connection to           |         Protected        |
        |   internal hosts    X---->|         Hosts            |
        |                           |                          |
        |   External attacker       |    192.168.1.x           |
        |   only sees               |    Hidden from           |
        |   203.0.113.5             |    Internet              |
```

### 3.2 Disadvantages of NAT (NAT의 단점)

```
+-------------------------------------------------------------------+
| Disadvantage (단점)     | Description (설명)                      |
+-------------------------------------------------------------------+
| End-to-End              | Breaks the original Internet model      |
| Connectivity            | 원래의 인터넷 모델을 깨뜨림             |
| (종단간 연결성)         | External hosts cannot initiate          |
|                         | connections to internal hosts           |
|                         | 외부 호스트가 내부 호스트에 연결 시작 불가|
+-------------------------------------------------------------------+
| Application             | Some protocols embed IP addresses       |
| Compatibility           | (FTP, SIP, H.323)                       |
| (애플리케이션 호환성)   | 일부 프로토콜이 IP 주소를 내장          |
|                         | Requires ALG (Application Layer Gateway)|
|                         | ALG 필요                                |
+-------------------------------------------------------------------+
| Performance             | Translation adds processing overhead    |
| (성능)                  | 변환이 처리 오버헤드 추가               |
+-------------------------------------------------------------------+
| Troubleshooting         | Complicates network debugging           |
| (문제 해결)             | 네트워크 디버깅을 복잡하게 함           |
+-------------------------------------------------------------------+

Peer-to-Peer Problem:
(P2P 문제:)

    Host A                 NAT A          NAT B               Host B
  192.168.1.5               |               |            192.168.2.10
       |                    |               |                   |
       |   Both hosts behind NAT - cannot connect directly!     |
       |   (양쪽 호스트가 NAT 뒤에 있음 - 직접 연결 불가!)      |
       |                    |               |                   |
       X--------------------> <-------------X                   |

Solutions: NAT Traversal techniques (STUN, TURN, ICE)
(해결책: NAT 통과 기술)
```

### 3.3 Port Forwarding (포트 포워딩)

```
Port forwarding allows external access to internal servers:
(포트 포워딩은 내부 서버에 대한 외부 접근을 허용합니다:)

Configuration:
(구성:)

External 203.0.113.5:80 --> Forward to --> Internal 192.168.1.100:80
External 203.0.113.5:22 --> Forward to --> Internal 192.168.1.50:22

    Internet                NAT Router              Internal Servers
        |                       |
        |  Request to           |
        |  203.0.113.5:80       |    +------------------+
        |--------------------->|---->| Web Server       |
        |                       |    | 192.168.1.100:80 |
        |                       |    +------------------+
        |                       |
        |  Request to           |    +------------------+
        |  203.0.113.5:22       |    | SSH Server       |
        |--------------------->|---->| 192.168.1.50:22  |
        |                       |    +------------------+

NAT Port Forwarding Table:
(NAT 포트 포워딩 테이블:)

+-------------------+-------------------+
| External Port     | Internal Address  |
| (외부 포트)       | (내부 주소)       |
+-------------------+-------------------+
| 80                | 192.168.1.100:80  |
| 22                | 192.168.1.50:22   |
| 443               | 192.168.1.100:443 |
+-------------------+-------------------+
```

---

## 4. DHCP Concept (DHCP 개념)

### 4.1 What is DHCP? (DHCP란?)

DHCP (Dynamic Host Configuration Protocol) automatically assigns IP addresses and network configuration to devices on a network.

DHCP(동적 호스트 구성 프로토콜)는 네트워크의 장치에 IP 주소와 네트워크 구성을 자동으로 할당합니다.

```
Without DHCP (Manual Configuration):
(DHCP 없이 (수동 구성):)

Administrator must configure each device manually:
(관리자가 각 장치를 수동으로 구성해야 함:)

Device 1: IP=192.168.1.10, Mask=255.255.255.0, GW=192.168.1.1, DNS=8.8.8.8
Device 2: IP=192.168.1.11, Mask=255.255.255.0, GW=192.168.1.1, DNS=8.8.8.8
Device 3: IP=192.168.1.12, Mask=255.255.255.0, GW=192.168.1.1, DNS=8.8.8.8
...
(Tedious and error-prone! / 지루하고 오류가 발생하기 쉬움!)

With DHCP (Automatic Configuration):
(DHCP 사용 (자동 구성):)

    +-------------+                      +--------------+
    | New Device  | -- "I need an IP" -->| DHCP Server  |
    |             | <-- "Here's your" -- | 192.168.1.1  |
    |             |     configuration    |              |
    +-------------+                      +--------------+

    Device automatically receives:
    (장치가 자동으로 수신:)
    - IP Address (IP 주소)
    - Subnet Mask (서브넷 마스크)
    - Default Gateway (기본 게이트웨이)
    - DNS Servers (DNS 서버)
    - Lease Time (임대 시간)
```

### 4.2 DHCP Components (DHCP 구성 요소)

```
+------------------+------------------------------------------+
| Component        | Description (설명)                       |
| (구성 요소)      |                                          |
+------------------+------------------------------------------+
| DHCP Server      | Manages IP address pool and              |
| (DHCP 서버)      | configuration parameters                 |
|                  | IP 주소 풀과 구성 매개변수 관리          |
+------------------+------------------------------------------+
| DHCP Client      | Device requesting configuration          |
| (DHCP 클라이언트)| 구성을 요청하는 장치                     |
+------------------+------------------------------------------+
| DHCP Relay Agent | Forwards DHCP messages between           |
| (DHCP 릴레이     | different networks                       |
|  에이전트)       | 다른 네트워크 간 DHCP 메시지 전달        |
+------------------+------------------------------------------+
| IP Address Pool  | Range of addresses available for         |
| (IP 주소 풀)     | assignment                               |
|                  | 할당 가능한 주소 범위                    |
+------------------+------------------------------------------+
| Lease            | Time period for which IP is assigned     |
| (임대)           | IP가 할당되는 기간                       |
+------------------+------------------------------------------+

DHCP Server Configuration Example:
(DHCP 서버 구성 예시:)

    Pool: 192.168.1.100 - 192.168.1.200
    Subnet Mask: 255.255.255.0
    Default Gateway: 192.168.1.1
    DNS: 8.8.8.8, 8.8.4.4
    Lease Time: 24 hours
```

---

## 5. DHCP DORA Process (DHCP DORA 과정)

### 5.1 DORA Overview (DORA 개요)

DORA stands for the four-step DHCP process: Discover, Offer, Request, Acknowledge.

DORA는 네 단계의 DHCP 과정을 나타냅니다: Discover(발견), Offer(제안), Request(요청), Acknowledge(승인).

```
DHCP DORA Process:
(DHCP DORA 과정:)

    DHCP Client                                    DHCP Server
         |                                              |
         |  [D] DHCP Discover (Broadcast)               |
         |  "I need an IP address!"                     |
         |  (IP 주소가 필요합니다!)                     |
         |  Src: 0.0.0.0    Dst: 255.255.255.255       |
         |--------------------------------------------->|
         |                                              |
         |  [O] DHCP Offer                              |
         |  "Here's 192.168.1.100 for you"              |
         |  (여기 192.168.1.100을 드립니다)             |
         |<---------------------------------------------|
         |                                              |
         |  [R] DHCP Request (Broadcast)                |
         |  "I'll take 192.168.1.100"                   |
         |  (192.168.1.100을 사용하겠습니다)            |
         |--------------------------------------------->|
         |                                              |
         |  [A] DHCP Acknowledge                        |
         |  "Confirmed! Lease granted"                  |
         |  (확인! 임대 승인됨)                         |
         |<---------------------------------------------|
         |                                              |
    IP Configured: 192.168.1.100
    (IP 구성됨: 192.168.1.100)
```

### 5.2 DORA Step-by-Step (DORA 단계별)

```
Step 1: DHCP Discover
(1단계: DHCP 발견)

+----------------------------------------------------+
| Ethernet Frame                                     |
| Src MAC: Client MAC    Dst MAC: FF:FF:FF:FF:FF:FF  |
+----------------------------------------------------+
| IP Header                                          |
| Src IP: 0.0.0.0        Dst IP: 255.255.255.255     |
+----------------------------------------------------+
| UDP Header                                         |
| Src Port: 68           Dst Port: 67                |
+----------------------------------------------------+
| DHCP Discover Message                              |
| - Transaction ID                                   |
| - Client MAC Address                               |
| - Requested Options (IP, Mask, GW, DNS)            |
+----------------------------------------------------+

- Client broadcasts because it doesn't know server's address
  (클라이언트는 서버 주소를 모르기 때문에 브로드캐스트)
- Uses 0.0.0.0 as source because it has no IP yet
  (아직 IP가 없으므로 0.0.0.0을 출발지로 사용)


Step 2: DHCP Offer
(2단계: DHCP 제안)

+----------------------------------------------------+
| DHCP Offer Message                                 |
| - Transaction ID (same as Discover)                |
| - Your IP Address: 192.168.1.100                   |
| - Server IP Address: 192.168.1.1                   |
| - Subnet Mask: 255.255.255.0                       |
| - Default Gateway: 192.168.1.1                     |
| - DNS Server: 8.8.8.8                              |
| - Lease Time: 86400 seconds (24 hours)             |
+----------------------------------------------------+

- Server offers available IP address
  (서버가 사용 가능한 IP 주소를 제안)
- Multiple servers can make offers
  (여러 서버가 제안할 수 있음)


Step 3: DHCP Request
(3단계: DHCP 요청)

+----------------------------------------------------+
| DHCP Request Message (Broadcast)                   |
| - Transaction ID                                   |
| - Requested IP Address: 192.168.1.100              |
| - Server Identifier: 192.168.1.1                   |
+----------------------------------------------------+

- Client broadcasts request so all servers know which offer accepted
  (클라이언트가 요청을 브로드캐스트하여 모든 서버가 어떤 제안이 수락되었는지 알 수 있음)
- Other servers retract their offers
  (다른 서버들은 제안을 철회)


Step 4: DHCP Acknowledge
(4단계: DHCP 승인)

+----------------------------------------------------+
| DHCP Acknowledge Message                           |
| - Transaction ID                                   |
| - Your IP Address: 192.168.1.100                   |
| - Configuration Parameters (confirmed)             |
| - Lease Time: 86400 seconds                        |
+----------------------------------------------------+

- Server confirms the lease
  (서버가 임대를 확인)
- Client can now use the IP address
  (클라이언트가 이제 IP 주소를 사용할 수 있음)
```

### 5.3 DHCP Lease Lifecycle (DHCP 임대 수명 주기)

```
Lease Timeline:
(임대 타임라인:)

   T0                T1 (50%)            T2 (87.5%)         T3 (100%)
    |---------------------|---------------------|--------------|
    |                     |                     |              |
  Lease              Renewal               Rebind           Lease
  Start              Timer                 Timer            Expires
  (임대 시작)        (갱신 타이머)         (재바인딩 타이머) (임대 만료)

Renewal Process (T1):
(갱신 과정 (T1):)

Client                                          Server
   |                                               |
   |  At 50% of lease time                        |
   |  (임대 시간의 50% 시점에)                    |
   |                                               |
   |  DHCP Request (unicast) ------------------>  |
   |  "Renew my lease please"                     |
   |                                               |
   |  <------------------- DHCP Acknowledge       |
   |  "Renewed! New lease time"                   |
   |                                               |

Rebind Process (T2):
(재바인딩 과정 (T2):)

If renewal fails at T1:
(T1에서 갱신이 실패하면:)

Client                                          Any Server
   |                                               |
   |  At 87.5% of lease time                      |
   |  (임대 시간의 87.5% 시점에)                  |
   |                                               |
   |  DHCP Request (broadcast) ----------------> |
   |  "Anyone renew my lease?"                   |
   |                                               |
   |  <------------------- DHCP Acknowledge       |
   |  "I can help!"                               |
   |                                               |

If T3 reached without renewal: IP released
(갱신 없이 T3에 도달하면: IP 해제)
```

### 5.4 DHCP Relay Agent (DHCP 릴레이 에이전트)

```
DHCP broadcasts don't cross router boundaries:
(DHCP 브로드캐스트는 라우터 경계를 넘지 않습니다:)

    Subnet A                Router                  Subnet B
   192.168.1.0/24             |                   192.168.2.0/24
        |                     |                        |
   +--------+                 |                   +--------+
   | Client |---Broadcast--->X (Blocked!)         | DHCP   |
   +--------+                 |                   | Server |
                              |                   +--------+

Solution: DHCP Relay Agent
(해결책: DHCP 릴레이 에이전트)

    Subnet A                Router (Relay)          Subnet B
   192.168.1.0/24             |                   192.168.2.0/24
        |                     |                        |
   +--------+                 |                   +--------+
   | Client |---Broadcast--->| |---Unicast------>| DHCP   |
   +--------+                 | |                | Server |
        |                     | |                +--------+
        |                     | |                     |
        |<---Unicast---------| |<---Unicast----------|
        |                     |                       |

The relay agent:
(릴레이 에이전트는:)
1. Receives broadcast from client (클라이언트로부터 브로드캐스트 수신)
2. Adds GIADDR (Gateway IP Address) field (GIADDR 필드 추가)
3. Forwards to DHCP server via unicast (유니캐스트로 DHCP 서버에 전달)
4. Receives response and forwards to client (응답을 받아 클라이언트에 전달)
```

---

## Summary (요약)

### Key Points (핵심 포인트)

1. **NAT Concept (NAT 개념)**
   - Translates private IP addresses to public addresses
   - Enables Internet access for private networks
   - 사설 IP 주소를 공인 주소로 변환
   - 사설 네트워크의 인터넷 접근 가능

2. **NAPT (Port-based NAT) (포트 기반 NAT)**
   - Uses port numbers to multiplex connections
   - Many internal hosts share one public IP
   - 포트 번호를 사용하여 연결을 다중화
   - 많은 내부 호스트가 하나의 공인 IP 공유

3. **NAT Pros and Cons (NAT 장단점)**
   - Pros: IP conservation, security, flexibility
   - Cons: Breaks end-to-end connectivity, application issues
   - 장점: IP 절약, 보안, 유연성
   - 단점: 종단간 연결성 파괴, 애플리케이션 문제

4. **DHCP Concept (DHCP 개념)**
   - Automatic IP address assignment
   - Provides IP, mask, gateway, DNS, and lease time
   - 자동 IP 주소 할당
   - IP, 마스크, 게이트웨이, DNS, 임대 시간 제공

5. **DHCP DORA Process (DHCP DORA 과정)**
   - Discover: Client broadcasts for server
   - Offer: Server proposes configuration
   - Request: Client accepts offer
   - Acknowledge: Server confirms lease
   - 발견: 클라이언트가 서버를 찾아 브로드캐스트
   - 제안: 서버가 구성을 제안
   - 요청: 클라이언트가 제안을 수락
   - 승인: 서버가 임대를 확인

```
+---------------------+------------------------------------------+
| Technology          | Key Function                             |
| (기술)              | (핵심 기능)                              |
+---------------------+------------------------------------------+
| NAT                 | Private <-> Public IP translation        |
|                     | 사설 <-> 공인 IP 변환                    |
+---------------------+------------------------------------------+
| NAPT                | Multiple hosts share one public IP       |
|                     | via port numbers                         |
|                     | 포트 번호로 여러 호스트가 하나의         |
|                     | 공인 IP 공유                             |
+---------------------+------------------------------------------+
| DHCP                | Automatic IP configuration               |
|                     | 자동 IP 구성                             |
+---------------------+------------------------------------------+
| DORA                | 4-step DHCP: Discover, Offer,            |
|                     | Request, Acknowledge                     |
|                     | 4단계 DHCP: 발견, 제안, 요청, 승인       |
+---------------------+------------------------------------------+
| Port Forwarding     | Allow external access to internal        |
| (포트 포워딩)       | servers through NAT                      |
|                     | NAT를 통해 내부 서버에 외부 접근 허용    |
+---------------------+------------------------------------------+
| DHCP Relay          | Forward DHCP across router boundaries    |
| (DHCP 릴레이)       | 라우터 경계를 넘어 DHCP 전달             |
+---------------------+------------------------------------------+
```
