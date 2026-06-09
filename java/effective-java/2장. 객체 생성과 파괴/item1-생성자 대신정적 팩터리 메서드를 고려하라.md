---
title: Item1. 생성자 대신 정적 팩터리 메서드를 고려하라
parent: EffectiveJava
nav_order: 1
description: 정적 팩터리 메서드의 5가지 장점과 2가지 단점, 그리고 네이밍 컨벤션을 정리한다.
keywords: 정적 팩터리 메서드, static factory method, Effective Java, valueOf, of, from, getInstance
---

# 정적 팩터리 메서드
> 지금 얘기하는 정적 팩터리 메서드는 디자인 패턴의 팩터리 메서드와 다르다.

클래스의 인스턴스를 얻는 전통적인 방법은 `public` 생성자다.
하지만 클래스는 생성자와 별도로 **정적 팩터리 메서드(static factory method)** 를 제공할 수 있다.

## 정적 팩터리 메서드의 장점 다섯가지

### 1. 이름을 가질 수 있다.

생성자에 넘기는 매개변수와 생성자 자체만으로 생성할 객체의 특성을 제대로 설명하지 못한다.
반면 정적 팩터리는 이름만 잘 지으면 반환될 객체의 특성을 쉽게 묘사 가능하다.
한 클래스에 시그니처가 같은 생성자가 여러 개 필요 할 것 같으면, 생성자를 정적 팩터리 메서드로 바꾸고 각각의 차이를 잘 드러내는 이름을 지어주면 좋다.

```java
public class User {
    private String name;
    private String email;
    private boolean isAdmin;

    private User(String name, String email, boolean isAdmin) {
        this.name = name;
        this.email = email;
        this.isAdmin = isAdmin;
    }

    public static User createAdmin(String name, String email) {
        return new User(name, email, true);
    }

    public static User createGuest(String name, String email) {
        return new User(name, email, false);
    }
}

// 생성자 방식 - true가 무엇을 의미하는지 알 수 없음
User user = new User("홍길동", "hong@example.com", true);

// 정적 팩터리 방식 - 이름만으로 의도가 명확
User admin = User.createAdmin("홍길동", "hong@example.com");
User guest = User.createGuest("김철수", "kim@example.com");
```

### 2. 호출될 때마다 인스턴스를 새로 생성하지는 않아도 된다.

불변 클래스는 인스턴스를 미리 만들어 놓거나 새로 생성한 인스턴스를 캐싱하여 재활용하는 식으로 불필요한 객체 생성을 피할 수 있다.
따라서 같은 객체가 자주 요청되는 상황이라면 성능을 상당히 끌어올려 준다.

이처럼 인스턴스의 생명주기를 통제하는 클래스를 **인스턴스 통제(instance-controlled) 클래스**라 한다.
인스턴스를 통제하면 싱글턴(Singleton)으로 만들 수도, 인스턴스화 불가(noninstantiable)로 만들 수도 있다.

```java
public final class Boolean {
    public static final Boolean TRUE = new Boolean(true);
    public static final Boolean FALSE = new Boolean(false);

    // 매번 새 인스턴스를 만들지 않고 캐싱된 인스턴스를 반환
    public static Boolean valueOf(boolean b) {
        return b ? TRUE : FALSE;
    }
}
```

### 3. 반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다.

반환할 객체의 클래스를 자유롭게 선택할 수 있는 유연성이 생긴다.
API를 만들 때 이 유연성을 응용하면 구현 클래스를 공개하지 않고도 그 객체를 반환할 수 있어 API를 작게 유지할 수 있다.

자바 컬렉션 프레임워크의 `java.util.Collections`가 대표적인 예다.
45개의 유틸리티 구현체를 공개하지 않고 정적 팩터리 메서드를 통해서만 제공한다.

```java
// 반환 타입은 인터페이스지만, 실제로는 하위 구현체를 반환
List<String> emptyList = Collections.emptyList();
List<String> syncList = Collections.synchronizedList(new ArrayList<>());
List<String> unmodifiable = Collections.unmodifiableList(new ArrayList<>());
```

### 4. 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다.

반환 타입의 하위 타입이기만 하면 어떤 클래스의 객체를 반환하든 상관없다.
클라이언트는 팩터리가 건네주는 객체가 어느 클래스의 인스턴스인지 알 수도 없고 알 필요도 없다.

`EnumSet` 클래스가 대표적인 예다. 원소가 64개 이하면 `long` 변수 하나로 관리하는 `RegularEnumSet`을, 65개 이상이면 `long` 배열로 관리하는 `JumboEnumSet`을 반환한다.

```java
// 원소 수에 따라 내부적으로 RegularEnumSet 또는 JumboEnumSet을 반환
// 클라이언트는 어떤 구현체인지 알 필요가 없다
EnumSet<Day> weekdays = EnumSet.of(Day.MON, Day.TUE, Day.WED, Day.THU, Day.FRI);
```

### 5. 정적 팩터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다.

이런 유연함은 **서비스 제공자 프레임워크(Service Provider Framework)** 를 만드는 근간이 된다.
대표적인 예가 `JDBC`다. `Connection`이 서비스 인터페이스 역할을, `DriverManager.registerDriver`가 제공자 등록 API 역할을, `DriverManager.getConnection`이 서비스 접근 API 역할을 한다.

```java
// JDBC 드라이버 구현체는 작성 시점에 존재하지 않았지만,
// 런타임에 등록된 드라이버를 통해 Connection 객체를 반환
Connection conn = DriverManager.getConnection("jdbc:mysql://...", user, password);
```

## 정적 팩터리 메서드의 단점

### 1. 정적 팩터리 메서드만 제공하면 하위 클래스를 만들 수 없다.

상속을 하려면 `public`이나 `protected` 생성자가 필요하다.
따라서 생성자를 `private`으로 막고 정적 팩터리 메서드만 제공하는 클래스는 상속이 불가능하다.
이는 오히려 컴포지션(composition)을 유도하므로 장점으로 볼 수도 있다.

### 2. 정적 팩터리 메서드는 프로그래머가 찾기 어렵다.

생성자처럼 API 문서에 명확히 드러나지 않아 사용자가 직접 찾아야 한다.
이를 완화하기 위해 널리 알려진 네이밍 컨벤션을 따르는 것이 좋다.

| 메서드명 | 설명 | 예시 |
|---|---|---|
| `from` | 매개변수 하나, 해당 타입의 인스턴스 반환 | `Date.from(instant)` |
| `of` | 여러 매개변수를 받아 인스턴스 반환 | `EnumSet.of(JACK, QUEEN)` |
| `valueOf` | `from`과 `of`의 더 자세한 버전 | `BigInteger.valueOf(Integer.MAX_VALUE)` |
| `instance` / `getInstance` | 명시한 인스턴스를 반환, 같은 인스턴스임을 보장하지 않음 | `StackWalker.getInstance(options)` |
| `create` / `newInstance` | 매번 새로운 인스턴스를 반환 | `Array.newInstance(classObject, length)` |
| `getType` | `getInstance`와 같으나 다른 클래스에 팩터리 메서드 정의 시 | `Files.getFileStore(path)` |
| `newType` | `newInstance`와 같으나 다른 클래스에 팩터리 메서드 정의 시 | `Files.newBufferedReader(path)` |
| `type` | `getType`과 `newType`의 간결한 버전 | `Collections.list(legacyLitany)` |
