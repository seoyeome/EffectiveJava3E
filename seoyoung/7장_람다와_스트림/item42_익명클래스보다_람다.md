## item42. 익명 클래스보다는 람다를 사용하라

- 예전에는 자바에서 함수 타입을 표현할 때 추상 메서드를 하나만 담은 인터페이스를 사용
```java
@FunctionalInterface
public interface IntBinaryOperator {
    int applyAsInt(int left, int right);
}
```
```java
IntBinaryOperator adder = (a, b) -> a + b;
System.out.println(adder.applyAsInt(5, 3));  // 출력: 8
```
### 익명클래스란
1. 이름이 없다.
2. 한 번만 사용된다.
3. new 키워드를 사용하여 바로 생성된다.
4. 클래스 또는 인터페이스를 확장 또는 구현할 수 있다.
### 익명클래스 형식
```java
Collections.sort(words, new Comparator<String>() {
    public int compare(String s1, String s2) {
        return Integer.compare(s1.length(), s2.length());
    }
});
```
### 익명클래스 람다식으로 대체
```java
Collections.sort(words, (s1, s2) -> Integer.compare(s1.length(), s2.length()));
```
- 람다의 모든 매개변수 타입은 생략하자.
- 코드가 세주 이상이면 람다를 쓰지말자
- 람다는 함수형 인터페이스에서만 쓰인다.

## 함수형 인터페이스
-  정확히 하나의 추상 메서드만을 포함하는 인터페이스입니다. (그러나 디폴트 메서드나 정적 메서드는 여러 개 있어도 된다.)

### Q. 람다를 직렬화하는 일은 극히 삼가야 한다는게 무슨 소리~?????? 람다를 직렬화 할 일이 있나....