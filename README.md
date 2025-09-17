# 🏃‍♂️ Tag Arena

**Tag Arena**는 Spring Boot와 WebSocket(STOMP), Redis를 활용해 만든 **실시간 멀티플레이 술래잡기 게임 서버**입니다.  
플레이어는 방에 접속해 자유롭게 이동하며, 술래는 일정 반경 내의 도망자를 태그하여 역할을 교체할 수 있습니다.  
모든 게임 상태는 서버 권위(Server Authoritative)로 관리되며, Redis와 Lua 스크립트를 통해 원자적·동시성 안전하게 처리됩니다.  

---

## 📖 프로젝트 개요
- **프로젝트명:** Tag Arena  
- **주제:** 실시간 좌표 기반 멀티플레이 술래잡기  
- **목표:**  
  - WebSocket 기반 실시간 통신 이해  
  - Redis를 통한 분산 상태 관리와 동시성 제어 학습  
  - 서버 권위 방식(Server Authoritative) 적용으로 치트 방지 및 확장성 고려  

---

## 🛠 기술 스택
### Backend
- **Java 17+**
- **Spring Boot 3.x**
- **Spring WebSocket (STOMP)**
- **Spring Messaging**
- **Spring Data Redis (Lettuce)**
- **Lua Script** (원자적 태깅 처리)
- (옵션) **Spring Security + JWT**

### Infra
- **Redis 7.x**
- **Docker Compose** (로컬 개발 환경)

### Client (예정)
- **HTML / JavaScript**
- **SockJS + STOMP.js**

---

## 📐 아키텍처
```
Client (Browser)
   └── SockJS + STOMP.js
         │
         ▼
Spring Boot (WebSocket/STOMP)
   ├─ GameController (@MessageMapping)
   ├─ GameService (좌표/태깅 로직)
   ├─ Redis (좌표/역할 상태 저장)
   └─ Lua Script (원자적 연산)

   Broadcast → /topic/room.{roomId}
```

---

## 🚀 주요 기능
- **방 생성 및 입장**: 플레이어 등록 및 최초 술래 자동 지정  
- **실시간 이동**: 좌표 갱신 후 서버에서 검증 → 전체 브로드캐스트  
- **술래 태깅**:  
  - 서버에서 반경 계산 및 쿨다운 검증  
  - Redis Lua 스크립트로 원자적 역할 전환  
- **상태 동기화**: `/topic/room.{roomId}`를 통해 모든 참가자에게 실시간 전파  
- (추가 예정) 아이템, 타이머, 랭킹 시스템  

---

## 📂 프로젝트 구조
```
src/main/java/com/example/tag/
 ├─ global/
 │   ├─ config/        # WebSocket, Redis, 보안 설정
 │   └─ util/          # 거리 계산 등 유틸
 ├─ domain/
 │   ├─ room/          # Room 엔티티 및 서비스
 │   ├─ player/        # PlayerState, PlayerService
 │   └─ game/          # GameService, DTO, 규칙
 ├─ infra/
 │   └─ redis/         # RedisRepo, Lua 스크립트 관리
 └─ api/
     └─ GameController.java
```

---

## 🔑 Redis Key 설계
```
room:{roomId}:players           # Set<playerId>
room:{roomId}:player:{pid}      # Hash{x,y,role,ts}
room:{roomId}:tagger            # String (현재 술래 playerId)
```

---

## 📦 실행 방법
### 1) Redis 실행
```bash
docker-compose up -d
```

### 2) 서버 실행
```bash
./gradlew bootRun
```

### 3) 클라이언트 접속
- WebSocket Endpoint: `ws://localhost:8080/ws-tag`  
- STOMP App Prefix: `/app/**`  
- STOMP Topic: `/topic/room.{roomId}`  

---

## 📌 학습 포인트
- **실시간 통신:** WebSocket(STOMP) 기반 이벤트 흐름  
- **분산 상태 관리:** Redis를 활용한 멀티플레이어 게임 동기화  
- **동시성 제어:** Lua 스크립트 기반 원자적 연산 처리  
- **서버 권위 패턴:** 좌표 검증 및 태깅 로직 서버 단일 제어  
- **확장성:** 멀티 인스턴스 확장 시 Redis Pub/Sub으로 브로드캐스트 가능  

---

## 📅 로드맵
- [x] 기본 방 생성 및 플레이어 입장  
- [x] 좌표 이동 동기화  
- [x] 술래 태깅 (Lua 스크립트 적용)  
- [ ] 클라이언트 UI (HTML/JS)  
- [ ] 아이템/버프 시스템  
- [ ] 랭킹/리플레이 기능  

---

## 📜 라이선스
이 프로젝트는 MIT 라이선스를 따릅니다.  
