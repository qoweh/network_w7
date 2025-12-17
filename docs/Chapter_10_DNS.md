# 10장 응용층 (Application Layer) - Part 1

이 문서는 `ch 10-1 application.pdf` 파일의 내용을 바탕으로 **10.1 개요**, **10.2 클라이언트/서버 패러다임**, **10.3 표준 응용(WWW, HTTP, FTP, Email, Telnet/SSH, DNS)**에 대해 상세하게 정리한 것입니다.

## 10.1 응용층 개요 (Overview)

### 1. 역할
- 프로세스 간의 **논리적 연결**을 제공합니다.
- 전송층 서비스를 사용하여 사용자에게 직접적인 서비스를 제공합니다.
- 새로운 프로토콜의 추가, 변경, 제거가 용이합니다.

### 2. 응용층 패러다임
- **클라이언트-서버 (Client-Server) 패러다임**:
    - **서버**: 항상 실행되며 서비스를 제공 (비용 발생).
    - **클라이언트**: 필요할 때 실행되어 서비스를 요청.
    - 예: WWW, FTP, Email.
- **P2P (Peer-to-Peer) 패러다임**:
    - 동등한 피어(Peer)들이 서비스를 제공하기도 하고 이용하기도 함.
    - 확장성이 좋으나 보안 및 관리가 어려움.
- **혼합 패러다임**: P2P 연결을 위해 초기 탐색에 클라이언트-서버 방식을 사용.

---

## 10.2 클라이언트/서버 패러다임

### 1. 소켓 (Socket)
- **API (Application Programming Interface)**: 응용 프로그램이 운영체제의 TCP/IP 서비스를 이용하기 위한 명령어 집합.
- **소켓 주소**: **IP 주소 + 포트 번호**.
    - 서버: 로컬 IP + Well-known Port (또는 Registered Port).
    - 클라이언트: 로컬 IP + Ephemeral Port (임시 포트).

### 2. 전송층 서비스 선택
- **UDP**: 비연결형, 비신뢰성, 빠름. (DNS, DHCP, 멀티미디어)
- **TCP**: 연결형, 신뢰성, 바이트 스트림. (HTTP, FTP, SMTP)
- **SCTP**: 연결형, 신뢰성, 메시지 중심, 멀티홈잉.

---

## 10.3 표준 응용 (Standard Applications)

### 1. WWW (World Wide Web) & HTTP
- **구성 요소**: HTML (문서 형식), URL (주소), HTTP (전송 프로토콜).
- **웹 문서**: 정적(Static), 동적(Dynamic, CGI/JSP/PHP), 액티브(Active, Java/JS).
- **HTTP (HyperText Transfer Protocol)**:
    - **포트**: 80번 (TCP).
    - **비영속적(Non-persistent) 연결**: 객체 하나당 연결 생성/종료 (오버헤드 큼).
    - **영속적(Persistent) 연결**: 하나의 연결로 여러 객체 전송 (효율적).
- **메시지 형식**:
    - **요청(Request)**: 요청 라인(메소드, URL, 버전) + 헤더 + 본문.
        - 메소드: GET, POST, HEAD, PUT, DELETE 등.
    - **응답(Response)**: 상태 라인(버전, 상태 코드, 문구) + 헤더 + 본문.
        - 상태 코드: 200(성공), 301(이동), 400(클라이언트 오류), 404(없음), 500(서버 오류).
- **쿠키 (Cookie)**:
    - 비상태(Stateless)인 HTTP를 보완하여 상태 정보를 저장.
    - 서버가 `Set-Cookie` 헤더로 전송 -> 클라이언트가 저장 후 재요청 시 `Cookie` 헤더로 전송.
    - 활용: 장바구니, 로그인 유지, 사용자 맞춤 광고.
- **웹 캐싱 (Proxy Server)**:
    - 클라이언트와 서버 사이에서 응답을 저장하여 속도 향상 및 트래픽 감소.

### 2. FTP (File Transfer Protocol)
- **역할**: 호스트 간 파일 전송.
- **두 개의 연결 사용 (TCP)**:
    - **제어 연결 (포트 21)**: 명령/응답 전달. 세션 내내 유지.
    - **데이터 연결 (포트 20)**: 실제 파일 전송. 파일 전송 때마다 열고 닫힘.
- **전송 모드**: 스트림 모드(기본), 블록 모드, 압축 모드.
- **보안**: 기본 FTP는 평문 전송이므로 **sftp** (SSH 기반) 또는 **FTPS** (SSL/TLS 기반) 사용 권장.

### 3. 전자우편 (E-mail)
- **구조**:
    - **UA (User Agent)**: 사용자 인터페이스 (Outlook 등).
    - **MTA (Message Transfer Agent)**: 메일 서버 간 전송 (SMTP).
    - **MAA (Message Access Agent)**: 수신함에서 메일 가져오기 (POP3, IMAP4).
- **프로토콜**:
    - **SMTP (Simple Mail Transfer Protocol)**: **Push** 방식. 클라이언트->서버, 서버->서버 전송. (포트 25).
    - **POP3 (Post Office Protocol v3)**: **Pull** 방식. 다운로드 후 삭제(또는 유지). (포트 110).
    - **IMAP4 (Internet Mail Access Protocol v4)**: **Pull** 방식. 서버에 메일 저장/관리, 헤더 미리보기 등 기능 풍부. (포트 143).
- **MIME (Multipurpose Internet Mail Extension)**: ASCII가 아닌 데이터(한글, 이미지, 영상 등)를 텍스트로 변환하여 전송하기 위한 확장 프로토콜.

### 4. TELNET & SSH
- **TELNET**: 원격 로그인 프로토콜. (포트 23). 평문 전송으로 보안 취약. **NVT (Network Virtual Terminal)** 문자 집합 사용.
- **SSH (Secure Shell)**: TELNET의 보안 대체제. (포트 22).
    - 암호화된 원격 로그인, 파일 전송(sftp), 포트 포워딩(터널링) 제공.

### 5. DNS (Domain Name System)
- **역할**: 사람이 읽기 쉬운 **도메인 이름**을 컴퓨터가 사용하는 **IP 주소**로 변환 (또는 그 반대).
- **이름 공간 (Name Space)**: 계층적 구조 (Root -> TLD -> Domain -> Subdomain).
- **DNS 서버 계층**:
    - **Root Server**: 최상위 관리.
    - **TLD Server**: com, org, kr 등 관리.
    - **Authoritative Server**: 특정 도메인의 실제 정보를 가진 서버.
- **동작 방식**:
    - **재귀적(Recursive) 질의**: 서버가 대신 끝까지 찾아줌.
    - **반복적(Iterative) 질의**: "나는 모르니 저 서버에 물어봐"라고 알려줌.
- **캐싱 (Caching)**: 한 번 조회한 정보는 TTL(Time-To-Live) 동안 저장하여 재사용.
- **자원 레코드 (Resource Record)**:
    - `A`: IPv4 주소.
    - `AAAA`: IPv6 주소.
    - `NS`: 네임 서버.
    - `MX`: 메일 서버.
    - `CNAME`: 별칭(Canonical Name).
