# Redis 특수 명령어

---

### 1. 데이터 만료(Expiration)

- 데이터를 특정시간 이후에 만료시키는 기능.
- TTL(Time To Live) : 데이터가 유효한 시간(초 단위).
- 특징 :
    - 데이터 조회 요청시에 만료된 데이터는 조회되지 않음.
    - 데이터가 만료되자마자 삭제하지는 않고, 만료로 표시했다가 백그라운드에서 주기적으로 삭제.
- 명령어 :

```redis
SET [key] [value]
-- 초단위 지정시간 이후에 만료시킴
EXPIRE [key] [시간(초단위)]
-- key에 해당하는 데이터의 남은시간을 조회
TTL [key]
-- 데이터를 저장함과 동시에 만료시간 설정
SETEX [key] [시간(초단위)] [value]
```

### 2. SET NX/XX

- NX : 해당 key가 존재하지 않는 경우에만 SET(새로운 데이터 저장).
- XX : 해당 key가 이미 존재하는 경우에만 SET.
- Null Reply : SET이 동작하지 않은 경우(NX/XX 명령이 조건에 부합하지 않은 경우) (nil) 응답.
- 명령어 :

```redis
-- 해당 key가 존재하지 않아야 성공
SET [key] [value] NX
-- 해당 key가 존재해야 성공
SET [key] [value] XX
```

### 3. Pub/Sub

- 시스템 사이에 메시지를 통신하는 패턴 중 하나.
- Publisher와 Subscriber가 서로 알지 못해도 통신이 가능하도록 decoupling된 패턴.
    - 두 시스템 간에 강한 coupling(결합도)을 낮출 수 있음.
- Publisher는 Subscriber에게 직접 메시지를 보내지 않고, Channel에 Publish.
- Subscriber는 관심이 있는 Channel을 필요에 따라 Subscribe하며 메시지 수신.
    - Publisher의 역할이 변경되어도 Subscriber는 이를 신경쓰지 않고, 자신이 구독중인 채널에만 집중하면 됨.
- stream vs pub/sub :
    - pub/sub : stream 메시지가 보관되는 stream과 달리 pub/sub은 subscribe하지 않을 때 발행된 메시지는 수신 불가하다.

<img width="469" alt="Untitled (10)" src="https://github.com/hgene0929/hgene0929/assets/90823532/d61f890d-bee6-476a-9101-976575a5cc70">

- 원리 : stock은 order 채널만 구독하면 되고, notification은 필요에 따라 모든 채널을 구독하는 형태.
- 명령어 :

```redis
SUBSCRIBE [채널명] [채널명] ...
PUBLISH [채널명] [채널명]
```

### 4. Pipeline

- 다수의 commands를 한번에 요청하여 네트워크 성능을 향상시키는 기술.
- Round-Trip Times 최소화 ⇒ 전체적인 지연시간을 줄일 수 있음.
    - 요청과 응답에 따라 네트워크를 통해 트래픽이 오고 가며 round-trip time이 증가하는데, 이를 최소화하는 기술.
- 대부분의 redis 클라이언트 라이브러리에서 지원.

### 5. Transaction

- 다수의 명령을 하나의 트랜잭션으로 처리 → 원자성 보장.
    - 원자성(atomicity) : all or nothing → 모든 작업이 적용되거나 하나도 적용되지 않거나.
- 작업 처리 중 에러가 발생하면 모든 작업 rollback.
- 하나의 트랜잭션이 처리되는 동안 다른 클라이언트의 요청이 중간에 끼어들 수 없음.
- pipeline vs transaction :
    - pipeline : 네트워크 퍼포먼스 향상을 위해 여러개의 명령어를 한번에 요청.
    - tranaction : 작업의 원자성을 보장하기 위해 다수의 명령을 하나처럼 처리하는 기술.
        - pipeline과 transaction을 동시에 사용가능.
- 명령어 :

```redis
-- 트랜잭션 시작
MULTI
-- 트랜잭션 범위의 작업 명령
INCR foo
-- 트랜잭션 롤백
DISCARD
-- 트랜잭션 커밋
EXEC
```
