# 아이템11. equals를 재정의하려거든 hashCode도 재정의하라.
- **equals를 재정의한 클래스 모두에서 hasCode도 재정의해야한다.**
일반 규약을 어기게 되어 HashMap이나 HashSet같은 컬렉션의 원소로 사용할 때 문제가 발생할 수 있다.

## 1. Object 명세의 3가지 규약
1. equals 비교에 사용되는 정보가 변경되지 않았다면 객체의 hashCode를 몇번을 호출해도 항상 같은 값을 반환해야 한다.
2. equals(object)를 통해 두 객체를 비교했을 때 객체가 같다고 판단했다면(true를 반환) 두 객체의 hashCode는 같다.
3. equals(object)가 두 객체를 다르게 판단했다 하더라도 (false를 반환) hashCode가 다를 필요는 없다. (Hash Collision) 단, 다른 객체에 대해서는 다른 값을 반환해야 해시 테이블의 성능이 좋아진다.

**논리적으로 같은 객체는 같은 hasCode를 반환해야 한다.**

## 2. hashcode의 동작방법
- 좋은 해쉬함수라면 서로 다른 인스턴스에 다른 해쉬코드를 반환한다.
- 주어진 인스턴스들을 균일하게 분배해야 한다. (32비트 정수 범위 내에서)
- 만약 같은 값을 반환한다면, 객체가 해시 테이블 버킷하나에 담기고, 그 객체들이 연결리스트처럼 동작한다.

## 3. 좋은 hashCode를 작성하는 방법
```java
@Override
public int hashCode() {
    int c = 31;
    // 1. int 변수 result를 선언한 후 첫번째 핵심 필드에 대한 hashCode로 초기화한다.
    int result = Integer.hashCode(firstNumber);

    // 2. 기본타입 필드라면 Type.hashCode()를 실핸한다.
    // Type은 기본타입의 Boxing 클래스이다.
    result = c * result + Integer.hashCode(secondNumber);

    // 3. 참조타입이라면 참조타입에 대한 hashCode함수를 호출한다.
    // 4. 값이 null이면 0을 더해준다.
    result = c * result + address == null ? 0 : address.hashCode();

    // 5. 필드가 배열이라면 핵심 원소를 각각 필드처럼 다룬다.
    for(String elem : arr) {
        result += c * result + elem == null ? 0 : elem.hashCode();
    }

    // 6. 배열의 모든 원소가 핵심필드면 Arrays.hashCode를 이용한다.
    result = c * result + Arrays.hashCode(arr);

    // 7. result = 31 * result + c 형태로 초기화하여 result 를 리턴한다.
    return result;
}
```
- hashCode를 구현했다면 이 메서드가 동치인 인스턴스에 대해 똑같은 해시코드를 반환할지 Testcase를 작성하자.
- 파생필드는 hashCode 계산에서 제외해도 된다.
- equals 비교에 사용되지 않은 필드는 반드시 제외한다.
- 31 * result 를 곱하는 순서에 따라 result 값이 달라진다.
- 곱하는 숫자 31인 이뉴는 31이 홀수이면서 소수(prime)이기 때문이다.
- 31을 이용하면 (i << 5) - i 와 같이 최적화할 수 있다.

* hashCode를 편하게 만들어주는 모듈
- Objects.hash() 
    - 내부적으로 auto boxing이 일어나 성능이 떨어진다.
    - 입력 인수를 담기 위한 배열 생성으로 속도가 더 느리다.
- Lombok 의 @EqualsAndHashCode
- Google 의 @AutoValue

## 4. hashCode를 재정의할 때 주의할 점 !
- 불변 객체에 대해서는 hashCode 생성비용이 많이 든다면, hashCode를 캐싱하는 것도 고려하자.
    - 스레드 안전성까지 고려해야 한다.
- 성능을 높이고자 hashCode를 계산할 때 핵심 필드를 생략해서는 안된다.
    - 속도는 빨라지겠지만, hash 품질이 나빠져서 해시테이블의 성능을 떨어뜨릴 수 있다. (Hashing Collision)
- hashCode 생성 규칙을 API 사용자에게 공표하지 말자.
    - 그래야 클라이언트가 hashCode 값에 의지한 코드를 짜지 않는다.
    - 다음 릴리즈 시, 생성을 개선할 여지가 있다.