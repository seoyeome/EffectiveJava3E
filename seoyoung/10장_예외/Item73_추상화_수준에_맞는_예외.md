## Item73. 추상화 수준에 맞는 예외를 던지라

### 상위 계층에서는 저수준 예외를 잡아 자신의 추상화 수준에 맞는 예외로 바꿔 던져야 한다.
- 예외 번역(exception translation)

### 73-1. 예외 번역
```java
try {
    ... // 저수준 추상화를 이용한다.
} catch (LowerLevelException e) {
    throw new HigherLevelException(...);
}
```
- 예외를 번역할 때, 저수준 예외가 디버깅에 도움이 된다면 예외 연쇄(exception chaining)를 사용하는게 좋다.

### 아래 계층에서의 예외를 피할 수 없다면, 상웨 계층에서 그 예외를 조용히 처리하여 문제를 API 호출자에까지 전파하지 않는 방법이 있다.
- java.util.logging 같은 적절한 로깅 기능을 화룡하여 기록해두면 좋다.

### 73-2. 예외 연쇄
```java
try {
    ... // 저수준 추상화를 이용한다.
} catch (LowerLevelException cause) {
    throw new HigherLevelException(cause);
}
```

```java
핵심 정리
아래 계층의 예외를 예방하거나 스스로 처리할 수 없고, 그 예외를 상위 계층에 그대로 노출하기 곤란하다면 예외 번역을 사용하라.
이 때 예외 연쇄를 이용하면 상위 계층에는 맥락에 어울리는 고수준 예외를 던지면서 근본 원인도 함께 알려주어 오류를 분석하기에 좋다.
```