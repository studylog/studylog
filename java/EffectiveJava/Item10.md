# 아이템10. equals는 일반 규약을 지켜 재정의하라
## 1. equals를 재정의하면 안되는 경우
- equals는 재정의하기 쉬워보이지만 곳곳에 함정이 있다. 문제를 회피하는 가장 쉬운 길은 **아예 재정의하지 않는 것**

### a. 각 인스턴스가 본질적으로 고유할 때
- 값 표현 객체가 없을 때(값이 아닌 동작을 표현하는 클래스, Thread가 좋은 예)
- Bean에 등록해두는 객체 repository, controller, service 등이 이에 해당
- DTO, Domain 객체는 값 검증이 필요할 수 있으니 equals를 재정의해야할 수도 있다. 

### b. 인스턴스의 논리적 동치성(logical equality)을 검사할 일이 없을 때
- 논리적 동치성 검사의 예시: java.utils.regex.Pattern의 equals는 내부의 정규표현식이 같은지를 검사하는 메서드

### c. 상위 클래스에서 재정의한 equals가 하위 클래스에도 들어맞을 때
- 같은 특징을 가지기 때문에 equals를 상속받아 사용하는 걸 권장
- Set, Map, List의 경우 Abstract(Type)의 equals를 쓴다.

### d. 클래스가 private거나 package-private여서 equals 메서드를 호출할 일이 없을 때
- equals가 실수로 호출되는 것을 막고 싶다면
```java
@Override
public boolean equals(Object o) {
    throws new AssertionError(); // equals 호출시 error();
}
```

### e. 싱글턴을 보장하는 클래스(인스턴스 통제 클래스, Enum(열거타입)) 인 경우
- 객체간 동등성, 동일성이 보장된다.

## 2. equals를 재정의해야하는 경우
- 객체 식별성(object identity): X - 두 객체가 물리적으로 같은가
- 논리적 동치성(logical equality): O

- 객체 식별성이 아니라 논리적 동치성을 확인해야 하는데, 상위 클래스의 equals가 논리적 동치성을 비교하도록 재정의하지 않았을 때 (주로 **값 클래스**)
```java
public class Fruit {
    private String name; // name이 같을 경우 두 객체는 같다(논리적 동치성)
}
```
### 2-1. 값 클래스의 equals를 재정의할 때 기대 효과
- 값을 비교
- Map, Set의 원소로 사용가능

### 2-2. 값 클래스여도 equals를 재정의할 필요가 없을 때
- **인스턴스 통제 클래스**: 값이 같은 인스턴스가 2개 이상 만들어지지 않음 (예. Static Factory Method Pattern, Enum)

## 3. Equals 메서드의 규약 - 동치관계
- **동치 클래스(equivalence class)**: 집합을 서로 같은 원소들로 이루어진 부분집합으로 나누는 연산
- equals 메서드가 쓸모있으려면 모든 원소가 같은 동치류에 속한 어떤 원소와도 서로 교환이 가능해야 한다.

### 3-1. 반사성(reflexivity)
- null이 아닌 모든 참조 값 x에 대해 x.equals(x)는 true이다.
- 단순히 말하면 객체는 자기 자신과 비교했을 때 같아야한다는 뜻
- 만약 x.equals(x)가 성립하지 않는 객체라면, 컬렉션에서 contain 메서드를 사용하는 경우 방금 넣은 객체도 찾을 수 없을 것이다.

### 3-2. 대칭성(symmetry)
- null이 아닌 모든 참조 값 x, y에 대해 x.equals(y)가 true이면, y.equals(x)가 true를 만족해야 한다.
```java
public final class CaseInsensitiveString {
  private final String s;

  public CaseInsensitiveString(String s) {
    this.s = Objects.requireNonNull(s);
  }

  @Override
  public boolean equals(Object o) {
    if(o instanceof CaseInsensitiveString) {
      return s.equalsIgnoreCase(((CaseInsensitiveString) o).s);
    }

    if(o instanceof String) { //한 방향으로만 작동!!
      return s.equalsIgnoreCase((String) o);
    }
    return false;
  }
}
```
```java
CaseInsensitiveString caseInsensitiveString = new CaseInsensitiveString("Test");
String test = "test";
System.out.println(caseInsensitiveString.equals(test)); //true
System.out.println(test.equals(caseInsensitiveString)); //false
```
- String 클래스에서는 CaseInsensitiveString의 존재를 모르기 때문에 false가 날 수 밖에 없는 상황
```java
List<CaseInsensitiveString> list = new ArrayList<>();
list.add(new CaseInsensitiveString("Test"));
System.out.println(list.contain("test")); //false or true
```
- 위의 내용을 수정한다면, String과의 비교는 포기해야 한다.
같은 CaseInsensitiveString 타입인 경우에만 비교하도록 한다.
```java
@Override
public boolean equals(Object o) {
    return o instanceof CaseInsensitiveString 
        && ((CaseInsensitiveString) o).s.equalsIgnoreCase(s);
}
```

### 3-3. 추이성(transitivity) 
- null이 아닌 모든 참조 값 x, y, z에 대해 x.equals(y)가 true이고, y.equals(z)가 true이면 x.equals(z)도 true가 되어야 한다는 조건이다.
- 상위 클래스에서 equals를 재정의했을 경우 equals를 재정의하면 안된다.
#### 문제1. 대칭성 위배 문제에 빠질 수 있다.
```java
// ColorPoint.java 의 equals
@Override public boolean equals(Object o){
    if(!(o instanceof ColorPoint))
        return false;
    return super.equals(o) && ((ColorPoint) o).color == color;
}

public static void main(){
    Point p = new Point(1,2);
    ColorPoint cp = new ColorPoint(1,2, Color.RED);
    p.equals(cp);	// true (Point의 equals로 계산)
    cp.equals(p);	// false (ColorPoint의 equals로 계산: color 필드 부분에서 false)
}
```
#### 문제2. 추이성 위배 문제에 빠질 수 있다.
```java
//ColorPoint.java의 equals
@Override public boolean equals(Obejct o){
    if(!(o instanceof Point))
        return false;
    if(!(o instanceof ColorPoint))
        return o.equals(this);
    return super.equals(o) && ((ColorPoint) o).color == color;
}

public static void main(){
    ColorPoint p1 = new ColorPoint(1,2, Color.RED);
    Point p2 = new Point(1,2);
    ColorPoint p3 = new ColorPoint(1,2, Color.BLUE);
    p1.equals(p2);	// true (ColorPoint의 equals 비교 //2번째 if문에서 Point의 equals로 변환)
    p2.equals(p3);	// true (Point의 equals 비교 // x,y 같으니 true)
    p1.equals(p3);	// false (ColorPoint의 equals 비교)
}
```

#### 문제3. 무한 재귀에 빠질 수 있다.
```java
//SmellPoint.java의 equals
@Override public boolean equals(Obejct o){
    if(!(o instanceof Point))
        return false;
    if(!(o instanceof SmellPoint))
        return o.equals(this);

    return super.equals(o) && ((SmellPoint) o).color == color;
}

public static void main(){
    ColorPoint p1 = new ColorPoint(1,2, Color.RED);
    SmellPoint p2 = new SmellPoint(1,2); 
    p1.equals(p2);
    // 처음에 ColorPoint의 equals로 비교 : 2번째 if문 때문에 SmellPoint의 equals로 비교
    // 이후 SmellPoint의 equals로 비교 : 2번째 if문 때문에 ColorPoint의 equals로 비교
    // 무한 재귀의 상태!
}
```
- 구체 클래스를 확장해 새로운 값을 추가하면서 equals 규약을 만족시킬 방법은 존재하지 않는다.
- 그렇다고 instanceof 검사 대신 getClass 검사를 하라는 것이 아니다.
```java
@Override
public boolean equals(Object o) {
    if (o == null || o.getClass() != getClass())
        return false;
    Point p = (Point) o;
    return x == p.x && y == p.y;
}
```
- **리스코프 치환원칙을 위배한다**: Point의 하위 클래스는 정의상 여전히 Point이기 때문에 어디서든 Point로 활용되어야 한다.
- **리스코프 치환원칙(Liskov Substitution Principle)**: 어떤 타입에 있어 중요한 속성이라면 그 하위 타입에서도 마찬가지로 중요하다. 따라서 그 타입의 모든 메서드가 하위 타입에서도 똑같이 잘 작동해야 한다.

#### 우회 방법1. 상속 대신 컴포지션을 활용하라
```java
public class ColorPoint {
    private final Point point;
    private final Color color;

    public Point asPoint() {
        return point;
    }

    @Override
    public boolean equals(Object o) {
        if (!(o instanceof ColorPoint))
            return false;
        ColorPoint cp = (ColorPoint) o;
        return cp.point.equals(point) && cp.color.equals(color);
    }
}
```

#### 우회 방법2. 추상클래스의 하위클래스 사용하기
- 추상 클래스의 하위 클래스에서는 equals 규약을 지키면서도 값을 추가할 수 있다.
- 상위 클래스를 직접 인스턴스로 만드는 게 불가능하기 때문에 하위 클래스끼리 비교가 가능해진다.

### 3-4. 일관성(consistency)
- null이 아닌 모든 참조 값 x, y에 대해 x.equals(y)를 반복해서 호출하면 항상 같은 값을 반환한다.
- 두 객체가 같다면(어느 하나 혹은 두 객체 모두가 수정되지 않는 한) 앞으로도 영원히 같아야 한다.
- 가변객체 = 비교 시점에 따라 서로 다를 수 있다.
- 불변객체 = 한 번 다르면 끝까지 달라야 한다.
- equals는 항시 메모리에 존재하는 객체만을 사용한 결정적 계산만 수행해야 한다.
- 클래스가 불변이든 가변이든 equals의 판단에 신뢰할 수 없는 자원이 끼어들면 안된다. 예) URL과 매핑된 호스트의 IP 주소

### 3-5. null 아님
- null이 아닌 모든 참조값 x에 대해, x.equals(null)은 false이다.
- 모든 객체가 null과 같지 않아야 한다.
1. 명시적 null 검사
```java
@Override
public boolean equals(Object o) {
    if (o == null) return false;
    ...
}
```
2. 묵시적 null 검사
```java
@Override
public boolean equals(Object o) {
    if (!(o instanceof MyType)) return false;

    MyType mt = (MyType) o;
    ...
}
```

## 4. equals 메서드를 구현하는 4단계
### 4-1. == 연산자를 사용해 입력이 자기 자신의 참조인지 확인
- object identity를 검사한다.

### 4-2. instanceof 연산자로 입력이 올바른 타입인지 확인
- 올바른 타입인 경우: equals가 정의된 클래스로 리턴이 되는가
- 올바른 타입이 아닌 경우: 구현한 서로 다른 클래스간 비교가 가능하게 해야 함.

### 4-3. 입력을 올바른 타입으로 형변환
- 4-2번에서 instanceof 연산자를 사용하였기 때문에 형변환이 가능함.

### 4-4. 입력 객체와 자기 자신의 대응되는 '핵심'필드들이 모두 일치하는지 하나씩 검사

## 5. equals 구현할 때 주의사항
### 5-1. 기본 타입과 참조 타입
- 기본 타입: == 연산자 비교
- 참조 타입: equals() 메소드 비교
- 배열 필드: 원소 각각을 지침대로 비교한다. 모두가 핵심 필드라면 Arrays.equals()를 사용한다.
- float, double 필드: Float.compare(), Double.compare() 비교 (부동 소수 값) / Float.equals()나 Double.equals()은 오토 박싱을 수반할 수 있어 성능상 좋지 않다.

### 5-2. null 정상값 취급 방지
- Object.equals()로 비교하여 NullPointException 발생을 예방하자

### 5-3. 필드의 표준형을 저장하자.
- 비교하기 복잡한 필드는 필드의 표준형을 저장한 후 비교: 불변 클래스에 제격

### 5-4. 필드 비교 순서는 equals 성능을 좌우한다.
- 다를 가능성이 높은 필드 우선
- 비교 비용이 싼 필드 우선
- 핵심 필드 / 파생 필드 구분

### 5-5. equals 재정의할 땐 hashCode도 반드시 재정의하자.

### 5-6. 너무 복잡하게 해결하여 하지 말자.

### 5-7. Object 외의 타입을 매개변수로 받는 equals 메서드는 선언하지 말자.
```java
public boolean equals(MyType o) // @Override가 되지 않는다 !
```

### 5-8. AutoValue 프레임워크
