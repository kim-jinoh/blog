---
title: Spring Docker 개발 가이드
parent: Java
date: 2025-10-21
tags: [java, spring, deployment, docker]
---

이 글은 로컬에서 Spring(Gradle) 애플리케이션을 Docker로 개발 환경에서 실행할 수 있도록 정리한 가이드입니다.

핵심 요약
- 개발 전용 구성: 호스트 파일을 컨테이너에 마운트해 개발 중 파일 변경이 컨테이너에 반영되도록 구성할 수 있습니다(IDE 자동 빌드 또는 `spring-boot-devtools` 사용 권장).
- 포트: 호스트 `8080` ↔ 컨테이너 `8080` 매핑(개발용)
- Compose 명령은 최신 방식인 `docker compose` 사용을 권장합니다.

파일 구성
- `Dockerfile` (개발용)
- `docker-compose.yml` (개발용)
- `docs/docker.md` (실행/재시작 명령)

개요
개발 목표는 "호스트의 코드 변경이 컨테이너에서 빠르게 반영되는 개발 환경"을 만드는 것입니다. 이를 위해 이미지를 빌드한 뒤 소스 전체를 컨테이너에 마운트하고, Gradle wrapper로 애플리케이션을 구동합니다.

개발용 `Dockerfile` (간략)
```dockerfile
FROM gradle:9.1.0-jdk25
WORKDIR /home/gradle/project
VOLUME ["/home/gradle/project"]
EXPOSE 8080
CMD ["/bin/sh", "-c", "./gradlew bootRun"]
```

설명
- `gradle:9.1.0-jdk25` 이미지를 사용해 개발 환경에서 Gradle과 JDK를 함께 제공합니다.
- 프로젝트 파일을 컨테이너 작업 디렉터리로 복사합니다. (필요 시 `RUN chmod +x gradlew` 추가)
- `VOLUME`으로 개발 중 소스 마운트를 허용해 호스트 변경이 컨테이너에 반영되도록 합니다(자동 빌드나 `spring-boot-devtools` 필요할 수 있음).

개발용 `docker-compose.yml` (간략)
```yaml
version: '3.8'
services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
    image: platform-app:dev
    ports:
      - "8080:8080"
    environment:
      - SPRING_PROFILES_ACTIVE=dev
    volumes:
      - ./:/home/gradle/project
    command: ["/bin/sh", "-c", "./gradlew bootRun"]
```

중요 포인트
- `ENTRYPOINT`와 `command` 충돌: 개발용으로 `command` 혹은 `entrypoint`를 덮어써서 `./gradlew bootRun`만 실행하도록 합니다. 프로덕션은 JAR를 직접 실행하는 방식으로 분리하세요.
- 포트 일치: 애플리케이션이 내부적으로 8080으로 실행되므로 `EXPOSE 8080`과 `ports: - "8080:8080"`을 맞춰야 합니다.
- Gradle wrapper와 JDK 호환성: 로컬/이미지의 JDK 버전과 Gradle wrapper가 호환되어야 합니다. 예) JDK25를 쓰면 Gradle 9.1 이상 권장. 참고: 공식 사이트(`https://start.spring.io/`)로 생성하면 기본 wrapper 버전이 `8.1.4`로 나올 수 있는데, 사용한 `gradle` 베이스 이미지에서 해당 태그를 찾을 수 없어 이미지 호환을 위해 wrapper를 `9.1`로 업그레이드했습니다(업그레이드 명령: `./gradlew wrapper --gradle-version 9.1.0`).

실행/재시작 주요 명령
- 빌드 및 실행(백그라운드):
```bash
docker compose up --build -d
```
- 완전 재시작:
```bash
docker compose down
docker compose up --build -d
```
- 빠른 재시작(이미지 재빌드 생략):
```bash
docker compose restart app
```
- 로그 확인:
```bash
docker compose logs -f app
```

문제 해결 팁
- 애플리케이션이 보이지 않을 때: `docker compose ps`와 `docker compose logs -f app`로 포트와 에러 로그를 확인하세요.
- `./gradlew` 실행 권한 문제: 호스트에서 `chmod +x gradlew` 또는 Dockerfile에 `RUN chmod +x gradlew` 추가.
- 호스트 마운트로 인해 컨테이너에 있던 빌드 산출물이 마운트로 가려질 수 있음(의도된 동작). 프로덕션 테스트는 빌드된 JAR로 별도 실행하세요.

docs 업데이트
- 문서(`docs/docker.md`)의 명령은 최신 Compose 플러그인 사용을 위해 `docker compose`로 표기해야 합니다.

마무리

테스트 환경
- 이 프로젝트는 https://start.spring.io/에서 생성한 Spring Boot 애플리케이션으로 테스트했습니다.
- Spring Boot 버전: 3.5.6
- JDK: 25
- Gradle wrapper: 초기값 `8.1.4` (start.spring.io 기본) → 이미지 호환성 문제로 `9.1`로 업그레이드함

마무리
개발 단계에서는 위와 같이 단순화된 dev-only Docker 구성이 빠른 피드백 루프를 제공합니다. 프로덕션 배포가 필요해지면 multi-stage 빌드로 JAR만 포함한 런타임 이미지를 만들고 Compose 파일을 분리하세요.


