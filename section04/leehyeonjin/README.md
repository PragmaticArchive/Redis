# Redis 데이터 타입 활용

---

### 1. String - One Time Password(임시 비밀번호)

- OTP : 인증을 위해 사용되는 임시 비밀번호(eg. 6자리 랜덤 숫자).

<img width="563" alt="Untitled (11)" src="https://github.com/hgene0929/hgene0929/assets/90823532/ea69593b-74d2-4fef-b404-0ec9db4b489d">

---

### 2. String - Distributed Lock(분산 락)

- 분산 환경의 다수의 프로세스에서 동일한 자원에 접근할 때, 동시성 문제 해결.
- RDB에 레코드 락을 설정하는 방법도 있으나, 레코드가 없거나 레코드 락으로 인한 RDB의 성능저하가 발생할 수 있으므로 필요에 따라 둘을 적절하게 배치하여 사용할 필요가 있다.

<img width="570" alt="Untitled (12)" src="https://github.com/hgene0929/hgene0929/assets/90823532/9e1f7892-2306-4d41-8644-ea55070646ce">

---

### 3. String - Fixed Window Rate Limiter(비율 계산기)

- 시스템 안정성/보안을 위해 요청의 수를 제한하는 기술.
- Ex : IP-Based, User-Based, Application-Based, etc.
    - 특정 IP별, 특정 사용자, 특정 애플리케이션을 기준으로 초당 요청 횟수 제한.
- Fixed-window Rate Limiting : 고정된 시간(eg. 1분) 안에 요청 수를 제한하는 방법.
    - 기준 분이 갱신되면 새롭게 데이터 요청 허용여부 측정시작.
    - 최악의 경우, 10분(10분 기준이라면)까지 데이터가 몰려있다가, 11분부터 아예 없을 수 있다.

<img width="585" alt="Untitled (13)" src="https://github.com/hgene0929/hgene0929/assets/90823532/0cb6c3e8-a06a-4a40-8483-29cc7e8167a9">
<img width="585" alt="Untitled (14)" src="https://github.com/hgene0929/hgene0929/assets/90823532/988e92f5-03db-4f24-9b3a-76c478cd3851">

---

### 4. List - SNS Activity Feed(소셜 네트워크 활동 피드)

- 활동 피드 : 사용자 또는 시스템과 관련된 활동이나 업데이트를 시간순으로 정렬하여 보여주는 기능.
- Fan-Out : 단일 데이터를 한 소스에서 여러 목적지로 동시에 전달하는 메시징 패턴.

<img width="684" alt="Untitled" src="https://github.com/hgene0929/hgene0929/assets/90823532/e53a7645-8b89-4e3b-a726-5957c491a334">

---

### 5. Set - Shopping Cart(장바구니)

- 장바구니 :
    - 사용자가 구매를 원하는 상품을 임시로 모아두는 가상의 공간.
    - 수시로 변경이 발생할 수 있고, 실제 구매로 이어지지 않을 수도 있다.
- redis set을 이용하면 중복 상품에 대한 처리를 쉽게 할 수 있고, 임시 데이터 처리를 효율적으로 할 수 있다.

<img width="522" alt="Untitled (1)" src="https://github.com/hgene0929/hgene0929/assets/90823532/5cf36e45-ecc1-480e-9347-dab4e209bf61">

---

### 6. Hash - Login Session(로그인 세션)

- 로그인 세션 : 사용자의 로그인 상태를 유지하기 위한 기술.
    - 사용자가 로그인하면 서버는 사용자를 식별하기 위한 세션 ID를 부여하고, 해당 ID를 통해 이후의 요청에 대한 별도의 인증 없이 자원을 제공하는 기술.
- 동시 로그인 제한 : 로그인시 세션의 개수를 제한하여, 동시에 로그인 가능한 디바이스 개수 제한.

<img width="638" alt="Untitled (2)" src="https://github.com/hgene0929/hgene0929/assets/90823532/548146cc-26df-415b-97af-87ebe6dee940">

---

### 7. Sorted Set - Sliding Window Rate Limiter(비율 계산기)

- Sliding Window Rate Limiter : 시간에 따라 Window를 이동시켜 동적으로 요청 수를 조절하는 기술.
- Fixed Window vs Sliding Window :
    - Fixed Window는 window 시간마다 허용량이 초기화 되지만, Sliding Windowsms 시간이 경과함에 따라 window가 같이 움직인다.

<img width="623" alt="Untitled (3)" src="https://github.com/hgene0929/hgene0929/assets/90823532/140c7f4e-099b-49bf-b370-7651c3d039c5">
<img width="656" alt="Untitled (4)" src="https://github.com/hgene0929/hgene0929/assets/90823532/d5c4925a-1928-4205-8249-b256ddb18568">

---

### 8. GeoSpatial - Geofencing(반경 탐색)

- Geofencing : 위치를 활용하여 지도 상의 가상의 경계 또는 지리적 영역을 정의하는 기술.
- 명령어 :

```redis
-- 반경거리 만큼의 위치 내에 있는 경도, 위도 지역목록 찾기
GEORADIUS [지역명] [경도] [위도] [반경거리]
```

### 9. Bitmap - User Online Status(온라인 상태 표시)

- Online Status : 사용자의 현재 상태를 표시하는 기술.
- 특징 : 실시간성을 완벽히 보장하지는 않는다. 수시로 변경되는 값이다.

<img width="533" alt="Untitled (5)" src="https://github.com/hgene0929/hgene0929/assets/90823532/f88fe5c2-2e82-4913-8422-3424c43848e1">

### 10. HyperLogLog - Visitors Count(방문자 수 계산)

- visitors count approximation : 방문자 수(또는 특정 횟수)를 대략적으로 추정하는 경우.
- 정확한 횟수를 셀 필요 없이 대략적인 어림치만 알고자 하는 경우.
- 명령어 :

```redis
PFADD today:users
        -- 방문자수 키에 user id 와 방문시간을 초단위로 구분하여 초마다 방문 횟수를 다르게 카운트
	user:1:1693494070
	user:1:1693494071
	user:2:1693494071
PFCOUNT today:users
```

### 11. BloomFilter - Unique Events(중복 이벤트 제거)

- Unique Events : 동일 요청이 중복으로 처리되지 않기 위해 빠르게 해당 item이 중복인지 확인하는 방법.
    - 일반적으로 redis가 관계형 db에 비해 빠르기 때문에 일반 rdb가 받게 될 부하를 redis에 분산시켜주는 기술.

<img width="649" alt="Untitled (6)" src="https://github.com/hgene0929/hgene0929/assets/90823532/2175443c-87e4-4771-9b5b-ab878d91138a">
<img width="649" alt="Untitled (7)" src="https://github.com/hgene0929/hgene0929/assets/90823532/5d48c3fd-9637-4caa-93e1-e1c376e296a5">
