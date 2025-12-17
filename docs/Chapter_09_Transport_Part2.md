# 9장 전송층 (Transport Layer) - Part 2

이 문서는 `ch 09 transport (2).pdf` 파일의 내용을 바탕으로 **9.4 TCP (오류 제어, 혼잡 제어, 타이머)**와 **9.5 SCTP**에 대해 상세하게 정리한 것입니다.

## 9.4 전송 제어 프로토콜 (TCP) - Part 2

### 1. 오류 제어 (Error Control)
TCP는 데이터 스트림을 신뢰성 있게 전달하기 위해 다음과 같은 기능을 수행합니다.
- **목표**: 순서에 맞고, 오류 없이, 손실이나 중복 없이 데이터 전달.
- **도구**:
    - **검사합 (Checksum)**: 훼손된 세그먼트 감지.
    - **확인응답 (Acknowledgement)**: 수신 확인.
    - **타임아웃 (Timeout)**: 재전송 트리거.

#### A. 확인응답 (ACK) 방식
- **누적 확인응답 (Cumulative ACK)**:
    - 정상적으로 수신된 데이터의 **다음 바이트 번호**를 알립니다.
    - 중간에 패킷이 손실되면, 그 이후 패킷을 받았더라도 손실된 패킷의 번호를 계속 요청합니다.
- **선택 확인응답 (SACK, Selective ACK)**:
    - 옵션으로 사용되며, 순서에 맞지 않게 수신된 데이터 블록(Gap)을 알릴 수 있습니다.

#### B. ACK 생성 규칙
1. **피기배킹 (Piggybacking)**: 데이터 보낼 때 ACK를 함께 보냄.
2. **ACK 지연 (Delayed ACK)**: 데이터가 없고 세그먼트 하나만 받은 경우, 500ms 정도 기다렸다가 ACK 전송 (트래픽 감소).
3. **재전송 방지**: 순서대로 도착한 세그먼트가 2개 이상 미확인 상태면 즉시 ACK 전송.
4. **빠른 재전송 (Fast Retransmission)**: 순서가 어긋난(Out-of-order) 세그먼트 도착 시 즉시 중복 ACK 전송.
5. **누락된 세그먼트 수신 시**: 즉시 ACK를 보내 다음 기대 번호를 알림.
6. **중복 세그먼트 수신 시**: 세그먼트는 버리고 즉시 ACK 전송 (ACK 손실 대비).

#### C. 재전송 (Retransmission)
- **RTO (Retransmission Time-Out)**: 타이머 만료 시 재전송. RTT(왕복 시간)를 기반으로 설정.
- **3-Duplicate ACKs**: 동일한 ACK가 3번 중복 수신되면, 타이머 만료 전이라도 즉시 재전송 (빠른 재전송).

### 2. 혼잡 제어 (Congestion Control)
네트워크 혼잡을 방지하고 해결하기 위한 메커니즘입니다. (흐름 제어는 수신자 버퍼 보호, 혼잡 제어는 네트워크 보호)

#### A. 혼잡 윈도우 (cwnd, Congestion Window)
- 송신자가 보낼 수 있는 데이터 양은 `min(rwnd, cwnd)`로 결정됩니다.
- `rwnd`: 수신자가 알리는 윈도우 크기 (흐름 제어).
- `cwnd`: 송신자가 네트워크 상황에 따라 조절하는 윈도우 크기 (혼잡 제어).

#### B. 혼잡 제어 알고리즘
1. **느린 시작 (Slow Start)**:
    - `cwnd`를 1 MSS(Maximum Segment Size)로 시작.
    - ACK 수신 시마다 `cwnd`를 1씩 증가 (RTT마다 2배로 **지수적 증가**).
    - 임계치(`ssthresh`)에 도달하면 혼잡 회피 단계로 전환.
2. **혼잡 회피 (Congestion Avoidance)**:
    - `ssthresh` 도달 후에는 RTT마다 `cwnd`를 1씩 증가 (**가산적 증가**).
    - 혼잡 발생을 늦추기 위함.
3. **빠른 회복 (Fast Recovery)**:
    - 3개의 중복 ACK 수신 시(약한 혼잡), `ssthresh`를 절반으로 줄이고 `cwnd`를 그 값으로 설정한 뒤 다시 증가 시작.
    - 타임아웃 발생 시(강한 혼잡)에는 `cwnd`를 1로 초기화하고 느린 시작부터 다시 시작.

#### C. 혼잡 제어 정책 (TCP 버전)
- **Tahoe TCP**: 타임아웃이나 3-Duplicate ACK 발생 시 무조건 `cwnd`를 1로 초기화 (느린 시작).
- **Reno TCP**:
    - 타임아웃 시: `cwnd` = 1 (느린 시작).
    - 3-Duplicate ACK 시: `cwnd` = `ssthresh` (빠른 회복).
- **AIMD (Additive Increase, Multiplicative Decrease)**:
    - 혼잡이 없으면 선형 증가(AI), 혼잡 감지 시 절반으로 감소(MD).

### 3. TCP 타이머
- **재전송 타이머 (RTO)**: 패킷 손실 감지용.
- **영속 타이머 (Persistence Timer)**: `rwnd=0` 통보 후, 윈도우 업데이트 ACK가 손실되어 발생하는 데드락 방지. 주기적으로 Probe 패킷 전송.
- **킵얼라이브 타이머 (Keepalive Timer)**: 장시간 유휴 연결 확인 (보통 2시간).
- **시간 대기 타이머 (Time-Wait Timer)**: 연결 종료 후 2MSL 동안 대기 (마지막 ACK 손실 대비 및 이전 패킷 소멸 대기).

---

## 9.5 스트림 제어 전송 프로토콜 (SCTP)

### 1. 개요
- UDP의 메시지 지향성과 TCP의 신뢰성/연결 지향성을 결합한 차세대 전송 프로토콜.
- 멀티미디어 통신 및 신뢰성 있는 전송에 적합.

### 2. 주요 특징
- **다중 스트림 (Multi-streaming)**:
    - 하나의 연결(Association) 내에 여러 개의 스트림을 논리적으로 분리.
    - **HOL(Head-of-Line) Blocking** 문제 해결 (한 스트림의 패킷 손실이 다른 스트림에 영향 주지 않음).
- **멀티홈잉 (Multihoming)**:
    - 하나의 연결에 여러 개의 IP 주소를 할당.
    - 주 경로 장애 시 예비 경로로 즉시 전환하여 내결함성(Fault Tolerance) 제공.
- **메시지 지향**: TCP처럼 바이트 스트림이 아니라, UDP처럼 메시지 경계를 유지함.

### 3. 패킷 구조
- **공통 헤더**: 소스/목적지 포트, 검증 태그(Verification Tag), 검사합(CRC-32).
- **청크 (Chunk)**: 데이터 또는 제어 정보를 담는 단위. 하나의 패킷에 여러 청크 포함 가능.
    - DATA, INIT, SACK, HEARTBEAT, SHUTDOWN 등.

### 4. 주요 절차
- **결합 설정 (4-Way Handshake)**:
    - INIT -> INIT ACK -> COOKIE ECHO -> COOKIE ACK.
    - **쿠키(Cookie)** 메커니즘을 사용하여 **SYN Flooding 공격**을 원천적으로 방지.
- **데이터 전송**:
    - **TSN (Transmission Sequence Number)**: 전체 데이터 청크의 순서 제어 (흐름/오류 제어용).
    - **SI (Stream Identifier)**: 스트림 구분.
    - **SSN (Stream Sequence Number)**: 각 스트림 내에서의 순서 번호.
- **결합 종료 (3-Way Handshake)**:
    - SHUTDOWN -> SHUTDOWN ACK -> SHUTDOWN COMPLETE.
    - TCP와 달리 Half-Close를 지원하지 않음.

### 5. 흐름 및 오류 제어
- **흐름 제어**: TCP와 유사하게 `rwnd` 기반으로 동작하지만, 바이트 단위가 아닌 청크 단위로 관리될 수 있음(구현에 따라 다름, 교재에서는 청크와 바이트 구분 설명).
- **오류 제어**: SACK 청크를 사용하여 손실된 청크(Gap)와 중복된 청크를 정확하게 보고.
