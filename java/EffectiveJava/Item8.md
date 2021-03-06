# 아이템8. finalizer와 cleaner 사용을 피해라
## 1. 자바의 객체 소멸자
- finalizer: 예측할 수 없고, 상황에 따라 위험할 수 있어 일반적으로 불필요하다. 오동작, 낮은 성능, 이식성 문제의 원인이다. **"쓰지 말자"** 자바 9에서는 사용 자제(deprecated) API로 지정
- cleaner: finalizer보단 덜 위험하지만, 여전히 예측할 수 없고, 느리고, 일반적으로 불필요하다.

* C++의 파괴자(destructor): 특정 객체와 관련된 자원을 회수하는 보편적인 방법 (Java의 try-with-resources, try-finally)
* Java의 가비지 컬렉터: 접근할 수 없게된 객체를 회수

## 2. finalizer와 cleaner 사용을 피해야 하는 이유
### 1) 즉시 수행된다는 보장이 없다.
객체에 접근할 수 없게 된 이후부터 실행되기까지 얼마나 걸릴지 알 수 없다. 제때 실행되어야 하는 작업은 절대 할 수 없다.   
수행 시점이 전적으로 GC 알고리즘에 달렸으며, 구현 방식에 따라 천차만별이다.     
finalizer 쓰레드는 다른 애플리케이션보다 우선순위가 낮다. 

### 2) 수행 여부도 보장하지 않는다.
접근할 수 없는 객체에 딸린 종료작업을 수행하지 못한 채 프로그램이 중단될 수도 있다. 
상태를 영구적으로 수정하는 작업에서 절대 finalizer나 cleaner에 의존해서는 안된다.   
System.gc, System.runFinalization 메서드에 현혹되지 말자. 실행 가능성은 높여주나 보장해주진 않는다.     

### 3) finalizer 동작 중 발생한 예외가 무시되며, 처리할 작업이 남아있어도 그 순간 종료된다.
잡지 못한 예외로 객체가 덜 마무리된 상태로 남아있을 수 있다. 훼손된 객체를 사용하려 할 때 예측할 수 없다.   

### 4) finalizer와 cleaner은 심각한 성능문제도 동반한다.
AutoCloseable 객체를 생성해 GC 수거시간: 12ns   
finalizer 수거시간: 550ns -> GC의 효율을 떨어뜨린다.    

### 5) finalizer 공격에 노출되어 심각한 보안 문제를 일으킬 수도 있다.
생성자나 직렬화 과정에서 예외가 발생하면 생성되다 만 객체에서 악의적인 하위 클래스의 finalizer가 수행될 수 있다.    
객체 생성을 막으려면 생성자에서 예외를 던지면 되는데, finalizer가 있으면 그렇지도 않다.     
final이 아닌 클래스를 finalizer 공격으로부터 방어하려면 아무 일도 하지 않는 finalize 메서드를 만들고 final로 선언하자.

## 3. finalizer, cleaner의 대안
AutoClosable을 구현하고, 클라이언트에서 인스턴스를 다 쓰면 close 메서드를 호출한다. 예외가 발생해도 잘 종료되도록 try-with-resources를 사용한다.

## 4. finalizer, cleaner의 쓰임새
### 1) 자원의 소유자가 close메서드를 호출하지 않는 것에 대비한 안전망 역할
안전망 역할의 finalizer를 작성할 때는 그럴만한 값어치가 있는지 심사숙고하자.    
자바 라이브러리의 일부 클래스는 안전망 역할의 finalizer를 제공한다: FileInputStream, FileOutputStream, ThreadPoolExecutor   

### 2) 네이티브 피어와 연결
네이티브 피어(native peer): 일반 자바 객체가 네이티브 메서드를 통해 기능을 위임한 네이티브 객체. 자바 객체가 아니라 가비지 컬렉터는 그 존재를 알지 못한다.      
성능 저하를 감당할 수 있고 네이티브 피어가 심각한 자원을 가지고 있지 않을 때에만 해당된다. 성능 저하를 감당할 수 없거나 네이티브 피어가 사용하는 자원을 즉시 회수해야 한다면 close 메서드를 사용하자.   