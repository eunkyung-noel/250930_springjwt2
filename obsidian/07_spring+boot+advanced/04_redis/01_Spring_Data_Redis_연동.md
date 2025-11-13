# 01 Spring Data Redis 연동

#SpringDataRedis #스프링데이터레디스

## 1. `build.gradle` 의존성 추가

Spring Boot에서 Redis를 쉽게 사용하기 위한 스타터를 추가합니다.

```gradle
// build.gradle
dependencies {
    // ... 기존 의존성 생략 ...

    // Spring Data Redis 의존성 추가
    implementation 'org.springframework.boot:spring-boot-starter-data-redis'
}
```

## 2. `application.yml` Redis 설정 추가

Redis 서버의 접속 정보를 추가합니다.

- [https://console.aiven.io/signup](https://console.aiven.io/signup)
- [https://aiven.io/valkey](https://aiven.io/valkey)

```yaml
# src/main/resources/application.yml
spring:
  # ... jwt 설정 생략 ...

  # Redis 설정
  data:
    redis:
      host: valkey-{...}.aivencloud.com
      port: { ... }
      username: default
      password: { ... }
      ssl:
        enabled: true
```

## 3. Redis 설정 클래스 작성 (`RedisConfig.java`)

Spring이 Redis와 통신할 때 사용할 `RedisTemplate`을 설정합니다. Key와 Value를 모두 `String`으로 직렬화(Serialize)하도록 설정하여 `redis-cli`에서 쉽게 데이터를 확인할 수 있도록 합니다.

```java
// config/RedisConfig.java
@Configuration
public class RedisConfig {

    @Bean
    public RedisTemplate<String, String> redisTemplate(RedisConnectionFactory connectionFactory) {
        RedisTemplate<String, String> template = new RedisTemplate<>();
        template.setConnectionFactory(connectionFactory);

        // Key와 Value의 직렬화 방식을 String으로 설정
        template.setKeySerializer(new StringRedisSerializer());
        template.setValueSerializer(new StringRedisSerializer());

        // Hash Key와 Hash Value의 직렬화 방식도 String으로 설정 (이번 예제에서는 사용하지 않음)
        template.setHashKeySerializer(new StringRedisSerializer());
        template.setHashValueSerializer(new StringRedisSerializer());

        // 설정 초기화
        template.afterPropertiesSet();
        return template;
    }
}
```
