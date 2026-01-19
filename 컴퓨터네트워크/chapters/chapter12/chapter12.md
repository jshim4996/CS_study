# Chapter 12. Link Layer (링크 계층)

---

## Table of Contents (목차)

1. [Link Layer Role (링크 계층의 역할)](#1-link-layer-role-링크-계층의-역할)
2. [MAC Address (MAC 주소)](#2-mac-address-mac-주소)
3. [Ethernet Frame Structure (이더넷 프레임 구조)](#3-ethernet-frame-structure-이더넷-프레임-구조)
4. [Switch Operation (스위치 동작)](#4-switch-operation-스위치-동작)
5. [ARP (Address Resolution Protocol)](#5-arp-address-resolution-protocol)
6. [Hub vs Switch vs Router](#6-hub-vs-switch-vs-router)
7. [Summary (요약)](#7-summary-요약)

---

## 1. Link Layer Role (링크 계층의 역할)

### What is the Link Layer? (링크 계층이란?)

The Link Layer (also called Data Link Layer or Layer 2) is responsible for **node-to-node communication** within the same local network segment.

링크 계층(데이터 링크 계층 또는 2계층)은 **동일 로컬 네트워크 세그먼트 내에서 노드 간 통신**을 담당합니다.

```
+--------------------------------------------------+
|              OSI Model Position                  |
+--------------------------------------------------+
|  Layer 7  |  Application  |  HTTP, FTP, SMTP    |
|  Layer 6  |  Presentation |  SSL, Encryption    |
|  Layer 5  |  Session      |  NetBIOS, RPC       |
|  Layer 4  |  Transport    |  TCP, UDP           |
|  Layer 3  |  Network      |  IP, ICMP, Routing  |
|  Layer 2  |  Data Link    |  <<< THIS LAYER >>> |  <-- Ethernet, MAC
|  Layer 1  |  Physical     |  Cables, Signals    |
+--------------------------------------------------+
```

### Key Responsibilities (주요 책임)

```
+---------------------------------------------------------------+
|                 Link Layer Functions                          |
+---------------------------------------------------------------+
|                                                               |
|  1. Framing (프레이밍)                                        |
|     - Encapsulate network layer packets into frames           |
|     - 네트워크 계층 패킷을 프레임으로 캡슐화                  |
|                                                               |
|  2. Physical Addressing (물리 주소 지정)                      |
|     - Use MAC addresses to identify devices                   |
|     - MAC 주소로 장치 식별                                    |
|                                                               |
|  3. Error Detection (오류 탐지)                               |
|     - Use CRC (Cyclic Redundancy Check)                       |
|     - CRC로 오류 검출                                         |
|                                                               |
|  4. Media Access Control (매체 접근 제어)                     |
|     - CSMA/CD for shared media                                |
|     - 공유 매체에서 CSMA/CD 사용                              |
|                                                               |
+---------------------------------------------------------------+
```

### Data Encapsulation Flow (데이터 캡슐화 흐름)

```
Application Data
       |
       v
+----------------------------------------------+
| Transport Header | Application Data          |  <-- Segment
+----------------------------------------------+
       |
       v
+------------------------------------------------------+
| IP Header | Transport Header | Application Data      |  <-- Packet
+------------------------------------------------------+
       |
       v
+----------------------------------------------------------------------+
| Ethernet | IP Header | Transport | App Data | FCS                   |
| Header   |           | Header    |          |                       |  <-- Frame
+----------------------------------------------------------------------+
     ^                                              ^
     |                                              |
  MAC addresses                              Error check
  (14 bytes)                                 (4 bytes)
```

---

## 2. MAC Address (MAC 주소)

### What is a MAC Address? (MAC 주소란?)

A MAC (Media Access Control) address is a **unique hardware identifier** assigned to network interface cards (NICs). It operates at Layer 2 and is used for local network communication.

MAC(Media Access Control) 주소는 네트워크 인터페이스 카드(NIC)에 할당된 **고유한 하드웨어 식별자**입니다. 2계층에서 동작하며 로컬 네트워크 통신에 사용됩니다.

### MAC Address Format (MAC 주소 형식)

```
+---------------------------------------------------------------+
|                    MAC Address Structure                       |
+---------------------------------------------------------------+
|                                                               |
|   48 bits (6 bytes) total                                     |
|                                                               |
|   AA : BB : CC : DD : EE : FF                                 |
|   |----OUI----|  |--Device ID--|                              |
|   (24 bits)       (24 bits)                                   |
|                                                               |
|   OUI (Organizationally Unique Identifier):                   |
|   - Assigned by IEEE to manufacturers                         |
|   - IEEE가 제조사에 할당                                      |
|                                                               |
|   Device ID (장치 ID):                                        |
|   - Assigned by manufacturer to each device                   |
|   - 제조사가 각 장치에 할당                                   |
|                                                               |
+---------------------------------------------------------------+

Examples (예시):
+------------------+-------------------------+
| OUI              | Manufacturer            |
+------------------+-------------------------+
| 00:1A:2B         | Apple, Inc.             |
| 00:50:56         | VMware, Inc.            |
| B8:27:EB         | Raspberry Pi Foundation |
| FC:AA:14         | Samsung Electronics     |
+------------------+-------------------------+
```

### MAC vs IP Address Comparison (MAC vs IP 주소 비교)

```
+----------------------+---------------------------+---------------------------+
| Feature              | MAC Address               | IP Address                |
|----------------------|---------------------------|---------------------------|
| Layer                | Layer 2 (Data Link)       | Layer 3 (Network)         |
| 계층                 | 2계층 (데이터 링크)       | 3계층 (네트워크)          |
+----------------------+---------------------------+---------------------------+
| Size                 | 48 bits (6 bytes)         | 32 bits (IPv4)            |
| 크기                 |                           | 128 bits (IPv6)           |
+----------------------+---------------------------+---------------------------+
| Format               | AA:BB:CC:DD:EE:FF         | 192.168.1.1               |
| 형식                 | (Hexadecimal)             | (Decimal dotted)          |
+----------------------+---------------------------+---------------------------+
| Assignment           | Manufacturer (burned-in)  | Network admin / DHCP      |
| 할당                 | 제조사 (하드웨어에 고정)  | 네트워크 관리자 / DHCP    |
+----------------------+---------------------------+---------------------------+
| Scope                | Local network only        | Global (across networks)  |
| 범위                 | 로컬 네트워크만           | 글로벌 (네트워크 간)      |
+----------------------+---------------------------+---------------------------+
| Changes?             | Usually permanent         | Can change (dynamic)      |
| 변경 여부            | 보통 영구적               | 변경 가능 (동적)          |
+----------------------+---------------------------+---------------------------+
```

### Special MAC Addresses (특수 MAC 주소)

```
+---------------------------+---------------------------+
| Address                   | Purpose                   |
+---------------------------+---------------------------+
| FF:FF:FF:FF:FF:FF         | Broadcast (모든 장치)     |
| 01:00:5E:xx:xx:xx         | IPv4 Multicast            |
| 33:33:xx:xx:xx:xx         | IPv6 Multicast            |
| 00:00:00:00:00:00         | Invalid/Unassigned        |
+---------------------------+---------------------------+
```

---

## 3. Ethernet Frame Structure (이더넷 프레임 구조)

### Ethernet II Frame Format (이더넷 II 프레임 형식)

```
+------------------------------------------------------------------------+
|                        Ethernet Frame Structure                         |
+------------------------------------------------------------------------+
|                                                                        |
|  +---------+--------+--------+------+---------+-----+                  |
|  |Preamble |  SFD   | Dest   |Source| Type/   |Data | FCS |            |
|  |         |        | MAC    | MAC  | Length  |     |     |            |
|  +---------+--------+--------+------+---------+-----+-----+            |
|  | 7 bytes |1 byte  |6 bytes |6bytes| 2 bytes |46-  |4 byt|            |
|  |         |        |        |      |         |1500 |es   |            |
|  |         |        |        |      |         |bytes|     |            |
|  +---------+--------+--------+------+---------+-----+-----+            |
|                                                                        |
+------------------------------------------------------------------------+

Total frame size: 64 - 1518 bytes (without preamble)
최소 64바이트 - 최대 1518바이트 (프리앰블 제외)
```

### Field Descriptions (필드 설명)

```
+----------------+--------+-----------------------------------------------+
| Field          | Size   | Description                                   |
|----------------|--------|-----------------------------------------------|
| Preamble       | 7 B    | Synchronization pattern (10101010...)         |
| 프리앰블       |        | 동기화 패턴                                   |
+----------------+--------+-----------------------------------------------+
| SFD            | 1 B    | Start Frame Delimiter (10101011)              |
| 시작 구분자    |        | 프레임 시작 표시                              |
+----------------+--------+-----------------------------------------------+
| Destination    | 6 B    | Recipient's MAC address                       |
| MAC            |        | 수신자 MAC 주소                               |
+----------------+--------+-----------------------------------------------+
| Source MAC     | 6 B    | Sender's MAC address                          |
|                |        | 송신자 MAC 주소                               |
+----------------+--------+-----------------------------------------------+
| Type/Length    | 2 B    | Protocol type (e.g., 0x0800 = IPv4)           |
| 타입/길이      |        | 프로토콜 타입 (예: 0x0800 = IPv4)             |
+----------------+--------+-----------------------------------------------+
| Payload        | 46-    | Data from upper layers                        |
| (Data)         | 1500 B | 상위 계층 데이터                              |
+----------------+--------+-----------------------------------------------+
| FCS            | 4 B    | Frame Check Sequence (CRC-32)                 |
|                |        | 프레임 검사 시퀀스                            |
+----------------+--------+-----------------------------------------------+
```

### Common EtherType Values (주요 EtherType 값)

```
+-------------+-------------------------+
| EtherType   | Protocol                |
+-------------+-------------------------+
| 0x0800      | IPv4                    |
| 0x0806      | ARP                     |
| 0x86DD      | IPv6                    |
| 0x8100      | VLAN-tagged frame       |
| 0x8847      | MPLS unicast            |
+-------------+-------------------------+
```

### CRC Error Detection (CRC 오류 검출)

```
Sender Side (송신측):
+----------------------------------------------------------+
|  1. Calculate CRC-32 over frame contents                 |
|  2. Append 4-byte FCS to frame                           |
|  3. Transmit frame                                       |
+----------------------------------------------------------+
              |
              v
        [Transmission]
              |
              v
Receiver Side (수신측):
+----------------------------------------------------------+
|  1. Receive frame                                        |
|  2. Calculate CRC-32 over received frame                 |
|  3. Compare with FCS field                               |
|     - Match: Frame is valid                              |
|     - Mismatch: Frame is corrupted, discard              |
+----------------------------------------------------------+
```

---

## 4. Switch Operation (스위치 동작)

### What is a Network Switch? (네트워크 스위치란?)

A switch is a **Layer 2 device** that forwards frames based on MAC addresses. It learns which devices are connected to which ports and makes intelligent forwarding decisions.

스위치는 MAC 주소를 기반으로 프레임을 전달하는 **2계층 장치**입니다. 어떤 장치가 어떤 포트에 연결되어 있는지 학습하고 지능적인 전달 결정을 합니다.

### MAC Address Table (MAC 주소 테이블)

```
+---------------------------------------------------------------+
|                    Switch MAC Address Table                    |
+---------------------------------------------------------------+
|                                                               |
|   +---------------+------------------+-------------+          |
|   | MAC Address   | Port             | Age (Timer) |          |
|   +---------------+------------------+-------------+          |
|   | AA:BB:CC:11   | Port 1           | 180 sec     |          |
|   | AA:BB:CC:22   | Port 2           | 45 sec      |          |
|   | AA:BB:CC:33   | Port 3           | 120 sec     |          |
|   | AA:BB:CC:44   | Port 4           | 30 sec      |          |
|   +---------------+------------------+-------------+          |
|                                                               |
|   - Entries expire after timeout (usually 5 min)             |
|   - 항목은 타임아웃 후 만료됨 (보통 5분)                      |
|                                                               |
+---------------------------------------------------------------+
```

### Switch Learning Process (스위치 학습 과정)

```
Step 1: Initial State (초기 상태)
+--------------------------------------------------+
|                   SWITCH                          |
|  +--------------------------------------------+  |
|  | MAC Table: (empty)                         |  |
|  +--------------------------------------------+  |
|     Port1    Port2    Port3    Port4             |
+------+--------+--------+--------+----------------+
       |        |        |        |
      [A]      [B]      [C]      [D]
   AA:11:11  BB:22:22  CC:33:33  DD:44:44


Step 2: A sends frame to B (A가 B로 프레임 전송)
+--------------------------------------------------+
|                   SWITCH                          |
|  +--------------------------------------------+  |
|  | MAC Table:                                 |  |
|  | AA:11:11 -> Port1  (LEARNED!)              |  |
|  +--------------------------------------------+  |
|     Port1    Port2    Port3    Port4             |
+------+--------+--------+--------+----------------+
       |   ====>|====>   |====>   |  FLOOD to all
      [A]      [B]      [C]      [D]               (unknown dest)
      src      dest

Switch doesn't know where B is, so it FLOODS
스위치가 B의 위치를 모르므로 FLOOD(범람)


Step 3: B replies to A (B가 A에게 응답)
+--------------------------------------------------+
|                   SWITCH                          |
|  +--------------------------------------------+  |
|  | MAC Table:                                 |  |
|  | AA:11:11 -> Port1                          |  |
|  | BB:22:22 -> Port2  (LEARNED!)              |  |
|  +--------------------------------------------+  |
|     Port1    Port2    Port3    Port4             |
+------+--------+--------+--------+----------------+
   <===|========|        |        |  FORWARD directly
      [A]      [B]      [C]      [D]
      dest     src

Switch knows A is on Port1, so it FORWARDS directly
스위치가 A가 Port1에 있음을 알아서 직접 FORWARD
```

### Switch Forwarding Logic (스위치 전달 로직)

```
+---------------------------------------------------------------+
|              Switch Frame Processing                          |
+---------------------------------------------------------------+
|                                                               |
|   Frame Received on Port X                                    |
|           |                                                   |
|           v                                                   |
|   +------------------+                                        |
|   | Learn: Update    |                                        |
|   | source MAC ->    |                                        |
|   | Port X in table  |                                        |
|   +------------------+                                        |
|           |                                                   |
|           v                                                   |
|   +------------------+                                        |
|   | Lookup: Is dest  |                                        |
|   | MAC in table?    |                                        |
|   +--------+---------+                                        |
|            |                                                  |
|     +------+------+                                           |
|     |             |                                           |
|    YES           NO                                           |
|     |             |                                           |
|     v             v                                           |
| +--------+   +----------+                                     |
| |Forward |   | Flood to |                                     |
| |to port |   | all ports|                                     |
| |in table|   | except X |                                     |
| +--------+   +----------+                                     |
|                                                               |
+---------------------------------------------------------------+
```

---

## 5. ARP (Address Resolution Protocol)

### What is ARP? (ARP란?)

ARP (Address Resolution Protocol) resolves **IP addresses to MAC addresses** within a local network. When a device wants to communicate with another device on the same network, it needs the destination's MAC address.

ARP(주소 결정 프로토콜)는 로컬 네트워크 내에서 **IP 주소를 MAC 주소로 변환**합니다. 장치가 같은 네트워크의 다른 장치와 통신하려면 목적지의 MAC 주소가 필요합니다.

### Why is ARP Needed? (ARP가 필요한 이유)

```
+---------------------------------------------------------------+
|                    The Problem                                 |
+---------------------------------------------------------------+
|                                                               |
|   Host A wants to send data to Host B                         |
|   A가 B에게 데이터를 보내려 함                                |
|                                                               |
|   A knows:                                                    |
|   - B's IP address: 192.168.1.20                              |
|   - B's IP 주소: 192.168.1.20                                 |
|                                                               |
|   A doesn't know:                                             |
|   - B's MAC address: ??:??:??:??:??:??                        |
|   - B의 MAC 주소: ??:??:??:??:??:??                           |
|                                                               |
|   But Ethernet needs MAC addresses to deliver frames!         |
|   하지만 이더넷은 프레임 전달에 MAC 주소가 필요!              |
|                                                               |
|   Solution: Use ARP to discover B's MAC address               |
|   해결책: ARP를 사용하여 B의 MAC 주소 발견                    |
|                                                               |
+---------------------------------------------------------------+
```

### ARP Operation (ARP 동작 과정)

```
Network: 192.168.1.0/24

+-------------+                                    +-------------+
|   Host A    |                                    |   Host B    |
| 192.168.1.10|                                    | 192.168.1.20|
| AA:AA:AA:AA |                                    | BB:BB:BB:BB |
+------+------+                                    +------+------+
       |                                                  |
       |  Step 1: ARP Request (Broadcast)                 |
       |  "Who has 192.168.1.20? Tell 192.168.1.10"       |
       |  Dest MAC: FF:FF:FF:FF:FF:FF                     |
       |=================================================>|
       |     (All hosts receive this broadcast)           |
       |                                                  |
       |  Step 2: ARP Reply (Unicast)                     |
       |  "192.168.1.20 is at BB:BB:BB:BB"                |
       |  Dest MAC: AA:AA:AA:AA                           |
       |<=================================================|
       |                                                  |
       |  Step 3: A caches the mapping                    |
       |  A가 매핑을 캐시에 저장                          |
       |  [192.168.1.20 -> BB:BB:BB:BB]                   |
       |                                                  |
```

### ARP Packet Format (ARP 패킷 형식)

```
+---------------------------------------------------------------+
|                    ARP Packet Structure                        |
+---------------------------------------------------------------+
|                                                               |
|  +------------------+------------------+                       |
|  | Hardware Type    | Protocol Type    |   (2 + 2 bytes)      |
|  +------------------+------------------+                       |
|  | HW Addr Len | Protocol Addr Len    |   (1 + 1 bytes)       |
|  +------------------+------------------+                       |
|  | Operation (1=Request, 2=Reply)     |   (2 bytes)           |
|  +------------------------------------+                       |
|  | Sender Hardware Address (MAC)      |   (6 bytes)           |
|  +------------------------------------+                       |
|  | Sender Protocol Address (IP)       |   (4 bytes)           |
|  +------------------------------------+                       |
|  | Target Hardware Address (MAC)      |   (6 bytes)           |
|  +------------------------------------+                       |
|  | Target Protocol Address (IP)       |   (4 bytes)           |
|  +------------------------------------+                       |
|                                                               |
+---------------------------------------------------------------+

Example ARP Request:
+------------------+--------------------------------------------+
| Field            | Value                                      |
+------------------+--------------------------------------------+
| Hardware Type    | 0x0001 (Ethernet)                          |
| Protocol Type    | 0x0800 (IPv4)                              |
| Operation        | 0x0001 (Request)                           |
| Sender MAC       | AA:AA:AA:AA:AA:AA                          |
| Sender IP        | 192.168.1.10                               |
| Target MAC       | 00:00:00:00:00:00 (unknown)                |
| Target IP        | 192.168.1.20                               |
+------------------+--------------------------------------------+
```

### ARP Cache (ARP 캐시)

```
+---------------------------------------------------------------+
|                    ARP Cache Table                             |
+---------------------------------------------------------------+
|                                                               |
|   $ arp -a                                                    |
|                                                               |
|   Interface: 192.168.1.10                                     |
|   +------------------+-----------------+---------+            |
|   | Internet Address | Physical Address| Type    |            |
|   +------------------+-----------------+---------+            |
|   | 192.168.1.1      | 00:11:22:33:44  | dynamic |            |
|   | 192.168.1.20     | BB:BB:BB:BB:BB  | dynamic |            |
|   | 192.168.1.30     | CC:CC:CC:CC:CC  | static  |            |
|   +------------------+-----------------+---------+            |
|                                                               |
|   - Dynamic entries expire (typically 2-20 minutes)           |
|   - 동적 항목은 만료됨 (보통 2-20분)                          |
|   - Static entries are manually configured                    |
|   - 정적 항목은 수동 설정                                     |
|                                                               |
+---------------------------------------------------------------+
```

### ARP Security Concerns (ARP 보안 문제)

```
+---------------------------------------------------------------+
|                    ARP Spoofing Attack                         |
+---------------------------------------------------------------+
|                                                               |
|        +--------+        +----------+        +--------+       |
|        | Victim |        | Attacker |        | Gateway|       |
|        |  .10   |        |   .99    |        |   .1   |       |
|        +---+----+        +----+-----+        +---+----+       |
|            |                  |                  |            |
|            |   Fake ARP Reply |                  |            |
|            |  ".1 is at       |                  |            |
|            |   attacker MAC"  |                  |            |
|            |<-----------------+                  |            |
|            |                  |                  |            |
|            |   Traffic meant for .1              |            |
|            +----------------->|                  |            |
|            |     goes to attacker instead!       |            |
|            |                  |                  |            |
|                                                               |
|   Defense: Use static ARP, ARP inspection, 802.1X             |
|   방어: 정적 ARP, ARP 검사, 802.1X 사용                       |
|                                                               |
+---------------------------------------------------------------+
```

---

## 6. Hub vs Switch vs Router

### Device Comparison Overview (장치 비교 개요)

```
+---------------------------------------------------------------+
|              Network Devices at Different Layers               |
+---------------------------------------------------------------+
|                                                               |
|   Layer 3 (Network)      +----------+                         |
|   네트워크 계층          | ROUTER   |  IP addresses           |
|                          +----------+                         |
|                               |                               |
|   Layer 2 (Data Link)    +----------+                         |
|   데이터 링크 계층       | SWITCH   |  MAC addresses          |
|                          +----------+                         |
|                               |                               |
|   Layer 1 (Physical)     +----------+                         |
|   물리 계층              |   HUB    |  Electrical signals     |
|                          +----------+                         |
|                                                               |
+---------------------------------------------------------------+
```

### Hub (허브)

```
+---------------------------------------------------------------+
|                         HUB                                    |
+---------------------------------------------------------------+
|                                                               |
|   - Layer 1 device (physical layer)                           |
|   - 1계층 장치 (물리 계층)                                    |
|                                                               |
|   - Simply repeats signals to ALL ports                       |
|   - 단순히 모든 포트로 신호 반복                              |
|                                                               |
|   - Creates a single collision domain                         |
|   - 하나의 충돌 도메인 생성                                   |
|                                                               |
|           +-------+                                           |
|     A --->|       |---> B (receives)                          |
|           |  HUB  |---> C (receives)                          |
|           |       |---> D (receives)                          |
|           +-------+                                           |
|                                                               |
|   A sends to B, but C and D also receive it!                  |
|   A가 B에게 보내도 C와 D도 수신함!                            |
|                                                               |
|   Problems:                                                   |
|   - Wastes bandwidth (낭비되는 대역폭)                        |
|   - Collisions (충돌)                                         |
|   - No privacy (프라이버시 없음)                              |
|                                                               |
+---------------------------------------------------------------+
```

### Switch (스위치)

```
+---------------------------------------------------------------+
|                        SWITCH                                  |
+---------------------------------------------------------------+
|                                                               |
|   - Layer 2 device (data link layer)                          |
|   - 2계층 장치 (데이터 링크 계층)                             |
|                                                               |
|   - Learns MAC addresses and forwards intelligently           |
|   - MAC 주소 학습 후 지능적으로 전달                          |
|                                                               |
|   - Each port is a separate collision domain                  |
|   - 각 포트가 별도의 충돌 도메인                              |
|                                                               |
|           +--------+                                          |
|     A --->| MAC    |---> B (only B receives)                  |
|           | Table  |     C (nothing)                          |
|           |        |     D (nothing)                          |
|           +--------+                                          |
|                                                               |
|   A sends to B, only B receives it                            |
|   A가 B에게 보내면 B만 수신                                   |
|                                                               |
|   Benefits:                                                   |
|   - Full bandwidth per port (포트당 전체 대역폭)              |
|   - No collisions between ports (포트 간 충돌 없음)           |
|   - Better security (향상된 보안)                             |
|                                                               |
+---------------------------------------------------------------+
```

### Router (라우터)

```
+---------------------------------------------------------------+
|                        ROUTER                                  |
+---------------------------------------------------------------+
|                                                               |
|   - Layer 3 device (network layer)                            |
|   - 3계층 장치 (네트워크 계층)                                |
|                                                               |
|   - Routes packets between different networks                 |
|   - 서로 다른 네트워크 간 패킷 라우팅                         |
|                                                               |
|   - Uses IP addresses for forwarding decisions                |
|   - IP 주소로 전달 결정                                       |
|                                                               |
|                                                               |
|   Network A               +--------+            Network B     |
|   192.168.1.0/24  <------>| ROUTER |<------> 10.0.0.0/24      |
|                           +--------+                          |
|                                                               |
|   - Separates broadcast domains                               |
|   - 브로드캐스트 도메인 분리                                  |
|                                                               |
|   - Provides NAT, firewall, WAN connectivity                  |
|   - NAT, 방화벽, WAN 연결 제공                                |
|                                                               |
+---------------------------------------------------------------+
```

### Detailed Comparison Table (상세 비교표)

```
+------------------+-------------------+-------------------+-------------------+
| Feature          | Hub               | Switch            | Router            |
|------------------|-------------------|-------------------|-------------------|
| OSI Layer        | Layer 1           | Layer 2           | Layer 3           |
| OSI 계층         | 1계층             | 2계층             | 3계층             |
+------------------+-------------------+-------------------+-------------------+
| Address Used     | None              | MAC Address       | IP Address        |
| 사용 주소        | 없음              | MAC 주소          | IP 주소           |
+------------------+-------------------+-------------------+-------------------+
| Intelligence     | None              | Learns MACs       | Routing tables    |
| 지능             | 없음              | MAC 학습          | 라우팅 테이블     |
+------------------+-------------------+-------------------+-------------------+
| Broadcast        | Forwards all      | Forwards all      | Blocks            |
| 브로드캐스트     | 모두 전달         | 모두 전달         | 차단              |
+------------------+-------------------+-------------------+-------------------+
| Collision Domain | Single (all ports)| Per port          | Per port          |
| 충돌 도메인      | 하나 (모든 포트)  | 포트별            | 포트별            |
+------------------+-------------------+-------------------+-------------------+
| Broadcast Domain | Single            | Single            | Per interface     |
| 브로드캐스트     | 하나              | 하나              | 인터페이스별      |
| 도메인           |                   |                   |                   |
+------------------+-------------------+-------------------+-------------------+
| Network Scope    | Same LAN          | Same LAN          | Between LANs      |
| 네트워크 범위    | 동일 LAN          | 동일 LAN          | LAN 간            |
+------------------+-------------------+-------------------+-------------------+
| Price            | Cheap             | Moderate          | Expensive         |
| 가격             | 저렴              | 중간              | 비쌈              |
+------------------+-------------------+-------------------+-------------------+
| Speed            | Shared            | Full per port     | Variable          |
| 속도             | 공유              | 포트당 전체       | 가변              |
+------------------+-------------------+-------------------+-------------------+
| Use Case         | Legacy/Testing    | LAN connectivity  | Internet/WAN      |
| 용도             | 레거시/테스트     | LAN 연결          | 인터넷/WAN        |
+------------------+-------------------+-------------------+-------------------+
```

### Collision Domain vs Broadcast Domain (충돌 도메인 vs 브로드캐스트 도메인)

```
+-----------------------------------------------------------------------+
|                          Hub Network                                   |
+-----------------------------------------------------------------------+
|                                                                       |
|     [A]----+                                                          |
|            |                                                          |
|     [B]----+----[HUB]----+----[C]                                     |
|            |             |                                            |
|     [D]----+             +----[E]                                     |
|                                                                       |
|     Collision Domain: 1 (all 5 hosts)                                 |
|     충돌 도메인: 1개 (모든 5개 호스트)                                |
|                                                                       |
|     Broadcast Domain: 1                                               |
|     브로드캐스트 도메인: 1개                                          |
|                                                                       |
+-----------------------------------------------------------------------+

+-----------------------------------------------------------------------+
|                        Switch Network                                  |
+-----------------------------------------------------------------------+
|                                                                       |
|     [A]----+                                                          |
|            |                                                          |
|     [B]----+--[SWITCH]---+----[C]                                     |
|            |             |                                            |
|     [D]----+             +----[E]                                     |
|                                                                       |
|     Collision Domains: 5 (one per port)                               |
|     충돌 도메인: 5개 (포트당 1개)                                     |
|                                                                       |
|     Broadcast Domain: 1                                               |
|     브로드캐스트 도메인: 1개                                          |
|                                                                       |
+-----------------------------------------------------------------------+

+-----------------------------------------------------------------------+
|                        Router Network                                  |
+-----------------------------------------------------------------------+
|                                                                       |
|     Network A                        Network B                        |
|     [A]----+                         +----[E]                         |
|            |                         |                                |
|     [B]--[SW1]---[ROUTER]---[SW2]----+----[F]                         |
|            |                         |                                |
|     [C]----+                         +----[G]                         |
|                                                                       |
|     Collision Domains: 6 (one per switch port)                        |
|     충돌 도메인: 6개 (스위치 포트당 1개)                              |
|                                                                       |
|     Broadcast Domains: 2 (Network A and Network B)                    |
|     브로드캐스트 도메인: 2개 (네트워크 A와 B)                         |
|                                                                       |
+-----------------------------------------------------------------------+
```

---

## 7. Summary (요약)

### Key Concepts (핵심 개념)

```
+-----------------------------------------------------------------------+
|                     Chapter 12 Summary                                 |
+-----------------------------------------------------------------------+
|                                                                       |
|  1. Link Layer (링크 계층)                                            |
|     - Layer 2 of OSI model                                            |
|     - Handles node-to-node communication                              |
|     - Uses MAC addresses                                              |
|     - Framing, error detection, media access control                  |
|                                                                       |
|  2. MAC Address (MAC 주소)                                            |
|     - 48-bit hardware identifier                                      |
|     - Format: AA:BB:CC:DD:EE:FF                                       |
|     - OUI (manufacturer) + Device ID                                  |
|     - Local scope only                                                |
|                                                                       |
|  3. Ethernet Frame (이더넷 프레임)                                    |
|     - Preamble + SFD + Dest MAC + Src MAC + Type + Data + FCS         |
|     - 64-1518 bytes                                                   |
|     - FCS uses CRC-32 for error detection                             |
|                                                                       |
|  4. Switch (스위치)                                                   |
|     - Layer 2 device                                                  |
|     - Learns MAC addresses                                            |
|     - Forwards frames intelligently                                   |
|     - Creates collision domain per port                               |
|                                                                       |
|  5. ARP (주소 결정 프로토콜)                                          |
|     - Resolves IP to MAC address                                      |
|     - Request (broadcast) + Reply (unicast)                           |
|     - Cached for efficiency                                           |
|     - Vulnerable to spoofing attacks                                  |
|                                                                       |
|  6. Hub vs Switch vs Router                                           |
|     - Hub: L1, broadcasts all, single collision domain                |
|     - Switch: L2, forwards by MAC, collision domain per port          |
|     - Router: L3, routes by IP, separates broadcast domains           |
|                                                                       |
+-----------------------------------------------------------------------+
```

### Quick Reference (빠른 참조)

| Topic | Key Point |
|-------|-----------|
| MAC Address | 48-bit, manufacturer assigned, local scope |
| Ethernet Frame | 64-1518 bytes, includes FCS for error check |
| Switch Learning | Source MAC -> Port mapping |
| ARP | IP to MAC resolution via broadcast request |
| Hub | L1, repeater, creates collisions |
| Switch | L2, intelligent forwarding |
| Router | L3, routes between networks |

---

*End of Chapter 12*
