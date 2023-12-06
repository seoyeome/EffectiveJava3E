## Item54. null이 아닌, 빈 컬렉션이나 배열을 반환하라

### 컬렉션이 비었으면 null을 반환한다. - 따라 하지 말 것!
```java
private final List<Cheese> cheesesInStock = ...;

/**
* @return 매장 안의 모든 치즈 목록을 반환한다.
* 단, 재고가 하나도 없다면 null을 반환한다.
*/
public List<Cheese> getCheeses() {
    return cheesesInStock.isEmpty() ? null
        : new ArrayList<>(cheesesInStock);
}
```
- null을 반환하면 클라이언트에서는 이를 처리하는 코드를 추가로 작성해야 한다.
- null을 반환하는 습관은 더 나쁜 결과를 초래한다.
- null을 반환하는 메서드에서는 그 문서화를 확실히 해야 한다.
- null을 반환하는 API는 사용하기 어렵고 오류 처리 코드도 늘어난다.

### 빈 컬렉션을 반환하는 올바른 예
```java
public List<Cheese> getCheeses() {
    return new ArrayList<>(cheesesInStock);
}
```

### 최적화 - 빈 컬렉션을 매번 새로 할당하지 않고, 불변인 빈 컬렉션을 반환하는 방법
```java
public List<Cheese> getCheeses() {
    return cheesesInStock.isEmpty() ? Collections.emptyList()
        : new ArrayList<>(cheesesInStock);
}
```
### 길이가 0일 수도 있는 배열을 반환하는 올바른 예
```java
public Cheese[] getCheeses() {
    return cheesesInStock.toArray(new Cheese[0]);
}
```
- 미리 길이 0짜리 배열을 미리 선언해두고 매번 그 배열을 봔환.......
- 길이 0인 배열은 모두 불변..?