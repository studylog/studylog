# 아이템5. 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라.
# Dependency Injection

많은 클래스가 하나 이상의 자원에 의존한다. 이때 사용하는 자원에 따라 동작이 달라지는 클래스(하나 이상의 자원에 의존)에는 정적 유틸리티 클래스나 싱글턴 방식이 적합하지 않다.    
**인스턴스를 생성할 때 생성자에 필요한 자원을 넘겨주는 방식을 사용하면, 클래스가 여러 자원 인스턴스를 지원하고, 클라이언트가 원하는 자원을 사용할 수 있다.**    
=> **의존 객체 주입**의 한 형태로 유연성과 재사용성, 테스트 용이성을 높였다.    

### Dependency Injection
- 의존적인 객체를 직접 생성하거나 제어하는 것이 아니라, 특정 객체에 필요한 객체를 외부에서 결정해서 연결시키는 것을 의미

```java
public class SpellChecker {

    private final Lexicon dictionary;

    // dependency injection
    public SpellChecker(Lexicon dictionary) {
        this.dictionary = Objects.requireNonNull(dictionary); // null이면 NPE 아닌경우 objects 반환
    }

    public boolean isValid(String word) { ... }
    public List<String> suggestions(String typo) { ... }
}

interface Lexicon { ... }
public class MyDictionary implements Lexicon { ... }
```

```java
Lexicon dic = new MyDictionary();
SpellChecker checker = new SpellChecker(dic);

checker.isValid(word);
```

- **불변을 보장**하여 여러 클라이언트가 의존 객체를 사용할 수 있다.
- 의존 객체 주입은 생성자, 정적 팩터리, Builder 모두 똑같이 적용할 수 있다.

## 생성자에 자원을 넘겨주는 방식
### Factory
- 호출할 때마다 특정 타입의 인스턴스를 반복해서 만들어주는 객체를 말하는데, Factory Method Pattern은 의존 객체 주입 패턴을 응용해서 구현한 것이다.

### Supplier<T> 인터페이스
```java
@FunctionalInterface
public interface Supplier<T> {

    /**
     * Gets a result.
     *
     * @return a result
     */
    T get();
}
```
- 팩터리를 표현한 완벽한 예시, 이 방식을 사용하여 클라이언트는 자신이 명시한 타입의 하위 타입이라면 무엇이든 생성할 수 있는 팩터리를 만들 수 있다.
```java
Mosaic create(Supplier<? extends Tile> tileFactory) { ... }
```

- 한정적 와일드 카드 타입(bounded wildcard type)으로 팩터리 타입 매개변수를 제한한다. (ITypeFactory.class)
```java
public class IType {
    private static final int TYPE_Z = 0;
    private static final int TYPE_A = 1;
    private static final int TYPE_B = 2;

    final static Map<Integer, Supplier<? extends ITypeFactory>> map = new HashMap<>();
    static {
	    map.put(TYPE_Z, ITypeFactory::new);
        map.put(TYPE_A, A::new);
        map.put(TYPE_B, B::new);
    }
}

class ITypeFactory {}
class A extends ITypeFactory {}
class B extends ITypeFactory {}
```

### 의존객체 주입 프레임워크
- 대거(Dagger), 주스(Guice), 스프링(Spring)
- 의존 객체 주입이 유연성과 테스트 용이성을 개선해주지만, 의존성이 너무 많은 프로젝트에서는 코드를 어지럽게 하며, 의존 객체 주입 프레임워크를 사용해 코드의 어지러움을 해소할 수 있다.