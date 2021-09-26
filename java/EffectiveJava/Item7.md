# 아이템7. 다 쓴 객체 참조를 해제하라
Java의 장점 중 하나는 가비지 컬렉션을 지원하는 언어라는 점  
하지만, 가비지 컬렉션을 통해 소멸 대상이 되는 객체가 되기 위해서는 어떠한 reference 변수에서 가르키지 않아야 한다. 다 쓴 객체에 대한 참조를 해제하지 않으면 가비지 컬렉션의 대상이 되지 않아 계속 메모리가 할당되는 메모리 누수 현상이 발생된다.    
가비지 컬렉션을 지원하는 언어에서는 메모리 누수를 찾기가 까다롭다. 객체 참조(reference)를 하나 살려두면, 가비지 컬렉터는 그 객체뿐만 아니라 그 객체 내에서 참조하고 있는 객체까지 회수할 수 없다.   

## 1. 직접 할당 해제
- 객체 참조 변수를 null로 초기화한다.
- 실제 Heap 메모리에 존재하는 객체는 어떤 참조(reference)도 가지지 않기 때문에 가비지 컬렉션의 소멸 대상이 된다.
- 클래스 내에서 메모리를 관리하는 객체라면 이 방법을 통해 다 쓴 객체는 할당을 해제하는 것이 옳다.

## 2. Scope을 통한 자동 할당 해제
- 보통은 변수 선언과 동시에 초기화를 사용한다. 그 변수에 대한 scope가 종료되는 순간 reference가 해제되어 가비지 컬렉션의 대상이 된다.
- try-catch 와 같은 구문에서는 finally 구문에서 변수에 대한 참조를 해제한다.

## 메모리 누수의 주범
1. 자기 메모리를 직접 관리하는 클래스
- 클래스 내에서 인스턴스에 대한 참조(reference)를 관리하는 객체
- 프로그래머가 항시 메모리 누수에 주의한다.

2. 캐시 (Map)
- 캐시에 객체 참조를 넣고, 객체를 다 쓴 후에도 놔두는 경우
- 해결책)
    - WeakHashMap을 이용해 캐시를 만든다. 외부에서 키를 참조하는 동안만 엔트리가 살아있는 캐시
    - 시간이 지날수록 엔트리 가치를 떨어뜨리는 방식 -> ScheduledThreadPoolExecutor(쓰지 않는 엔트리 청소)

3. 리스너(listener), 콜백(callback)
- 클라이언트가 콜백만 등록하고 해지하지 않는 경우, 계속 쌓인다.
- 해결책)
    - 콜백을 약한 참조(weak reference)로 저장한다. WeakHashMap에 키로 저장한다.

## Java Reference
### GC의 reachability
- reachable: 어떤 객체에 유효한 참조가 있다. (root set: 유효한 최초의 참조)
- unreachable: 어떤 객체에 유효한 참조가 없다. (GC 대상)

### java.lang.ref 패키지의 객체 참조 종류 4가지
1. Strong Reference
- 우리가 흔히 사용하는 참조
- String str = new String("hello"); 와 같은 형태
- Strong Reference는 GC의 대상이 아니다.
- Strong Reference 관계의 객체가 GC의 대상이 되기 위해서 null로 초기화해 객체에 대한 reachability 상태를 unreachable 상태로 만들어줘야 한다.

2. Soft Reference
- 객체의 reachability가 strongly reachable 객체가 아닌 객체 중 Soft Reference만 있는 상태
- SoftReference<Class> ref = new SoftReference<>(new String("hello")); 와 같은 형태
- Soft Reference는 대게 GC대상이 아니다가 out of memory 에러가 나기 직전에 Soft Reference 관계에 있는 객체들은 GC 대상이 된다.

3. Weak Reference
- 객체의 reachability가 strongly reachable 객체가 아닌 객체 중 Soft Reference가 없고 Weak Reference만 있는 상태
- WeakReference<Class> ref = new WeakReference<Class>(new String("hello")); 와 같은 형태
- WeakReference는 GC 동작마다 회수된다.
- WeakReference 객체 내의 weakly reachable 객체에 대한 참조가 null로 설정되면, GC에 의해 메모리 회수
```java
WeakReference<Sample> wr = new WeakReference<Sample>(new Sample());
Sample ex = wr.get(); // 참조
ex = null; // weakly reachable 객체 = null -> 메모리 회수
```

4. Phantomly Reference
- 객체의 reachability가 strongly reachable 객체가 아닌 객체 중 Soft Reference와 Weak Referencerk 모두 해당되지 않는 객체
- finalize 되었지만 메모리가 아직 회수 되지 않은 객체