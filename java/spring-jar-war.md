---
title: "JAR vs WAR — Spring 애플리케이션 배포 방식 비교"
date: 2025-10-18
tags: [java, spring, deployment]
---

### 개요

Java 웹 애플리케이션을 배포할 때 흔히 마주하는 두 가지 아카이브 형식이 있습니다: **JAR**와 **WAR**. 둘은 목적과 구조, 배포 방식에서 차이가 있으며, 특히 Spring(특히 Spring Boot)을 사용할 때 어떤 형식을 선택하느냐에 따라 개발·운영 흐름이 달라집니다. 이 글에서는 핵심 차이와 실전에서의 선택 기준을 정리합니다.

### JAR란?

- **JAR (Java ARchive)**: 여러 클래스 파일과 리소스를 하나의 파일로 묶은 표준 Java 아카이브입니다.

- 일반적인 라이브러리 배포에 널리 쓰이며, Spring Boot에서는 **내장 서블릿 컨테이너(예: Tomcat, Jetty)를 포함한 실행 가능한 JAR**을 만들어 독립 실행형 애플리케이션으로 배포할 수 있습니다.

특징:

- 보통 `META-INF/MANIFEST.MF`에 실행 진입점(Main-Class) 정보를 넣어 실행 가능하게 만듭니다.

- 컨테이너가 불필요해 컨테이너 설정/관리 부담이 적고, Docker 이미지로 만들기 편리합니다.

### WAR란?

- **WAR (Web Application Archive)**: Java 웹 애플리케이션을 배포하기 위한 표준 아카이브로, `WEB-INF/` 디렉터리 구조와 `web.xml`(필요 시)을 포함합니다.

- 전통적인 서블릿 컨테이너(예: Tomcat, Jetty, WildFly)에 배포됩니다.

특징:

- 애플리케이션 서버가 제공하는 공용 라이브러리나 설정을 활용할 수 있습니다.

- 여러 애플리케이션을 동일한 서버에서 호스팅할 때 유리합니다.

### 구조적 차이 비교

- 파일 구조
  - JAR: `META-INF/`, `com/yourcompany/...` 등의 일반 Java 패키지
  - WAR: `WEB-INF/web.xml` (옵션), `WEB-INF/classes/`, `WEB-INF/lib/`
- 실행 방식
  - JAR: 자체 실행(내장 서버) 또는 클래스패스로 사용
  - WAR: 외부 서블릿 컨테이너에 배포하여 실행

### Spring (Spring Boot) 관점에서의 차이

- Spring Boot + JAR
  - `spring-boot-starter`와 `spring-boot-maven-plugin`을 사용하면 실행 가능한 단일 JAR로 패키징되어 `java -jar app.jar`로 실행됩니다.
  - 운영 환경에서 컨테이너화(예: Docker)를 통해 빠르게 배포/스케일링하기 좋습니다.

- Spring Boot + WAR
  - `<packaging>war</packaging>`를 지정하고, 외부 컨테이너에서 동작하도록 하려면 `SpringBootServletInitializer`를 상속받아 설정해야 합니다.
  - 기존 기업 환경이나 중앙 Tomcat에 배포하는 레거시 요구사항이 있는 경우 유리합니다.

간단한 예시(빌드 도구별 설정)

JAR(기본, Spring Boot 실행 가능 JAR) - Gradle (Groovy DSL):

```groovy
plugins {
    id 'org.springframework.boot' version '3.1.6'
    id 'io.spring.dependency-management' version '1.1.0'
    id 'java'
}

group = 'com.example'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '17'

repositories {
    mavenCentral()
}

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}

// 실행 가능한 JAR은 기본적으로 Spring Boot Gradle 플러그인이 처리합니다
```

WAR(외부 컨테이너에 배포) - Gradle (Groovy DSL):

```groovy
plugins {
    id 'org.springframework.boot' version '3.1.6'
    id 'io.spring.dependency-management' version '1.1.0'
    id 'java'
    id 'war'
}

group = 'com.example'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '17'

repositories {
    mavenCentral()
}

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
    providedRuntime 'org.springframework.boot:spring-boot-starter-tomcat'
}

// WAR로 패키징하려면 'war' 플러그인을 사용하고 Tomcat을 providedRuntime으로 지정합니다
```

그리고 메인 애플리케이션 클래스는 WAR용으로 다음과 같이 작성합니다:

```java
public class MyApplication extends SpringBootServletInitializer {
    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder application) {
        return application.sources(MyApplication.class);
    }

    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }
}
```

### 운영 관점에서의 장단점

- **JAR (실행 가능한 JAR)**
  - 장점: 독립 실행, 컨테이너화 쉬움, 배포 파이프라인 단순화, 마이크로서비스에 적합
  - 단점: 여러 애플리케이션을 하나의 물리적 서버에서 관리해야 할 때 중복된 런타임이 발생할 수 있음

- **WAR**
  - 장점: 기존 앱서버 인프라 활용 가능, 여러 앱을 동일 컨테이너에서 관리 가능
  - 단점: 서버 관리·설정 부담, 컨테이너 독립적인 배포 파이프라인 구성 시 제약

### 마무리 체크리스트

- **JAR**: 빠른 배포, 컨테이너 친화적, Spring Boot 내장 서버 사용
- **WAR**: 전통적 서버 기반 배포, 앱서버 기능(관리·모듈화) 활용
