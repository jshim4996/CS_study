# Chapter 09. IP (Internet Protocol)

## Overview (개요)

IP (Internet Protocol) is the principal communications protocol in the Internet Protocol Suite for routing packets across network boundaries. It provides addressing and routing functions that enable internetworking.

IP(인터넷 프로토콜)는 네트워크 경계를 넘어 패킷을 라우팅하기 위한 인터넷 프로토콜 스위트의 주요 통신 프로토콜입니다. 인터넷워킹을 가능하게 하는 주소 지정 및 라우팅 기능을 제공합니다.

---

## 1. IP Address Concept (IP 주소 개념)

### 1.1 What is an IP Address? (IP 주소란?)

An IP address is a numerical label assigned to each device connected to a computer network that uses the Internet Protocol for communication.

IP 주소는 통신을 위해 인터넷 프로토콜을 사용하는 컴퓨터 네트워크에 연결된 각 장치에 할당된 숫자 레이블입니다.

```
IP Address Functions (IP 주소 기능):

1. Identification (식별)
   - Uniquely identifies a device on the network
   - 네트워크에서 장치를 고유하게 식별

2. Location Addressing (위치 주소 지정)
   - Provides the location of the device in the network topology
   - 네트워크 토폴로지에서 장치의 위치를 제공

          Internet
             |
    +--------+--------+
    |                 |
+-------+         +-------+
|Network|         |Network|
|  A    |         |  B    |
+-------+         +-------+
    |                 |
192.168.1.x      10.0.0.x
```

### 1.2 IP Address Structure (IP 주소 구조)

```
IPv4 Address: 32 bits (4 bytes)
(IPv4 주소: 32비트 (4바이트))

Binary:
11000000.10101000.00000001.00001010

Decimal (Dotted Notation):
192      .168      .1        .10

+----------+----------+
| Network  |   Host   |
|  Part    |   Part   |
+----------+----------+
    |            |
Identifies   Identifies
the network  the device

네트워크     장치를
식별         식별
```

---

## 2. IPv4 Address System (IPv4 주소 체계)

### 2.1 Address Classes (주소 클래스)

Traditional IPv4 addressing uses five classes (A through E).

전통적인 IPv4 주소 지정은 5개의 클래스(A부터 E)를 사용합니다.

```
+-------+-----------------+------------------+------------------+
| Class | First Octet     | Network/Host     | Address Range    |
| (클래스)| (첫 번째 옥텟)  | (네트워크/호스트)| (주소 범위)      |
+-------+-----------------+------------------+------------------+
|   A   | 0xxxxxxx        | N.H.H.H          | 0.0.0.0 -        |
|       | (0-127)         | 8/24 bits        | 127.255.255.255  |
+-------+-----------------+------------------+------------------+
|   B   | 10xxxxxx        | N.N.H.H          | 128.0.0.0 -      |
|       | (128-191)       | 16/16 bits       | 191.255.255.255  |
+-------+-----------------+------------------+------------------+
|   C   | 110xxxxx        | N.N.N.H          | 192.0.0.0 -      |
|       | (192-223)       | 24/8 bits        | 223.255.255.255  |
+-------+-----------------+------------------+------------------+
|   D   | 1110xxxx        | Multicast        | 224.0.0.0 -      |
|       | (224-239)       | (멀티캐스트)     | 239.255.255.255  |
+-------+-----------------+------------------+------------------+
|   E   | 1111xxxx        | Reserved         | 240.0.0.0 -      |
|       | (240-255)       | (예약됨)         | 255.255.255.255  |
+-------+-----------------+------------------+------------------+
```

### 2.2 Class Visualization (클래스 시각화)

```
Class A: Large Networks (대형 네트워크)
+--------+------------------------+
|Network |         Host           |
| 8 bits |        24 bits         |
+--------+------------------------+
   |
   +--> 126 networks, 16,777,214 hosts each

Class B: Medium Networks (중형 네트워크)
+----------------+----------------+
|    Network     |      Host      |
|    16 bits     |    16 bits     |
+----------------+----------------+
   |
   +--> 16,384 networks, 65,534 hosts each

Class C: Small Networks (소형 네트워크)
+------------------------+--------+
|        Network         |  Host  |
|        24 bits         | 8 bits |
+------------------------+--------+
   |
   +--> 2,097,152 networks, 254 hosts each
```

### 2.3 Special Addresses (특수 주소)

```
+------------------+------------------+--------------------------------+
| Address          | Name             | Description (설명)             |
+------------------+------------------+--------------------------------+
| 0.0.0.0          | This Network     | Current network                |
|                  | (이 네트워크)    | 현재 네트워크                  |
+------------------+------------------+--------------------------------+
| 127.0.0.1        | Loopback         | Local host testing             |
|                  | (루프백)         | 로컬 호스트 테스트             |
+------------------+------------------+--------------------------------+
| 255.255.255.255  | Limited          | Broadcast to current network   |
|                  | Broadcast        | 현재 네트워크로 브로드캐스트   |
|                  | (제한 브로드캐스트)|                              |
+------------------+------------------+--------------------------------+
| x.x.x.0          | Network Address  | Identifies the network         |
|                  | (네트워크 주소)  | 네트워크 식별                  |
+------------------+------------------+--------------------------------+
| x.x.x.255        | Broadcast        | Broadcast to network x.x.x     |
|                  | (브로드캐스트)   | x.x.x 네트워크로 브로드캐스트  |
+------------------+------------------+--------------------------------+

Loopback Address Example (루프백 주소 예):

    Application
         |
         v
    +----------+
    |   TCP    |
    +----------+
         |
         v
    +----------+
    |    IP    |-----> 127.0.0.1 (Loopback)
    +----------+              |
         ^                    |
         +--------------------+
         (Data returns to same host)
         (데이터가 같은 호스트로 돌아옴)
```

---

## 3. Subnets and Subnet Masks (서브넷과 서브넷 마스크)

### 3.1 Subnet Concept (서브넷 개념)

Subnetting divides a large network into smaller, more manageable segments.

서브네팅은 큰 네트워크를 더 작고 관리하기 쉬운 세그먼트로 나눕니다.

```
Without Subnetting:
(서브네팅 없이:)

       Organization Network: 192.168.0.0/16
       +------------------------------------------+
       |                                          |
       |   All 65,534 hosts in one network       |
       |   (모든 65,534개의 호스트가 하나의      |
       |    네트워크에)                           |
       +------------------------------------------+

With Subnetting:
(서브네팅으로:)

       Organization Network: 192.168.0.0/16
       +-------------+-------------+-------------+
       |  Subnet 1   |  Subnet 2   |  Subnet 3   |
       | 192.168.1.0 | 192.168.2.0 | 192.168.3.0 |
       |   /24       |   /24       |   /24       |
       | (254 hosts) | (254 hosts) | (254 hosts) |
       +-------------+-------------+-------------+
```

### 3.2 Subnet Mask (서브넷 마스크)

The subnet mask defines which part of the IP address is the network and which is the host.

서브넷 마스크는 IP 주소의 어느 부분이 네트워크이고 어느 부분이 호스트인지 정의합니다.

```
IP Address:    192.168.1.100
Subnet Mask:   255.255.255.0

Binary Representation:
(2진수 표현:)

IP Address:    11000000.10101000.00000001.01100100
Subnet Mask:   11111111.11111111.11111111.00000000
               |<------- Network ------>|<-Host->|

AND Operation to find Network Address:
(네트워크 주소를 찾기 위한 AND 연산:)

IP Address:    11000000.10101000.00000001.01100100
Subnet Mask:   11111111.11111111.11111111.00000000
               ----------------------------------------
Network:       11000000.10101000.00000001.00000000
               = 192.168.1.0
```

### 3.3 Common Subnet Masks (일반적인 서브넷 마스크)

```
+------------------+------------------+-------------+-------------+
| Subnet Mask      | CIDR Notation    | Networks    | Hosts       |
| (서브넷 마스크)  | (CIDR 표기법)    | (네트워크)  | (호스트)    |
+------------------+------------------+-------------+-------------+
| 255.0.0.0        | /8               | 1           | 16,777,214  |
| 255.255.0.0      | /16              | 1           | 65,534      |
| 255.255.255.0    | /24              | 1           | 254         |
| 255.255.255.128  | /25              | 2           | 126 each    |
| 255.255.255.192  | /26              | 4           | 62 each     |
| 255.255.255.224  | /27              | 8           | 30 each     |
| 255.255.255.240  | /28              | 16          | 14 each     |
| 255.255.255.248  | /29              | 32          | 6 each      |
| 255.255.255.252  | /30              | 64          | 2 each      |
+------------------+------------------+-------------+-------------+

Note: Host count = 2^(host bits) - 2
      (Subtract 2 for network and broadcast addresses)
참고: 호스트 수 = 2^(호스트 비트) - 2
      (네트워크 및 브로드캐스트 주소를 위해 2를 뺌)
```

---

## 4. CIDR Notation (CIDR 표기법)

### 4.1 CIDR Overview (CIDR 개요)

CIDR (Classless Inter-Domain Routing) replaces the old class-based system with a more flexible approach.

CIDR(Classless Inter-Domain Routing)은 이전의 클래스 기반 시스템을 더 유연한 방식으로 대체합니다.

```
CIDR Notation: IP_Address/Prefix_Length
(CIDR 표기법: IP_주소/접두사_길이)

Example: 192.168.1.0/24

        192.168.1.0/24
             |      |
             |      +-- Prefix length (24 bits for network)
             |          (접두사 길이 (네트워크용 24비트))
             |
             +-- Network address
                 (네트워크 주소)
```

### 4.2 CIDR Benefits (CIDR의 이점)

```
Classful (Old Way):
(클래스 기반 (이전 방식):)

Organization needs 500 hosts:
- Class C (254 hosts): Too small
- Class B (65,534 hosts): Wastes 65,034 addresses!
조직에 500개의 호스트가 필요:
- 클래스 C (254 호스트): 너무 작음
- 클래스 B (65,534 호스트): 65,034개의 주소 낭비!

CIDR (New Way):
(CIDR (새로운 방식):)

Organization needs 500 hosts:
- Use /23 (512 addresses, 510 usable hosts)
- Only 10 addresses wasted!
조직에 500개의 호스트가 필요:
- /23 사용 (512개 주소, 510개 사용 가능한 호스트)
- 단 10개의 주소만 낭비!

+-------+----------+-----------------+
| /23   | 2 bits   | 512 addresses   |
|       | for host | (510 hosts)     |
+-------+----------+-----------------+
```

### 4.3 CIDR Calculation Examples (CIDR 계산 예)

```
Example 1: 192.168.10.50/28

Step 1: Convert /28 to subnet mask
        28 bits = 11111111.11111111.11111111.11110000
                = 255.255.255.240

Step 2: Find network address
        192.168.10.50  = 11000000.10101000.00001010.00110010
        255.255.255.240= 11111111.11111111.11111111.11110000
        AND            = 11000000.10101000.00001010.00110000
                       = 192.168.10.48

Step 3: Calculate host range
        Network:   192.168.10.48
        First host: 192.168.10.49
        Last host:  192.168.10.62
        Broadcast:  192.168.10.63

        Usable hosts: 2^4 - 2 = 14

+-------------------+-------------------+
| Network Address   | 192.168.10.48     |
| (네트워크 주소)   |                   |
+-------------------+-------------------+
| First Usable      | 192.168.10.49     |
| (첫 번째 사용 가능)|                  |
+-------------------+-------------------+
| Last Usable       | 192.168.10.62     |
| (마지막 사용 가능) |                  |
+-------------------+-------------------+
| Broadcast         | 192.168.10.63     |
| (브로드캐스트)    |                   |
+-------------------+-------------------+
```

---

## 5. Public vs Private IP (공인 IP vs 사설 IP)

### 5.1 Private IP Address Ranges (사설 IP 주소 범위)

```
RFC 1918 Private Address Ranges:
(RFC 1918 사설 주소 범위:)

+---------------+----------------------+-------------------+
| Class         | Range                | CIDR              |
| (클래스)      | (범위)               | (CIDR)            |
+---------------+----------------------+-------------------+
| A             | 10.0.0.0 -           | 10.0.0.0/8        |
|               | 10.255.255.255       |                   |
+---------------+----------------------+-------------------+
| B             | 172.16.0.0 -         | 172.16.0.0/12     |
|               | 172.31.255.255       |                   |
+---------------+----------------------+-------------------+
| C             | 192.168.0.0 -        | 192.168.0.0/16    |
|               | 192.168.255.255      |                   |
+---------------+----------------------+-------------------+
```

### 5.2 Public vs Private Comparison (공인 vs 사설 비교)

```
                      Internet
                         |
                    +---------+
                    | Router  | Public IP: 203.0.113.5
                    |  (NAT)  |
                    +---------+
                         |
         +---------------+---------------+
         |               |               |
     +-------+       +-------+       +-------+
     | PC 1  |       | PC 2  |       | PC 3  |
     +-------+       +-------+       +-------+
   192.168.1.10    192.168.1.11    192.168.1.12

   (Private IPs - 사설 IP)

+-------------------+-----------------------------------+
| Public IP         | Private IP                        |
| (공인 IP)         | (사설 IP)                         |
+-------------------+-----------------------------------+
| Globally unique   | Can be reused in different        |
| (전역적으로 고유) | networks (다른 네트워크에서       |
|                   | 재사용 가능)                      |
+-------------------+-----------------------------------+
| Routable on       | Not routable on Internet          |
| Internet          | (인터넷에서 라우팅 불가)          |
| (인터넷 라우팅 가능)|                                 |
+-------------------+-----------------------------------+
| Assigned by ISP   | Assigned by network admin         |
| (ISP가 할당)      | (네트워크 관리자가 할당)          |
+-------------------+-----------------------------------+
| Limited supply    | Unlimited (in private space)      |
| (제한된 공급)     | (사설 공간에서 무제한)            |
+-------------------+-----------------------------------+
```

---

## 6. IPv6 Overview (IPv6 개요)

### 6.1 Why IPv6? (왜 IPv6인가?)

```
IPv4 Limitations (IPv4의 한계):
- Only 2^32 = ~4.3 billion addresses
- 2^32 = ~43억 개의 주소만 가능
- Address exhaustion (IANA exhausted in 2011)
- 주소 고갈 (IANA가 2011년에 고갈)

IPv6 Solution (IPv6 해결책):
- 2^128 = ~340 undecillion addresses
- 2^128 = ~340간 개의 주소
- Enough for every grain of sand on Earth!
- 지구의 모든 모래알에 할당해도 충분!
```

### 6.2 IPv6 Address Format (IPv6 주소 형식)

```
IPv6: 128 bits (16 bytes)
(IPv6: 128비트 (16바이트))

Format: Eight groups of four hexadecimal digits
(형식: 4개의 16진수 숫자로 된 8개의 그룹)

Full notation:
2001:0db8:0000:0000:0000:ff00:0042:8329

Simplified rules:
(간소화 규칙:)

1. Remove leading zeros in each group
   (각 그룹의 선행 0 제거)
   2001:db8:0:0:0:ff00:42:8329

2. Replace consecutive zero groups with ::
   (연속된 0 그룹을 ::로 대체)
   2001:db8::ff00:42:8329

   Note: Can only use :: once per address
   (참고: 주소당 ::는 한 번만 사용 가능)
```

### 6.3 IPv4 vs IPv6 Comparison (IPv4 vs IPv6 비교)

```
+-------------------+----------------------+----------------------+
| Feature           | IPv4                 | IPv6                 |
| (특징)            | (IPv4)               | (IPv6)               |
+-------------------+----------------------+----------------------+
| Address Length    | 32 bits              | 128 bits             |
| (주소 길이)       |                      |                      |
+-------------------+----------------------+----------------------+
| Notation          | Decimal              | Hexadecimal          |
| (표기법)          | 192.168.1.1          | 2001:db8::1          |
+-------------------+----------------------+----------------------+
| Address Space     | ~4.3 billion         | ~340 undecillion     |
| (주소 공간)       | (~43억)              | (~340간)             |
+-------------------+----------------------+----------------------+
| Header Size       | 20-60 bytes          | 40 bytes (fixed)     |
| (헤더 크기)       |                      | (고정)               |
+-------------------+----------------------+----------------------+
| Checksum          | Yes                  | No (removed)         |
| (체크섬)          | (있음)               | (제거됨)             |
+-------------------+----------------------+----------------------+
| Configuration     | Manual/DHCP          | Auto-config/DHCPv6   |
| (구성)            | (수동/DHCP)          | (자동 구성/DHCPv6)   |
+-------------------+----------------------+----------------------+
```

### 6.4 IPv6 Address Types (IPv6 주소 유형)

```
+-------------------+----------------------+------------------------+
| Type              | Prefix               | Description            |
| (유형)            | (접두사)             | (설명)                 |
+-------------------+----------------------+------------------------+
| Global Unicast    | 2000::/3             | Public addresses       |
| (글로벌 유니캐스트)|                     | (공인 주소)            |
+-------------------+----------------------+------------------------+
| Link-Local        | fe80::/10            | Local network only     |
| (링크 로컬)       |                      | (로컬 네트워크 전용)   |
+-------------------+----------------------+------------------------+
| Unique Local      | fc00::/7             | Private addresses      |
| (고유 로컬)       |                      | (사설 주소)            |
+-------------------+----------------------+------------------------+
| Multicast         | ff00::/8             | One-to-many            |
| (멀티캐스트)      |                      | (일대다)               |
+-------------------+----------------------+------------------------+
| Loopback          | ::1                  | Localhost              |
| (루프백)          |                      | (로컬호스트)           |
+-------------------+----------------------+------------------------+
```

---

## Summary (요약)

### Key Points (핵심 포인트)

1. **IP Address (IP 주소)**
   - Unique identifier for devices on a network
   - Two parts: Network and Host portions
   - 네트워크 상 장치의 고유 식별자
   - 두 부분: 네트워크와 호스트 부분

2. **IPv4 Classes (IPv4 클래스)**
   - Class A (1-126): Large networks
   - Class B (128-191): Medium networks
   - Class C (192-223): Small networks
   - Classes D and E: Special purposes

3. **Subnetting (서브네팅)**
   - Divides networks into smaller segments
   - Subnet mask defines network/host boundary
   - 네트워크를 더 작은 세그먼트로 분할
   - 서브넷 마스크가 네트워크/호스트 경계를 정의

4. **CIDR (CIDR)**
   - Flexible addressing without class restrictions
   - Format: IP/prefix (e.g., 192.168.1.0/24)
   - 클래스 제한 없는 유연한 주소 지정
   - 형식: IP/접두사 (예: 192.168.1.0/24)

5. **Public vs Private (공인 vs 사설)**
   - Public: Internet routable, globally unique
   - Private: Internal use, requires NAT for Internet
   - 공인: 인터넷 라우팅 가능, 전역적으로 고유
   - 사설: 내부 사용, 인터넷 접속에 NAT 필요

6. **IPv6 (IPv6)**
   - 128-bit addresses for virtually unlimited space
   - Solves IPv4 exhaustion problem
   - 128비트 주소로 사실상 무제한 공간
   - IPv4 고갈 문제 해결

```
+------------------+------------------------------------------+
| Concept          | Key Formula/Rule                         |
| (개념)           | (핵심 공식/규칙)                         |
+------------------+------------------------------------------+
| Usable Hosts     | 2^(host bits) - 2                        |
| (사용 가능 호스트)| 2^(호스트 비트) - 2                      |
+------------------+------------------------------------------+
| Network Address  | IP AND Subnet Mask                       |
| (네트워크 주소)  | IP AND 서브넷 마스크                     |
+------------------+------------------------------------------+
| Broadcast        | Network Address + All Host Bits = 1      |
| (브로드캐스트)   | 네트워크 주소 + 모든 호스트 비트 = 1     |
+------------------+------------------------------------------+
| CIDR /n          | n bits for network, (32-n) for host      |
| (CIDR /n)        | n비트는 네트워크, (32-n)은 호스트        |
+------------------+------------------------------------------+
```
