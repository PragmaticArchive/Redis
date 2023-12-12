# spring data redis

---

### 1. Spring Boot + Redis 사용하기

- 스프링 부트에서 Redis를 사용하기 위해서는 의존성 추가가 필요하다(build.gradle).

```groovy
implementation 'org.springframework.boot:spring-boot-starter-data-redis'
```

- 의존성 추가가 완료되면, 사용할 Redis의 Host 와 Port 를 지정해줘야한다(application.yml).

```yaml
spring:
  data:
    redis:
      host: localhost # localhost : 로컬에서 사용, 그외의 서버 사용시 해당 서버 호스트 지정
      port: 6379
```

- 위의 설정이 완료되면, spring 애플리케이션에서 redis와 연동하기 위한 별도의 값을 세팅하고 스프링 빈으로 등록해주어야 한다.

  java의 redis client 라이브러리에는 Jedis와 Lettuce가 있는데, Lettuce가 성능이 더 좋기 때문에 Lettuce로 RedisConnectionFactory를 통해 연동한다.


> spring boot 2.0부터 Redis Templater과 String Template은 자동생성 되기 때문에 따로 스프링 빈으로 등록하지 않아도 된다.
>

```java
/*
* application.yml 파일의 속성을 생성자를 통해 가져와 변수에 저장해둠
*/
@Getter
@ConfigurationProperties(prefix = "spring.data.redis")
public class RedisProperties {

	private final String host;
	private final int port;

	public RedisProperties(String host, int port) {
		this.host = host;
		this.port = port;
	}
}
```

```java
/*
* redis 설정파일
*/
@RequiredArgsConstructor
@Configuration
public class RedisConfiguration {

	private final RedisProperties redisProperties;

	@Bean
	public RedisConnectionFactory redisConnectionFactory() {
		// java의 redis client들(Jedis | Lettuce) 중 성능이 더 우수한 Lettuce를 통해 java 애플리케이션과 redis 연동
		return new LettuceConnectionFactory(redisProperties.getHost(), redisProperties.getPort());
	}
	
	@Bean
	public RedisTemplate<String, Object> redisTemplate() {
		// spring data redis를 사용하는 방법들(repository | redisTemplate) 중 한가지인 redisTemplate을 미리 생성하여 이후 사용가능하도록 스프링빈으로 등록해둠
		RedisTemplate<String, Object> redisTemplate = new RedisTemplate<>();
		redisTemplate.setConnectionFactory(redisConnectionFactory());
		
		// 일반적인 key:value 타입의 데이터용 serializer
		redisTemplate.setKeySerializer(new StringRedisSerializer());
		redisTemplate.setValueSerializer(new StringRedisSerializer());
		
		// Hash를 사용할 경우의 serializer
		redisTemplate.setHashKeySerializer(new StringRedisSerializer());
		redisTemplate.setHashValueSerializer(new StringRedisSerializer());
		
		// 모든 경우의 serializer
		redisTemplate.setDefaultSerializer(new StringRedisSerializer());
		
		return redisTemplate;
	}
}
```

---

### 2. Spring Data Redis 사용방법

- Spring 애플리케이션에서 Redis를 사용하는 방법은 크게 2가지로 구분된다.
    1. RedisRepository 사용하기.
    2. RedisTemplate 사용하기.

---

**RedisRepository 사용하기**

- Repository로 사용할 경우, Entity를 생성하여 쉽게 사용가능하다.

```java
/*
* redisRepository 사용을 위한 Entity
*/
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@AllArgsConstructor(access = AccessLevel.PRIVATE)
@RedisHash(value = "refresh_token") // (1)
public class RefreshToken {
	
	@Id // (2)
	private String authId;
	
	@Indexed // (3)
	private String token;
	
	private String role;
	
	@TimeToLive // (4)
	private long expiresIn;
	
	public RefreshToken update(String token, long expiresIn) {
		this.token = token;
		this.expiresIn = expiresIn;
		return this;
	}
}
```

1. 설정한 값을 Redis의 Key 값 prefix로 사용한다.
   redis는 Key값에 콜론(:)을 두어 의미를 구분하는데 이때, 콜론의 접두어로 사용되는 부분이다.
2. Key 값이 되며, 위에서 설정한 prefix:{id} <- 이부분에 추가되어 세부적인 의미를 구분한다(auto-increment 된다).
3. redis는 기본적으로 Key값을 통해 값을 찾아오는 방식의 조회만 가능한데, @Indexed 애노테이션이 붙어있는 필드도 추가적으로 검색이 가능하다.
4. redis는 만료시간 설정이 가능한데 이때 사용되는 만료시간(초단위)이다.

```java
public interface RefreshTokenRepository extends CrudRepository<RefreshToken, String> {
	Optional<RefreshToken> findByToken(String token);
	Optional<RefreshToken> findByAuthId(String authId);
}
```

- CrudRepository 인터페이스를 상속받는다.
- 이때, 해당 Repository의 Entity에 위치한 @Id 또는 @Indexed 애노테이션을 적용한 필드들에만 CrudRepository가 제공하는 findBy~ 구문을 적용할 수 있다.

---

**RedisTemplate 사용하기**

```java
/*
* 외부 서비스 클래스에서 redisTemplate이 필요할 때 사용할 수 있도록 util 클래스 생성
*/
@RequiredArgsConstructor
@Component
public class RedisUtils {
	
	// (1)
	private final RedisTemplate<String, Object> redisTemplate;

	// (2)
	public void setData(String key, String value, Long expiresIn) {
		// (3)
		redisTemplate.opsForValue().set(key, value, expiresIn, TimeUnit.MILLISECONDS);
	}
	
	// (4)
	public String getData(String key) {
		return (String) redisTemplate.opsForValue().get(key);
	}
	
	// (5)
	public void deleteData(String key) {
		redisTemplate.delete(key);
	}
}
```

1. 이전에 redis 설정파일에 생성하여 스프링 빈으로 등록해둔 redisTemplate을 의존성 주입받는다.
2. setData : key, value, 만료시간(TTL)을 파라미터로 받아와 redis에 저장하기 위한 커스텀 메서드.
3. redisTemplate은 사용하는 자료구조(redis에는 10개의 자료구조가 있다)마다 제공하는 메서드가 다르기 때문에 객체를 만들어서 redis의 자료구조에 맞는 메서드를 사용하면된다.


    | 메서드명 | redis 자료구조 |
    | --- | --- |
    | opsForValue | String |
    | opsForList | List |
    | opsForSet | Set |
    | opsForZSet | Sorted Set |
    | opsForHash | Hash |
4. getData : key 를 파라미터로 받아와 redis에서 데이터를 조회해오기 위한 커스텀 메서드.
5. deleteData : key 를 파라미터로 받아와 redis에서 데이터를 삭제하기 위한 커스텀 메서드.

---

### 3. Redis Cache 사용하기

- Redis는 데이터베이스로도 사용되고, Message Broker로도 사용되지만 Cache Manager의 용도로도 많이 사용된다.
- Redis가 제공하는 Caching 관련 애노테이션은 다음과 같다.

---

**@EnableCaching**

- Spring Boot에게 캐싱기능이 필요함을 전달한다.
- Spring Boot Starter class에 적용된다.

---

**@Cacheable**

- 리턴값을 기준으로 데이터가 캐시에 있으면 그대로 반환, 없으면 저장 후 반환한다.
- 일반적으로 조회와 같은 api에 주로 사용된다.

| Element | Description                                                                                             | Type |
| --- |---------------------------------------------------------------------------------------------------------| - |
| cacheName | 캐시 이름(설정 메서드 리턴값이 저장되는)                                                                                 | String[] |
| value | cacheName의 별칭                                                                                           | String[] |
| key | 동적인 키 값을 사용하는 spEL 표현식<br>동일한 cacheName을 사용하지만 구분될 필요가 있을 경우 사용되는 값                                     | String |
| condition | spEL 표현식이 참일 경우에만 캐싱 적용<br>-or, and 등 조건식, 논리연산 가능                                                      | String|
| unless | 캐싱을 막기 위해 사용되는 spEL 표현식<br>condition과 반대로 참일 경우에만 캐싱이 적용되지 않음                                           | String                                                             |
| cacheManager | 사용할 CacheManager 지정<br>(EHCacheCacheManager, RedisCacheManager 등)                                       | String                                                             |
| sync | 여러 스레드가 동일한 키에 대한 값을 로드하려고 할 경우, 기본 메서드의 호출을 동기화<br>즉, 캐시 구현체가 Thread safe 하지 않은 경우, 캐시에 동기화를 걸 수 있는 속성 | boolean                                                             |

---

**@CachePut**

- 캐시에 데이터를 저장할 때만 사용한다.
- @Cacheable과 달리 캐시에 저장된 데이터를 사용하지 않는다.
- 일반적으로 수정과 같은 api에 주로 사용된다.

| Element | Description                                                         | Type |
| --- |---------------------------------------------------------------------| --- |
| cacheName | 입력할 캐시 이름                                                           | String[] |
| value | cacheName의 별칭                                                       | String[] |
| key | 동적인 키 값을 사용하는 spEL 표현식<br>동일한 cacheName을 사용하지만 구분될 필요가 있을 경우 사용되는 값 | String |
| cacheManager | 사용할 CacheManager 지정<br>(EHCacheCacheManager, RedisCacheManager 등)   | String                                                              |
| condition | spEL 표현식이 참일 경우에만 캐싱 적용<br>-or, and 등 조건식, 논리연산 가능                  | String                                                              |
| unless | 캐싱을 막기 위해 사용되는 spEL 표현식<br>condition과 반대로 참일 경우에만 캐싱이 적용되지 않음       | String                                                              |

---

**@CacheEvict**

- 메서드가 호출될 때 캐시에 있는 데이터가 삭제된다.
- 일반적으로 삭제와 같은 api에 주로 사용된다.

| Element | Description                                                         | Type |
| --- |---------------------------------------------------------------------| --- |
| cacheName | 입력할 캐시 이름                                                           | String[] |
| value | cacheName의 별칭                                                       | String[] |
| key | 동적인 키 값을 사용하는 spEL 표현식<br>동일한 cacheName을 사용하지만 구분될 필요가 있을 경우 사용되는 값 | String |
| allEntries | 캐시 내의 모든 리소스를 삭제할지의 여부                                              | boolean |
| condition | spEL 표현식이 참일 경우에만 캐싱 적용<br>-or, and 등 조건식, 논리연산 가능                  | String                                                              |
| cacheManager | 사용할 CacheManager 지정<br>(EHCacheCacheManager, RedisCacheManager 등)   | String                                                              |
| beforeInvocation | true - 메서드 수행 이전 캐시 리소스 삭제<br>false - 메서드 수행 후 캐시 리소스 삭제            | boolean                                                             |

---

**Spring Boot + Redis Caching 적용**

```java
@EnableCaching
@ConfigurationPropertiesScan
@SpringBootApplication
public class SpringDataRedisApplication {

	public static void main(String[] args) {
		SpringApplication.run(SpringDataRedisApplication.class, args);
	}
}
```

```java
/*
	* redisCaching을 사용하기 위해 RedisCacheManager 을 생성하여 빈으로 등록
*/
@EnableCaching
@Configuration
public class RedisConfiguration {

	@Bean
	public CacheManager cacheManager(RedisConnectionFactory redisConnectionFactory) {
		RedisCacheConfiguration redisCacheConfiguration = RedisCacheConfiguration.defaultCacheConfig()
			.serializeKeysWith(RedisSerializationContext.SerializationPair.fromSerializer(new StringRedisSerializer()))
			.serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(new GenericJackson2JsonRedisSerializer()))
			.entryTtl(Duration.ofMinutes(3L));
		
		return RedisCacheManager
			.RedisCacheManagerBuilder
			.fromConnectionFactory(redisConnectionFactory)
			.cacheDefaults(redisCacheConfiguration)
			.build();
	}
}
```

- 데이터를 가져오고 보낼 때, 우리가 만든 도메인 모델을 Serailize 해주기 위한 설정이 필요하다.
- 이때 생성한 RedisConnectionFactory 메서드를 앞으로 사용할 redis 애노테이션에 명시해주어야 한다.
- entryTtl() 을 통해 유효 캐시 시간을 제한할 수 있다.

```java
@RequestMapping("/log")
@RestController
@RequiredArgsConstructor
public class LogController {

    private final LogService logService;

    @GetMapping
    public ResponseEntity<List<LogResponse>> getAll() {
        List<LogResponse> logs = logService.searchAll();
        return ResponseEntity.ok(logs);
    }

    @DeleteMapping
    public void deleteAll() {
        logService.removeAll();
    }
}
```

```java
@Service
public class LogService {

	private final LogFacade logFacade;

	@Cacheable(cacheNames = "searchAll", key = "#root.target + #root.methodName", sync = true, cacheManager = "cacheManager")
	public ResponseEntity<List<LogResponse>> searchAll() {
		return logFacade.findAllOrderByDateAtDesc()
			.stream()
			.map(LogResponse::new)
			.collect(Collectors.toList());
	}

	@CacheEvict(cacheNames = "searchAll", allEntries = true, beforeInvocation = true, cacheManager = "rcm")
	public void removeAll() {
		logFacade.removeAll();
	}
}
```
