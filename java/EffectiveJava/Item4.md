# 아이템4. 인스턴스화를 막으려거든 private 생성자를 사용하라
# Private Constructor

## 1. 정적 메서드와 정적 필드만을 담은 클래스의 쓰임새
1. 기본 타입 값이나 배열 관련 메서드를 모아 둘 때 (java.lang.Math, java.util.Arrays)
2. 특정 인터페이스를 구현한 객체를 생성해주는 정적 메서드(팩터리)를 모아둘 때 (java.util.Collections)
3. final 클래스와 관련한 메서드: final 클래스를 상속해서 하위 클래스에 메서드를 넣는 것은 불가능하므로, final 클래스와 관련 메서드들을 모아놓을 때도 사용
```java
public class Arrays {

    /**
     * The minimum array length below which a parallel sorting
     * algorithm will not further partition the sorting task. Using
     * smaller sizes typically results in memory contention across
     * tasks that makes parallel speedups unlikely.
     */
    private static final int MIN_ARRAY_SORT_GRAN = 1 << 13;

    // Suppresses default constructor, ensuring non-instantiability.
    private Arrays() {}
```

## 2. 인스턴스화를 막는 방법
정적 멤버만 담을 때는 인스턴스로 쓰려고 설계한 것이 아님    
- 문제) 생성자를 명시하지 않을 때 자동으로 기본 생성자가 만들어짐.    
- 해결) private 생성자를 추가해서 클래스의 인스턴스화를 막는다.
- 상속이 불가능 (하위가 상위 생성자 접근을 할 수 없다.)
- 직관적이지 않을 수 있으니 적절한 주석을 달 것

```java
public class ImageUtility {
    
    // 기본 생성자가 만들어지는 것을 방어(인스턴스화 방지용)
    private ImageUtility() {
        throw new AssertionError();
    }
}
```