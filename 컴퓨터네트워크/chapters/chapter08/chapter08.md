# Chapter 08. TCP Reliability (TCP 신뢰성)

## Overview (개요)

TCP provides reliable data transmission over unreliable IP networks. This chapter explores the mechanisms TCP uses to ensure data is delivered correctly, in order, and without loss.

TCP는 신뢰할 수 없는 IP 네트워크 위에서 신뢰성 있는 데이터 전송을 제공합니다. 이 장에서는 TCP가 데이터를 올바르게, 순서대로, 손실 없이 전달하기 위해 사용하는 메커니즘을 살펴봅니다.

---

## 1. Sequence Numbers and Acknowledgments (순서 번호와 확인 응답)

### 1.1 Sequence Numbers (순서 번호)

Sequence numbers identify the position of data bytes in the stream. They enable:
- Ordering of segments
- Detection of missing data
- Identification of duplicate segments

순서 번호는 스트림에서 데이터 바이트의 위치를 식별합니다. 이를 통해:
- 세그먼트 순서 지정
- 누락된 데이터 감지
- 중복 세그먼트 식별

```
Initial Sequence Number (ISN) = 1000

Application Data: "Hello World!" (12 bytes)

+--------+--------+--------+--------+--------+--------+
| Seq    | 1000   | 1005   | 1008   | ...              |
+--------+--------+--------+--------+--------+--------+
| Data   | Hello  | Wor    | ld!    | ...              |
| Bytes  | (5)    | (3)    | (4)    | ...              |
+--------+--------+--------+--------+--------+--------+

Sender                                         Receiver
   |                                               |
   |  Seq=1000, Data="Hello" (5 bytes)  ------->  |
   |                                               |
   |  Seq=1005, Data="Wor" (3 bytes)    ------->  |
   |                                               |
   |  Seq=1008, Data="ld!" (4 bytes)    ------->  |
   |                                               |
```

### 1.2 Acknowledgment Numbers (확인 응답 번호)

The ACK number indicates the next expected byte, not the last received byte.

ACK 번호는 마지막으로 수신된 바이트가 아닌 다음 예상 바이트를 나타냅니다.

```
Sender                                         Receiver
   |                                               |
   |  Seq=1000, 100 bytes  -------------------->  |
   |                                               |
   |  <------------------- ACK=1100               |
   |  (Expects byte 1100 next)                    |
   |                                               |
   |  Seq=1100, 150 bytes  -------------------->  |
   |                                               |
   |  <------------------- ACK=1250               |
   |  (Expects byte 1250 next)                    |
```

### 1.3 Cumulative Acknowledgment (누적 확인 응답)

TCP uses cumulative ACKs - acknowledging all bytes up to a certain point.

TCP는 누적 ACK를 사용합니다 - 특정 지점까지의 모든 바이트를 확인합니다.

```
Sender                                         Receiver
   |                                               |
   |  Seq=1000 (100 bytes) ------------------->   |
   |                                               |
   |  Seq=1100 (100 bytes) -----X (Lost!)         |
   |                                               |
   |  Seq=1200 (100 bytes) ------------------->   |
   |                                               |
   |  <------------------- ACK=1100               |
   |  (Still waiting for byte 1100)               |
   |                                               |
   |  Seq=1300 (100 bytes) ------------------->   |
   |                                               |
   |  <------------------- ACK=1100               |
   |  (Duplicate ACK - still waiting for 1100)    |
```

---

## 2. Retransmission Mechanisms (재전송 메커니즘)

### 2.1 Timeout-Based Retransmission (타임아웃 기반 재전송)

TCP uses a Retransmission Timeout (RTO) timer to detect lost segments.

TCP는 손실된 세그먼트를 감지하기 위해 재전송 타임아웃(RTO) 타이머를 사용합니다.

```
Sender                                         Receiver
   |                                               |
   |  Seq=1000 -------------------------------->  |
   |  Start RTO Timer                             |
   |                                               |
   |  <-------------------------------- ACK=1100  |
   |  Stop Timer, Start new timer for next segment|
   |                                               |
   |  Seq=1100 ----X (Lost!)                      |
   |  Start RTO Timer                             |
   |                                               |
   |  ... Timer expires ...                       |
   |                                               |
   |  Seq=1100 (Retransmit) ------------------>   |
   |  Restart RTO Timer                           |
```

### 2.2 RTO Calculation (RTO 계산)

```
RTT (Round-Trip Time) Measurement:
(왕복 시간 측정:)

     Sender                    Receiver
        |                          |
  T1 -->|  Segment -------->      |
        |                          |
  T2 <--|  <-------- ACK          |
        |                          |

     RTT_sample = T2 - T1

SRTT (Smoothed RTT):
    SRTT = (1 - alpha) * SRTT + alpha * RTT_sample
    (Typically alpha = 0.125)

RTTVAR (RTT Variance):
    RTTVAR = (1 - beta) * RTTVAR + beta * |SRTT - RTT_sample|
    (Typically beta = 0.25)

RTO Calculation:
    RTO = SRTT + 4 * RTTVAR

Example:
    SRTT = 100ms, RTTVAR = 20ms
    RTO = 100 + 4 * 20 = 180ms
```

### 2.3 Fast Retransmit (빠른 재전송)

Fast retransmit triggers retransmission after receiving 3 duplicate ACKs, without waiting for timeout.

빠른 재전송은 타임아웃을 기다리지 않고 3개의 중복 ACK를 수신한 후 재전송을 트리거합니다.

```
Sender                                         Receiver
   |                                               |
   |  Seq=1000 -------------------------------->  |
   |                                               |
   |  Seq=1100 -------X (Lost!)                   |
   |                                               |
   |  Seq=1200 -------------------------------->  |
   |  <-------------------------------- ACK=1100  | (Dup ACK #1)
   |                                               |
   |  Seq=1300 -------------------------------->  |
   |  <-------------------------------- ACK=1100  | (Dup ACK #2)
   |                                               |
   |  Seq=1400 -------------------------------->  |
   |  <-------------------------------- ACK=1100  | (Dup ACK #3)
   |                                               |
   |  === 3 Duplicate ACKs: Fast Retransmit! === |
   |                                               |
   |  Seq=1100 (Retransmit) ------------------>   |
   |                                               |
   |  <-------------------------------- ACK=1500  | (Cumulative ACK)
```

---

## 3. Sliding Window (슬라이딩 윈도우)

The sliding window mechanism allows multiple segments to be sent without waiting for individual ACKs.

슬라이딩 윈도우 메커니즘은 개별 ACK를 기다리지 않고 여러 세그먼트를 전송할 수 있게 합니다.

### 3.1 Window Concept (윈도우 개념)

```
Sender's View:
(송신자의 관점:)

        Window Size = 4 segments

        +---+---+---+---+---+---+---+---+---+---+
Bytes:  | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 |10 |
        +---+---+---+---+---+---+---+---+---+---+
          ^           ^           ^
          |           |           |
        Sent &      Send       Cannot
        ACKed       Window      Send Yet

        <---Sent--->|<-Window->|<--Not Sent-->
            (1)       (2,3,4,5)    (6,7,8...)
```

### 3.2 Window Sliding (윈도우 슬라이딩)

```
Initial State:
(초기 상태:)

    +---+---+---+---+---+---+---+---+
    | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 |
    +---+---+---+---+---+---+---+---+
        [   Window = 4   ]

After ACK for segment 1:
(세그먼트 1에 대한 ACK 후:)

    +---+---+---+---+---+---+---+---+
    | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 |
    +---+---+---+---+---+---+---+---+
     ^      [   Window = 4   ]
    ACKed

After ACK for segments 2, 3:
(세그먼트 2, 3에 대한 ACK 후:)

    +---+---+---+---+---+---+---+---+
    | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 |
    +---+---+---+---+---+---+---+---+
     ^---^---^     [   Window = 4   ]
       ACKed
```

### 3.3 Efficiency Comparison (효율성 비교)

```
Stop-and-Wait (Without Sliding Window):
(슬라이딩 윈도우 없이:)

Sender                    Receiver      Time
   |  Seg 1 ----->           |           |
   |         <----- ACK 1    |           |
   |  Seg 2 ----->           |           |
   |         <----- ACK 2    |           |
   |  Seg 3 ----->           |           |
   |         <----- ACK 3    |           v

   Utilization = Data Time / Total Time = Low

Sliding Window (Window = 3):
(슬라이딩 윈도우 (윈도우 = 3):)

Sender                    Receiver      Time
   |  Seg 1 ----->           |           |
   |  Seg 2 ----->           |           |
   |  Seg 3 ----->           |           |
   |         <----- ACK 1    |           |
   |  Seg 4 ----->           |           |
   |         <----- ACK 2    |           |
   |  Seg 5 ----->           |           v

   Utilization = Data Time / Total Time = High
```

---

## 4. Flow Control (흐름 제어)

Flow control prevents a fast sender from overwhelming a slow receiver.

흐름 제어는 빠른 송신자가 느린 수신자를 압도하는 것을 방지합니다.

### 4.1 Receive Window (수신 윈도우)

```
Receiver's Buffer:
(수신자의 버퍼:)

+--------------------------------------------------+
|  Application   |   Received    |     Available   |
|    has read    |   but not     |     Buffer      |
|                |   read yet    |     Space       |
+--------------------------------------------------+
                                  <-- rwnd ------->
                                  (Receive Window)

The receiver advertises 'rwnd' to the sender.
수신자는 송신자에게 'rwnd'를 알립니다.
```

### 4.2 Flow Control in Action (흐름 제어 동작)

```
Sender                                         Receiver
   |                                               |
   |                                    rwnd=4000  |
   |  <--------------------- ACK, rwnd=4000       |
   |                                               |
   |  Send 2000 bytes ---------------------------->|
   |                                    rwnd=2000  |
   |  <--------------------- ACK, rwnd=2000       |
   |                                               |
   |  Send 2000 bytes ---------------------------->|
   |                                    rwnd=0     |
   |  <--------------------- ACK, rwnd=0          |
   |                                               |
   |  (Sender must stop - window is 0!)           |
   |  (송신자는 멈춰야 함 - 윈도우가 0!)           |
   |                                               |
   |         ... Application reads data ...       |
   |                                               |
   |  <--------------------- ACK, rwnd=2000       |
   |  (Window update - can send again)            |
   |  (윈도우 업데이트 - 다시 전송 가능)          |
```

### 4.3 Zero Window Problem and Persist Timer (제로 윈도우 문제와 지속 타이머)

```
Problem:
(문제:)

Sender                                         Receiver
   |                                               |
   |  <--------------------- ACK, rwnd=0          |
   |                                               |
   |  (Waiting for window update)                 |
   |                                               |
   |             (Window Update LOST!)             |
   |                                               |
   |  (Sender waits forever)                      |
   |  (Receiver waits for data)                   |
   |  --> DEADLOCK! (교착 상태!)                  |

Solution: Persist Timer (지속 타이머)
(해결책: 지속 타이머)

Sender                                         Receiver
   |                                               |
   |  <--------------------- ACK, rwnd=0          |
   |  Start Persist Timer                         |
   |                                               |
   |  ... Timer expires ...                       |
   |                                               |
   |  Zero Window Probe (1 byte) ------------->   |
   |                                               |
   |  <--------------------- ACK, rwnd=0 or >0    |
   |                                               |
```

---

## 5. Congestion Control (혼잡 제어)

Congestion control prevents the sender from overwhelming the network.

혼잡 제어는 송신자가 네트워크를 압도하는 것을 방지합니다.

### 5.1 Congestion vs Flow Control (혼잡 제어 vs 흐름 제어)

```
+------------------+--------------------------------+
| Flow Control     | Congestion Control             |
| (흐름 제어)      | (혼잡 제어)                    |
+------------------+--------------------------------+
| Prevents sender  | Prevents sender from           |
| from overwhelming| overwhelming the NETWORK       |
| the RECEIVER     |                                |
|                  |                                |
| 수신자 보호      | 네트워크 보호                  |
+------------------+--------------------------------+
| Uses: rwnd       | Uses: cwnd                     |
| (receive window) | (congestion window)            |
+------------------+--------------------------------+

Effective Window = min(rwnd, cwnd)
실제 윈도우 = min(rwnd, cwnd)
```

### 5.2 Congestion Window (cwnd) (혼잡 윈도우)

```
TCP maintains a congestion window (cwnd) that limits how much data
can be outstanding (sent but not yet acknowledged).

TCP는 미처리 데이터(전송되었지만 아직 확인되지 않은)의 양을
제한하는 혼잡 윈도우(cwnd)를 유지합니다.

Send Rate roughly = cwnd / RTT

Example:
    cwnd = 10 KB, RTT = 100 ms
    Send Rate = 10 KB / 100 ms = 100 KB/s
```

---

## 6. Slow Start (느린 시작)

Slow Start is used when starting a connection or after a timeout. Despite the name, it grows exponentially.

느린 시작은 연결을 시작하거나 타임아웃 후에 사용됩니다. 이름과 달리 지수적으로 증가합니다.

### 6.1 Slow Start Algorithm (느린 시작 알고리즘)

```
Initial: cwnd = 1 MSS (Maximum Segment Size)
         ssthresh = 65535 bytes (or based on path MTU)

For each ACK received:
    cwnd = cwnd + 1 MSS

This results in exponential growth:
(이는 지수적 증가를 초래합니다:)

Round  | cwnd (MSS) | Segments Sent | ACKs Received
-------|------------|---------------|---------------
  1    |     1      |      1        |      1
  2    |     2      |      2        |      2
  3    |     4      |      4        |      4
  4    |     8      |      8        |      8
  5    |    16      |     16        |     16
  ...  |   2^n      |     2^n       |     2^n
```

### 6.2 Slow Start Visualization (느린 시작 시각화)

```
cwnd
(segments)
    ^
    |                                    ssthresh
    |                               .....*........
    |                          *
    |                     *
    |                *
    |           *
    |      *
    |  *
    +-------------------------------------------> Time
                                                  (RTT)
       |<-- Slow Start -->|<-- Congestion -->|
                              Avoidance
```

---

## 7. AIMD (Additive Increase, Multiplicative Decrease)

AIMD is the congestion avoidance algorithm used after slow start threshold is reached.

AIMD는 느린 시작 임계값에 도달한 후 사용되는 혼잡 회피 알고리즘입니다.

### 7.1 AIMD Algorithm (AIMD 알고리즘)

```
Additive Increase (가산 증가):
- When cwnd >= ssthresh
- For each ACK: cwnd = cwnd + MSS * (MSS / cwnd)
- Effectively: cwnd increases by 1 MSS per RTT

Multiplicative Decrease (승산 감소):
- On packet loss (3 duplicate ACKs):
    ssthresh = cwnd / 2
    cwnd = ssthresh (Fast Recovery)

- On timeout:
    ssthresh = cwnd / 2
    cwnd = 1 MSS (Start Slow Start)
```

### 7.2 AIMD Visualization (AIMD 시각화)

```
cwnd
  ^
  |        *
  |       * *                    *
  |      *   *                  * *
  |     *     *                *   *
  |    *       *              *     *
  |   *         *            *       *
  |  *           *          *         *
  | *             *        *           *
  |*               *      *             *
  +---------------------------------------------> Time

  |<-Slow->|<-- AIMD -->|<-- AIMD -->|
    Start   (AI then MD)  (AI then MD)

  * = Packet loss event (패킷 손실 이벤트)
```

### 7.3 Complete TCP Congestion Control (전체 TCP 혼잡 제어)

```
                  +-------------------+
                  |    cwnd = 1 MSS   |
                  | ssthresh = large  |
                  +-------------------+
                           |
                           v
                  +-------------------+
             +--->|    SLOW START     |<---+
             |    | cwnd += 1 per ACK |    |
             |    +-------------------+    |
             |             |               |
             |    cwnd >= ssthresh?        | Timeout
             |             |               |
             |      Yes    v      No       |
             |    +--------+--------+      |
             |    |                 |      |
             |    v                 v      |
    Timeout  |  +-------------------+      |
    ------   |  | CONGESTION        |      |
    ssthresh |  | AVOIDANCE         |------+
    = cwnd/2 |  | cwnd += 1 per RTT |
    cwnd = 1 |  +-------------------+
             |           |
             |    3 Dup ACKs?
             |           |
             |           v
             |  +-------------------+
             +--| FAST RECOVERY     |
                | ssthresh = cwnd/2 |
                | cwnd = ssthresh   |
                +-------------------+
```

### 7.4 Congestion Control Timeline (혼잡 제어 타임라인)

```
cwnd   ssthresh = 16
(MSS)
  ^
32|                                    .........timeout
  |                                   .
  |                                  .  ssthresh=16
16|.......3 dup ACKs.............. .   cwnd=1
  |      .     Congestion       . .
  |     .       Avoidance      .  .
  |    .           /          .   .
  |   .  Slow    /           .    .
  |  . Start    /           .     .
8 | .         /            .      ..ssthresh=8
  |.        /             .         .
4 |       /              .          .
  |      /              .           . Slow Start
2 |     /              .            . (again)
  |    /              .             .
1 |. /               .              .
  +-------------------------------------------> Time
      ^              ^               ^
      |              |               |
   Slow Start    Fast Recovery    Timeout
   ends          (3 dup ACKs)     (RTO expires)
```

---

## Summary (요약)

### Key Points (핵심 포인트)

1. **Sequence Numbers & ACKs (순서 번호와 ACK)**
   - Sequence numbers identify byte positions
   - ACK indicates next expected byte (cumulative)
   - 순서 번호는 바이트 위치를 식별
   - ACK는 다음 예상 바이트를 나타냄 (누적)

2. **Retransmission (재전송)**
   - Timeout-based: RTO timer expires
   - Fast Retransmit: 3 duplicate ACKs
   - 타임아웃 기반: RTO 타이머 만료
   - 빠른 재전송: 3개의 중복 ACK

3. **Sliding Window (슬라이딩 윈도우)**
   - Allows multiple segments in flight
   - Improves throughput and efficiency
   - 여러 세그먼트가 전송 중일 수 있음
   - 처리량과 효율성 향상

4. **Flow Control (흐름 제어)**
   - Receiver advertises rwnd
   - Prevents receiver buffer overflow
   - 수신자가 rwnd를 알림
   - 수신자 버퍼 오버플로우 방지

5. **Congestion Control (혼잡 제어)**
   - Slow Start: Exponential growth initially
   - AIMD: Linear increase, half on loss
   - Effective window = min(cwnd, rwnd)
   - 느린 시작: 초기에 지수 성장
   - AIMD: 선형 증가, 손실 시 절반
   - 실제 윈도우 = min(cwnd, rwnd)

```
+------------------------+---------------------------+
| Mechanism (메커니즘)    | Purpose (목적)            |
+------------------------+---------------------------+
| Sequence Numbers       | Order & Identify data     |
| (순서 번호)            | (데이터 순서 및 식별)     |
+------------------------+---------------------------+
| ACKs                   | Confirm receipt           |
| (확인 응답)            | (수신 확인)               |
+------------------------+---------------------------+
| Retransmission         | Handle lost segments      |
| (재전송)               | (손실된 세그먼트 처리)    |
+------------------------+---------------------------+
| Sliding Window         | Efficient transmission    |
| (슬라이딩 윈도우)      | (효율적인 전송)           |
+------------------------+---------------------------+
| Flow Control           | Protect receiver          |
| (흐름 제어)            | (수신자 보호)             |
+------------------------+---------------------------+
| Congestion Control     | Protect network           |
| (혼잡 제어)            | (네트워크 보호)           |
+------------------------+---------------------------+
```
