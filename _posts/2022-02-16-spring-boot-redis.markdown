---
layout: post
title: "SpringBoot - Redis Client 적용기"
categories:
  - SpringBoot
tags:
  - springBoot
  - redis
  - lettuce

toc: true
toc_sticky: true

date: 2022-02-16
last_modified_at: 2022-02-21
comments: true
---

## 들어가며


스프링부트 Redis 적용 방법을 공유하고자 게시물을 작성하였다.
예제코드로 적용 방법을 알아보자

## 🌟 Redis 의존 설정


Spring은 다양한 Redis 의존설정 방법을 제공한다. 

1 - Gradle이나 Maven 설정 파일에 의존 추가하기 

```scheme
dependencies {
	implementation 'org.springframework.boot:spring-boot-starter-data-redis'
}
```
<br/>
2 - Intellij tool 을 사용하여 SpringBoot 프로젝트 생성시점에 Spring Data Redis 를 추가하는 방법이다.

   ![redis-intellij](https://user-images.githubusercontent.com/56028408/154784437-6aff0467-1b9e-401d-a3da-928f16524194.png){: width="550" height="500"}
    

3 - Spring initializr 로 프로젝트 생성 시점에 Spring Data Redis 추가하기

   ![initalizr-redis](https://user-images.githubusercontent.com/56028408/154784439-d6dafd63-1c91-47b5-b38f-4533574a972b.png){: width="700" height="500"}


<br/>

`💡 참고로 Springboot 2 부터는 Lettuce가 기본 Redis Client로 사용된다. 본 프로젝트도 Lettuce를 사용하였다`


## 🌟 A**pplication.yml** 설정


정말 간단하게 Redis 의존을 추가했다. 이제 남은 건 A**pplication.yml** 작성과 Redis Configuration Bean 만 등록하면 된다. 아마도

ymal 작성은 매우 간단하다.

```scheme
spring:
	redis:
		lettuce:
			pool:
				max-active: 1000
				max-idle: 500
				min-idle: 2
				max-wait: 1000
	port: 6379
	host: 127.0.0.1
	password: 'vldpsxl1q2w3e'
```

Redis 에 관한 정보만 입력한다고 가정하면 이게 끝이다.  설정 정보는 아래 표 참조 바람  
<br/>

**Redis Option**

| 파라미터 | 파라미터 설명 | 기본값 | 기타 |
| --- | --- | --- | --- |
| spring.redis.pool.max-active | pool에 할당할 수 있는 최대 커넥션 수 (음수는 무제한) | 8 |  |
| spring.redis.pool.max-idle | pool의 “idle” 커넥션 최대수 (음수는 무제한) | 8 |  |
| spring.redis.pool.max-wait | 사용 가능한 커넥션이 없을 때 호출자가 대기해야하는 최대 시간 (단위 : 밀리초, 음수는 무제한 차단) | -1 |  |
| spring.redis.pool.min-idle | pool에서 관리하는 “idle” 커넥션 최소수 (양수일 떄만 유효) | 0 |  |
| spring.redis.port | redis server port | 6379 |  |
| spring.redis.host | redis server host | 127.0.0.1 |  |
| spring.redis.password | redis server login password |  |  |
| spring.redis.timeout | 커넥션 타임아웃 (단위: 밀리세컨드) | 0 |  |
| spring.redis.database | 커넥션 팩토리에 사용되는 데이터베이스 인덱스 | 0 |  |

## 🌟 RedisConnectionFactory, RedisTemplate 설정하기

순조롭게 Yaml 파일 설정도 끝이났다.  이제는 Java를 사용해서 기존에 xml 파일로 설정했던 RedisFactory, RedisTemplate를 변경해보자.

### XML

```xml
<!-- Redis 설정 고통 부분-->
<!-- Redis Serializer -->
<bean id="stringRedisSerializer" class="org.springframework.data.redis.serializer.StringRedisSerializer" />
<bean id="jsonRedisSerializer" class="org.springframework.data.redis.serializer.GenericJackson2JsonRedisSerializer" />
<bean id="jackson2JsonRedisSerializer" class="org.springframework.data.redis.serializer.Jackson2JsonRedisSerializer">
    <constructor-arg>
        <value>java.util.Map</value>
    </constructor-arg>
</bean>

<!-- Redis Template -->
<bean id="redisTemplate" class="org.springframework.data.redis.core.RedisTemplate">
    <property name="connectionFactory" ref="jedisConnFactory" />
    <property name="keySerializer" ref="stringRedisSerializer" />
    <property name="valueSerializer" ref="jsonRedisSerializer" />
    <property name="hashKeySerializer" ref="stringRedisSerializer" />
    <property name="hashValueSerializer" ref="jsonRedisSerializer" />
</bean>
```

### JAVA

**RedisTemplate 설정**

```java
@Configuration
@EnableRedisRepositories
public class RedisTemplateConfig {

		@Value("${spring.redis.host}")
    private String redisHost;
		@Value("${spring.redis.port}")
    private int redisPort;

		@Bean
    public RedisConnectionFactory redisConnectionFactory() {
        return new LettuceConnectionFactory(redisHost,redisPort);
		}

    @Bean
    public RedisTemplate<String,Object> redisTemplate() {
        RedisTemplate<String,Object> redisTemplate = new RedisTemplate<>();
				redisTemplate.setKeySerializer(new StringRedisSerializer());
				redisTemplate.setValueSerializer(new GenericJackson2JsonRedisSerializer());
				redisTemplate.setHashKeySerializer(new StringRedisSerializer());
				redisTemplate.setHashValueSerializer(new GenericJackson2JsonRedisSerializer());
				return redisTemplate;
		}
}
```

기존 xml 파일은 번거롭게 Serializer 클래스를 매번 bean으로 등록해서 redisTemplate에 주입했다.
그런데 자바코드는 간단하게 인스턴스만 생성해서 주입하고 있는 것을 확인할 수 있다.

<br/>

`💡 참고로 RedisTemplate는 RedisTemplate뿐만 아니라 StringRedisTemplate 도 존재한다.  게시물 주제와 연관이 없으므로 자세한 내용은 생략한다.`

<br/>

## 🌟 Redis Message Listener

이제 Redis Message Listener를 등록해보자.

### XML

```xml
<!--첫번째-->
 <bean id="redisKeyExpireMessageListener" class="org.springframework.data.redis.listener.adapter.MessageListenerAdapter">
        <constructor-arg>
            <bean class="framework.redis.RedisKeyExpireMessageListener" />
        </constructor-arg>
    </bean>

<!--두번째-->
  <bean id="redisContainer1" class="org.springframework.data.redis.listener.RedisMessageListenerContainer">
      <property name="connectionFactory" ref="jedisConnFactory"/>
      <property name="messageListeners">
          <map>
              <entry key-ref="redisKeyExpireMessageListener">
<!--세번째-->
                  <bean class="org.springframework.data.redis.listener.PatternTopic">
                      <constructor-arg value="__keyevent@*:expired"/>
                  </bean>
              </entry>
          </map>
      </property>
  </bean>
```

1. `MessageListenerAdapter` 빈에  `MessageListener` 를 주입한다.
2. `RedisMessageListenerContainer` 빈 등록 `connectionFactory`, `messageListeners` 주입
3. `redisKeyExpireMessageListener` 가 수신할 채널 혹은 패턴을 등록한다. 패턴은 이벤트 또는 채널명이 될 수 있다.

### JAVA

**MessageListenerAdapter 추가**

```java
	@Bean
  public MessageListenerAdapter messageListenerAdapter(RedisKeyExpireMessageListener redisKeyExpireMessageListener) {
      return new MessageListenerAdapter(redisKeyExpireMessageListener);
  }

  @Bean
  public RedisMessageListenerContainer redisMessageListenerContainer(
          RedisConnectionFactory redisConnectionFactory
          , MessageListenerAdapter messageListenerAdapter, PatternTopic patternTopic) {
      RedisMessageListenerContainer container = new RedisMessageListenerContainer();
      container.setConnectionFactory(redisConnectionFactory);
      container.addMessageListener(messageListenerAdapter, patternTopic);
      return container;
  }

	@Bean
  public PatternTopic patternTopic() {
      return new PatternTopic("__keyevent@*:expired");
  }
```

**MessageListenerAdapter 에 주입할 MessageListener 구현**

```java
@Component
public class RedisKeyExpireMessageListener implements MessageListener {
    @Override
    public void onMessage(Message message,byte[] pattern) {
				
    }
}
```
<br/>
❗ spring boot 의 `@EnableAutoConfiguration`

    spring boot 는  프로젝트에 추가된 라이브러리를 기반으로 실행에 필요한 환경을 자동으로 설정해준다.
    어떻게 보면 정말 편리한 기능이지만, 섬세한 설정이 필요한 경우엔 직접 설정하는 것이 안전하다.
    

## 🌟 Redis Cache Management


### XML

```xml
<!-- redis Cache 설정 -->
  <bean id="redisCachePrefix" class="org.springframework.data.redis.cache.DefaultRedisCachePrefix" >
      <constructor-arg name="delimiter" value="_" />
  </bean>

  <bean id="cacheManager" class="org.springframework.data.redis.cache.RedisCacheManager" primary="true">
      <constructor-arg ref="redisTemplate" />
      <property name="defaultExpiration" value="1000" />
      <property name="usePrefix" value="true"/>
      <property name="cachePrefix" ref="redisCachePrefix" />
  </bean>
```

### JAVA
**@EnableCaching 어노테이션 추가**
```java
@SpringBootApplication
@EnableCaching
public class SpringBootStudyApplication {

  public static void main(String[] args) {
      SpringApplication.run(SpringBootStudyApplication.class, args);
  }
}
```

@EnableCaching 어노테이션은 @Cacheable 메소드가 호출될 때 사용되는 CacheInterceptor 및 Proxy 또는 AspectJ 같은 캐시 관리에 필요한 스프링 구성요소를 등록하는 역할을 한다.

@EnableCaching 어노테이션 뿐만 아니라 xml 파일에서 cacheManger 를 선언해서 사용할 때도 동일하게 동작한다.

<br/>

**CacheManagerConfig.class**
```java
@Configuration
@RequiredArgsConstructor
public class CacheManagerConfig {

  private final RedisCacheConfig redisCacheConfig;

	@Bean
  public CacheManager cacheManager(RedisConnectionFactory redisConnectionFactory) {
      return RedisCacheManager.RedisCacheManagerBuilder
              .fromConnectionFactory(redisConnectionFactory)
              .cacheDefaults(getDefaultRedisCacheConfiguration())
              .build();
	}

  private RedisCacheConfiguration getDefaultRedisCacheConfiguration() {
      return RedisCacheConfiguration.defaultCacheConfig()
              .entryTtl(Duration.ofSeconds(redisCacheConfig.getTtl()))
              .prefixCacheNameWith(redisCacheConfig.getCachePrefix());
	}

}
```

**RedisCacheConfig.class**

```java
@Component
@ConfigurationProperties(prefix="config.redis.cache")
@Getter
@Setter
public class RedisCacheConfig {
    private String cachePrefix;
    private long ttl;
}
```

`application.yml`

```java
config:
  redis:
      cache:
        cache-ttl: 1000
        cache-prefix: "_"
```

## 🌟 Lettuce를 선택한 이유

마지막으로 Redis Client로 Lettuce를 선택한 이유를 설명하고자 한다. 

### Lettuce란?

lettuce는 Netty 기반의 Redis Client로 동기, 비동기 및 대응적 사용을 위해 확장 가능한 Thread-Safe Redis Client이다. 

트랜잭션을 사용하지 않는 경우 하나의 연결을 공유할 수 있다. 추가로 Sentinel, Cluster, Pipelining, Auto-Reconnect 및 Redis 데이터 모델과 같은 고급 Redis 기능을 지원한다.

### Lettuce 장점

- connection 인스턴스를 여러 쓰레드에서 공유할 수 있기 때문에 Thread-safe하다.    
- Lettuce는 비동기로 요청을 처리하기 때문에 성능적인 이점이 있다. (비동기, 동기 둘 다 지원하기 때문에 유연하게 사용가능)
- Jedis보다 다양한 DataType지원

<br/>
<details>
  <summary>❗connection 인스턴스를 여러 쓰레드에서 공유할 수 있는 것이 장점인 이유는?</summary>
  <div markdown="1">
  
    Redis 서버 인스턴스는 여러 클라이언트에 연결하여 명령을 동시에 보내고 받을 수 있지만, 명령을 실행할 때 각 인스턴스는 단일 스레드이다.
        
    이는 애플리케이션이 멀티스레딩+단일 연결을 통해 Redis를 운영할 수 있다면 Redis 서버의 전체 연결 수를 효율화 할 수 있고,
        
    여러 애플리케이션이 동일한 Redis 서버를 공유할 때 더 나은 안정성과 성능을 얻을 수 있다는 것을 의미한다.
        
    응용 프로그램의 경우 여러 인스턴스를 유지 관리하는 리소스 소비가 줄어든다.
</div>
</details>    

### Lettuce 단점

- connection 인스턴스의 공유라는 점에서 Thread-safe 한 것이기 때문에 Single-Thread로 동작하는 애플리케이션에서 레디스 데이터에 접근할 때는 또다르게 고려할 점이 생긴다.
- 하나의 connection을 공유하기 때문에 connection pool을 생성하지 않으면 transaction을 사용할 수 없다.

<br/> 

## 🌟 Jedis와 Lettuce 비교

그렇다면 Jedis와 Lettuce의 차이점이 무엇이길래 Springboot2 이후로는 Lettuce가 기본 Redis Client로 선택되었는지 알아보자
<br/>
<br/>
<br/>

**Jedis보다 뛰어난 성능**

| Type     | TPS(낮을수록 응답속도 빠름) | Redis CPU | Connections | 응답 속도 |
| --- | ------ | ----- | ----- | ----- |
|Jedis |31.000 |20% |515 |1000ms |
|Lettuce|100,000 |7% |6 |7.5ms |
  
<br/>
<br/>
<br/>

**Jedis와 Lettuce 기능비교**

| Supported Feature | Lettuce | Jedis |
| --- | --- | --- |
| Standalone Connection | O | O |
| Master/Replica Connection | O |  |
| Redis Sentinel | Master Lookup, Sentinel Authentication, Replica Redis | Master Lookup |
| Redis Cluster | Cluster Connection, Cluster Node Connection, Replica Reads | Cluster Connection, Cluster Node Connection |
| Transport Channers | TCP, OS-native TCP(epoll, kqueue), Unix Domain Sockets | TCP |
| Connection Pooling | O(using commons-pool2) | O(using commons-pool2) |
| Other Connection Features | Singleton-connection sharing for non-blocking commands | JedisShardInfo support |
| SSL Support | O | O |
| Pub/Sub | O | O |
| Pipelining | O | O |
| Transactions | O | O |
| DataType Support | Key, String, List, Set, Sorted Set, Hash Server, Stream, Scriping, Geo, HyperLogLog | Key, String, List, Set, Sorted Set, Scriping, Geo, HyperLogLog |
| Reactive (non-blocking) API | O | O |

<br/>
<br/>  

***참고 → O 표시는 지원되는 기능이다.***

표로 jedis와 Lettuce의 기능을 비교해본 결과, 두 가지 차이점을 찾을 수 있다.

- Jedis에서 지원하는 모든 기능을 Lettuce에서도 지원한다.
- Jedis에서 지원하지 않는 기능도 Lettuce에서 지원한다.

<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>

# 🌟****Reference****

[http://arahansa.github.io/docs_spring/redis.html](http://arahansa.github.io/docs_spring/redis.html)

[https://wnwngus.tistory.com/64](https://wnwngus.tistory.com/64)

[https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/cache/annotation/EnableCaching.html](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/cache/annotation/EnableCaching.html)

[https://segmentfault.com/a/1190000040331000/en](https://segmentfault.com/a/1190000040331000/en)

[https://mandlife.tistory.com/entry/Springboot-redis-설정-Jedis보다는-Lettuce](https://mandlife.tistory.com/entry/Springboot-redis-%EC%84%A4%EC%A0%95-Jedis%EB%B3%B4%EB%8B%A4%EB%8A%94-Lettuce)
