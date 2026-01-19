# Chapter 07. TCP Basics (TCP 기초)

## Overview (개요)

TCP (Transmission Control Protocol) is a core protocol of the Internet Protocol Suite that provides reliable, ordered, and error-checked delivery of data between applications running on hosts communicating via an IP network.

TCP(전송 제어 프로토콜)는 인터넷 프로토콜 스위트의 핵심 프로토콜로, IP 네트워크를 통해 통신하는 호스트의 애플리케이션 간에 신뢰성 있고, 순서가 보장되며, 오류가 검사된 데이터 전달을 제공합니다.

---

## 1. TCP Characteristics (TCP 특성)

### 1.1 Reliable Transmission (신뢰성 있는 전송)

TCP guarantees that data sent from one end will be received correctly at the other end. This is achieved through:

TCP는 한쪽 끝에서 보낸 데이터가 다른 쪽 끝에서 올바르게 수신되는 것을 보장합니다. 이는 다음을 통해 달성됩니다:

- **Acknowledgments (확인 응답)**: Receiver sends ACK for received segments
- **Retransmission (재전송)**: Lost or corrupted segments are retransmitted
- **Checksums (체크섬)**: Detects data corruption during transmission
- **Sequence Numbers (순서 번호)**: Ensures data is delivered in order

```
Sender                                    Receiver
   |                                          |
   |  -------- Data (Seq=100) -------->      |
   |                                          |
   |  <------- ACK (Ack=200) ---------       |
   |                                          |
   |  -------- Data (Seq=200) -------->      |  (Lost!)
   |                                          |
   |         (Timeout - No ACK)               |
   |                                          |
   |  -------- Data (Seq=200) -------->      |  (Retransmit)
   |                                          |
   |  <------- ACK (Ack=300) ---------       |
```

### 1.2 Connection-Oriented (연결 지향)

TCP establishes a connection before data transfer begins. This connection is:

TCP는 데이터 전송을 시작하기 전에 연결을 설정합니다. 이 연결은:

- **Virtual Circuit (가상 회선)**: Not a physical path, but a logical connection
- **Full-Duplex (전이중)**: Data can flow in both directions simultaneously
- **Point-to-Point (점대점)**: Each TCP connection has exactly two endpoints

```
+------------------+                    +------------------+
|    Application   |                    |    Application   |
+------------------+                    +------------------+
         |                                       |
         v                                       v
+------------------+                    +------------------+
|       TCP        |<======= Virtual ======>|       TCP        |
|    (Endpoint)    |<======= Circuit ======>|    (Endpoint)    |
+------------------+                    +------------------+
         |                                       |
         v                                       v
+------------------+                    +------------------+
|       IP         |                    |       IP         |
+------------------+                    +------------------+
```

### 1.3 Stream-Oriented (스트림 지향)

TCP treats data as a continuous stream of bytes, not as individual messages.

TCP는 데이터를 개별 메시지가 아닌 연속적인 바이트 스트림으로 취급합니다.

```
Application sends:    "Hello" (5 bytes) + "World" (5 bytes)
                           |
                           v
TCP may transmit:     "HelloWor" + "ld"  (or any combination)
                           |
                           v
Application receives: Continuous byte stream "HelloWorld"
```

---

## 2. TCP Header Structure (TCP 헤더 구조)

The TCP header is typically 20-60 bytes in length (20 bytes minimum without options).

TCP 헤더는 일반적으로 20-60바이트 길이입니다 (옵션 없이 최소 20바이트).

```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|          Source Port          |       Destination Port        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                        Sequence Number                        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                    Acknowledgment Number                      |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| Data  |       |C|E|U|A|P|R|S|F|                               |
|Offset |Reserved|W|C|R|C|S|S|Y|I|           Window             |
|       |       |R|E|G|K|H|T|N|N|                               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|           Checksum            |         Urgent Pointer        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                    Options                    |    Padding    |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                             Data                              |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

### Header Field Descriptions (헤더 필드 설명)

| Field (필드) | Size (크기) | Description (설명) |
|-------------|-------------|-------------------|
| Source Port (출발지 포트) | 16 bits | Sender's port number / 송신자의 포트 번호 |
| Destination Port (목적지 포트) | 16 bits | Receiver's port number / 수신자의 포트 번호 |
| Sequence Number (순서 번호) | 32 bits | Position of first byte in segment / 세그먼트의 첫 바이트 위치 |
| Acknowledgment Number (확인 응답 번호) | 32 bits | Next expected byte / 다음 예상 바이트 |
| Data Offset (데이터 오프셋) | 4 bits | Header length in 32-bit words / 32비트 워드 단위의 헤더 길이 |
| Flags (플래그) | 9 bits | Control flags / 제어 플래그 |
| Window (윈도우) | 16 bits | Receive window size / 수신 윈도우 크기 |
| Checksum (체크섬) | 16 bits | Error detection / 오류 검출 |
| Urgent Pointer (긴급 포인터) | 16 bits | Position of urgent data / 긴급 데이터 위치 |

### TCP Flags (TCP 플래그)

```
+-----+---------------------------------------------+
| Flag| Description (설명)                          |
+-----+---------------------------------------------+
| SYN | Synchronize sequence numbers (동기화)       |
| ACK | Acknowledgment field is valid (확인 응답)   |
| FIN | No more data from sender (종료)             |
| RST | Reset the connection (연결 재설정)          |
| PSH | Push data immediately (즉시 전달)           |
| URG | Urgent pointer is valid (긴급)              |
| ECE | ECN-Echo (명시적 혼잡 통보 에코)             |
| CWR | Congestion Window Reduced (혼잡 윈도우 감소) |
+-----+---------------------------------------------+
```

---

## 3. 3-Way Handshake (3단계 핸드셰이크)

The 3-way handshake is used to establish a TCP connection. It ensures both sides are ready to communicate and synchronizes their sequence numbers.

3단계 핸드셰이크는 TCP 연결을 설정하는 데 사용됩니다. 양쪽 모두 통신할 준비가 되었는지 확인하고 순서 번호를 동기화합니다.

### Connection Establishment Process (연결 설정 과정)

```
     Client                                      Server
        |                                           |
        |              CLOSED                       |
        |                                        LISTEN
        |                                           |
   [1]  |  --------- SYN (Seq=x) ----------->      |
        |                                           |
        |              SYN_SENT                  SYN_RCVD
        |                                           |
   [2]  |  <------ SYN+ACK (Seq=y, Ack=x+1) ---    |
        |                                           |
   [3]  |  --------- ACK (Ack=y+1) ----------->    |
        |                                           |
        |            ESTABLISHED              ESTABLISHED
        |                                           |
        |  <======== Data Transfer =========>      |
        |                                           |
```

### Step-by-Step Explanation (단계별 설명)

**Step 1: SYN (Client -> Server)**
- Client sends a SYN segment with initial sequence number (ISN)
- 클라이언트가 초기 순서 번호(ISN)와 함께 SYN 세그먼트를 전송

**Step 2: SYN-ACK (Server -> Client)**
- Server responds with SYN-ACK
- ACK acknowledges client's ISN (x+1)
- Server includes its own ISN (y)
- 서버가 SYN-ACK로 응답
- ACK는 클라이언트의 ISN을 확인 (x+1)
- 서버는 자신의 ISN을 포함 (y)

**Step 3: ACK (Client -> Server)**
- Client acknowledges server's ISN (y+1)
- Connection is now established
- 클라이언트가 서버의 ISN을 확인 (y+1)
- 연결이 이제 설정됨

### Why 3-Way? (왜 3단계인가?)

```
Two-way would fail:
(2단계는 실패할 것입니다:)

Client                              Server
   |                                   |
   |  --- SYN (Seq=100) --->          |   (1)
   |                                   |
   |  <-- SYN+ACK (Seq=200, Ack=101)  |   (2)
   |                                   |
   |  (Server doesn't know if         |
   |   client received the SYN-ACK)   |
   |                                   |

The third ACK confirms that the client received the SYN-ACK.
세 번째 ACK는 클라이언트가 SYN-ACK를 수신했음을 확인합니다.
```

---

## 4. 4-Way Handshake (4단계 핸드셰이크)

The 4-way handshake is used to gracefully terminate a TCP connection. Either side can initiate the termination.

4단계 핸드셰이크는 TCP 연결을 정상적으로 종료하는 데 사용됩니다. 어느 쪽이든 종료를 시작할 수 있습니다.

### Connection Termination Process (연결 종료 과정)

```
     Client                                      Server
        |                                           |
        |            ESTABLISHED              ESTABLISHED
        |                                           |
   [1]  |  ---------- FIN (Seq=u) ----------->     |
        |                                           |
        |              FIN_WAIT_1               CLOSE_WAIT
        |                                           |
   [2]  |  <--------- ACK (Ack=u+1) -----------    |
        |                                           |
        |              FIN_WAIT_2               CLOSE_WAIT
        |                                           |
        |        (Server may send remaining data)   |
        |                                           |
   [3]  |  <--------- FIN (Seq=v) -------------    |
        |                                           |
        |              TIME_WAIT                LAST_ACK
        |                                           |
   [4]  |  ---------- ACK (Ack=v+1) ----------->   |
        |                                           |
        |              TIME_WAIT                  CLOSED
        |                                           |
        |         (Wait 2*MSL)                      |
        |                                           |
        |               CLOSED                      |
```

### Step-by-Step Explanation (단계별 설명)

**Step 1: FIN (Client -> Server)**
- Client sends FIN to indicate no more data to send
- Client enters FIN_WAIT_1 state
- 클라이언트가 더 이상 보낼 데이터가 없음을 나타내기 위해 FIN 전송
- 클라이언트는 FIN_WAIT_1 상태로 진입

**Step 2: ACK (Server -> Client)**
- Server acknowledges the FIN
- Server enters CLOSE_WAIT state
- Client enters FIN_WAIT_2 state
- 서버가 FIN을 확인
- 서버는 CLOSE_WAIT 상태로 진입
- 클라이언트는 FIN_WAIT_2 상태로 진입

**Step 3: FIN (Server -> Client)**
- Server sends FIN when ready to close
- Server enters LAST_ACK state
- 서버가 종료할 준비가 되면 FIN 전송
- 서버는 LAST_ACK 상태로 진입

**Step 4: ACK (Client -> Server)**
- Client acknowledges server's FIN
- Client enters TIME_WAIT state
- After 2*MSL timeout, client enters CLOSED
- 클라이언트가 서버의 FIN을 확인
- 클라이언트는 TIME_WAIT 상태로 진입
- 2*MSL 타임아웃 후 클라이언트는 CLOSED로 진입

### Why 4-Way? (왜 4단계인가?)

```
TCP is Full-Duplex, so each direction must be closed separately:
(TCP는 전이중이므로 각 방향을 별도로 닫아야 합니다:)

      Client ------ FIN ------> Server    (Client closes its send)
      Client <----- ACK ------- Server    (Server acknowledges)

      (Server can still send data here / 서버는 여기서 여전히 데이터를 보낼 수 있음)

      Client <----- FIN ------- Server    (Server closes its send)
      Client ------ ACK ------> Server    (Client acknowledges)
```

---

## 5. TCP State Transitions (TCP 상태 전이)

TCP connections go through various states during their lifetime.

TCP 연결은 수명 동안 다양한 상태를 거칩니다.

### State Diagram (상태 다이어그램)

```
                              +---------+ ---------\      active OPEN
                              |  CLOSED |            \    -----------
                              +---------+<---------\   \   create TCB
                                |     ^              \   \  snd SYN
                   passive OPEN |     |   CLOSE        \   \
                   ------------ |     | ----------       \   \
                    create TCB  |     | delete TCB        \   \
                                V     |                    \   V
                              +---------+            +-----------+
                              |  LISTEN |            | SYN_SENT  |
                              +---------+            +-----------+
                   rcv SYN      |     |     CLOSE          |     |
                   ----------   |     |    -------         |     |
                   snd SYN,ACK  |     |    delete TCB      |     |
                      +--------+       |                   |     |
                      |                |                   |     |
                      V                |                   V     |
                 +-----------+         |          +------------+ |
                 | SYN_RCVD  |<--------+----------|            | |
                 +-----------+snd SYN,ACK/        | rcv SYN,ACK| |
                      |                           +------------+ |
                      |                                          |
          rcv ACK     |                                rcv ACK   |
          -------     |                                -------   |
          x           V                                x         V
                  +-----------+                    +-----------+
                  |ESTABLISHED|                    |ESTABLISHED|
                  +-----------+                    +-----------+
```

### TCP States Description (TCP 상태 설명)

| State (상태) | Description (설명) |
|-------------|-------------------|
| CLOSED | No connection exists / 연결이 존재하지 않음 |
| LISTEN | Server waiting for connection request / 서버가 연결 요청을 기다림 |
| SYN_SENT | Client sent SYN, waiting for SYN-ACK / 클라이언트가 SYN을 보내고 SYN-ACK를 기다림 |
| SYN_RCVD | Server received SYN, sent SYN-ACK / 서버가 SYN을 받고 SYN-ACK를 보냄 |
| ESTABLISHED | Connection is open, data transfer / 연결이 열려 있고 데이터 전송 중 |
| FIN_WAIT_1 | Sent FIN, waiting for ACK / FIN을 보내고 ACK를 기다림 |
| FIN_WAIT_2 | Received ACK for FIN, waiting for FIN / FIN에 대한 ACK를 받고 FIN을 기다림 |
| CLOSE_WAIT | Received FIN, sent ACK / FIN을 받고 ACK를 보냄 |
| CLOSING | Both sides sent FIN simultaneously / 양쪽이 동시에 FIN을 보냄 |
| LAST_ACK | Sent FIN, waiting for final ACK / FIN을 보내고 마지막 ACK를 기다림 |
| TIME_WAIT | Waiting for packets to die off / 패킷이 소멸되기를 기다림 |

### TIME_WAIT State (TIME_WAIT 상태)

The TIME_WAIT state is crucial for two reasons:

TIME_WAIT 상태는 두 가지 이유로 중요합니다:

```
1. Ensures final ACK is received:
   (최종 ACK가 수신되었는지 확인:)

   Client                              Server
      |                                   |
      |  <-------- FIN (Seq=v) --------  |
      |                                   |
      |  --------- ACK (Lost!) ------->  |  (ACK lost!)
      |                                   |
      |            TIME_WAIT              |
      |                                   |
      |  <------ FIN (Retransmit) -----  |  (Server retransmits)
      |                                   |
      |  --------- ACK ----------->      |
      |                                   |

2. Allows old duplicate segments to expire:
   (이전 중복 세그먼트가 만료되도록 함:)

   - Wait time: 2 * MSL (Maximum Segment Lifetime)
   - Typical MSL: 30 seconds to 2 minutes
   - 대기 시간: 2 * MSL (최대 세그먼트 수명)
   - 일반적인 MSL: 30초 ~ 2분
```

---

## Summary (요약)

### Key Points (핵심 포인트)

1. **TCP Characteristics (TCP 특성)**
   - Reliable (신뢰성): Guaranteed delivery through ACKs and retransmissions
   - Connection-oriented (연결 지향): Requires connection setup before data transfer
   - Stream-oriented (스트림 지향): Data treated as continuous byte stream

2. **TCP Header (TCP 헤더)**
   - 20-60 bytes with source/destination ports, sequence/ACK numbers, flags
   - Important flags: SYN, ACK, FIN, RST, PSH, URG

3. **3-Way Handshake (3단계 핸드셰이크)**
   - SYN -> SYN-ACK -> ACK
   - Establishes connection and synchronizes sequence numbers

4. **4-Way Handshake (4단계 핸드셰이크)**
   - FIN -> ACK -> FIN -> ACK
   - Gracefully terminates connection in both directions

5. **TCP States (TCP 상태)**
   - LISTEN, SYN_SENT, SYN_RCVD, ESTABLISHED for connection setup
   - FIN_WAIT_1/2, CLOSE_WAIT, LAST_ACK, TIME_WAIT, CLOSED for termination

```
+-----------------------+----------------------------------------+
| Connection Phase      | Key States                             |
| (연결 단계)           | (주요 상태)                            |
+-----------------------+----------------------------------------+
| Setup (설정)          | CLOSED -> SYN_SENT -> ESTABLISHED      |
|                       | LISTEN -> SYN_RCVD -> ESTABLISHED      |
+-----------------------+----------------------------------------+
| Termination (종료)    | ESTABLISHED -> FIN_WAIT_1 -> FIN_WAIT_2|
|                       | -> TIME_WAIT -> CLOSED                 |
+-----------------------+----------------------------------------+
```
