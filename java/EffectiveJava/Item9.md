# 아이템9. try-finally 보다는 try-with-resources를 사용하라
## 1. 자원이 닫힘을 보장하는 수단, try-finally의 단점
- 코드 가독성에 있어 지저분하다.
- 두번째 예외가 첫번째 예외를 집어삼켜버러 실제 시스템에서 디버깅을 어렵게한다.
```java
static void copy(String src, String dst) throws IOException {
    InputStream in = new FileInputStream(src);

    try {
        OutStream out = new FileOutStream(dst);
        try {
            byte[] buf = new byte[BUFFER_SIZE];
            int n;
            while((n = in.read(buf)) >= 0) 
                out.write(buf, 0, n);
        }finally {
            out.close();
        }
    }finally {
        in.close();
    }
}
```

## 2. 자원 회수의 최선책 try-with-resources
- AutoCloseable 인터페이스 구현: close() 메서드 하나만 정의
```java
public interface AutoCloseable {
    void close() throws Exception;
}
```
```java
static void copy(String src, String dst) throws IOException {
    try (InputStream in = new FileInputStream(src);
        OutStream out = new FileOutStream(dst)) {
            byte[] buf = new byte[BUFFER_SIZE];
            int n;
            while((n = in.read(buf)) >= 0) 
                out.write(buf, 0, n);
    } catch (IOException e) {
        return defaultValue;
    }
}
```
- 읽기 쉽고 문제 진단에 유리하다.
- catch를 이용해 try문을 중첩하지 않고도 다수의 예외 처리가 가능하다.
- 숨겨진 예외도 버려지지 않고, suppressed 꼬리표를 달고 출력된다.

readLine()과 close() 호출 양쪽에서 예외가 발생하면, close() 예외는 숨겨지고 readLine()에서 발생한 예외가 기록된다.  
