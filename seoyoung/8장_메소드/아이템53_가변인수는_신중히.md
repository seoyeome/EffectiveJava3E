## Item53. 가변인수는 신중히 사용하라


### 인수가 1개 이상이어야 하는 가변인수 메서드 - 잘못 구현한 예
```java
static int min(int... args) {
    if (args.length == 0)
        throw new IllegalArgumentException("인수가 1개 이상 필요합니다.");
    int min = args[0];
    for (int i = 1; i < args.length; i++)
        if (args[i] < min)
            min = args[i];
    return min;
}
```
--> Integer.MAX_VALUE가 왜 나오는거지??

### 인수가 1개 이상이어야 할 때 가변인수를 제대로 사용하는 방법
```java
static int min(int firstArg, int... remainingArgs) {
    int min = firstArg;
    for (int arg : remainingArgs)
        if (arg < min)
            min = arg;
    return min;
}
```
- 가변인수는 인수 개수가 정해지지 않았을 때 아주 유용하다.
- printf는 가변인수와 한 묶음으로 자바에 도입
- 핵심 리플렉션 재정비????????

### 핵심정리
```
인수 개수가 일정하지 않은 메서드를 정의해야 한다면 가변인수가 반드시 필요하다.
메서드를 정의할 때 필수 매개변수는 가변인수 앞에 두고, 가변인수를 사용할 때는 성능 문제까지 고려하자.
```
