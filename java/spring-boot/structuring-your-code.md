---
title: "Java의 빌드 툴들: Maven vs Gradle — 조사하며 알게 된 것들"
parent: SpringBoot
date: 2025-12-08
categories: [java, build, spring boot, code structure]
---

# structuring Your Code

이 문서는 Spring Boot 애플리케이션의 코드 구조와 권장 관례를 정리한 내용입니다. 원문은 Spring Boot 공식문서의 "Structuring Your Code"를 참고.

## 1. 기본 권장 원칙 (요약)

- 가능한 한 루트(최상위) 패키지에 `@SpringBootApplication`이 붙은 메인 클래스를 두어, 컴포넌트 스캔(component scan)과 엔티티 스캔(entity scan)이 프로젝트 내부로 제한되도록 합니다.
- 자바의 권장 패키지 네이밍 규칙(도메인 역순: `com.example.project`)을 따릅니다.
- 기본 패키지(default package)를 사용하지 마십시오. (package 선언이 없는 클래스는 default package에 속하며, 컴포넌트 스캔 등에서 의도치 않은 문제가 발생할 수 있습니다.)

## 2. "default" 패키지 사용 금지

`package` 선언을 생략하면 클래스는 기본 패키지(default package)에 포함됩니다. 기본 패키지는 다음과 같은 문제를 일으킬 수 있습니다:

- Spring의 `@ComponentScan`, `@ConfigurationPropertiesScan`, `@EntityScan` 등이 모든 JAR의 클래스들을 읽게 되어 성능 저하 또는 충돌 가능성이 높아집니다.
- 패키지 기반 접근 제한자 및 리팩토링 도구의 동작이 예측 불가능해질 수 있습니다.

따라서 항상 명시적인 패키지 선언을 사용하세요. 예: `com.example.myapp`

## 3. 메인 애플리케이션 클래스 위치

메인 애플리케이션 클래스(보통 `@SpringBootApplication`이 붙음)는 애플리케이션의 "루트 패키지"에 위치시키는 것을 권장합니다. 이 위치는 다음을 의미합니다:

- 해당 패키지와 그 하위 패키지들만 컴포넌트 스캔 대상이 되어, 외부 라이브러리의 컴포넌트가 의도치 않게 스캔되는 것을 방지합니다.

예시 디렉토리 구조 (권장)

```text
com
 +- example
     +- myapplication
         +- MyApplication.java
         |
         +- customer
         |   +- Customer.java
         |   +- CustomerController.java
         |   +- CustomerService.java
         |   +- CustomerRepository.java
         |
         +- order
             +- Order.java
             +- OrderController.java
             +- OrderService.java
             +- OrderRepository.java
```

`MyApplication.java` (표준 메인 클래스 예시)

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class MyApplication {

	public static void main(String[] args) {
		SpringApplication.run(MyApplication.class, args);
	}

}
```

원문 참조: Spring Boot — Structuring Your Code (`https://docs.spring.io/spring-boot/3.5/reference/using/structuring-your-code.html`)
