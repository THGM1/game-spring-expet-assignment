# WebCraft Game Server 
Spring Boot 기반의 실시간 멀티플레이어 게임 서버입니다. REST API로 플레이어 등록과 월드 관리를,
WebSocket으로 실시간 채팅·이동·접속자 조회 등을 처리합니다.

## 기술 스택

- **Language / Framework**: Java, Spring Boot
- **DB**: MySQL (JPA / Hibernate)
- **Cache / Presence**: Redis (Sorted Set 기반 접속 상태 관리)
- **실시간 통신**: Spring WebSocket
- **테스트**: JUnit 5, Mockito, MockMvc

## REST API


| Method | Path | 설명       |
|---|---|----------|
| POST | `/players` | 플레이어 등록  |
| GET | `/worlds` | 월드 목록 조회 |
| POST | `/worlds` | 월드 생성    |
| GET | `/worlds/{worldId}/chats` | 최근 채팅 조회 |

## WebSocket 프로토콜

### 연결

```
ws://localhost:8080/ws/worlds/{worldId}?nickname={nickname}
```

- `worldId`: 월드 목록/생성 응답의 `id`
- `nickname`: `POST /players`로 등록된 닉네임

핸드셰이크에 성공 시 `101 Switching Protocols`

### 종료 코드

| 코드   | 상황 |
|------|---|
| 1000 | 정상 종료 |
| 4000 | 닉네임 누락 또는 등록되지 않은 닉네임 |
| 4001 | 접속할 수 없는 월드 ID |
| 4002 | 같은 월드에 같은 닉네임이 이미 접속 중 (기존 연결은 유지) |

허용되지 않은 Origin은 핸드셰이크에서 `403`, 서버 준비 전 요청은 `503`으로 거절

### 클라이언트 → 서버 메시지

| type | 설명 |
|---|---|
| `ping` | 연결 확인. 15초 주기로 전송 권장 |
| `chat` | 채팅 전송 (`content`, 1~200자) |
| `move` | 위치/시선/이동 상태 전달 |
| `onlineUsers` | 현재 월드 접속자 목록 조회 |

### 서버 → 클라이언트 메시지

| type | 설명 |
|---|---|
| `pong` | ping에 대한 응답 |
| `chat` | 같은 월드 전체에 브로드캐스트되는 채팅 (`sender`, `content`, `timestamp`) |
| `onlineUsers` | 조회 요청자에게만 반환되는 접속자 목록 (`users`, `count`) |
| `error` | 오류 코드 응답 (`code`) |

#### WebSocket 오류 코드

| code | 설명 |
|---|---|
| INVALID_JSON | 받은 텍스트를 JSON으로 읽을 수 없음 |
| INVALID_MESSAGE | JSON 객체가 아니거나 필수 필드/값이 잘못됨 |
| UNKNOWN_TYPE | type이 없거나 처리할 핸들러가 없음 |
| INTERNAL_ERROR | 처리 중 예기치 못한 서버 오류 |
| CHAT_COOLDOWN | 10초 구간 채팅 허용 횟수(5건) 초과 |

## 참고

- API 명세: https://f-api.github.io/game-spring-api-docs/expert/api-docs.html