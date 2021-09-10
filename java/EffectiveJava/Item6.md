# 아이템6. 불필요한 객체 생성을 피하라

## 1. 객체 재사용
똑같은 기능의 객체를 매번 생성하는 것보다 객체 하나를 생성해서 재사용하는 편이 좋다. 불변 객체는 언제든지 재사용할 수 있다.     
```java
// bad example - 호출될 때마다 새로운 인스턴스 생성
String s = new String("bad example"); // Heap 영역에 존재

// good example - 하나의 인스턴스를 사용
String s = "good example"; // String constant pool 영역에 존재 (Perm > Heap)
```
- 같은 JVM에서 똑같은 문자열 리터럴을 사용하는 경우 모든 코드가 같은 객체를 재사용하는 것이 보장된다.      
- String constant pool 영역에 있는지 검색 후, String 객체를 재사용한다. (== 연산자로 비교 가능 ! )

## 2. 정적 팩터리 메서드를 제공하는 불변 클래스
```java
public Boolean(String s) { // 생성자 - Java9에서 deprecated
    this(parseBoolean(s));
}

public static Boolean valueOf(String s) { // 정적 팩터리 메서드
    return parseBoolean(s) ? TRUE : FALSE;
}
```
```java
boolean b = new Boolean("false");
boolean b = Boolean.valueOf("true"); 
```
- 생성자는 매번 새로운 객체를 생성하지만, 팩터리 메서드는 그렇지 않으므로 Boolean(String s) 생성자 대신 Boolean.valueOf(String s) 팩터리 메서드를 사용하는 것이 좋다.

## 3. 생성 비용이 비싼 객체라면 캐싱하여 재사용
```java
// 정규 표현식을 활용해 유효한 로마 숫자인지 확인하는 메서드
static boolean isRomanNumeral(String s) {
    return s.matches("^(?=.)M*(C[MD]|D?C{0,3})" + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
}
```
- 문제) String.matches() 메서드를 사용
```java
public boolean matches(String regex) {
    return Pattern.matches(regex, this);
}
```
- String.matches 메서드 내부에서 만드는 정규표현식용 Pattern 인스턴스는 한 번 쓰고 버려져 곧 바로 가비지 컬렉션의 대상이 된다.
- Pattern은 입력받은 정규 표현식에 해당하는 유한 상태 머신(finite state machine)을 만들어 인스턴스 생성 비용이 높다.
- 해결) 생성 비용이 많이 드는 객체가 반복해서 필요하다면, 캐싱하여 재사용하는 것을 권장한다.
```java
public class RomanNumerals {
    private static final Pattern ROMAN = Pattern.compile("^(?=.)M*(C[MD]|D?C{0,3})" + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");

    static boolean isRomanNumeralFast(String s) {
        return ROMAN.matcher(s).matches();
    }
}
```
- 클래스가 초기화된 후 이 메서드를 한 번도 호출하지 않는다면, ROMAN 필드는 필요없이 초기화된 것이다. lazy initialization으로, 지연 초기화는 코드를 복잡하게 만드는데, 성능은 크게 개선되지 않을 때가 많으므로 권하지 않는다.

## 4. 어댑터 패턴 사용
- Map 인터페이스의 keySet 메서드는 Map 객체안의 키를 전부 담은 Set 뷰를 반환한다.
- 뷰 객체를 여러개 만드는 것이 아니라 매번 같은 인스턴스를 반환한다. 
```java
@DisplayName("keyset은 같은 Map을 바라본다.")
@Test
void keyset() {
    Map<String, Object> map = new HashMap<>();
    map.put("Effective Java", "Hello");
    
    Set<String> Set1 = map.keySet();
    Set<String> Set2 = map.keySet();

  assertThat(Set1).isSameAs(Set2);
}
```

## 5. 오토박싱(autoboxing)
- auto boxing은 기본 타입과 박싱된 기본 타입을 섞어 쓸 때 자동으로 상호 변환해주는 기술이다. 
- 기본타입과 그에 대응하는 박싱된 기본타입의 구분을 흐려주지만 완전히 없애주진 않는다. 성능에서 좋아진다 볼 수 없다.
```java
private static long sum(){
    Long sum = 0L;
    
    for (long i = 0; i< Integer.MAX_VALUE; i++) {
        sum += i;
    }
    return sum;
}
```
- sum 변수를 long이 아닌 Long으로 선언하여 불필요한 인스턴스가 sum += i 연산이 이루어질 때마다 생성되는 것이다. 단순히 sum 타입을 long으로 변경해주어도 성능이 개선된다.
- 박싱된 기본타입보다 기본 타입을 사용하고, 의도치 않은 오토박싱이 숨어들지 않도록 주의해야 한다.

## 6. 본인만의 객체 풀(pool)을 만들지 말자
- 가벼운 객체를 다룰때는 직접 만든 객체 풀보다 훨씬 빠르다. (JVM GC) 데이터베이스 연결하는데 생성비용이 워낙 비싸니 재사용이 낫다.
- 방어적 복사가 필요한 상황에서 객체를 재사용했을 때 피해가 필요없는 객체를 반복 생성했을 때 피해보다 훨씬 크다.