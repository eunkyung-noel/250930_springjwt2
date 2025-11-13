# 02. 순수 Spring 예제

#spring #ioc #di #example #예제

Spring Boot 없이 순수 Spring Framework만을 사용하여 IoC와 DI가 어떻게 동작하는지 코드를 통해 직접 확인해 보겠습니다. 이 예제는 자바 기반 설정(Java-based Configuration)을 사용하여 Spring 컨테이너를 구성합니다.

---

## 1. 프로젝트 설정

### 가. Maven 의존성 (`pom.xml`)

가장 먼저, Spring의 핵심 기능인 `spring-context` 모듈에 대한 의존성을 추가해야 합니다. 이 모듈은 IoC 컨테이너와 DI 기능을 포함하고 있습니다.

```xml
<!-- pom.xml -->
<dependencies>
    <!-- https://mvnrepository.com/artifact/org.springframework/spring-context -->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>6.1.10</version> <!-- 작성 시점의 최신 안정 버전 -->
    </dependency>
</dependencies>
```

---

## 2. 코드 작성

### 가. Repository 및 Service 클래스

먼저 Spring 컨테이너가 관리할 빈(Bean)이 될 클래스들을 작성합니다.

- `MemoryRepo`: 데이터를 저장하는 역할을 하는 간단한 리포지토리 클래스입니다.
- `OrderService`: `MemoryRepo`에 의존하여 주문 처리 로직을 수행하는 서비스 클래스입니다.

```java
// MemoryRepo.java

import org.springframework.stereotype.Component;

/**
 * @Component: 이 클래스를 스프링 컨테이너의 빈(Bean)으로 등록합니다.
 * 스프링은 이 애너테이션을 보고 MemoryRepo 클래스의 인스턴스를 생성하고 관리합니다.
 */
@Component
class MemoryRepo {
    public void save(String data) {
        System.out.println(" [MemoryRepo] 데이터를 메모리에 저장했습니다: " + data);
    }
}
```

```java
// OrderService.java

import org.springframework.stereotype.Component;

/**
 * @Component: 이 클래스 또한 스프링 컨테이너의 빈으로 등록합니다.
 */
@Component
class OrderService {
    // 💡 DI(Dependency Injection): OrderService는 MemoryRepo에 의존합니다.
    private final MemoryRepo memoryRepo;

    /**
     * 💡 생성자 주입 (Constructor Injection)
     * 스프링 컨테이너는 OrderService 빈을 생성할 때 이 생성자를 호출합니다.
     * 이때, 매개변수 타입(MemoryRepo)에 맞는 빈을 찾아 자동으로 주입해줍니다.
     * 생성자가 하나만 있을 경우 @Autowired 애너테이션은 생략 가능합니다.
     */
    public OrderService(MemoryRepo memoryRepo) {
        System.out.println(" [OrderService] 빈이 생성되고 MemoryRepo가 주입되었습니다.");
        this.memoryRepo = memoryRepo; // 주입받은 객체를 필드에 할당
    }

    public void processOrder(String item) {
        System.out.println(" [OrderService] " + item + " 주문을 처리합니다.");
        // 주입받은 MemoryRepo 객체를 사용하여 데이터를 저장합니다.
        this.memoryRepo.save(item);
    }
}
```

### 나. Spring 설정 클래스 (`AppConfig.java`)

다음으로, Spring 컨테이너가 어떤 설정을 따라야 할지 알려주는 자바 설정 클래스를 만듭니다.

```java
// AppConfig.java

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

/**
 * @Configuration: 이 클래스를 스프링 설정 클래스로 지정합니다.
 * @ComponentScan: 스프링에게 지정된 패키지를 스캔하여
 * @Component, @Service, @Repository 등으로 등록된 클래스들을 찾아 빈으로 등록하라고 지시합니다.
 */
@Configuration
@ComponentScan("com.example") // 예시 패키지명
class AppConfig {
    // 순수 스프링에서는 @Bean 메서드를 통해 빈을 직접 등록할 수도 있으나,
    // 이 예제에서는 컴포넌트 스캔 방식을 사용합니다.
}
```

### 다. 메인 실행 클래스 (`Main.java`)

마지막으로, Spring 컨테이너를 초기화하고 컨테이너에 등록된 빈을 가져와 사용하는 메인 클래스를 작성합니다.

```java
// Main.java

import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class Main {
    public static void main(String[] args) {
        System.out.println("----- 스프링 컨테이너 초기화 시작 -----");

        // 💡 스프링 컨테이너 생성 및 초기화
        // AnnotationConfigApplicationContext는 @Configuration 클래스를 설정 정보로 사용합니다.
        // 이 과정에서 컨테이너는 AppConfig에 지정된 패키지를 스캔하고, 빈을 등록 및 주입합니다.
        try (var context = new AnnotationConfigApplicationContext(AppConfig.class)) {
            System.out.println("----- 스프링 컨테이너 초기화 완료 -----");
            System.out.println();

            // 💡 빈(Bean) 조회
            // 컨테이너가 관리하는 OrderService 빈(객체)을 조회합니다.
            // 우리는 직접 'new OrderService()'를 하지 않고 컨테이너에게 요청합니다.
            OrderService orderService = context.getBean(OrderService.class);

            // 💡 IoC와 DI 동작 확인
            // 조회한 객체의 메서드를 호출하여 기능을 사용합니다.
            orderService.processOrder("IoC와 DI 실습 예제");

        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

---

## 3. 실행 결과

위 코드를 실행하면 다음과 같은 순서로 출력이 나타납니다.

```
----- 스프링 컨테이너 초기화 시작 -----
 [OrderService] 빈이 생성되고 MemoryRepo가 주입되었습니다.
----- 스프링 컨테이너 초기화 완료 -----

 [OrderService] IoC와 DI 실습 예제 주문을 처리합니다.
 [MemoryRepo] 데이터를 메모리에 저장했습니다: IoC와 DI 실습 예제
```

- 컨테이너가 초기화되면서 `OrderService`와 `MemoryRepo` 빈이 생성되고, `OrderService`의 생성자를 통해 `MemoryRepo`가 주입되는 것을 확인할 수 있습니다.
- 개발자는 `new` 키워드를 사용하지 않았지만, Spring 컨테이너가 객체들을 생성하고 의존 관계를 설정해준 덕분에 `orderService.processOrder()` 메서드가 정상적으로 동작합니다. 이것이 바로 IoC와 DI의 핵심입니다.
