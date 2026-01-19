# Chapter 14. TLS/SSL and HTTPS

---

## Table of Contents (목차)

1. [TLS/SSL Concept (TLS/SSL 개념)](#1-tlsssl-concept-tlsssl-개념)
2. [TLS Handshake Process (TLS 핸드셰이크 과정)](#2-tls-handshake-process-tls-핸드셰이크-과정)
3. [Certificates and CA (인증서와 CA)](#3-certificates-and-ca-인증서와-ca)
4. [HTTPS Operation (HTTPS 동작)](#4-https-operation-https-동작)
5. [Certificate Chain (인증서 체인)](#5-certificate-chain-인증서-체인)
6. [Summary (요약)](#6-summary-요약)

---

## 1. TLS/SSL Concept (TLS/SSL 개념)

### What is TLS/SSL? (TLS/SSL이란?)

TLS (Transport Layer Security) and its predecessor SSL (Secure Sockets Layer) are cryptographic protocols that provide **secure communication** over a network. TLS encrypts data, verifies identities, and ensures data integrity.

TLS(전송 계층 보안)와 이전 버전인 SSL(보안 소켓 계층)은 네트워크를 통한 **안전한 통신**을 제공하는 암호화 프로토콜입니다. TLS는 데이터를 암호화하고, 신원을 확인하며, 데이터 무결성을 보장합니다.

```
+-----------------------------------------------------------------------+
|                    Without TLS vs With TLS                             |
+-----------------------------------------------------------------------+
|                                                                       |
|   WITHOUT TLS (HTTP):                                                 |
|   +--------+                                    +--------+            |
|   | Client |  "Password: secret123"            | Server |            |
|   |        | ================================> |        |            |
|   +--------+                                    +--------+            |
|                     ^                                                 |
|                     |                                                 |
|              +------+------+                                          |
|              |  Attacker   | "I can see: Password: secret123"         |
|              +-------------+                                          |
|                                                                       |
+-----------------------------------------------------------------------+
|                                                                       |
|   WITH TLS (HTTPS):                                                   |
|   +--------+                                    +--------+            |
|   | Client |  "x#$%^&*(encrypted)@#$%"         | Server |            |
|   |        | ================================> |        |            |
|   +--------+                                    +--------+            |
|                     ^                                                 |
|                     |                                                 |
|              +------+------+                                          |
|              |  Attacker   | "I see: x#$%^&* (useless!)"              |
|              +-------------+                                          |
|                                                                       |
+-----------------------------------------------------------------------+
```

### SSL vs TLS History (SSL vs TLS 역사)

```
+-----------------------------------------------------------------------+
|                    SSL/TLS Version History                             |
+-----------------------------------------------------------------------+
|                                                                       |
|   +----------+--------+------------------------------------------+    |
|   | Protocol | Year   | Status                                   |    |
|   +----------+--------+------------------------------------------+    |
|   | SSL 1.0  | -      | Never released (security flaws)          |    |
|   +----------+--------+------------------------------------------+    |
|   | SSL 2.0  | 1995   | DEPRECATED - Serious vulnerabilities     |    |
|   +----------+--------+------------------------------------------+    |
|   | SSL 3.0  | 1996   | DEPRECATED - POODLE attack               |    |
|   +----------+--------+------------------------------------------+    |
|   | TLS 1.0  | 1999   | DEPRECATED - Should not be used          |    |
|   +----------+--------+------------------------------------------+    |
|   | TLS 1.1  | 2006   | DEPRECATED - Should not be used          |    |
|   +----------+--------+------------------------------------------+    |
|   | TLS 1.2  | 2008   | SUPPORTED - Widely used today            |    |
|   +----------+--------+------------------------------------------+    |
|   | TLS 1.3  | 2018   | RECOMMENDED - Latest and most secure     |    |
|   +----------+--------+------------------------------------------+    |
|                                                                       |
|   Note: "SSL" is often used colloquially to mean TLS                  |
|   참고: "SSL"은 종종 TLS를 의미하는 구어로 사용됨                     |
|                                                                       |
+-----------------------------------------------------------------------+
```

### What TLS Provides (TLS가 제공하는 것)

```
+-----------------------------------------------------------------------+
|                    TLS Security Properties                             |
+-----------------------------------------------------------------------+
|                                                                       |
|   +-------------------+----------------------------------------------+|
|   | Property          | Description                                  ||
|   +-------------------+----------------------------------------------+|
|   | Confidentiality   | Data is encrypted                            ||
|   | 기밀성            | 데이터 암호화                                ||
|   |                   | - Uses symmetric encryption (AES)            ||
|   |                   | - 대칭키 암호화 사용 (AES)                   ||
|   +-------------------+----------------------------------------------+|
|   | Authentication    | Server (and optionally client) identity      ||
|   | 인증              | verified via certificates                    ||
|   |                   | 서버 (선택적으로 클라이언트) 신원이          ||
|   |                   | 인증서로 확인됨                              ||
|   +-------------------+----------------------------------------------+|
|   | Integrity         | Data cannot be modified without detection    ||
|   | 무결성            | 데이터가 탐지 없이 수정될 수 없음            ||
|   |                   | - Uses MAC (Message Authentication Code)     ||
|   |                   | - MAC(메시지 인증 코드) 사용                 ||
|   +-------------------+----------------------------------------------+|
|                                                                       |
+-----------------------------------------------------------------------+
```

### TLS in the Protocol Stack (프로토콜 스택에서의 TLS)

```
+-----------------------------------------------------------------------+
|                    TLS Position in Stack                               |
+-----------------------------------------------------------------------+
|                                                                       |
|   Application Layer      +------------------+                         |
|                          |  HTTP, SMTP, FTP |                         |
|                          +------------------+                         |
|                                  |                                    |
|   Security Layer         +------------------+                         |
|                          |      TLS/SSL     |  <-- Sits here          |
|                          +------------------+                         |
|                                  |                                    |
|   Transport Layer        +------------------+                         |
|                          |       TCP        |                         |
|                          +------------------+                         |
|                                  |                                    |
|   Network Layer          +------------------+                         |
|                          |        IP        |                         |
|                          +------------------+                         |
|                                                                       |
|   TLS runs on top of TCP and below application protocols              |
|   TLS는 TCP 위에서 실행되고 애플리케이션 프로토콜 아래에 위치         |
|                                                                       |
+-----------------------------------------------------------------------+
```

---

## 2. TLS Handshake Process (TLS 핸드셰이크 과정)

### Overview of TLS Handshake (TLS 핸드셰이크 개요)

The TLS handshake establishes a secure connection before any application data is exchanged. It negotiates cipher suites, authenticates the server, and creates shared encryption keys.

TLS 핸드셰이크는 애플리케이션 데이터가 교환되기 전에 보안 연결을 설정합니다. 암호 스위트를 협상하고, 서버를 인증하며, 공유 암호화 키를 생성합니다.

```
+-----------------------------------------------------------------------+
|                    TLS Handshake Goals                                 |
+-----------------------------------------------------------------------+
|                                                                       |
|   1. Agree on protocol version (TLS 1.2, TLS 1.3)                     |
|      프로토콜 버전 합의                                               |
|                                                                       |
|   2. Select cryptographic algorithms (cipher suite)                   |
|      암호화 알고리즘 선택 (암호 스위트)                               |
|                                                                       |
|   3. Authenticate server (via certificate)                            |
|      서버 인증 (인증서를 통해)                                        |
|                                                                       |
|   4. Generate shared secret keys                                      |
|      공유 비밀 키 생성                                                |
|                                                                       |
+-----------------------------------------------------------------------+
```

### TLS 1.2 Handshake (Full) (TLS 1.2 핸드셰이크 - 전체)

```
+-----------------------------------------------------------------------+
|                    TLS 1.2 Full Handshake                              |
+-----------------------------------------------------------------------+
|                                                                       |
|   Client                                              Server          |
|   +------+                                            +------+        |
|   |      |                                            |      |        |
|   +------+                                            +------+        |
|      |                                                   |            |
|      |  1. ClientHello                                   |            |
|      |     - TLS version                                 |            |
|      |     - Supported cipher suites                     |            |
|      |     - Client random (32 bytes)                    |            |
|      |-------------------------------------------------->|            |
|      |                                                   |            |
|      |  2. ServerHello                                   |            |
|      |     - Selected TLS version                        |            |
|      |     - Selected cipher suite                       |            |
|      |     - Server random (32 bytes)                    |            |
|      |<--------------------------------------------------|            |
|      |                                                   |            |
|      |  3. Certificate                                   |            |
|      |     - Server's X.509 certificate                  |            |
|      |<--------------------------------------------------|            |
|      |                                                   |            |
|      |  4. ServerKeyExchange (if needed)                 |            |
|      |     - For DHE/ECDHE key exchange                  |            |
|      |<--------------------------------------------------|            |
|      |                                                   |            |
|      |  5. ServerHelloDone                               |            |
|      |<--------------------------------------------------|            |
|      |                                                   |            |
|      |  6. ClientKeyExchange                             |            |
|      |     - Pre-master secret (encrypted with           |            |
|      |       server's public key)                        |            |
|      |-------------------------------------------------->|            |
|      |                                                   |            |
|      |  [Both compute master secret]                     |            |
|      |  [둘 다 마스터 시크릿 계산]                       |            |
|      |                                                   |            |
|      |  7. ChangeCipherSpec                              |            |
|      |     - "I'm switching to encrypted mode"           |            |
|      |-------------------------------------------------->|            |
|      |                                                   |            |
|      |  8. Finished (encrypted)                          |            |
|      |     - Verify handshake integrity                  |            |
|      |-------------------------------------------------->|            |
|      |                                                   |            |
|      |  9. ChangeCipherSpec                              |            |
|      |<--------------------------------------------------|            |
|      |                                                   |            |
|      |  10. Finished (encrypted)                         |            |
|      |<--------------------------------------------------|            |
|      |                                                   |            |
|      |  === Secure connection established ===            |            |
|      |  === 보안 연결 수립됨 ===                         |            |
|      |                                                   |            |
|      |  [Application data (encrypted)]                   |            |
|      |<=================================================>|            |
|                                                                       |
|   Total: 2 round trips (2 RTT)                                        |
|   총: 2 왕복 (2 RTT)                                                  |
|                                                                       |
+-----------------------------------------------------------------------+
```

### TLS 1.3 Handshake (Improved) (TLS 1.3 핸드셰이크 - 개선됨)

```
+-----------------------------------------------------------------------+
|                    TLS 1.3 Handshake (1-RTT)                           |
+-----------------------------------------------------------------------+
|                                                                       |
|   Client                                              Server          |
|   +------+                                            +------+        |
|   |      |                                            |      |        |
|   +------+                                            +------+        |
|      |                                                   |            |
|      |  1. ClientHello                                   |            |
|      |     - Supported cipher suites                     |            |
|      |     - Key share (client's DH public key)          |            |
|      |     - Supported groups                            |            |
|      |-------------------------------------------------->|            |
|      |                                                   |            |
|      |  2. ServerHello                                   |            |
|      |     - Selected cipher suite                       |            |
|      |     - Key share (server's DH public key)          |            |
|      |  + EncryptedExtensions                            |            |
|      |  + Certificate                                    |            |
|      |  + CertificateVerify                              |            |
|      |  + Finished                                       |            |
|      |<--------------------------------------------------|            |
|      |                                                   |            |
|      |  3. Finished                                      |            |
|      |-------------------------------------------------->|            |
|      |                                                   |            |
|      |  === Secure connection established ===            |            |
|      |  === 보안 연결 수립됨 ===                         |            |
|      |                                                   |            |
|      |  [Application data (encrypted)]                   |            |
|      |<=================================================>|            |
|                                                                       |
|   Total: 1 round trip (1 RTT) - 50% faster!                           |
|   총: 1 왕복 (1 RTT) - 50% 더 빠름!                                   |
|                                                                       |
+-----------------------------------------------------------------------+
```

### TLS 1.3 0-RTT Resumption (TLS 1.3 0-RTT 재개)

```
+-----------------------------------------------------------------------+
|                    TLS 1.3 0-RTT Resumption                            |
+-----------------------------------------------------------------------+
|                                                                       |
|   When client has previously connected to server:                     |
|   클라이언트가 이전에 서버에 연결한 적이 있을 때:                     |
|                                                                       |
|   Client                                              Server          |
|      |                                                   |            |
|      |  1. ClientHello + Early Data (encrypted!)         |            |
|      |     - Uses pre-shared key (PSK) from previous     |            |
|      |       session                                     |            |
|      |-------------------------------------------------->|            |
|      |                                                   |            |
|      |  2. ServerHello + Finished                        |            |
|      |<--------------------------------------------------|            |
|      |                                                   |            |
|      |  3. Finished                                      |            |
|      |-------------------------------------------------->|            |
|      |                                                   |            |
|      |  [Continue with encrypted data]                   |            |
|                                                                       |
|   Client can send data immediately - 0 RTT!                           |
|   클라이언트가 즉시 데이터 전송 가능 - 0 RTT!                         |
|                                                                       |
|   Warning: 0-RTT data is vulnerable to replay attacks                 |
|   주의: 0-RTT 데이터는 재전송 공격에 취약                             |
|                                                                       |
+-----------------------------------------------------------------------+
```

### TLS 1.2 vs TLS 1.3 Comparison (TLS 1.2 vs TLS 1.3 비교)

```
+-------------------------+----------------------+----------------------+
| Feature                 | TLS 1.2              | TLS 1.3              |
+-------------------------+----------------------+----------------------+
| Handshake RTT           | 2 RTT                | 1 RTT (or 0-RTT)     |
| 핸드셰이크 RTT          |                      |                      |
+-------------------------+----------------------+----------------------+
| Key Exchange            | RSA, DHE, ECDHE      | ECDHE, DHE only      |
| 키 교환                 |                      | (no RSA key exch.)   |
+-------------------------+----------------------+----------------------+
| Cipher Suites           | Many legacy options  | Only 5 secure ones   |
| 암호 스위트             | 많은 레거시 옵션     | 안전한 5개만         |
+-------------------------+----------------------+----------------------+
| Forward Secrecy         | Optional             | Mandatory            |
| 순방향 비밀성           | 선택적               | 필수                 |
+-------------------------+----------------------+----------------------+
| RC4, 3DES               | Allowed (weak)       | Removed              |
|                         | 허용 (취약)          | 제거됨               |
+-------------------------+----------------------+----------------------+
| Static RSA key exchange | Allowed (no PFS)     | Removed              |
| 정적 RSA 키 교환        | 허용 (PFS 없음)      | 제거됨               |
+-------------------------+----------------------+----------------------+
```

### Cipher Suite Example (암호 스위트 예시)

```
+-----------------------------------------------------------------------+
|                    Cipher Suite Breakdown                              |
+-----------------------------------------------------------------------+
|                                                                       |
|   Example: TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384                       |
|                                                                       |
|   TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384                               |
|       |     |         |   |    |   |                                  |
|       |     |         |   |    |   +-- Hash for PRF/HMAC              |
|       |     |         |   |    +------ Mode (Galois/Counter Mode)     |
|       |     |         |   +----------- Key size (256 bits)            |
|       |     |         +--------------- Symmetric cipher (AES)         |
|       |     +------------------------- Authentication (RSA cert)      |
|       +------------------------------- Key exchange (ECDH Ephemeral)  |
|                                                                       |
+-----------------------------------------------------------------------+
|                                                                       |
|   TLS 1.3 Cipher Suite (simplified):                                  |
|   TLS_AES_256_GCM_SHA384                                              |
|       |   |    |    |                                                 |
|       |   |    |    +-- Hash                                          |
|       |   |    +------- Mode                                          |
|       |   +------------ Key size                                      |
|       +---------------- Cipher                                        |
|                                                                       |
|   Key exchange is separate in TLS 1.3                                 |
|   TLS 1.3에서는 키 교환이 별도로 지정됨                               |
|                                                                       |
+-----------------------------------------------------------------------+
```

---

## 3. Certificates and CA (인증서와 CA)

### What is a Digital Certificate? (디지털 인증서란?)

A digital certificate binds a **public key to an identity** (like a domain name). It's issued by a trusted Certificate Authority (CA) and proves that the server is who it claims to be.

디지털 인증서는 **공개키를 신원**(도메인 이름 등)에 연결합니다. 신뢰할 수 있는 인증 기관(CA)에서 발급하며 서버가 주장하는 대로의 서버임을 증명합니다.

```
+-----------------------------------------------------------------------+
|                    Why Certificates are Needed                         |
+-----------------------------------------------------------------------+
|                                                                       |
|   Problem without certificates:                                       |
|   인증서가 없을 때의 문제:                                            |
|                                                                       |
|   Client                 Attacker                  Real Server        |
|   +-----+                +------+                  +------+           |
|   |     |  "Give me your |      |                  |      |           |
|   |     |   public key"  |      |                  |      |           |
|   |     | -------------> |      |                  |      |           |
|   |     |                |      |                  |      |           |
|   |     |  "Here's MY    |      |                  |      |           |
|   |     |   public key"  |      |                  |      |           |
|   |     | <------------- |      |                  |      |           |
|   |     |                |      |                  |      |           |
|   |     |  [Client encrypts with attacker's key]   |      |           |
|   |     |  [클라이언트가 공격자 키로 암호화]       |      |           |
|   +-----+                +------+                  +------+           |
|                                                                       |
|   Man-in-the-Middle attack! (중간자 공격!)                            |
|                                                                       |
+-----------------------------------------------------------------------+
|                                                                       |
|   Solution: Certificates verify identity                              |
|   해결책: 인증서가 신원을 확인                                        |
|                                                                       |
|   Certificate says:                                                   |
|   "This public key belongs to example.com                             |
|    and is vouched for by Trusted CA"                                  |
|                                                                       |
|   인증서 내용:                                                        |
|   "이 공개키는 example.com의 것이며                                   |
|    신뢰할 수 있는 CA가 보증함"                                        |
|                                                                       |
+-----------------------------------------------------------------------+
```

### X.509 Certificate Structure (X.509 인증서 구조)

```
+-----------------------------------------------------------------------+
|                    X.509 Certificate Fields                            |
+-----------------------------------------------------------------------+
|                                                                       |
|   +---------------------------------------------------------------+   |
|   | Version: 3                                                    |   |
|   +---------------------------------------------------------------+   |
|   | Serial Number: 01:23:45:67:89:AB:CD:EF                        |   |
|   +---------------------------------------------------------------+   |
|   | Signature Algorithm: SHA256withRSA                            |   |
|   +---------------------------------------------------------------+   |
|   | Issuer: CN=DigiCert Global CA, O=DigiCert Inc                 |   |
|   +---------------------------------------------------------------+   |
|   | Validity:                                                     |   |
|   |   Not Before: 2024-01-01 00:00:00 UTC                         |   |
|   |   Not After:  2025-01-01 23:59:59 UTC                         |   |
|   +---------------------------------------------------------------+   |
|   | Subject: CN=www.example.com, O=Example Corp                   |   |
|   +---------------------------------------------------------------+   |
|   | Subject Public Key Info:                                      |   |
|   |   Algorithm: RSA                                              |   |
|   |   Public Key: (2048 bit)                                      |   |
|   |     Modulus: 00:b1:a2:c3:d4:e5:f6...                          |   |
|   |     Exponent: 65537                                           |   |
|   +---------------------------------------------------------------+   |
|   | Extensions:                                                   |   |
|   |   Subject Alternative Name:                                   |   |
|   |     DNS: www.example.com                                      |   |
|   |     DNS: example.com                                          |   |
|   |   Key Usage: Digital Signature, Key Encipherment              |   |
|   |   Extended Key Usage: TLS Web Server Authentication           |   |
|   +---------------------------------------------------------------+   |
|   | Signature: (CA's digital signature over all above)            |   |
|   +---------------------------------------------------------------+   |
|                                                                       |
+-----------------------------------------------------------------------+
```

### Certificate Authority (CA) (인증 기관)

```
+-----------------------------------------------------------------------+
|                    Certificate Authority Role                          |
+-----------------------------------------------------------------------+
|                                                                       |
|   CA is a trusted third party that:                                   |
|   CA는 다음을 수행하는 신뢰할 수 있는 제3자:                          |
|                                                                       |
|   1. Verifies identity of certificate requesters                      |
|      인증서 요청자의 신원 확인                                        |
|                                                                       |
|   2. Issues signed certificates                                       |
|      서명된 인증서 발급                                               |
|                                                                       |
|   3. Maintains certificate revocation lists (CRL)                     |
|      인증서 폐지 목록(CRL) 관리                                       |
|                                                                       |
|   +-------------------------------------------------------------------+
|   |                    Certificate Issuance Process                  |
|   +-------------------------------------------------------------------+
|   |                                                                   |
|   |   Website Owner                        Certificate Authority      |
|   |      +-----+                               +-----+                |
|   |      |     |  1. Generate key pair         |     |                |
|   |      |     |  2. Create CSR (Certificate   |     |                |
|   |      |     |     Signing Request)          |     |                |
|   |      |     | -------------------------->   |     |                |
|   |      |     |                               |     |                |
|   |      |     |  3. Verify domain ownership   |     |                |
|   |      |     |  4. Verify organization       |     |                |
|   |      |     |     (for OV/EV certs)         |     |                |
|   |      |     |                               |     |                |
|   |      |     |  5. Issue signed certificate  |     |                |
|   |      |     | <--------------------------   |     |                |
|   |      +-----+                               +-----+                |
|   |                                                                   |
|   +-------------------------------------------------------------------+
|                                                                       |
+-----------------------------------------------------------------------+
```

### Types of Certificates (인증서 유형)

```
+-----------------------------------------------------------------------+
|                    Certificate Types                                   |
+-----------------------------------------------------------------------+
|                                                                       |
|   +-------------+------------------+---------------------------------+ |
|   | Type        | Validation       | Description                     | |
|   +-------------+------------------+---------------------------------+ |
|   | DV          | Domain           | Verifies domain ownership only  | |
|   | (Domain     | Validation       | 도메인 소유권만 확인            | |
|   | Validation) |                  | - Cheapest, fastest             | |
|   |             |                  | - 가장 저렴, 가장 빠름          | |
|   +-------------+------------------+---------------------------------+ |
|   | OV          | Organization     | Verifies domain + organization  | |
|   | (Org        | Validation       | 도메인 + 조직 확인              | |
|   | Validation) |                  | - Moderate price                | |
|   |             |                  | - 중간 가격                     | |
|   +-------------+------------------+---------------------------------+ |
|   | EV          | Extended         | Rigorous verification           | |
|   | (Extended   | Validation       | 엄격한 검증                     | |
|   | Validation) |                  | - Green bar (legacy browsers)   | |
|   |             |                  | - Most expensive                | |
|   |             |                  | - 가장 비쌈                     | |
|   +-------------+------------------+---------------------------------+ |
|                                                                       |
|   Other types:                                                        |
|   - Wildcard: *.example.com (covers all subdomains)                   |
|   - Multi-domain (SAN): multiple domains in one cert                  |
|   - Self-signed: For testing (browsers will warn!)                    |
|                                                                       |
+-----------------------------------------------------------------------+
```

### Major Certificate Authorities (주요 인증 기관)

```
+-----------------------------------------------------------------------+
|                    Major CAs                                           |
+-----------------------------------------------------------------------+
|                                                                       |
|   Commercial CAs (상용 CA):                                           |
|   +---------------------------+--------------------------------------+|
|   | CA                        | Notes                                ||
|   +---------------------------+--------------------------------------+|
|   | DigiCert                  | Largest commercial CA                ||
|   | Sectigo (Comodo)          | Popular for SSL certificates         ||
|   | GlobalSign                | Enterprise focused                   ||
|   | Entrust                   | Government and enterprise            ||
|   +---------------------------+--------------------------------------+|
|                                                                       |
|   Free CAs (무료 CA):                                                 |
|   +---------------------------+--------------------------------------+|
|   | Let's Encrypt             | Free, automated, open                ||
|   |                           | 무료, 자동화, 오픈                   ||
|   +---------------------------+--------------------------------------+|
|                                                                       |
|   Let's Encrypt has revolutionized HTTPS adoption!                    |
|   Let's Encrypt가 HTTPS 보급에 혁명을 일으킴!                         |
|                                                                       |
+-----------------------------------------------------------------------+
```

---

## 4. HTTPS Operation (HTTPS 동작)

### What is HTTPS? (HTTPS란?)

HTTPS (HTTP Secure) is HTTP over TLS. It provides encrypted, authenticated communication between browsers and web servers.

HTTPS(HTTP Secure)는 TLS를 통한 HTTP입니다. 브라우저와 웹 서버 간의 암호화되고 인증된 통신을 제공합니다.

```
+-----------------------------------------------------------------------+
|                    HTTP vs HTTPS                                       |
+-----------------------------------------------------------------------+
|                                                                       |
|   HTTP:                                                               |
|   +------+         Plaintext          +--------+                      |
|   |Client| <======================>   | Server |                      |
|   +------+     (anyone can read)      +--------+                      |
|                                                                       |
|   Port: 80                                                            |
|   URL: http://example.com                                             |
|                                                                       |
+-----------------------------------------------------------------------+
|                                                                       |
|   HTTPS:                                                              |
|   +------+   [TLS]  Encrypted  [TLS]  +--------+                      |
|   |Client| <======================>   | Server |                      |
|   +------+     (secure tunnel)        +--------+                      |
|                                                                       |
|   Port: 443                                                           |
|   URL: https://example.com                                            |
|                                                                       |
+-----------------------------------------------------------------------+
```

### HTTPS Connection Flow (HTTPS 연결 흐름)

```
+-----------------------------------------------------------------------+
|                    Full HTTPS Connection                               |
+-----------------------------------------------------------------------+
|                                                                       |
|   Client (Browser)                            Server (example.com)    |
|      |                                              |                 |
|      |  1. DNS lookup: example.com -> 93.184.216.34 |                 |
|      |--------------------------------------------->|                 |
|      |                                              |                 |
|      |  2. TCP 3-way handshake (port 443)           |                 |
|      |      SYN -------------------------->         |                 |
|      |      <------------------------ SYN-ACK       |                 |
|      |      ACK -------------------------->         |                 |
|      |                                              |                 |
|      |  3. TLS Handshake                            |                 |
|      |      ClientHello ------------------->        |                 |
|      |      <------------------- ServerHello        |                 |
|      |      <------------------- Certificate        |                 |
|      |                                              |                 |
|      |  4. Client verifies certificate              |                 |
|      |     - Is it signed by trusted CA?            |                 |
|      |     - Is it not expired?                     |                 |
|      |     - Does domain match?                     |                 |
|      |                                              |                 |
|      |  5. Key exchange (continued handshake)       |                 |
|      |      ClientKeyExchange ------------>         |                 |
|      |      ChangeCipherSpec ------------->         |                 |
|      |      Finished --------------------->         |                 |
|      |      <--------------- ChangeCipherSpec       |                 |
|      |      <--------------------- Finished         |                 |
|      |                                              |                 |
|      |  === TLS tunnel established ===              |                 |
|      |                                              |                 |
|      |  6. HTTP Request (encrypted)                 |                 |
|      |      GET / HTTP/1.1 --------------->         |                 |
|      |      Host: example.com                       |                 |
|      |                                              |                 |
|      |  7. HTTP Response (encrypted)                |                 |
|      |      <-------------- HTTP/1.1 200 OK         |                 |
|      |      <-------------- Content-Type: text/html |                 |
|      |      <-------------- [HTML content]          |                 |
|                                                                       |
+-----------------------------------------------------------------------+
```

### Browser Certificate Verification (브라우저 인증서 확인)

```
+-----------------------------------------------------------------------+
|                    Certificate Verification Steps                      |
+-----------------------------------------------------------------------+
|                                                                       |
|   Browser receives server certificate                                 |
|   브라우저가 서버 인증서 수신                                         |
|           |                                                           |
|           v                                                           |
|   +-------------------------------------------------------------------+
|   |  Check 1: Is certificate expired?                                |
|   |           인증서가 만료되었는가?                                  |
|   |                                                                   |
|   |  Current Date: 2024-06-15                                         |
|   |  Valid From: 2024-01-01  Valid Until: 2025-01-01                  |
|   |  Result: OK (within validity period)                              |
|   +-------------------------------------------------------------------+
|           |                                                           |
|           v                                                           |
|   +-------------------------------------------------------------------+
|   |  Check 2: Does domain match?                                     |
|   |           도메인이 일치하는가?                                    |
|   |                                                                   |
|   |  Requested: www.example.com                                       |
|   |  Certificate: CN=www.example.com, SAN=example.com                 |
|   |  Result: OK (domain matches)                                      |
|   +-------------------------------------------------------------------+
|           |                                                           |
|           v                                                           |
|   +-------------------------------------------------------------------+
|   |  Check 3: Is it signed by trusted CA?                            |
|   |           신뢰할 수 있는 CA가 서명했는가?                         |
|   |                                                                   |
|   |  Certificate signed by: DigiCert Global CA                        |
|   |  DigiCert in browser's trust store? YES                           |
|   |  Result: OK (trusted CA)                                          |
|   +-------------------------------------------------------------------+
|           |                                                           |
|           v                                                           |
|   +-------------------------------------------------------------------+
|   |  Check 4: Is certificate revoked?                                |
|   |           인증서가 폐지되었는가?                                  |
|   |                                                                   |
|   |  Check CRL or OCSP                                                |
|   |  Result: OK (not revoked)                                         |
|   +-------------------------------------------------------------------+
|           |                                                           |
|           v                                                           |
|       ALL CHECKS PASSED = Show padlock icon                           |
|       모든 검사 통과 = 자물쇠 아이콘 표시                             |
|                                                                       |
+-----------------------------------------------------------------------+
```

### Common HTTPS Errors (일반적인 HTTPS 오류)

```
+-----------------------------------------------------------------------+
|                    HTTPS Error Types                                   |
+-----------------------------------------------------------------------+
|                                                                       |
|   +----------------------------+-------------------------------------+|
|   | Error                      | Cause / Solution                    ||
|   +----------------------------+-------------------------------------+|
|   | NET::ERR_CERT_COMMON_      | Domain in URL doesn't match         ||
|   | NAME_INVALID               | certificate CN or SAN               ||
|   | 일반 이름 불일치           | - Check domain name spelling        ||
|   +----------------------------+-------------------------------------+|
|   | NET::ERR_CERT_DATE_INVALID | Certificate expired                 ||
|   | 날짜 무효                  | - Renew certificate                 ||
|   |                            | - Check system clock                ||
|   +----------------------------+-------------------------------------+|
|   | NET::ERR_CERT_AUTHORITY_   | Self-signed or unknown CA           ||
|   | INVALID                    | - Use certificate from trusted CA   ||
|   | 인증 기관 무효             |                                     ||
|   +----------------------------+-------------------------------------+|
|   | NET::ERR_CERT_REVOKED      | Certificate was revoked             ||
|   | 인증서 폐지됨              | - Get new certificate               ||
|   +----------------------------+-------------------------------------+|
|   | Mixed Content Warning      | HTTP resources on HTTPS page        ||
|   | 혼합 콘텐츠 경고           | - Use HTTPS for all resources       ||
|   +----------------------------+-------------------------------------+|
|                                                                       |
+-----------------------------------------------------------------------+
```

---

## 5. Certificate Chain (인증서 체인)

### What is a Certificate Chain? (인증서 체인이란?)

A certificate chain (or chain of trust) links a website's certificate to a trusted root CA through one or more intermediate CAs.

인증서 체인(또는 신뢰 체인)은 하나 이상의 중간 CA를 통해 웹사이트의 인증서를 신뢰할 수 있는 루트 CA에 연결합니다.

```
+-----------------------------------------------------------------------+
|                    Certificate Chain Structure                         |
+-----------------------------------------------------------------------+
|                                                                       |
|                     +-------------------+                             |
|                     |     Root CA       |  Stored in browser/OS       |
|                     | (Self-signed)     |  브라우저/OS에 저장됨       |
|                     +-------------------+                             |
|                            |                                          |
|                            | Signs                                    |
|                            v                                          |
|                     +-------------------+                             |
|                     | Intermediate CA   |  Signed by Root CA          |
|                     |                   |  루트 CA가 서명             |
|                     +-------------------+                             |
|                            |                                          |
|                            | Signs                                    |
|                            v                                          |
|                     +-------------------+                             |
|                     | End-Entity Cert   |  Your website certificate   |
|                     | (www.example.com) |  웹사이트 인증서            |
|                     +-------------------+                             |
|                                                                       |
|   Trust path: Website cert -> Intermediate -> Root (trusted!)         |
|   신뢰 경로: 웹사이트 인증서 -> 중간 -> 루트 (신뢰!)                  |
|                                                                       |
+-----------------------------------------------------------------------+
```

### Why Intermediate CAs? (왜 중간 CA가 필요한가?)

```
+-----------------------------------------------------------------------+
|                    Why Use Intermediate CAs                            |
+-----------------------------------------------------------------------+
|                                                                       |
|   1. SECURITY (보안)                                                  |
|      - Root CA private key is kept OFFLINE                            |
|      - 루트 CA 개인키는 오프라인으로 보관                             |
|      - If intermediate is compromised, only revoke intermediate       |
|      - 중간 인증서가 손상되면 중간 인증서만 폐지                      |
|      - Root remains safe                                              |
|      - 루트는 안전하게 유지                                           |
|                                                                       |
|   2. FLEXIBILITY (유연성)                                             |
|      - Different intermediates for different purposes                 |
|      - 다양한 목적에 따른 다른 중간 인증서                            |
|      - Can have multiple layers of intermediates                      |
|      - 여러 계층의 중간 인증서 가능                                   |
|                                                                       |
|   3. SCALABILITY (확장성)                                             |
|      - Root signs fewer certificates (just intermediates)             |
|      - 루트는 적은 인증서만 서명 (중간 인증서만)                      |
|      - Intermediates handle volume                                    |
|      - 중간 인증서가 볼륨 처리                                        |
|                                                                       |
+-----------------------------------------------------------------------+
```

### Chain Verification Process (체인 검증 과정)

```
+-----------------------------------------------------------------------+
|                    Chain Verification                                  |
+-----------------------------------------------------------------------+
|                                                                       |
|   Server sends: [End-Entity Cert] + [Intermediate Cert(s)]            |
|   서버 전송: [최종 인증서] + [중간 인증서]                            |
|                                                                       |
|   Browser verification:                                               |
|   브라우저 검증:                                                      |
|                                                                       |
|   Step 1: Verify end-entity cert                                      |
|   +---------------------------------------------------------------+   |
|   | www.example.com certificate                                   |   |
|   | Issuer: DigiCert SHA2 Secure Server CA                        |   |
|   | Signature: [verify with intermediate's public key] -> OK      |   |
|   +---------------------------------------------------------------+   |
|                           |                                           |
|                           v                                           |
|   Step 2: Verify intermediate cert                                    |
|   +---------------------------------------------------------------+   |
|   | DigiCert SHA2 Secure Server CA certificate                    |   |
|   | Issuer: DigiCert Global Root CA                               |   |
|   | Signature: [verify with root's public key] -> OK              |   |
|   +---------------------------------------------------------------+   |
|                           |                                           |
|                           v                                           |
|   Step 3: Check if root is trusted                                    |
|   +---------------------------------------------------------------+   |
|   | DigiCert Global Root CA                                       |   |
|   | Is this in browser's trusted root store? -> YES               |   |
|   +---------------------------------------------------------------+   |
|                           |                                           |
|                           v                                           |
|           CHAIN VERIFIED - Connection is secure                       |
|           체인 검증됨 - 연결이 안전함                                 |
|                                                                       |
+-----------------------------------------------------------------------+
```

### Real-World Certificate Chain Example (실제 인증서 체인 예시)

```
+-----------------------------------------------------------------------+
|                    Example: google.com Certificate Chain               |
+-----------------------------------------------------------------------+
|                                                                       |
|   +---------------------------------------------------------------+   |
|   | Level 0: End-Entity Certificate                               |   |
|   | Subject: CN=*.google.com                                      |   |
|   | Issuer: CN=GTS CA 1C3                                         |   |
|   | Valid: 2024-01-01 to 2024-03-31                                |   |
|   | Key: ECDSA P-256                                              |   |
|   +---------------------------------------------------------------+   |
|                           |                                           |
|                           v                                           |
|   +---------------------------------------------------------------+   |
|   | Level 1: Intermediate CA                                      |   |
|   | Subject: CN=GTS CA 1C3                                        |   |
|   | Issuer: CN=GTS Root R1                                        |   |
|   | Valid: 2020-08-13 to 2027-09-30                                |   |
|   +---------------------------------------------------------------+   |
|                           |                                           |
|                           v                                           |
|   +---------------------------------------------------------------+   |
|   | Level 2: Root CA (in browser trust store)                     |   |
|   | Subject: CN=GTS Root R1                                       |   |
|   | Issuer: CN=GTS Root R1 (self-signed)                          |   |
|   | Valid: 2016-06-22 to 2036-06-22                                |   |
|   +---------------------------------------------------------------+   |
|                                                                       |
+-----------------------------------------------------------------------+
```

### Certificate Revocation (인증서 폐지)

```
+-----------------------------------------------------------------------+
|                    Certificate Revocation Methods                      |
+-----------------------------------------------------------------------+
|                                                                       |
|   When a certificate is compromised, it must be revoked               |
|   인증서가 손상되면 폐지해야 함                                       |
|                                                                       |
|   +-------------------+----------------------------------------------+|
|   | Method            | Description                                  ||
|   +-------------------+----------------------------------------------+|
|   | CRL               | Certificate Revocation List                  ||
|   | (인증서 폐지 목록)| - CA publishes list of revoked certs         ||
|   |                   | - Client downloads and checks                ||
|   |                   | - Problem: Can be large, slow updates        ||
|   +-------------------+----------------------------------------------+|
|   | OCSP              | Online Certificate Status Protocol           ||
|   | (온라인 인증서    | - Real-time check with CA                    ||
|   |  상태 프로토콜)   | - Client asks "Is cert X revoked?"           ||
|   |                   | - Problem: Privacy (CA knows what you visit) ||
|   +-------------------+----------------------------------------------+|
|   | OCSP Stapling     | Server attaches OCSP response                ||
|   | (OCSP 스태플링)   | - Server gets OCSP response from CA          ||
|   |                   | - Attaches to TLS handshake                  ||
|   |                   | - Best of both worlds                        ||
|   +-------------------+----------------------------------------------+|
|                                                                       |
+-----------------------------------------------------------------------+
```

---

## 6. Summary (요약)

### Key Concepts (핵심 개념)

```
+-----------------------------------------------------------------------+
|                     Chapter 14 Summary                                 |
+-----------------------------------------------------------------------+
|                                                                       |
|  1. TLS/SSL Concept (TLS/SSL 개념)                                    |
|     - Cryptographic protocol for secure communication                 |
|     - Provides: confidentiality, authentication, integrity            |
|     - TLS 1.3 is current standard (2018)                              |
|     - Sits between TCP and application layer                          |
|                                                                       |
|  2. TLS Handshake (TLS 핸드셰이크)                                    |
|     - Establishes secure connection                                   |
|     - TLS 1.2: 2 RTT                                                  |
|     - TLS 1.3: 1 RTT (or 0-RTT for resumption)                        |
|     - Negotiates cipher suite, authenticates server                   |
|                                                                       |
|  3. Certificates and CA (인증서와 CA)                                 |
|     - X.509 certificates bind public key to identity                  |
|     - CA verifies identity and signs certificates                     |
|     - Types: DV, OV, EV                                               |
|     - Let's Encrypt provides free certificates                        |
|                                                                       |
|  4. HTTPS (HTTPS 동작)                                                |
|     - HTTP over TLS (port 443)                                        |
|     - Browser verifies certificate before connecting                  |
|     - Common errors: expired, domain mismatch, untrusted CA           |
|                                                                       |
|  5. Certificate Chain (인증서 체인)                                   |
|     - Root CA -> Intermediate CA -> End-entity cert                   |
|     - Root CAs are pre-trusted in browser/OS                          |
|     - Intermediate CAs provide security and flexibility               |
|     - Revocation via CRL, OCSP, or OCSP Stapling                      |
|                                                                       |
+-----------------------------------------------------------------------+
```

### TLS Quick Reference (TLS 빠른 참조)

```
+-------------------------+--------------------------------------------+
| Component               | Purpose                                    |
+-------------------------+--------------------------------------------+
| TLS                     | Secure communication protocol              |
| Certificate             | Proves server identity                     |
| CA                      | Issues and signs certificates              |
| Root CA                 | Self-signed, trusted by browser            |
| Intermediate CA         | Signed by root, signs end certs            |
| Cipher Suite            | Set of algorithms for TLS                  |
| Handshake               | Establishes secure connection              |
| HTTPS                   | HTTP over TLS                              |
+-------------------------+--------------------------------------------+
```

### Security Checklist (보안 체크리스트)

```
+-----------------------------------------------------------------------+
|                    TLS Security Best Practices                         |
+-----------------------------------------------------------------------+
|                                                                       |
|   [x] Use TLS 1.3 (or at minimum TLS 1.2)                             |
|   [x] Disable SSL 2.0, SSL 3.0, TLS 1.0, TLS 1.1                      |
|   [x] Use certificates from trusted CAs                               |
|   [x] Renew certificates before expiration                            |
|   [x] Enable OCSP Stapling                                            |
|   [x] Use strong cipher suites                                        |
|   [x] Enable HSTS (HTTP Strict Transport Security)                    |
|   [x] Redirect HTTP to HTTPS                                          |
|   [x] Use 2048-bit+ RSA or ECDSA certificates                         |
|   [x] Configure complete certificate chain                            |
|                                                                       |
+-----------------------------------------------------------------------+
```

---

*End of Chapter 14*
