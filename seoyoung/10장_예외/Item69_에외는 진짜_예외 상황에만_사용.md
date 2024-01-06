## Item69. 에외는 진짜 예외 상황에만 사용하라

### 69-1 에외를 완전히 잘못 사용한 예 - 따라 하지 말 것!
```java
try {
    int i = 0;
    while(true)
        range[i++].climb();
} catch (ArrayIndexOutOfBoundsException e) {
}
```
- 이 코드는 배열의 원소를 순회하는데, 아주 끔찍한 방식으로 하고 있다.
- 무한루프를 돌다가 배열의 끝에 도달하면 ArrayIndexOutOfBoundsException이 발생하며 끝을 내는 것

```java
for (Mountain m : range)
    m.climb();
```

1. 예외는 예외 상황에 쓸 용도로 설계되었으므로 JVM 구현자 입장에서는 명확한 검사만큼 빠르게 만들어야 할 동기가 약하다.
2. 코드를 try-catch 블록 안에 넣으면 JVM이 적용할 수 있는 최적화가 제한된다.