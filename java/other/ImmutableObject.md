# 불변 객체(Immutable Object) 및 final을 사용해야 하는 이유
> 클래스들은 가변적이여야 하는 매우 타당한 이유가 있지 않는 한 반드시 불변으로 만들어야 한다. 만약 클래스를 불변으로 만드는 것이 불가능하다면, 가능한 변경 가능성을 최소화하라.

## 1. 불변 객체(Immutable Object) 및 final을 사용해야 하는 이유
### 불변 객체란?
- 객체 생성 이후 내부의 상태가 변하지 않는 객체이다. 불변 객체는 read-only 메서드만을 제공하며, 객체의 내부 상태를 제공하는 메소드를 제공하지 않거나 제공하는 경우 방어적 복사(defensive-copy)를 통해 제공한다.
- Java의 대표적인 불변 객체로 String이 있다.
- Java에서는 배열이나 객체 등의 참조(reference)를 전달한다. 그렇기 때문에 참조를 통해 값을 수정하면 내부의 상태가 변하기 때문에 내부를 복사하여 전달하고 있는데, 이를 방어적 복사(defensive-copy)라고 한다.

### 불변 객체 및 final을 사용해야 하는 이유
#### 1. Thread-safe하여 병렬 프로그래밍에 유용하며, 동기화를 고려하지 않아도 된다. 
- 멀티 스레드 환경에서 동기화 문제가 발생하는 이유는 공유 자원을 동시에 쓰기(write) 때문이다. 하지만 만약 공유 자원이 불변의 자원이라면 동기화를 고려하지 않아도 된다. 이는 안정성을 보장할 뿐만 아니라 동기화를 하지 않아 성능상의 이점도 가져다 준다.

#### 2. 실패 원자적인(Failure Atomic) 메서드를 만들 수 있다.
- 가변 객체를 통해 어떠한 작업을 하는 도중 예외가 발생하면 해당 객체가 불안정한 상태에 빠질 수 있다. 그리고 불안정한 상태를 갖는 객체는 또 다른 에러를 유발할 수 있다. 하지만 불변 객체라면 어떠한 예외가 발생하여도 메서드 호출 전의 상태를 유지할 수 있을 것이다. 그리고 예외가 발생하여도 발생하지 않은 것처럼 다음 로직을 처리할 수 있다.

#### 3. Cache나 Map의 key, Set의 요소로 활용하기 적합하다.
- 만약 캐시나 Mapd의 key와 Set의 요소인 객체가 변경되었다면 이를 갱신하는 작업을 추가로 해주어야 한다. 하지만 객체가 불변이면 한 번 데이터가 저장된 이후에 다른 부가 작업들을 고려하지 않아도 될 것이고, 이는 캐시나 다른 자료구조를 사용하는 데 용이하다.

#### 4. 부수 효과(side effect)를 피해 오류 가능성을 최소화할 수 있다.
불변 객체는 기본적으로 값의 수정이 불가능하기 때문에 변경가능성이 적으며, 객체의 생성과 사용이 제한된다. 메소드들은 자연스럽게 순수 함수들로 구성될 것이고, 다른 메서드가 호출되어도 객체의 상태가 유지되기 때문에 안전하게 객체를 다시 사용할 수 있다. 

#### 5. 다른 사람이 작성한 함수를 예측가능하며 안전하게 사용할 수 있다. 

#### 6. 가비지 컬렉션의 성능을 높일 수 있다.
GC가 수행될 때, 가비지 컬렉터가 컨테이너 하위의 불변 객체들은 Skip할 수 있도록 도와준다. 왜냐하면 해당 컨테이너가 살아있다는 것은 하위의 불변 객체들 역시 처음에 할당된 그 상태로 참조되고 있다는 것을 의미하기 때문이다.

> there are many benefits of programming with immutable objects.
> 1. Immutable objects are thread-safe so you will not have any synchronization issues.
> 2. Immutable objects are good Map keys and Set elements, since these typically do not change once created.
> 3. Immutability makes it easier to write, use and reason about the code (class invariant is established once and then unchanged)
> 4. Immutability makes it easier to parallelize your program as there are no conflicts among objects.
> 5. The internal state of your program will be consistent even if you have exceptions.
> 6. References to immutable objects can be cached as they are not going to change.

## 2. Java에서 불변 객체를 생성하는 법
### [final 키워드]
Java에서는 불변성을 확보할 수 있도록 final 키워드를 제공하고 있다. Java에서 변수들은 기본적으로 가변적인데, 변수에 final 키워드를 붙이면 참조값을 변경 못하도록 하여 불변 변수로 만들 수 있다.  
```java
final String name = "Old";
name = "New"; // 컴파일 에러
```
Java에서는 final이 붙은 변수의 값을 변경하려고 하면 컴파일 에러가 발생한다. 하지만 final 키워드가 내부의 객체 상태를 변경하지 못하도록 하는 것은 아니다.    
예를 들어 아래와 같이 final로 선언된 List에는 새로운 객체가 더해져도(상태가 변해도) 문제가 없다.    
```java
final List<String> list = new ArrayList<>();
list.add("a");
```
참조에 의해 값이 변경될 수 있는 점을 유의하여 개발해야 한다.

### [불변 클래스 예시]
불변 객체를 생성하기 위해서     
1. 클래스를 final로 선언
2. 모든 클래스 변수를 private와 final로 선언
3. 객체를 생성하기 위한 생성자 또는 정적 팩토리 메서드를 추가
4. 참조에 의해 변경가능성이 있는 경우 방어적 복사를 이용하여 전달

```java
public final class ImmutableClass { 
    private final int age; 
    private final String name; 
    private final List<String> list; 
    
    private ImmutableClass(int age, String name) { 
        this.age = age; 
        this.name = name; 
        this.list = new ArrayList<>(); 
    } 
    
    public static ImmutableClass of(int age, String name) { 
        return new ImmutableClass(age, name); 
    } 
    
    public int getAge() { 
        return age; 
    } 
    
    public String getName() { 
        return name; 
    } 
    
    public List<String> getList() { 
        return Collections.unmodifiableList(list); 
    } 
}
```
- 내부 생성자를 만드는 대신 객체의 생성을 위해 **정적 팩토리 메소드**를 제공 - of 메서드
- Java에서는 생성자를 선언하지 않으면 기본 생성자가 자동으로 생성되는데, 그러면 다른 클래스에서 해당 객체를 자유롭게 호출할 수 있다. 그렇기 때문에 내부 생성자를 만드는 대신 정적 팩토리 메서드를 통해 객체를 생성하도록 강요하는 것이 좋다.
- 참조를 전달하여 클라이언트에 의해 수정가능성이 있는 list를 **방어적 복사**하여 제공 - getList() 메서드
- 배열이나 다른 객체 또는 컬렉션은 참조가 전달되어 수정가능성이 있다. 그렇기 때문에 참조를 통해 변경이 가능한 경우에는 방어적 복사를 통해 값을 반환해야 한다. 

## 출처
- https://www.baeldung.com/java-immutable-object
- https://www.linkedin.com/pulse/20140528113353-16837833-6-benefits-of-programming-with-immutable-objects-in-java/
- https://www.yegor256.com/2014/06/09/objects-should-be-immutable.html
- https://mangkyu.tistory.com/131