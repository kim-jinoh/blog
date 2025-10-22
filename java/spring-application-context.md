---
title: Spring ApplicationContext 깊이 이해하기
layout: post
parent: Java
categories: [java, spring, ApplicationContext, bean, DI, Container]
---

## Spring `ApplicationContext`란?
`ApplicationContext`는 Spring 프레임워크의 핵심 컨테이너 인터페이스로, 애플리케이션에서 사용하는 빈(Bean)의 생성·관리·의존성 주입을 담당합니다. `BeanFactory`를 확장한 형태로, 단순한 빈 팩토리 기능 외에도 메시지 소스, 이벤트 발행, 리소스 로딩, 환경 프로퍼티 관리 등 다양한 인프라 기능을 제공합니다.

## 핵심 역할 및 기능
- **빈 생성 및 생명주기 관리**: 빈의 생성, 의존성 주입, 초기화, 소멸까지 전 과정을 관리합니다. `BeanPostProcessor`, `InitializingBean.afterPropertiesSet()`, `@PostConstruct`, `@PreDestroy` 등을 지원합니다.
- **의존성 주입(Dependency Injection)**: 생성자 주입, 세터 주입, 필드 주입(`@Autowired`) 등 다양한 DI 방식을 제공합니다. 실무에서는 특히 생성자 주입을 권장합니다.
- **빈 조회 및 검색**: `getBean(...)`을 통해 이름 또는 타입으로 빈을 조회할 수 있습니다.
- **환경과 프로파일 관리(Environment & Profiles)**: `Environment`를 통해 프로퍼티 값을 읽고 `@Profile`을 이용해 환경별 빈 등록을 할 수 있습니다.
- **리소스 로딩(ResourceLoader)**: `classpath:`나 `file:` 스키마를 통해 리소스를 추상화하여 로드합니다.
- **국제화(MessageSource)**: 메시지 번들 기반의 i18n을 지원합니다.
- **애플리케이션 이벤트 발행(ApplicationEventPublisher)**: 애플리케이션 내부에서 이벤트를 발행하고, 리스너를 통해 이벤트를 처리할 수 있습니다.
- **컨텍스트 계층화(Hierarchical context)**: 부모-자식 컨텍스트를 통해 역할 분리(예: 루트 컨텍스트 vs 웹 컨텍스트)를 구성할 수 있습니다.
- **AOP 및 프록시 인프라 지원**: 자동 프록시 생성 등을 통해 횡단 관심사 처리를 지원합니다.

## 언제 ApplicationContext를 선택해야 하나?
일반적인 스프링 애플리케이션(특히 웹 애플리케이션)은 `ApplicationContext`를 사용합니다. `BeanFactory`는 보다 가벼운 컨테이너가 필요할 때 고려할 수 있지만, 대부분의 경우 `ApplicationContext`가 제공하는 풍부한 기능이 더 유용합니다.

## 간단 사용법
### Java 설정 기반
```java
// AppConfig.java
@Configuration
@ComponentScan(basePackages = "com.example")
public class AppConfig {
    @Bean
    public String appName() {
        return "MyApp";
    }
}

// Main.java
public class Main {
    public static void main(String[] args) {
        AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext(AppConfig.class);

        String appName = ctx.getBean("appName", String.class);
        System.out.println("appName = " + appName);

        ctx.close();
    }
}
```

### XML 설정 기반
```xml
<!-- applicationContext.xml -->
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="myService" class="com.example.MyService"/>

</beans>
```

```java
// MainXml.java
public class MainXml {
    public static void main(String[] args) {
        ClassPathXmlApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");
        MyService myService = ctx.getBean(MyService.class);
        ctx.close();
    }
}
```

## 빈 생명주기 예시
- 초기화: `@PostConstruct`, `InitializingBean.afterPropertiesSet()`, `init-method`
- 소멸: `@PreDestroy`, `DisposableBean.destroy()`, `destroy-method`
- 빈 전처리: `BeanPostProcessor`를 구현하면 빈 초기화 전후에 커스텀 로직을 실행할 수 있습니다.

```java
@Component
public class ExampleBean {
    @PostConstruct
    public void init() { /* 초기화 로직 */ }

    @PreDestroy
    public void cleanup() { /* 정리 로직 */ }
}
```

## 권장사항
- **생성자 주입**(DI)을 사용하여 불변성과 테스트 용이성 확보.
- `@ComponentScan` 범위를 필요한 패키지로 제한하여 초기화 시간을 단축.
- `@Profile`을 활용해 환경별 설정을 분리.
- 큰 리소스 로딩은 비동기 또는 지연 로딩으로 처리하여 스타트업 성능 개선.

## 참고 API/클래스
- `ApplicationContext`, `ConfigurableApplicationContext`
- `AnnotationConfigApplicationContext`, `ClassPathXmlApplicationContext`
- `Environment`, `ApplicationEventPublisher`, `MessageSource`

---