# 컴퓨터 네트워크 학습 계획

> 목표: 웹 통신의 동작 원리를 이해하고, 네트워크 문제를 진단할 수 있는 수준

---

## AI 학습 가이드

### 역할
- AI는 이 커리큘럼으로 학습을 시켜주는 **시니어 개발자이자 교수**

### 학습 진행 흐름 (5단계)

**1단계: 진행도 확인** - "오늘 어디 할 차례야"
**2단계: 설명** - "학습 내용 보여줘" → 해당 챕터 .md 읽기
**3단계: 예제** - "예제 보여줘" → examples/ 폴더 확인
**4단계: 과제** - "과제 줘"
**5단계: 평가** - [답안 제출] → 통과 = [x] 체크

### 평가 기준
□ 학습한 개념을 이해하고 적용했는가

---

## STEP 1. 네트워크 기초

### Chapter 01. 네트워크 개요
> **doc**: `chapters/chapter01/chapter01.md`
> **examples**: `chapters/chapter01/examples/`

- [ ] 네트워크란 무엇인가
- [ ] 인터넷 구조
- [ ] 프로토콜 개념
- [ ] OSI 7계층 모델
- [ ] TCP/IP 4계층 모델
- [ ] OSI vs TCP/IP 비교
- [ ] 캡슐화와 역캡슐화

---

## STEP 2. 애플리케이션 계층

### Chapter 02. 웹과 HTTP
> **doc**: `chapters/chapter02/chapter02.md`
> **examples**: `chapters/chapter02/examples/`

- [ ] 클라이언트-서버 모델
- [ ] HTTP 개념과 특징
- [ ] HTTP 메시지 구조
- [ ] HTTP 메서드: GET, POST, PUT, DELETE
- [ ] 상태 코드: 2xx, 3xx, 4xx, 5xx
- [ ] HTTP 헤더
- [ ] 쿠키와 세션
- [ ] REST API 개념

---

### Chapter 03. DNS
> **doc**: `chapters/chapter03/chapter03.md`
> **examples**: `chapters/chapter03/examples/`

- [ ] DNS 개념과 필요성
- [ ] 도메인 계층 구조
- [ ] DNS 질의 과정
- [ ] DNS 레코드 타입: A, AAAA, CNAME, MX
- [ ] DNS 캐싱
- [ ] hosts 파일

---

### Chapter 04. 기타 애플리케이션 프로토콜
> **doc**: `chapters/chapter04/chapter04.md`
> **examples**: `chapters/chapter04/examples/`

- [ ] FTP
- [ ] SMTP, POP3, IMAP (이메일)
- [ ] SSH

---

## STEP 3. 전송 계층

### Chapter 05. 전송 계층 개요
> **doc**: `chapters/chapter05/chapter05.md`
> **examples**: `chapters/chapter05/examples/`

- [ ] 전송 계층 역할
- [ ] 포트 번호
- [ ] 다중화와 역다중화
- [ ] 잘 알려진 포트: 80, 443, 22

---

### Chapter 06. UDP
> **doc**: `chapters/chapter06/chapter06.md`
> **examples**: `chapters/chapter06/examples/`

- [ ] UDP 특징
- [ ] UDP 헤더 구조
- [ ] UDP 사용 사례
- [ ] UDP vs TCP 선택 기준

---

### Chapter 07. TCP 기초
> **doc**: `chapters/chapter07/chapter07.md`
> **examples**: `chapters/chapter07/examples/`

- [ ] TCP 특징
- [ ] TCP 헤더 구조
- [ ] 3-way Handshake (연결 설정)
- [ ] 4-way Handshake (연결 종료)
- [ ] TCP 상태 전이

---

### Chapter 08. TCP 신뢰성
> **doc**: `chapters/chapter08/chapter08.md`
> **examples**: `chapters/chapter08/examples/`

- [ ] 시퀀스 번호와 ACK
- [ ] 재전송
- [ ] 슬라이딩 윈도우
- [ ] 흐름 제어
- [ ] 혼잡 제어 개념
- [ ] Slow Start, AIMD

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
