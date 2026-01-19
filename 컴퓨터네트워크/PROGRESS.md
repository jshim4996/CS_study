# Computer Networks Study Plan (컴퓨터 네트워크 학습 계획)

> Goal: Understand how web communication works and diagnose network issues
> 목표: 웹 통신의 동작 원리를 이해하고, 네트워크 문제를 진단할 수 있는 수준

---

## AI Study Guide (AI 학습 가이드)

### Role (역할)
- AI is a **senior developer and professor** teaching this curriculum
- AI는 이 커리큘럼으로 학습을 시켜주는 **시니어 개발자이자 교수**

### Language Rule (언어 규칙)
- **Default: Explain in English (기본: 영어로 설명)**
- When user says "번역" or "translate" → Provide Korean translation

### Study Flow (학습 진행 흐름) - 5 Steps

**Step 1: Check Progress** - "What's next?"
**Step 2: Explanation** - "Explain this" → Read chapter .md
**Step 3: Example** - "Show example" → Check examples/ folder
**Step 4: Assignment** - "Give me an assignment"
**Step 5: Evaluation** - [Submit answer] → Pass = [x] check

### Evaluation Criteria (평가 기준)
□ Did you understand and apply the concept? (학습한 개념을 이해하고 적용했는가)

---

## STEP 1. Network Fundamentals (네트워크 기초)

### Chapter 01. Introduction to Networks (네트워크 개요)
> **doc**: `chapters/chapter01/chapter01.md`
> **examples**: `chapters/chapter01/examples/`

- [ ] What is a Network (네트워크란 무엇인가)
- [ ] Internet Architecture (인터넷 구조)
- [ ] Protocol Concept (프로토콜 개념)
- [ ] OSI 7-Layer Model (OSI 7계층 모델)
- [ ] TCP/IP 4-Layer Model (TCP/IP 4계층 모델)
- [ ] OSI vs TCP/IP Comparison (OSI vs TCP/IP 비교)
- [ ] Encapsulation and Decapsulation (캡슐화와 역캡슐화)

---

## STEP 2. Application Layer (애플리케이션 계층)

### Chapter 02. Web and HTTP (웹과 HTTP)
> **doc**: `chapters/chapter02/chapter02.md`
> **examples**: `chapters/chapter02/examples/`

- [ ] Client-Server Model (클라이언트-서버 모델)
- [ ] HTTP Concept and Characteristics (HTTP 개념과 특징)
- [ ] HTTP Message Structure (HTTP 메시지 구조)
- [ ] HTTP Methods: GET, POST, PUT, DELETE (HTTP 메서드)
- [ ] Status Codes: 2xx, 3xx, 4xx, 5xx (상태 코드)
- [ ] HTTP Headers (HTTP 헤더)
- [ ] Cookies and Sessions (쿠키와 세션)
- [ ] REST API Concept (REST API 개념)

---

### Chapter 03. DNS
> **doc**: `chapters/chapter03/chapter03.md`
> **examples**: `chapters/chapter03/examples/`

- [ ] DNS Concept and Purpose (DNS 개념과 필요성)
- [ ] Domain Hierarchy (도메인 계층 구조)
- [ ] DNS Query Process (DNS 질의 과정)
- [ ] DNS Record Types: A, AAAA, CNAME, MX (DNS 레코드 타입)
- [ ] DNS Caching (DNS 캐싱)
- [ ] hosts File (hosts 파일)

---

### Chapter 04. Other Application Protocols (기타 애플리케이션 프로토콜)
> **doc**: `chapters/chapter04/chapter04.md`
> **examples**: `chapters/chapter04/examples/`

- [ ] FTP
- [ ] SMTP, POP3, IMAP (Email)
- [ ] SSH

---

## STEP 3. Transport Layer (전송 계층)

### Chapter 05. Transport Layer Overview (전송 계층 개요)
> **doc**: `chapters/chapter05/chapter05.md`
> **examples**: `chapters/chapter05/examples/`

- [ ] Transport Layer Role (전송 계층 역할)
- [ ] Port Numbers (포트 번호)
- [ ] Multiplexing and Demultiplexing (다중화와 역다중화)
- [ ] Well-known Ports: 80, 443, 22 (잘 알려진 포트)

---

### Chapter 06. UDP
> **doc**: `chapters/chapter06/chapter06.md`
> **examples**: `chapters/chapter06/examples/`

- [ ] UDP Characteristics (UDP 특징)
- [ ] UDP Header Structure (UDP 헤더 구조)
- [ ] UDP Use Cases (UDP 사용 사례)
- [ ] UDP vs TCP: When to Choose (UDP vs TCP 선택 기준)

---

### Chapter 07. TCP Basics (TCP 기초)
> **doc**: `chapters/chapter07/chapter07.md`
> **examples**: `chapters/chapter07/examples/`

- [ ] TCP Characteristics (TCP 특징)
- [ ] TCP Header Structure (TCP 헤더 구조)
- [ ] 3-way Handshake (연결 설정)
- [ ] 4-way Handshake (연결 종료)
- [ ] TCP State Transitions (TCP 상태 전이)

---

### Chapter 08. TCP Reliability (TCP 신뢰성)
> **doc**: `chapters/chapter08/chapter08.md`
> **examples**: `chapters/chapter08/examples/`

- [ ] Sequence Numbers and ACK (시퀀스 번호와 ACK)
- [ ] Retransmission (재전송)
- [ ] Sliding Window (슬라이딩 윈도우)
- [ ] Flow Control (흐름 제어)
- [ ] Congestion Control Concept (혼잡 제어 개념)
- [ ] Slow Start, AIMD (슬로우 스타트, AIMD)

---

## STEP 4. Network Layer (네트워크 계층)

### Chapter 09. IP
> **doc**: `chapters/chapter09/chapter09.md`
> **examples**: `chapters/chapter09/examples/`

- [ ] IP Address Concept (IP 주소 개념)
- [ ] IPv4 Address System (IPv4 주소 체계)
- [ ] Subnets and Subnet Masks (서브넷과 서브넷 마스크)
- [ ] CIDR Notation (CIDR 표기법)
- [ ] Public IP vs Private IP (공인 IP vs 사설 IP)
- [ ] IPv6 Overview (IPv6 개요)

---

### Chapter 10. IP Packets and Routing (IP 패킷과 라우팅)
> **doc**: `chapters/chapter10/chapter10.md`
> **examples**: `chapters/chapter10/examples/`

- [ ] IP Packet Structure (IP 패킷 구조)
- [ ] TTL - Time To Live
- [ ] Fragmentation and Reassembly (단편화와 재조립)
- [ ] Routing Concept (라우팅 개념)
- [ ] Routing Table (라우팅 테이블)
- [ ] Static vs Dynamic Routing (정적 vs 동적 라우팅)

---

### Chapter 11. NAT and DHCP
> **doc**: `chapters/chapter11/chapter11.md`
> **examples**: `chapters/chapter11/examples/`

- [ ] NAT Concept and Operation (NAT 개념과 동작 원리)
- [ ] NAPT - Port-based NAT (포트 기반 NAT)
- [ ] NAT Pros and Cons (NAT의 장단점)
- [ ] DHCP Concept (DHCP 개념)
- [ ] DHCP Process: DORA (DHCP 동작 과정)

---

## STEP 5. Link Layer (링크 계층)

### Chapter 12. Link Layer (링크 계층)
> **doc**: `chapters/chapter12/chapter12.md`
> **examples**: `chapters/chapter12/examples/`

- [ ] Link Layer Role (링크 계층 역할)
- [ ] MAC Address (MAC 주소)
- [ ] Ethernet Frame Structure (이더넷 프레임 구조)
- [ ] Switch Operation (스위치 동작 원리)
- [ ] ARP - Address Resolution Protocol
- [ ] Hub vs Switch vs Router (허브 vs 스위치 vs 라우터)

---

## STEP 6. Network Security (네트워크 보안)

### Chapter 13. Cryptography Basics (암호화 기초)
> **doc**: `chapters/chapter13/chapter13.md`
> **examples**: `chapters/chapter13/examples/`

- [ ] Why Encryption (암호화 필요성)
- [ ] Symmetric Key Encryption: AES (대칭키 암호화)
- [ ] Asymmetric Key Encryption: RSA (비대칭키 암호화)
- [ ] Hash Functions: SHA (해시 함수)
- [ ] Digital Signatures (디지털 서명)

---

### Chapter 14. TLS/SSL and HTTPS
> **doc**: `chapters/chapter14/chapter14.md`
> **examples**: `chapters/chapter14/examples/`

- [ ] TLS/SSL Concept (TLS/SSL 개념)
- [ ] TLS Handshake Process (TLS handshake 과정)
- [ ] Certificates and CA (인증서와 CA)
- [ ] HTTPS Operation (HTTPS 동작 원리)
- [ ] Certificate Chain (인증서 체인)

---

## STEP 7. Web Advanced Topics (웹 실무 심화)

### Chapter 15. HTTP Evolution (HTTP 발전)
> **doc**: `chapters/chapter15/chapter15.md`
> **examples**: `chapters/chapter15/examples/`

- [ ] HTTP/1.0 vs HTTP/1.1
- [ ] Keep-Alive
- [ ] HTTP/2: Multiplexing, Header Compression, Server Push
- [ ] HTTP/3: QUIC-based

---

### Chapter 16. Web Performance Optimization (웹 성능 최적화)
> **doc**: `chapters/chapter16/chapter16.md`
> **examples**: `chapters/chapter16/examples/`

- [ ] HTTP Caching: Cache-Control, ETag
- [ ] CDN Concept and Operation (CDN 개념과 동작)
- [ ] Compression: gzip, Brotli (압축)
- [ ] Connection Pooling

---

### Chapter 17. Real-time Communication (실시간 통신)
> **doc**: `chapters/chapter17/chapter17.md`
> **examples**: `chapters/chapter17/examples/`

- [ ] Polling vs Long Polling
- [ ] Server-Sent Events (SSE)
- [ ] WebSocket Concept (WebSocket 개념)
- [ ] WebSocket Handshake
- [ ] WebSocket vs HTTP
- [ ] Game Networking Basics (게임 네트워크 기초)

---

## Practical Connections (실무 연결 포인트)

| Network Concept | Practical Application |
|-----------------|----------------------|
| HTTP/HTTPS | API design, Secure communication |
| TCP/UDP | Protocol selection, Performance |
| DNS | Domain setup, Troubleshooting |
| Caching/CDN | Web performance optimization |
| WebSocket | Real-time features, Chat, Games |

---

## For Later (나중에 필요할 때)

| Topic | When |
|-------|------|
| BGP, OSPF details | Network engineering |
| VPN, Tunneling | Security/Infra design |
| SDN | Cloud networking |
| gRPC, Protocol Buffers | Microservices |
