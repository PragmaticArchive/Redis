### 기본 명령어(mac os 기준)

```bash
brew services start redis # redis 서버 실행
==> Successfully started `redis` (label: homebrew.mxcl.redis)
redis-cli # redis 사용을 위한 프로그램
127.0.0.1:6379>
brew services stop redis # redis 서버 종료
==> Successfully stopped `redis` (label: homebrew.mxcl.redis)
```
```redis
-- lecture라는 key에 inflearn-redis라는 value 한 쌍을 저장
127.0.0.1:6379> SET lecture inflearn-redis
OK
127.0.0.1:6379> GET lecture # key를 통해 저장한 value 조회
"inflearn-redis"
127.0.0.1:6379> DEL lecture # key를 통해 저장한 key-value 삭제
(integer) 1
127.0.0.1:6379> GET lecture
-- redis에서 데이터가 없음을 의미
(nil)
-- redis-cli 종료
127.0.0.1:6379> exit
```

### 목차
1. [String 데이터 타입](#string)
2. [List 데이터 타입](#list)
3. [Set 데이터 타입](#set)
4. [Hash 데이터 타입](#hash)
5. [Sorted Set 데이터 타입](#sorted-set)
6. [Stream 데이터 타입](#stream)
7. [Geospatial 데이터 타입](#geospatial)
8. [Bitmap 데이터 타입](#bitmap)
9. [Hyperloglog 데이터 타입](#hyperloglog)
10. [Bloomfilter 데이터 타입](#bloomfilter)
11. [데이터만료 Redis 특수명령어](#데이터만료)
12. [NX/XX Redis 특수명령어](#NX/XX)
13. [pub/sub Redis 특수명령어](#pub/sub)
14. [transaction Redis 특수명령어](#transaction)

### string

```redis
127.0.0.1:6379> SET lecture inflearn-redis
OK
-- 한꺼번에 여러개의 string 데이터 저장
127.0.0.1:6379> MSET price 100 language ko
OK
-- 한꺼번에 여러개의 value 조회
127.0.0.1:6379> MGET lecture price language
1) "inflearn-redis"
2) "100"
3) "ko"
-- 숫자형 string 데이터에 +1
127.0.0.1:6379> INCR price
(integer) 101
-- 숫자형 string 데이터에 숫자만큼 +
127.0.0.1:6379> INCRBY price 9
(integer) 110
-- JSON 형태의 직렬화된 객체형태의 string 데이터도 저장가능
127.0.0.1:6379> SET inflearn-redis '{"price": 100, "language": "ko"}'
OK
-- JSON 형태의 데이터가 직렬화되어 저장된 결과 조회(파싱 필요함)
127.0.0.1:6379> GET inflearn-redis
"{\"price\": 100, \"language\": \"ko\"}"
-- 데이터 저장시 :(콜론)을 이용하여 의미 분류
127.0.0.1:6379> SET inflearn-redis:ko:price 200
OK
```

### list

```redis
-- queue라는 key에 job1,2,3 value들을 push(왼->오)
127.0.0.1:6379> LPUSH queue job1 job2 job3
(integer) 3
-- 가장 오른쪽의 데이터 pop(왼쪽에서 넣었으니까 큐와 같이 작동)
127.0.0.1:6379> RPOP queue
"job1"
127.0.0.1:6379> LPUSH stack job1 job2 job3
(integer) 3
-- 가장 왼쪽의 데이터 pop(왼쪽에서 넣었으니까 스택처럼 작동)
127.0.0.1:6379> LPOP stack
"job3"
127.0.0.1:6379> LPUSH queue job4 job5
(integer) 4
-- 0(왼쪽에서 첫번째) ~ -1(오른쪽에서 첫번째) 인덱스 데이터 모두 조회
127.0.0.1:6379> LRANGE queue 0 -1
1) "job5"
2) "job4"
3) "job3"
4) "job2"
-- 0 ~ 1인덱스 데이터만 남겨두고 모두 삭제
127.0.0.1:6379> LTRIM queue 0 1
OK
```

### set

```redis
-- set형태의 데이터 추가(중복제거)
127.0.0.1:6379> SADD user:1:fruits apple banana orange orange
(integer) 3
-- 해당 집합 내부 데이터 조회(순서보장X)
127.0.0.1:6379> SMEMBERS user:1:fruits
1) "apple"
2) "banana"
3) "orange"
-- 해당 집합의 원소 개수 조회
127.0.0.1:6379> SCARD user:1:fruits
(integer) 3
-- 해당 집합에 해당 데이터가 포함되는지 여부 조회
127.0.0.1:6379> SISMEMBER user:1:fruits banana
(integer) 1
127.0.0.1:6379> SADD user:2:fruits apple lemon
(integer) 2
-- 두 집합 간의 교집합 원소 조회
127.0.0.1:6379> SINTER user:1:fruits user:2:fruits
1) "apple"
-- 두 집합 중 첫번째 집합에만 포함, 두번째에는 포함 X
127.0.0.1:6379> SDIFF user:1:fruits user:2:fruits
1) "banana"
2) "orange"
-- 두 집합의 합집합
127.0.0.1:6379> SUNION user:1:fruits user:2:fruits
1) "apple"
2) "banana"
3) "orange"
4) "lemon"
```

### hash

```redis
-- lecture라는 key에 name, price, language 필드의 데이터를 저장
127.0.0.1:6379> HSET lecture name inflearn-redis price 100 language ko
(integer) 3
-- lecture 키의 price라는 필드의 value 1개만 조회
127.0.0.1:6379> HGET lecture price
"100"
127.0.0.1:6379> HGET lecture name
"inflearn-redis"
127.0.0.1:6379> HGET lecture language
"ko"
-- 한꺼번에 여러개의 필드의 value 조회가능
127.0.0.1:6379> HMGET lecture name price language invalid
1) "inflearn-redis"
2) "100"
3) "ko"
-- 만약에 존재하지 않는 필드값을 조회할 경우 유효하지 않음을 의미하는 nil을 출력
4) (nil)
127.0.0.1:6379> HINCRBY lecture price 10
(integer) 110
```

### sorted-set

```redis
-- score에 따라 오름차순 저장(같으면 사전순)
127.0.0.1:6379> ZADD points 10 TeamA 10 TeamB 50 TeamC
(integer) 3
127.0.0.1:6379> ZRANGE points 0 -1
1) "TeamA"
2) "TeamB"
3) "TeamC"
-- REV 옵션에 의해 역순, WITHSCORES에 의해 점수 함께 출력
127.0.0.1:6379> ZRANGE points 0 -1 REV WITHSCORES
1) "TeamC"
2) "50"
3) "TeamB"
4) "10"
5) "TeamA"
6) "10"
-- 인덱스 조회
127.0.0.1:6379> ZRANK points TeamB
(integer) 1
127.0.0.1:6379> ZRANK points TeamC
(integer) 2
127.0.0.1:6379> ZRANK points TeamA
(integer) 0
```

### stream

```redis
-- events라는 스트림을 생성하고, unique id를 자동할당하여 뒤에 저장할 이벤트 내용을 추가해주었다
127.0.0.1:6379> XADD events * action like user_id 1 product_id 1
"1702285226161-0"
-- 가장 처음에 들어왔던 이벤트 ~ 가장 마지막에 들어온 이벤트까지 조회
127.0.0.1:6379> XRANGE events - +
1) 1) "1702285226161-0"
   2) 1) "action"
      2) "like"
      3) "user_id"
      4) "1"
      5) "product_id"
      6) "1"
127.0.0.1:6379> XADD events * action like user_id 2 product_id 2
"1702285527439-0"
127.0.0.1:6379> XRANGE events - +
1) 1) "1702285226161-0"
   2) 1) "action"
      2) "like"
      3) "user_id"
      4) "1"
      5) "product_id"
      6) "1"
-- 이벤트를 추가하고 나니까, 해당 스트림에 이벤트가 추가됨
2) 1) "1702285527439-0"
   2) 1) "action"
      2) "like"
      3) "user_id"
      4) "2"
      5) "product_id"
      6) "2"
127.0.0.1:6379> XDEL events 1702285226161-0
(integer) 1
```

### geospatial

```redis
-- redis에 좌표정보를 추가할 때는 경도를 먼저 명시해주어야 정상작동한다
127.0.0.1:6379> GEOADD seoul:station 126.923917 37.556944 hong-dae 127.027583 37.497928 gang-nam
(integer) 2
-- 두 장소 간의 거리(좌표기준) 계산 결과 조회 -> KM 옵션을 통해 단위도 지정가능
127.0.0.1:6379> GEODIST seoul:station hong-dae gang-nam KM
"11.2561"

-- geofencing
127.0.0.1:6379> GEOADD gang-name:burgers 127.025705 37.501272 five-guys 127.025699 37.502775 shake-shack 127.028747 37.498668 mc-donalds 127.027531 37.498847 burger-king
(integer) 4
-- 지정한 반경 내에 있는 경도, 위도를 가진 데이터 조회
127.0.0.1:6379> GEORADIUS gang-name:burgers 127.027583 37.497928 0.5 KM
1) "burger-king"
2) "mc-donalds"
3) "five-guys"
```

### bitmap

```redis
-- 1로 bit 추가
127.0.0.1:6379> SETBIT user:log-in:23-01-01 123 1
(integer) 0
-- 0으로 bit 제거
127.0.0.1:6379> SETBIT user:log-in:23-01-01 123 0
(integer) 1
127.0.0.1:6379> SETBIT user:log-in:23-01-01 123 1
(integer) 0
127.0.0.1:6379> SETBIT user:log-in:23-01-01 456 1
(integer) 0
127.0.0.1:6379> SETBIT user:log-in:23-01-02 123 1
(integer) 0
127.0.0.1:6379> BITCOUNT user:log-in:23-01-01
-- 23-01-01에 로그인한 유저는 2명이다
(integer) 2
127.0.0.1:6379> BITCOUNT user:log-in:23-01-02
-- 23-01-02에 로그인한 유저는 1명이다
(integer) 1
127.0.0.1:6379> BITOP AND result user:log-in:23-01-01 user:log-in:23-01-02
-- BITOP + AND/OR/XOR 등으로 두 오프셋 이어붙이기 가능 -> 결과는 새롭게 비트맵 만들어서 저장
(integer) 58
127.0.0.1:6379> BITCOUNT result
-- AND 위에서 한 결과(01과 02에 둘다로그인한 유저수)는 1
(integer) 1
127.0.0.1:6379> GETBIT result 123
-- 오프셋을 추가한 결과 123은 01과 02 둘다 로그인함
(integer) 1
127.0.0.1:6379> GETBIT result 456
-- 오프셋을 추가한 결과 456은 01과 02 둘다 로그인 안함(하나만 로그인 함)
(integer) 0
```

### hyperloglog

```redis
127.0.0.1:6379> PFADD fruits apple orange grape kiwi
(integer) 1
127.0.0.1:6379> PFCOUNT fruits
(integer) 4
-- 중복제거
127.0.0.1:6379> PFADD fruits apple
(integer) 0
-- 데이터 개수가 엄청 많아지면 정확하지 않은 카디널리티 근삿값 반환
127.0.0.1:6379> PFCOUNT fruits
(integer) 4

-- hyperloglog vs set
bash> for ((i=1; i<=1000; i++)); do redis-cli SADD k1 $i; done
127.0.0.1:6379> MEMORY USAGE k1
(integer) 48304
127.0.0.1:6379> SCARD k1
(integer) 1000
bash> for ((i=1; i<=1000; i++)); do redis-cli PFADD k2 $i; done
127.0.0.1:6379> MEMORY USAGE k2
-- set에 비해 아주 적은 메모리 사용
(integer) 2104
127.0.0.1:6379> SCARD k1
-- but, 정확하지 않은 카디널리티
(integer) 1001
```

### bloomfilter

```bash
# docker에서 redis 서버 띄우고 접속(docker 사용, 별도의 모듈설치대신 도커 사용)
docker run -p 63790:6379 -d --rm redis/redis-stack-server # 도커를 통해 해당하는 이미지 컨테이너 띄우기(없으면 알아서 다운로드해준다)
edf630063b401b7f6dde8936d0a1db4a1eed40ab250c076d5375230d34ec94df
docker ps # 도커 프로세스 확인해보면, 우리가 띄운 서버가 잘 실행중임
CONTAINER ID   IMAGE                      COMMAND            CREATED          STATUS          PORTS                     NAMES
edf630063b40   redis/redis-stack-server   "/entrypoint.sh"   33 seconds ago   Up 32 seconds   0.0.0.0:63790->6379/tcp   elated_raman
redis-cli -p 63790 # 63790 포트를 통해서 redis-cli 에 접속하여 redis를 사용한다
127.0.0.1:63790>
```
```redis
-- 여러개를 추가하기 때문에 MADD(멀티애드)
127.0.0.1:63790> BF.MADD fruits apple orange grape
1) (integer) 1
2) (integer) 1
3) (integer) 1
127.0.0.1:63790> BF.EXISTS fruits apple
-- 있는데 없다고 하지는 않음
(integer) 1
127.0.0.1:63790> BF.EXISTS fruits banana
-- 이때, 여기서 false positive가 발생한다면 없는 데이터가 있다고 1을 반환할 수도 있다
(integer) 0
```

### 데이터만료

```redis
127.0.0.1:6379> SET greeting hello
OK
127.0.0.1:6379> TTL greeting
-- TTL 명령어의 결과로 -1이 반환 -> 만료시간이 설정되어있지 않음
(integer) -1
-- 10초로 만료시간 설정
127.0.0.1:6379> EXPIRE greeting 10
(integer) 1
127.0.0.1:6379> TTL greeting
-- 5초 남음
(integer) 5
127.0.0.1:6379> TTL greeting
(integer) 4
127.0.0.1:6379> TTL greeting
(integer) 2
127.0.0.1:6379> GET greeting
-- 만료시간이 지나 greeting 데이터가 유효하지 않음
(nil)
-- 데이터 저장과 동시에 만료시간 설정
127.0.0.1:6379> SETEX greeting2 10 hi
OK
127.0.0.1:6379> GET greeting2
"hi"
127.0.0.1:6379> TTL greeting2
(integer) 1
127.0.0.1:6379> GET greeting2
(nil)
```

### NX/XX

```redis
127.0.0.1:6379> SET greeting hello NX
-- greeting이라는 key를 가진 데이터가 없기 때문에 저장 성공
OK
127.0.0.1:6379> GET greeting
"hello"
127.0.0.1:6379> SET greeting hi XX
-- greeting이라는 key를 가진 데이터가 있기 때문에 저장 성공
OK
127.0.0.1:6379> GET greeting
"hi"
127.0.0.1:6379> SET invalid abcd XX
-- greeting이라는 key를 가진 데이터가 없기 때문에 저장 실패
(nil)
127.0.0.1:6379> SET greeting hello NX
-- greeting이라는 key를 가진 데이터가 있기 때문에 저장 실패
(nil)
```

### pub/sub

```redis
-- order & payment 2개의 채널 구독
127.0.0.1:6379> SUBSCRIBE ch:order ch:payment
1) "subscribe"
2) "ch:order"
3) (integer) 1
1) "subscribe"
2) "ch:payment"
-- 2개의채널을 구독중임
3) (integer) 2
-- order 채널에 새로운 메시지 발행
127.0.0.1:6379> PUBLISH ch:order new-order
(integer) 1
1) "message"
2) "ch:order"
-- 위 발행 결과, message 타입의 order 채널로부터 new-order라는 메시지가 구독됨
3) "new-order"
127.0.0.1:6379> PUBLISH ch:payment new-payment
(integer) 1
1) "message"
2) "ch:payment"
3) "new-payment"
127.0.0.1:6379> PUBLISH ch:delievery new-delivery
-- 해당 채널을 구독중인 subscriber가 없어서 아무도 안읽음
(integer) 0
```

### transaction

```redis
-- TX1 시작
127.0.0.1:6379> MULTI
OK
127.0.0.1:6379(TX)> INCR foo
QUEUED
127.0.0.1:6379(TX)> DISCARD
-- TX1 끝(롤백 - 적용 X)
OK
127.0.0.1:6379> GET foo
-- 롤백했기 때문에 적용안돼서 (nil) 반환
(nil)
-- TX2 시작
127.0.0.1:6379> MULTI
OK
127.0.0.1:6379(TX)> INCR foo
QUEUED
127.0.0.1:6379(TX)> EXEC
-- TX2 끝(커밋 - 적용 O)
1) (integer) 1
127.0.0.1:6379> GET foo
-- 커밋했기 때문에 적용돼서 1 반환
"1"
-- TX3 시작
127.0.0.1:6379> MULTI
OK
127.0.0.1:6379(TX)> INCR foo
QUEUED
127.0.0.1:6379(TX)> INCR foo2
QUEUED
127.0.0.1:6379(TX)> DISCARD
-- TX3 끝
OK
127.0.0.1:6379> GET foo
"1"
127.0.0.1:6379> GET foo2
(nil)
-- TX4 시작
127.0.0.1:6379> MULTI
OK
127.0.0.1:6379(TX)> INCR foo
QUEUED
127.0.0.1:6379(TX)> INCR foo2
QUEUED
127.0.0.1:6379(TX)> EXEC
-- TX4 끝
1) (integer) 2
2) (integer) 1
127.0.0.1:6379> GET foo
"2"
127.0.0.1:6379> GET foo2
"1"
```
