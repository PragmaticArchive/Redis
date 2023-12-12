# Redis 데이터타입

---

### 1. Strings

- 문자열, 숫자, serialized object(JSON string) 등 저장.
- 명령어 :

```redis
-- key-value 형태의 문자열 데이터 1쌍 저장
SET [key] [value]
-- key-value 형태의 문자열 데이터 여러쌍 저장(multi-set), 
-- 이때 value는 단순문자열 뿐만 아니라 serialized object(직렬화된 객체)도 저장 가능
-- 또한 key에 콜론(:)을 이용하여 의미를 분류
MSET [key] [value] [key] [value] ...
-- 여러개의 key 들의 value 한꺼번에 조회(multi-get)
MGET [key] [key] [key] ...
-- key의 value + 1, 
-- 이때 해당 key의 value는 숫자형 문자열 데이터(eg. "1","2", ...)
INCR [key]
-- key의 value + number
INCRBY [key] [number]
```

### 2. Lists

- String을 Linked List로 저장.
- push/pop에 최적화 O(1).
- Queue(FIFO) / Stack(FILO) 구현에 사용.
- 명령어 :

```redis
-- list 형태의 자료구조에 여러개의 item 추가,
-- 이때 데이터는 (왼쪽 -> 오른쪽)순으로 추가됨
LPUSH [list] [item] [item] [item]
-- list 형태의 자료구조의 오른쪽에서 데이터 pop,
-- 이때 데이터가 오른쪽부터 채워지기 때문에 가장 오래된 데이터부터 pop하는 형태(like.queue)
RPOP [list]
-- list 형태의 자료구조의 왼쪽에서 데이터 pop,
-- 이때 데이터가 왼쪽부터 채워지기 때문에 가장 최근의 데이터부터 pop하는 형태(like.stack)
LPOP [list]
-- 리스트 인덱스(왼쪽부터 : 0,1,2,... | 오른쪽부터 : -1,-2,-3,...)
LRANGE [key] [start_idx] [end_idx]
-- 0~0까지 범위의 데이터를 남겨두고 모두 삭제
LTRIM [key] [start_idx] [end_idx]
```

### 3. Sets

- 중복없이 Unique한 string을 저장하는 정렬되지 않은 집합.
- Set Operation 사용 가능(eg. intersection, union, difference).
- 명령어 :

```redis
SADD [set] [item]
SMEMBERS [set]
-- set의 카디널리티(보유한 아이템 개수) 조회
SCARD [set]
-- value가 key의 집합에 포함되는지 여부 조회
SISMEMBER [set] [item]
-- 교집합 조회
SINTER [set] [set]
-- 차집합(앞 - 뒤)
SDIFF [set] [set]
-- 합집합
SUNION [set] [set]
```

### 4. Hashes

- field-value 구조를 갖는 데이터 타입( 프로그래밍 언어의 딕셔너리 or 맵과 유사 ).
- 다양한 속성을 갖는 객체의 데이터를 저장할 때 유용.
- 명령어 :

```redis
HSET [key] [field] [value] [field] [value] ...
-- 하나의 필드
HGET [key] [field]
-- 여러개 필드
HMGET [key] [field] [field]
HINCRBY [key] [field] [number]
```

### 5. Sorted Sets

- 중복없이 Unique한 string을 연관된 score를 통해 정렬된 집합(Set의 기능 + 추가로 score 속성 저장).
- 내부적으로 skip list + hash table로 이루어져 있고, score 값에 따라 정렬 유지.
- score가 동일하면 사전순 정렬.
- 명령어 :

```redis
ZADD [set] [score] [value] [score] [value] ...
ZRANGE [set] [start_idx] [end_idx]
-- 역순
ZRANGE [set] [start_idx] [end_idx] REV
-- 점수(우선순위)와 함께 출력
ZRANGE [set] [start_idx] [end_idx] WITHSCORES
-- key의 해당 value의 랭크(0부터 시작한 인덱스) 반환
ZRANK [set] [value]
```

### 6. Streams

<img width="347" alt="Untitled (7)" src="https://github.com/hgene0929/hgene0929/assets/90823532/f42bc0f3-a839-4d75-9820-c279420e2339">

- append-only log에 consumer groups와 같은 기능을 더한 자료 구조.
    - append-only : 데이터 삭제는 없고 항상 추가만 가능하다.
- 추가 기능 :
    - unique id를 통해 하나의 entry를 읽을 때, O(1) 시간 복잡도.
    - consumer group을 통해 분산 시스템에서 다수의 consumer가 event 처리.
        - 다수의 consumer가 메시지를 동시에 처리하면서도, 동일한 메시지를 중복처리하는 문제 쉽게 해결.
- 명령어 :

```redis
-- * 옵션은 unique id 자동할당
XADD [stream] * [event]
-- -(가장 처음) ~ +(가장 마지막)
XRANGE [stream] - +
XDEL [stream] [unique_id]
```

### 7. Geospatials Indexes

- 좌표를 저장하고, 검색하는 데이터 타입.
- 명령어 :

```redis
GEOADD [key] [경도] [위도] [장소명]
-- 두 장소간 거리 조회(옵션을 통해 출력단위 지정)
GEODIST [key] [장소명] [장소명] [장소명] [옵션(KM)]
```

### 8. Bitmaps

- 실제 데이터 타입은 아니고, String에 binary operation을 적용한 것(일종의 인터페이스).
- 최대 42억개 binary 데이터 표현 = 2^32(4,294,967,296).
- 적은 메모리를 사용하여 binary 상태값을 저장하는데 사용.
- 명령어 :

```redis
SETBIT [key] [offset] [bit]
-- key의 bit 수 조회
BITCOUNT [key]
-- key 모두의 bit 합한 결과 조회 -> 결과명의 bitmap에 저장
BITOP AND [결과저장Bitmap] [key] [key]
-- offset의 결과 조회
GETBIT [결과저장Bitmap] [offset]
```

### 9. HyperLogLog

- 집합의 cardinality를 추정할 수 있는 확률형 자료구조(결과값이 실제와 어느정도 오차값이 발생할 수 있음).
- 정확성을 일부 포기하는 대신 저장공간을 효율적으로 사용(평균 에러 0.81%).
    - 매우 정확한 데이터말고, 근삿값만 알아도 되는 경우 사용.

<img width="327" alt="Untitled (8)" src="https://github.com/hgene0929/hgene0929/assets/90823532/6551afa0-87b1-467d-a718-72c87ed0b75c">

- 원리 : 멤버의 값을 해싱하여 버킷이라는 단위로 분류하여 해시값에 맞게 표시.
    - 해싱충돌이 발생하는 경우, 정확하지 않은 결과 반환.
- HyperLogLog vs Set :
    - HyperLogLog : 실제값을 저장하지 않기 때문에 매우 적은 메모리.
    - Set에 비해 상대적으로 매우 적은 메모리로 값 계산 가능.
    - 다만 실제값을 저장하지 않기 때문에 값을 다시 조회해야 하는 경우에는 사용불가.
- 명령어 :

```redis
PFADD [key] [value] [value] [value] ...
-- key의 카디널리티 조회
PFCOUNT [key]
```

### 10. BloomFilter

- element가 집한 안에 포함되었는지 확인할 수 있는 확률형 자료 구조(= membership test).
- 정확성을 일부 포기하는 대신 저장공간을 효율적으로 사용.
- false positive : element가 집합에 실제로 포함되지 않은데 포함되었다고 잘못 예측하는 경우.
- bloomfilter vs set :
    - bloomfilter : 실제값을 저장하지 않기 때문에 매우 적은 메모리 사용.

<img width="331" alt="Untitled (9)" src="https://github.com/hgene0929/hgene0929/assets/90823532/c0dfbfa2-f9a0-4c1f-8463-221bfa7ec79e">

- 원리 : 값을 해싱하여 여러개의 key를 만들고, bloomfilter에 key의 위치를 표시한다. 이후 어떤 아이템이 존재하는지 여부를 확인할 때는 다시 해시키를 생성하여 해당 위치를 확인한다.
    - 실제로는 아이템이 해당 해시키에 존재하지 않는데도, 존재한다고 표시해버리는 false positive 문제가 발생할 수 있음을 인지하고 있어야 한다.
- 명령어 :

```redis
BF.MADD [key] [item] [item] ...
-- 해당 아이템이 키의 블룸필터에 존재하는지 여부 조회
BF.EXISTS [key] [item]
```
