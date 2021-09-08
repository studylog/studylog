# 아이템3. private 생성자나 열거타입으로 싱글턴임을 보증하라.

## 싱글턴이란?
* 싱글턴(singleton): 인스턴스를 오직 하나만 생성할 수 있는 클래스
- 함수와 같은 무상태(stateless) 객체
- 설계상 유일해야하는 시스템 컴포넌트
- 클래스를 싱글턴으로 만들면 이를 사용하는 클라이언트를 테스트하기 어려울 수 있음

* Mock 객체란?
- 실제 객체를 다양한 조건으로 인해 제대로 구현하기 어려울 경우 **가짜 객체를 만들어 사용**하는데, 이를 Mock 객체라 한다.

* Mock 객체가 필요한 경우
- 테스트 작성을 위한 환경 구축이 어려운 경우
- 테스트가 특정 경우나 순간에 의존적인 경우
- 시간이 걸리는 경우

## 1. public static 멤버가 final 필드
```java
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();

    private Elvis() { ... }
}
```
- private 생성자가 public static final 필드를 초기화할 때 딱 한번만 호출된다.
- public 혹은 protected 생성자가 없으므로, Elvis 클래스가 초기화될 때 만들어진 인스턴스는 하나뿐임이 보장된다.

### 장점
1. public 필드 방식은 해당 클래스가 싱글턴임이 API에 명백하게 드러난다. (final이므로 다른 객체 참조 불가)
2. 간결하다.

## 2. static factory 방식
```java
public class Elvis {
    private static final Elvis INSTANCE = new Elvis();

    private Elvis() { ... }

    public static Elvis getInstance() {
        return INSTANCE;
    }
}
```
- Elvis.getInstance()는 항상 같은 객체의 참조를 반환하므로 인스턴스가 하나임을 보장한다.

### 장점
1. 현재는 singleton 객체를 리턴하는 정적 메서드지만, 향후에 필요에 따라 변경할 수 있는 확장성이 있다. 유일한 메서드를 반환하는 팩터리 메서드가 호출하는 스레드별로 다른 인스턴스를 넘겨주도록 리턴하는 방법과 같이 확장성이 열려있다.
2. 정적 팩터리를 제네릭 싱글턴 팩터리로 만들 수 있다.
3. 정적 팩터리의 메서드 참조를 공급자(supplier)로 사용할 수 있다.

- 다음과 같은 장점이 필요하지 않다면, public static final 필드 방식이 더 좋다.

### Reflection 방어
- public static final 방식과 static factory 방식은 권한이 있는 클라이언트가 Reflection API인 AccessibleObject.setAccessible을 사용해 private 생성자를 호출할 수 있는 문제점이 있다. 이러한 공격을 방어하려면 두번째 객체가 생성되려 할 때 다음과 같이 예외처리가 가능하다.
```java
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();

    private Elvis() { 
        if( INSTANCE != null) {
            throw new RuntimeException("Can't create Constructor");
        }
          //... 
    }
}
```

### singleton class 직렬화
- singleton class를 직렬화하려면 단순히 Serializable을 구현하는 것만으로는 부족하다. 모든 인스턴스 필드를 transient(일시적) 약어를 선언하고 readResolve 메서드를 제공해야 한다. 이렇게 하지 않으면 역직렬화시 새로운 인스턴스가 생성된다.
```java
public class Elvis implements Serializable {
    private static final Elvis INSTANCE = new Elvis();

    private Elvis() { ... }

    public class Elvis getInstance() {
        return INSTANCE;
    }

    private Object readResolve() { // singleton임을 보장
        return INSTANCE; 
        // 역직렬화가 되어 새로운 인스턴스가 생성되더라도 INSTANCE를 반환하여 싱글턴 보장
        // 새로운 인스턴스는 GC에 의해 UnReachable 형태로 판별되어 제거
    }
}
```

## 3. Enum 방식
```java
public enum Elvis {
    INSTANCE;
}
```
- 원소가 하나인 Enum 타입을 선언해 singleton을 만들 수 있다.

### 장점
1. public static 방식보다 더 간결하다.
2. 추가 코드 없이 직렬화 가능(기본적으로 직렬화되어있음.)
3. Reflection 공격과 아주 복잡한 직렬화 상황에도 제 2의 인스턴스가 생기는 일을 완벽히 방어

- 대부분의 상황에서 원소가 하나뿐인 열거 타입이 singleton을 만드는 가장 좋은 방법이다. 하지만, 만들려는 singleton이 Enum 이외의 클래스를 상속해야하는 경우 이 방법은 사용할 수 없다.