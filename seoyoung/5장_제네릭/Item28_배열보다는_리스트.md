## 28. 배열보다는 리스트를 사용하라.
- 배열과 제네릭 타입의 중요한 차이

### 1.1. 배열은 공변이다. 즉, Sub가 Super의 하위 타입이라면 배열 Sub[]는 Super[]의 하위 타입이 된다.(함께 변한다는 뜻)

### 1.2. 제네릭은 불공변이다. 즉, 서로 다른 타입 Type1과 Type2가 있을 때, List<Type1>은 List<Type2>의 하위 타입도 아니고 상위 타입도 아니다.

```java
Object[] objectArray = new Long[1];
objectArray[0] = "타입이 달라 넣을 수 없다."; // ArrayStoreException RuntimeError 예외 발생
```

```java
List<Object> ol = new ArrayList<Long>(); // 호환되지 않는 타입이다.
ol.add("타입이 달라 넣을 수 없다."); // 컴파일 오류 발생
```

### 2.1. 배열은 실체화(reify)된다. 즉, 배열은 런타임에도 자신이 담기로 한 원소의 타입을 인지하고 확인한다.


### 제네릭 배열 생성을 허용하지 않는 이유 - 컴파일 되지 않는다
```java
List<String>[] stringLists = new List<String>[1];
List<Integer> intList = List.of(42);
Object[] objects = stringLists;
objects[0] = intList;
String s = stringLists[0].get(0); // ClassCastException 예외 발생
```

