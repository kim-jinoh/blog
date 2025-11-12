---
title: "Spring Bean Scope 정리: singleton, prototype, request, session, application"
date: 2025-11-12
parent: Java
tags: [spring, bean, scope]
---

Spring에서 빈의 "스코프(scope)"는 빈 인스턴스의 생성 주기와 가시 범위를 결정합니다. 기본 스코프는 `singleton`이며, 필요에 따라 `prototype`, 웹 환경에서의 `request`, `session`, `application` 등을 사용합니다. 아래에서 각 스코프의 목적, 특징, 사용 예를 정리합니다.

## 1. singleton (기본)

- 설명: 애플리케이션 컨텍스트 당 빈 인스턴스가 하나만 생성됩니다.
- 특징: 상태를 가지면 안 되거나(무상태), 스레드 안전을 고려해야 합니다.

```java
@Component
public class MySingletonService {
    // 싱글톤 빈, 애플리케이션 전역에서 하나의 인스턴스 사용
}
```

## 2. prototype

- 설명: 빈 요청마다 새로운 인스턴스가 생성됩니다.
- 특징: 컨테이너는 생성만 관리하고, 소멸(라이프사이클) 관리는 개발자가 담당합니다.

```java
@Component
@Scope("prototype")
public class MyPrototypeBean {
    // 매번 주입/요청 시 새로운 인스턴스가 생성됨
}
```

- 사용 주의: 싱글톤 빈이 프로토타입 빈을 직접 주입받으면 주입 시점에 한 번만 생성됩니다. 이 경우 아래와 같은 패턴을 사용해야 의도한대로 매번 새로운 인스턴스를 얻을 수 있습니다.

### 2-1. ObjectProvider / javax.inject.Provider 사용 예

- ObjectProvider는 스프링에서 제공하는 지연 조회(lazy lookup) 인터페이스입니다. `getObject()`(또는 `getIfAvailable()`) 호출 시마다 새로운 프로토타입 인스턴스를 얻을 수 있습니다.

```java
@Component
public class SingletonUsingProvider {
    private final ObjectProvider<MyPrototypeBean> prototypeProvider;

    public SingletonUsingProvider(ObjectProvider<MyPrototypeBean> prototypeProvider) {
        this.prototypeProvider = prototypeProvider;
    }

    public void doWork() {
        MyPrototypeBean bean = prototypeProvider.getObject(); // 매번 새로운 인스턴스
        bean.execute();
    }
}
```

- `javax.inject.Provider`도 비슷하게 사용할 수 있습니다:

```java
@Component
public class SingletonUsingJsr330Provider {
    private final Provider<MyPrototypeBean> provider;

    public SingletonUsingJsr330Provider(Provider<MyPrototypeBean> provider) {
        this.provider = provider;
    }

    public void doWork() {
        MyPrototypeBean bean = provider.get(); // 매번 새로운 인스턴스
        bean.execute();
    }
}
```
---

## 3. request (웹 전용)

- 설명: HTTP 요청(request) 하나당 빈 인스턴스가 하나 생성되어 요청이 끝나면 소멸됩니다.
- 특징: 웹 요청별 상태를 갖는 빈에 사용.

```java
@Component
@Scope(value = WebApplicationContext.SCOPE_REQUEST, proxyMode = ScopedProxyMode.TARGET_CLASS)
public class MyRequestScopedBean {
    // 요청마다 새 인스턴스
}
```

- `proxyMode = ScopedProxyMode.TARGET_CLASS`를 지정하면 싱글톤 빈에 프록시가 주입되어, 실제 호출 시점에 현재 요청에 해당하는 타깃 인스턴스를 참조합니다. 따라서 싱글톤에서 요청 스코프 빈을 안전하게 사용할 수 있습니다.

### proxyMode(ScopedProxyMode) 동작 예

```java
@Component
public class ControllerLikeService {
    private final MyRequestScopedBean requestBean;

    public ControllerLikeService(MyRequestScopedBean requestBean) {
        this.requestBean = requestBean; // 실제는 프록시가 주입되어 요청마다 다른 인스턴스를 참조
    }

    public void handle() {
        requestBean.doSomething(); // 호출 시 실제 요청 스코프 빈으로 위임
    }
}
```

- 위와 같이 프록시를 사용하면 싱글톤이나 다른 긴수명 빈에 안전하게 주입할 수 있습니다.

---

## 4. session (웹 전용)

- 설명: HTTP 세션당 빈 인스턴스가 하나 생성됩니다.
- 특징: 세션 단위의 사용자 상태 유지에 적합.

```java
@Component
@Scope(value = WebApplicationContext.SCOPE_SESSION, proxyMode = ScopedProxyMode.TARGET_CLASS)
public class MySessionScopedBean {
    // 세션별 인스턴스
}
```

---

## 5. application (웹 전용, 서블릿 컨텍스트)

- 설명: 서블릿 컨텍스트(애플리케이션) 당 하나의 인스턴스를 가집니다. 전역(singleton)과 유사하지만 웹 컨텍스트 단위입니다.

```java
@Component
@Scope(value = WebApplicationContext.SCOPE_APPLICATION, proxyMode = ScopedProxyMode.TARGET_CLASS)
public class MyApplicationScopedBean {
}
```

---

## 6. 스코프 관련 실전 팁

- 기본은 `singleton`. 상태가 필요하면 신중히 설계하세요.
- 싱글톤에서 요청/세션 빈을 직접 주입하면 기대와 다른 동작(주입 시점 고정)이 발생합니다. 이 경우 `proxyMode` 또는 `ObjectProvider<T>` 같은 지연 검색 패턴을 사용하세요.
- prototype 빈은 컨테이너가 소멸을 관리하지 않습니다.
- 테스트 시에는 스코프 설정이 테스트 컨텍스트에 영향을 줄 수 있으니 주의하세요.

## 7. 예: 싱글톤에서 요청 스코프 빈 사용 (정리)

- Provider(ObjectProvider) 패턴: 싱글톤에서 필요한 시점에 매번 새로운 인스턴스를 조회하여 사용한다.
- Scoped proxy 패턴: 프록시를 주입해 호출 시점에 현재 스코프의 실제 인스턴스로 위임한다.

### 선택 가이드: Provider vs Scoped-proxy

- 언제 Provider/ObjectProvider/Lookup를 선택할까?
  - 프로토타입 빈을 주입받아 '명시적으로 매번 새 인스턴스'가 필요할 때.
  - 호출 빈도나 생성 시점을 제어하고 싶을 때(오버헤드 최소화).
  - 프록시 관련 복잡도나 잠재적 성능 비용을 피하고 싶을 때.
  - 예: 작업 단위마다 완전히 분리된 객체가 필요할 경우.

- 언제 Scoped-proxy를 선택할까?
  - request/session 같은 웹 스코프 빈을 싱글톤(또는 긴 수명 빈)에 투명하게 주입해야 할 때.
  - 호출 지점에서 스코프를 의식하지 않고 사용하길 원할 때(프록시가 위임 처리).
  - 코드 변경을 최소화하면서 기존 설계에 스코프를 도입하고 싶을 때.

- 요약
  - Provider: 명시적, 의도 명확, 리팩터링·디버깅이 쉬움. 호출 시점 제어 가능.
  - Scoped-proxy: 편리하고 투명하지만 프록시 오버헤드가 있고 런타임에 동작이 숨겨질 수 있음.

---

## 9. 결론

- 스코프 선택은 빈의 상태 유지 방식과 라이프사이클 책임에 직접적인 영향을 줍니다. 가능한 불변(무상태)으로 설계하되, 상태가 필요하면 적절한 스코프와 프록시/Provider 패턴을 사용하세요.
