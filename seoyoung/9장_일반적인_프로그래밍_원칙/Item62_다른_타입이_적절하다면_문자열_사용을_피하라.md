## Item62. 다른 타입이 적절하다면 문자열 사용을 피하라

### 문자열은 다른 값 타입을 대신하기에 적합하지 않다.
- 받은 데이터가 수치형이라면 int, float, BigInteger 등 적절한 수치 타입으로 변환해야 한다.
- 예/아니오 질문이라면 적절한 열거 타입이나 boolean으로 변환해야 한다.

### 문자열은 열거 타입을 대신하기에 적합하지 않다.
### 문자열은 혼합 타입을 대신하기에 적합하지 않다.

### 62-1 혼합 타입을 문자열로 처리한 부적절한 예
```java
String compoundKey = className + "#" + i.next();
```
- 단점이 많은 방식
- 두 요소를 구분해주는 문자 #이 두 요소중 하나에서 쓰였다면 혼란스러운 결과를 초래한다.
- 각 요소를 개별로 접근하려면 문자열을 파싱해야 해서 느리고, 오류 가능성도 커진다.
- 적절한 equals, toString, compareTo 메서드를 제공하지 못한다.

### 문자열은 권한을 표현하기에 적합하지 않다.
### 62-2 잘못된 예 - 문자열을 사용해 권한을 구분
```java
public class ThreadLocal {
    private ThreadLocal() { } // 객체 생성 불가

    // 현 스레드의 값을 키로 구분해 저장
    public static void set(String key, Object value);

    // (키가 가리키는) 현 스레드의 값을 반환
    public static Object get(String key);
}
```

### 62-3 Key 클래스로 권한을 구분
```java
public class ThreadLocal {
    private ThreadLocal() { } // 객체 생성 불가

    public static class Key { // (권한)
        Key() { }
    }

    // 위조 불가능한 고유 키를 생성한다.
    public static Key getKey() {
        return new Key();
    }

    public static void set(Key key, Object value);
    public static Object get(Key key);
}
```

--> public void set 함수에서 {} 구현부분이 존재하지 않는 이유 -> 설계를 나타낸다..?
예시가 이상해....
### 62-4 리팩터링하여 Key를 ThreadLocal로 변경
```java
public final class ThreadLocal {
    public ThreadLocal();
    public void set(Object value);
    public Object get();
}
```

```
핵심정리
문자열을 쓰고 싶은 유혹을 뿌리쳐라.
문자열은 잘못 사용하면 번거롭고, 덜 유연하고, 느리고, 오류 가능성도 크다.
문자열을 잘못 사용하는 흔한 예로는 기본 타입, 열거 타입, 혼합 타입이 있다.
```
### 문자열 권한의 문제...
1. 보안 취약성: 문자열은 변경 가능하고, 예측 가능한 값이기 때문에 보안에 취약할 수 있습니다. 문자열 기반의 권한 체계는 외부로부터의 위조나 변조가 더 쉬워질 수 있으며, 이는 보안 위험을 증가시킵니다. 
2. 실수의 가능성: 문자열을 사용하면 오타나 대소문자 구분 오류 등으로 인해 실수가 발생할 가능성이 높아집니다. 이런 실수는 권한 부여의 오류로 이어질 수 있으며, 시스템의 안정성을 해칠 수 있습니다. 
3. 타입 안정성 부족: 문자열은 타입 안정성이 낮습니다. 즉, 컴파일 시점에 권한 값의 정확성을 검증하기 어렵습니다. 이는 런타임에 예기치 못한 오류를 발생시킬 수 있습니다. 
4. 확장성과 유지 보수의 어려움: 문자열 기반의 권한 시스템은 확장성이 낮고 유지 보수가 어렵습니다. 권한이 변경되거나 새로운 권한이 추가될 때마다, 모든 문자열 리터럴을 검토하고 수정해야 할 수 있습니다.