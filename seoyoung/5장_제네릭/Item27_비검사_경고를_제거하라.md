## 27. 비검사 경고를 제거하라

```java
Set<Lark> exaltation = new HashSet();
```
- 위 코드는 제네릭을 사용하지 않아 경고가 발생한다.
- 할 수 있는 한 모든 비검사 경고를 제거하자.
- 경고를 제거할 수는 없지만 타입이 안전하다고 확신할 수 있다면 @SuppressWarnings("unchecked") 애너테이션을 달아 경고를 숨기자.
- @SuppressWarnings 애너테이션은 항상 가능한 한 좁은 범위에 적용하자.
- @SuppressWarnings("unchecked") 애너테이션을 사용할 때는 그 경고를 무시해도 안전한 이유를 항상 주석으로 남겨야 한다.