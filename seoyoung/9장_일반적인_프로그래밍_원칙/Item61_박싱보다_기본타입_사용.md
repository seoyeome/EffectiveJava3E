## Item61. 박싱된 기본 타입보다는 기본 타입을 사용하라

### 자바 기본 데이터 타입
- int
- double
- boolean

### 참조 타입
- String
- List
- Integer
- Double
- Boolean

```
오토박싱과 오토언박싱 덕분에 두 타입을 크게 구분하지 않고 사용할 수 있지만 차이가 사라지는 것은 아니다.
어떤 타입을 사용하는지 중요하다.
```
### 기본 타입과 박싱된 기본 타입의 주된 차이
1. 기본 타입은 값만 가지고 있으나, 박싱된 기본 타입은 값에 더해 식별성이란 속성을 갖는다.
2. 기본 타입의 값은 언제나 유효하나, 박싱된 기본 타입은 유효하지 않은 값, Null을 가질 수 있다.
3. 기본 타입이 박싱된 기본 타입보다 시간과 메모리 사용면에서 더 효율적이다.

### 잘못 구현된 비교자
```java
Comparator<Integer> naturalOrder = 
        (i, j) -> (i < j) ? -1 : (i == j ? 0 : 1);
```
-> i < j 는 비교가 잘 되지만 i == j 에서 '객체 참조' 식별성을 검사하는데 false가 나와 1을 반환 -> 첫 번째 Integer 값이 두 번째보다 크다는 것,
박싱된 기본 타입에 == 연산자를 사용하면 오류가 발생
- 실무에서 기본 타입을 다루는 비교자가 필요하다면 Comparator.naturalOrder()를 사용하자.

### 61-2 문제를 수정한 비교자
```java
Comparator<Integer> naturalOrder = (iBoxed, jBoxed) -> {
    int i = iBoxed, j = jBoxed; // 오토 언박싱
    return i < j ? -1 : (i == j ? 0 : 1);
};
```

### 61-4 끔찍이 느린 코드
```java
public static void main(String[] args) {
    Long sum = 0L;
    for (long i = 0; i <= Integer.MAX_VALUE; i++) {
        sum += i;
    }
    System.out.println(sum);
}
```
- 지역변수 sum을 박싱된 기본 타입으로 선언하여 느려졌다.
- 오류나 경고 없이 컴파일됮미나, 박싱과 언박싱이 반복해서 일어나 성능이 느려진다.

### 문제의 원인
- 프로그래머가 기본 타입과 박싱된 기본 타입의 차이를 무시하여 생긴 일

### 박싱된 기본 타입 써야하는 경우
1. 컬렉션의 원소, 키, 값으로 사용한다. 컬렉션은 기본 타입을 담을 수 없으므로 어쩔 수 없이 박싱된 기본 타입을 써야한다. 매개변수화 타입이나 매개변수화 메서드의 타입 매개변수로는 박싱된 기본 타입을 써야한다.
2. 리플렉션을 통해 메서드를 호출할 때 박싱된 기본 타입을 사용해야 한다.

```
핵심정리
기본 타입과 박싱된 기본 타입 중 하나를 선택해야 한다면 가능하면 기본 타입을 사용하라.
기본 타입은 간단하고 빠르다.
박싱된 기본 타입을 써야 한다면 주의를 기울이자.
** 오토박싱이 박싱된 기본 타입을 사용할 때의 번거로움을 줄여주지만, 완전히 없애주지는 않는다. **
```