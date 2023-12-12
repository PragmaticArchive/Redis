# Redis 사용시 주의사항

---

### 1. O(N) 명령어

- O(N) 명령어 : 대부분의 명령어는 O(1) 시간복잡도를 갖지만, 일부 명령어의 경우 O(N).
- 주의사항 : redis는 single thread로 명령어를 순차적으로 수행하기 때문에, 오래 걸리는 O(N) 명령어 수행시, 전체적인 애플리케이션 성능 저하가 발생할 수 있다.
- 예시 :
    - KEYS : 지정된 패턴과 일치하는 모든 키 조회.
    - SMEMBERS : Set의 모든 member 반환(N = Set Cardinality).
        - 하나의 Set에 10,000개 이상의 아이템을 추가하지 않는 것 권장.
        - 그 이상 추가해야 한다면 Set을 분리하는 것이 나음.
    - HGETALL : Hash의 모든 필드 반환(N = Size of Hash).
    - SORT : List, Set, ZSet의 아이템을 정렬하여 반환.

---

### 2. Thundering Herd Problem

- Thundering Herd Problem : 병렬 요청이 공유 자원에 대해서 접근할 때, 급격한 과부하가 발생하는 문제.
- 주로 대규모 트래픽을 다루는 문제에서 발생.
    - 대규모 트래픽을 다루는 서비스는 초당 request 횟수가 매우 많기 때문에 더욱 캐시를 세밀하게 조정해야 한다.
    - cache miss 때문에 발생할 가능성이 높다 ⇒ 이를 방지하기 위해서 별도의 프로세스에서 cron job을 통해 통계 캐시를 주기적으로 최신화 시켜주는 방법이 있다.

<img width="550" alt="Untitled (8)" src="https://github.com/hgene0929/hgene0929/assets/90823532/e48f812a-b488-4ab4-bbbe-79d6015a2230">

---

### 3. Stale Cache Invalidation

- Cache Invalidation : 캐시의 유효성이 손실되었거나 변경되었을 때, 캐시를 변경하거나 삭제하는 기술.
    - 캐시는 임시 데이터이기 때문에 잘못하면 원본 데이터와 싱크가 안맞을 수 있다.
    - 따라서 캐시의 유효성이 손실되었거나 데이터 자체가 변경되었을 때는 캐시를 변경하거나 삭제하는 식의 관리가 필요하다.
