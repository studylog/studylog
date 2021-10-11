# 아이템13. clone 재정의는 주의해서 진행하라.
- Cloneable은 복제해도 되는 인터페이스임을 명시하는 용도의 mixin 인터페이스이다. 하지만 아쉽게도 의도한 목적을 이루지 못했다. 가장 큰 문제는 clone 메서드가 선언된 곳이 Cloneable이 아닌 Object이고, 그마저도 protected로 되어있다. 즉, Cloneable을 구현하는 것만으로는 외부에서 clone 메서드를 호출할 수 없다.

## 1. Cloneable 인터페이스
```java
/**
 * A class implements the <code>Cloneable</code> interface to
 * indicate to the {@link java.lang.Object#clone()} method that it
 * is legal for that method to make a
 * field-for-field copy of instances of that class.
 * <p>
 * Invoking Object's clone method on an instance that does not implement the
 * <code>Cloneable</code> interface results in the exception
 * <code>CloneNotSupportedException</code> being thrown.
 * <p>
 * By convention, classes that implement this interface should override
 * {@code Object.clone} (which is protected) with a public method.
 * See {@link java.lang.Object#clone()} for details on overriding this
 * method.
 * <p>
 * Note that this interface does <i>not</i> contain the {@code clone} method.
 * Therefore, it is not possible to clone an object merely by virtue of the
 * fact that it implements this interface.  Even if the clone method is invoked
 * reflectively, there is no guarantee that it will succeed.
 *
 * @author  unascribed
 * @see     java.lang.CloneNotSupportedException
 * @see     java.lang.Object#clone()
 * @since   1.0
 */
public interface Cloneable {
}
```
- 자바의 Cloneable 인터페이스를 보면 아무런 메서드도 없다.
- 아무것도 없지만, 사실은 Object의 clone 메서드의 동작방식을 결정한다.
- Cloneable을 구현한 클래스의 인스턴스에서 clone을 호출하면, 그 객체의 필드들을 하나하나 복사한 객체를 반환하며, 그렇지 않은 클래스의 인스턴스에서 호출하면, ClassNotSupportedException을 던진다.

## 2. Object 클래스의 clone 규약
```java
@HotSpotIntrinsicCandidate
protected native Object clone() throws CloneNotSupportedException;
```
- Object에 명시된 clone 규약이 주석으로 쓰여져 있다.
- x.clone() != x 는 참이다. 복사한 객체와 원본 객체는 서로 다른 객체이다.
- x.clone().getClass() == x.getClass()는 일반적으로 참이다. 하지만 반드시 만족해야하는 것은 아니다. 관례상, 이 방법으로 반환된 객체는 독립성이 있어야 한다. 이를 만족하려면 super.clone()으로 얻은 객체의 필드 중 하나 이상을 반환 전에 수정할 수도 있다.
- x.clone().equals(x)는 참이다. 복사한 객체와 원본 객체는 논리적 동치성이 같다.
- Cloneable을 구현하지 않은 경우, CloneNotSupportedException이 throw된다.
- 모든 Array는 Cloneable을 구현하도록 고려되었다. clone() 메서드는 T[] (T는 기본타입 또는 참조타입)를 리턴하도록 설계되었다.
- 기본적으로 Object.clone()은 clone 대상 클래스에 대해 새로운 객체를 new로 생성
- 모든 필드들에 대해 초기화를 진행한다.
- 하지만 필드에 대한 객체를 다시 생성하지 않는 Shallow copy 방식으로 수행한다. (deep copy 아님.)
- 클래스에 대한 복제본을 원하지 않는 경우 clone 메서드를 재정의하여 CloneNotSupportedException을 throw하도록 한다.

## 3. clone 메서드 재정의시, 주의할 점 !
### 1. 기본적인 clone 메서드 재정의
```java
class PhoneNumber implements Cloneable {
    
    @Override
    public PhoneNumber clone() {
        try {
            return (PhoneNumber) super.clone();
        } catch(ClassNotSupportedException e) {
            // 아무처리를 하지 않거나, RuntimeException으로 감싸는 것이 사용하기 편하다.
        }
    }
}
```
- super.clone()을 실행하면 PhoneNumber에 대한 완벽한 복제가 이루어진다.
- super.clone()의 리턴타입은 Object이지만, 자바의 공변 변환 타이핑 기능을 이용해 PhoneNumber 타입으로 캐스팅하여 리턴하는 것이 가능하다.
- try-catch 부분으로 감싼 것은 super.clone() 메서드에서 ClassNotSupportedException이라는 checked exception을 리턴하기 때문에 처리해주었다. 하지만 PhoneNumbe가 Cloneable을 구현하기 때문에 절대 실패하지 않는다. 따라서 이부분은 RuntimeException으로 처리하거나, 아무것도 설정하지 않아야 한다.

### 2. 가변 상태를 갖는 필드에 대한 복제 
```java
public class Stack implements Cloneable{
  private Object[] elements;
  private int size = 0;
  private static final int DEFAULT_INITIAL_CAPACITY = 16;

  public Stack() {
    this.elements = new Object[DEFAULT_INITIAL_CAPACITY];
  }

  public void push(Object o) {}
  }
  ...

  @Override
  public Stack clone() {
    try {
      Stack result = (Stack) super.clone();
      result.elements = 
    } catch(CloneNotSupportedException e) {
    }
  }
}
```
-  단순히 clone() 메서드를 이용해 super.clone()만 실행하게 된다면, new Stack()을 통해 새로운 객체가 생성되고 필드 모두 원본 객체와 동일하게 초기화가 될 것이다. 하지만, Object의 clone 기본 규약에는 deep copy가 아닌 shallow copy를 이요해 초기화를 진행한다고 적혀있다. 따라서, 배열과 같은 가변필드는 원본 필드와 객체를 공유하게 된다.
- clone() 메서드는 사실상 생성자와 같은 효과를 낸다. 즉, clone은 원본 객체에 아무런 해를 끼치지 않는 동시에 복제된 객체의 불변식을 보장해야 한다. 그렇기 때문에 제대로 복제하기 위해서는 elements라는 배열을 똑같이 복사해서 만들어줘야 한다.

### 3. 배열 복사
- 배열을 복제하는 방법 중 가장 권장하는 방법은 array.clone()을 이용해 복사하는 방법이다. 사실, 배열은 clone기능을 가장 제대로 사용하는 유일한 예이다.
하지만, array 필드가 final이 적용되어 있다면 array.clone()을 통해 초기화를 할 수 없다. (final이기 때문에 객체 생성이후 초기화 불가)
- Cloneable 아키텍처는 가변 객체를 참조하는 필드는 final로 선언하라 라는 일반 용법과 충돌한다. 그래서 복제할 수 있는 클래스를 만들기 위해 일부 필드에서 final 한정자를 제거해야할 수도 있다. 

### 4. stack-overflow 문제
```java
public class HashTable implements Cloneable  {
  private Entry[] buckets = ...;
  private static class Entry {
    final Object key;
    Object value;
    Entry next;

    Entry(Object key, Object value, Entry next) {
      this.key = key;
      this.value = value;
      this.next = next;
    }
  }

  @Override
  public HashTable clone() {
    try {
      HashTable result = (HashTable) super.clone();
      result.buckets = buckets.clone();
      return result;
    } catch(CloneNotSupportedException e) {
      throw new Assertion();
    }
  }
}
```
- 복제본은 자신만의 버킷 배열은 갖지만, 배열 내의 Entry는 원본과 같은 연결리스트를 참조하여, 불변성이 깨지게된다.
- 그래서 HashTable.Entry 클래스는 내부적으로 deep copy를 지원하도록 보강되었다. 연결리스트에 대한 next를 복제할 때 재귀적으로 clone을 호출하도록 되어있는데, 재귀호출 때문에 연결리스트의 size만큼 stack frame을 소비하여, 리스트가 길면 stack-overflow 에러를 발생시킬 위험이 있다.
- 이 문제를 해결하기 위해 재귀 호출을 통한 deep copy 대신 반복자를 써서 순회하는 방법으로 수정해야 한다.

### 5. 생성자 내에서는 재정의될 수 있는 메서드를 호출하지 말자.
- 만약 clone이 하위 클래스에서 재정의한 메서드를 호출하면, 하위 클래스는 복사과정에서 자신의 상태를 교정할 기회를 잃게 되어 원본과 복제본의 상태가 달라질 수 있다.

### 6. 스레드 안전성을 고려한다면 적절히 동기화해야 한다.
- 스레드 안정성을 고려한가면 clone() 메서드에 대해 적절히 동기화 처리를 해야한다.
```java
@Override
public synchronized Object clone() {
  try {
    Object result = super.clone();
  } catch(CloneNotSupportedException e) {
  }
}
```
## 4. clone() 재정의 방법
1. Cloneable을 구현하는 모든 클래스는 clone()을 재정의해야 한다.
2. 접근 제한자는 public으로
3. 반환타입은 클래스 자신으로
4. 가장 먼저 super.clone()을 호출한 후 필요한 필드를 적절히 수정한다.
5. 이후 깊은 구조까지 클론한다. (보통 재귀지만, 항상 정답은 아니다.)
6. 기본 타입 필드, 불변 객체 참조만 갖는 클래스면 아무 필드도 수정할 필요가 없다.
7. 고유 ID는 비록 기본 타입, 불변이어도 수정해야 한다.

## 5. clone()재정의 대체법
* 복사 생성자: 단순히 자신과 같은 클래스의 인스턴스를 인수로 받는 생성자
```java
public Yum(Yum yum) {...}
```
* 복사 팩터리
```java
public static Yum newInstance(Yum yum) {...}
```
* 복사 생성자와 복사 팩터리의 장점
1. 위험한 객체 생성 매커니즘을 사용하지 않는다. (생성자없이 객체 생성)
2. 문서화된 규약에 기대지 않는다.
3. final 필드 용법과 충돌하지 않는다.
4. 불필요한 검사 예외를 던지지 않는다.
5. 형변환 필요가 없다.
6. 해당 클래스가 구현한 '인터페이스' 타입의 인스턴스를 인수로 받을 수 있다.
    - 인터페이스 기반 복사 생성자 = 변환 생성자 (conversion constructor)
    - 인터페이스 기반 복사 팩터리 = 변환 팩터리 (conversion factory)