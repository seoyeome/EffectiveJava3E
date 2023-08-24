## 26. 로 타입은 사용하지 말라
- 클래스와 인터페이스 선언에 타입 매개변수(type-parameter)가 쓰이면, 이를 **제네릭 클래스** 혹은 **제네릭 인터페이스**라 한다.
- 로 타입이란 제네릭 타입에서 타입 매개변수를 전혀 사용하지 않을 때를 말한다.
- 예를 들어, `List<E>`의 로 타입은 `List`이다.
- 로 타입은 타입 선언에서 제네릭 타입 정보가 전부 지워진 것처럼 동작하는데, 제네릭이 도래하기 전 코드와 호환되도록 하기 위한 궁여지책이라 할 수 있다.

```java
//Stamp 인스턴스만 취급한다.
private final Collection stamps = ...;
```
- 이 코드를 사용하면 실수로 도장(Stamp) 대신 동전(Coin)을 넣어도 아무 오류 없이 컴파일되고 실행된다

```java
//실수로 동전을 넣는다.
stamps.add(new Coin(...)); //"unchecked call" 경고를 내뱉는다.
```

```java
for (Iterator i = stamps.iterator(); i.hasNext(); ) {
    Stamp stamp = (Stamp) i.next(); //ClassCastException을 던진다.
    stamp.cancel();
}
```

- 오류는 가능한 한 발생 즉시, 이상적으로는 컴파일할 때 발견하는 것이 좋다.

```java
private final Collection<Stamp> stamps = ...;
```
- 컴파일러는 stamps에는 Stamp의 인스턴스만 넣어야 함을 컴파일러가 인지하게 된다.


- 로 타입인 List와 매개변수화 타입인 List<Object>의 차이
- List는 제네릭 타입 시스템에 속하지 않는다.
- List<Object>는 모든 타입의 객체를 허용한다는 것을 컴파일러에 명확히 전달한 것이다.

```java
List<String> stringList = new ArrayList<>();
List<Object> objectList = stringList;  // 이렇게 할 수 있다고 가정하면
objectList.add(42);  // Integer를 추가하는 것이 가능해진다.
String s = stringList.get(0);  // 실행 시간에 오류 발생. 왜냐하면 42는 String이 아니다.

```