---
title: "Spring Bean: @Qualifier와 @Primary(fallback) 정리"
date: 2025-11-11
parent: Java
tags: [spring, spring-boot, bean, qualifier, primary]
---

Spring에서 같은 타입의 빈이 여러 개 등록되어 있을 때 어떤 빈을 주입할지 지정하는 대표적인 방법으로 `@Qualifier`와 `@Primary`가 있습니다. 이 포스트에서는 각 어노테이션의 목적, 간단한 사용법, 그리고 런타임에서의 동작(결과)을 예제와 함께 설명합니다.

## 1. 문제 상황: 같은 타입 빈이 여러 개일 때

예를 들어 `GreetingService` 인터페이스를 구현한 빈이 두 개 등록되어 있다고 합시다.

```java
public interface GreetingService {
    String greet();
}

@Component
public class EnglishGreetingService implements GreetingService {
    @Override
    public String greet() { return "Hello"; }
}

@Component
public class KoreanGreetingService implements GreetingService {
    @Override
    public String greet() { return "안녕하세요"; }
}
```

스프링이 클래스 기반으로 생성하는 기본 빈 이름은 보통 클래스명에서 첫 글자를 소문자로 바꾼 형태입니다. 예: `KoreanGreetingService` → `koreanGreetingService`.

이 상태에서 단순히 타입으로만 주입하면 스프링은 어느 빈을 주입해야 할지 모릅니다.

```java
// 컴파일은 되지만 런타임에서 예외 발생
@Autowired
private GreetingService greetingService;
```

결과: org.springframework.beans.factory.NoUniqueBeanDefinitionException 가 발생합니다 — 같은 타입의 빈이 2개 이상 존재하여 어떤 것을 주입할지 모름.

---

## 2. @Qualifier: 특정 빈을 명시적으로 선택

`@Qualifier`는 주입할 빈의 이름(또는 커스텀 식별자)을 지정해서 명확히 하나를 선택하게 합니다. 위 예제에 적용하면(기본 빈 이름 사용):

```java
// 필드 주입 예시
@Autowired
@Qualifier("koreanGreetingService")
private GreetingService greetingService;

// 생성자 주입 예시 (권장)
public class SomeController {
    private final GreetingService greetingService;

    @Autowired
    public SomeController(@Qualifier("koreanGreetingService") GreetingService greetingService) {
        this.greetingService = greetingService;
    }
}
```

결과: `KoreanGreetingService`가 주입되어 `greetingService.greet()`는 `"안녕하세요"`를 반환합니다.

팁:

- `@Component`에 이름을 직접 지정하지 않으면 스프링의 기본 네이밍 규칙(클래스명 camelCase)이 적용됩니다.
- 문자열 대신 `@Qualifier`에 커스텀 어노테이션을 만들어 사용하면 더 타입 안전하고 의도를 명확히 할 수 있습니다.

### 커스텀 `@Qualifier`(메타-어노테이션) 사용 예

문자열 기반 `@Qualifier` 대신 커스텀 어노테이션을 만들어 사용하는 패턴은 리팩터링 안전성과 의도 명확성에서 장점이 있습니다. 아래는 간단한 예입니다.

```java
@Target({ElementType.FIELD, ElementType.PARAMETER, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Qualifier
public @interface Korean {}
```

사용 예:

```java
@Component
@Korean
public class KoreanGreetingService implements GreetingService {
    @Override
    public String greet() { return "안녕하세요"; }
}

// 주입 시
public class SomeController {
    private final GreetingService greetingService;

    public SomeController(@Korean GreetingService greetingService) {
        this.greetingService = greetingService;
    }
}
```

얻게 되는 이득:

- **리팩터링 안전성**: 문자열 리터럴에 의존하지 않으므로 클래스명/메서드명을 변경해도 주입이 깨질 가능성이 줄어듭니다.
- **타입-안정성(의도 명확성)**: 어노테이션을 통해 역할(의도)을 표현하므로 코드 읽기가 쉬워집니다.
- **중복 적용/그룹화**: 여러 구현체에 동일한 커스텀 qualifier를 달아 그룹처럼 다룰 수 있습니다.
- **오타·문자열 관리 비용 감소**: 문자열 상수나 하드코딩을 줄여 유지보수가 쉬워집니다.

---

## 3. @Primary: 명시적 'fallback' (우선 선택) 빈

`@Primary`는 같은 타입의 빈이 여러 개 있을 때 **명시적인 선택(@Qualifier 없음)** 이라면 우선적으로 선택되는 빈을 지정합니다. 예:

```java
@Component
@Primary
public class DefaultGreetingService implements GreetingService {
    @Override
    public String greet() { return "Hi (default)"; }
}

@Component
public class KoreanGreetingService implements GreetingService {
    @Override
    public String greet() { return "안녕하세요"; }
}
```

이제 단순히 타입으로 주입하면 `DefaultGreetingService`가 주입됩니다.

```java
@Autowired
private GreetingService greetingService; // DefaultGreetingService가 주입됨
```

결과: `greetingService.greet()`는 `"Hi (default)"`를 반환합니다.

중요: `@Qualifier`가 지정되어 있으면 `@Primary`는 무시됩니다. 즉 우선순위는

1. `@Qualifier`에 의해 직접 지정된 빈
2. `@Primary`로 표시된 빈
3. (둘 다 없으면) NoUniqueBeanDefinitionException

---

## 4. @Bean 등록 시에도 동일하게 적용

`@Configuration` 내부 `@Bean` 메서드에도 `@Primary`와 `@Qualifier`(또는 `name`)를 사용할 수 있습니다.

```java
@Configuration
public class GreetingConfig {
    @Bean
    @Primary
    public GreetingService defaultGreeting() { return new DefaultGreetingService(); }

    @Bean
    public GreetingService koreanGreeting() { return new KoreanGreetingService(); }
}
```

이 설정도 앞서 설명한 규칙과 동일하게 동작합니다. (`@Bean` 메서드의 기본 이름은 메서드명입니다.)

---

## 5. 실전 팁과 권장사항

- 가능하면 생성자 주입을 사용하세요(테스트하기 쉽고 불변성을 보장).
- 여러 구현체가 존재하는 경우 이름(또는 커스텀 qualifier 어노테이션)으로 명확히 분기하세요. `@Primary`는 "일반적인 기본 동작"이 필요할 때 유용합니다.
- `@Qualifier`와 `@Primary`를 함께 사용하면, 기본은 `@Primary`로 두고 특수한 경우에만 `@Qualifier`로 지정하는 설계가 깔끔합니다.

---

간단 요약:

- **@Qualifier**: 특정 빈을 직접 지정 — 높은 우선순위.
- **@Primary**: 여러 빈 중 기본 선택(명시적 지정이 없을 때 사용).

읽어주셔서 감사합니다 — 예제 코드를 직접 만들어 보고, 어떤 방식이 프로젝트 구조에 더 맞는지 결정해 보세요.

