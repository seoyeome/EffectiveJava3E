## Item78. 공유 중인 가변 데이터는 동기화해 사용하라

- synchronized 키워드는 해당 메서드나 블록을 한번에 한 스레드씩 수행하도록 보장한다.
- 많은 프로그래머가 동기화를 배타적 실행, 즉 한 스레드가 변경하는 중이라서 상태가 일관되지 않은 순간의 객체를 다른 스레드가 보지 못하게 막는 용도로만 생각한다.
- 한 객체가 일관된 상태를 가지고 생성되고, 이 객체에 접근하는 메서드는 그 객체에 lock을 건다.
- 락을 건 메서드는 객체의 상태를 확인하고 필요하면 수정한다.
- 객체를 하나의 일관된 상태에서 다른 일관된 상태로 변화시킨다.
- 동기화를 제대로 사용하면 어떤 메서드도 이 객체의 상태가 일관되지 않은 순간을 볼 수 없을 것이다.

**동기화에는 중요한 기능이 하나 더 있다.**
- 동기화 없이는 한 스레드가 만든 변화를 다른 스레드에서 확인하지 못할 수 있다.
- 동기화는 일관성이 깨진 상태를 볼 수 없게 하는 것은 물론, 동기화된 메서드나 블록에 들어간 스레드가 같은 락의 보호하에 수행된 모든 이전 수정의 최종 결과를 보게 해준다.

### 동기화는 배타적 실행뿐 아니라 스레드 사이의 안정적인 통신에 꼭 필요하다.
- 자바 언어 명세는 스레드가 필드를 읽을 때 항상'수정이 완전히 반영된' 값을 얻는다고 보장하지만, 한 스레드가 저장한 값이 다른 스레드에게 '보이는가'는 보장하지 않는다.
- 한 스레드가 만든 변화가 다른 스레드에게 언제 어떻게 보이는지를 규정한 자바의 메모리 모델 떄문

### Thread.stop은 사용하지 말자!
- 다른 스레드를 멈추는 올바른 방법은 다음과 같다.
- 첫 번째 스레드는 자신의 boolean 필드를 폴링하면서 그 값이 true가 되면 멈춘다.
- 이 필드를 false로 초기화해놓고, 다른 스레드에서 이 스레드를 멈추고자 할 때 true로 변경하는 식이다.
- Boolean 필드를 읽고 쓰는 작업은 원자적이라 어떤 프로그래머는 이런 필드에 접근할 때 동기화를 제거하기도 한다(?)

### 원적(Atomic) 작업
- 프로그래밍에서 '원자적'이라는 용어는 어떤 작업이 중간 단계 없이 완전히 이루어지거나 전혀 이루어지지 않음을 의미합니다.
- 즉, 작업이 분할 불가능하고, 중간에 다른 작업에 의해 방해받지 않는 성질을 가집니다.
- Boolean 필드의 읽기와 쓰기는 대부분의 컴퓨터 시스템에서 원자적으로 처리됩니다.
- 즉, 한 번의 기계어 명령으로 완료되므로, 도중에 다른 스레드에 의해 방해받지 않습니다.

### 동기화 제거
- 멀티스레딩 환경에서는 여러 스레드가 동시에 같은 데이터에 접근할 때 데이터의 일관성을 유지하기 위해 동기화를 사용합니다.
- 하지만, 어떤 작업이 원자적일 경우, 데이터의 일관성이 자연스럽게 보장되므로 별도의 동기화 메커니즘(예: 락)이 필요 없게 됩니다.
- 따라서, Boolean 필드에 접근할 때 일부 프로그래머는 동기화 과정을 생략하기도 합니다.

### 잘못된 코드 - 이 프로그램은 얼마나 오래 실행될까?
```java
public class StopThread {
    private static boolean stopRequested;

    public static void main(String[] args) throws InterruptedException {
        Thread backgroundThread = new Thread(() -> {
            int i = 0;
            while (!stopRequested) {
                i++;
            }
        });
        backgroundThread.start();

        TimeUnit.SECONDS.sleep(1);
        stopRequested = true;
    }
}
```
- 동기화하지 않으면 메인 스레드가 수정한 값을 백그라운드 스레드가 언제쯤에나 보게 될지 보증할 수 없다.
- 동기화가 빠지면 가상머신이 다음과 같은 최적화를 수해알 수도 있는 것이다.

```java
while (!stopRequested) {
    i++;
}i
```

```java
if (!stopRequested) {
    while (true) {
        i++;
    }
}
```

### 적절히 동기화해 스레드가 정상 종료한다.
```java
public class StopThread {
    private static boolean stopRequested;

    private static synchronized void requestStop() {
        stopRequested = true;
    }

    private static synchronized boolean stopRequested() {
        return stopRequested;
    }

    public static void main(String[] args) throws InterruptedException {
        Thread backgroundThread = new Thread(() -> {
            int i = 0;
            while (!stopRequested()) {
                i++;
            }
        });
        backgroundThread.start();

        TimeUnit.SECONDS.sleep(1);
        requestStop();
    }
}
```
- 쓰기 메서드만 동기화해서는 충분하지 ㅇ낳다.
- 쓰기와 읽기 모두가 동기화되지 않으면 동작을 보장하지 않는다.
- stopRequested 필드를 volatile로 선언하면 동기화를 생략해도 된다.
- volatile 한정자는 배타적 수행과는 상관없지만 항상 가장 최근에 기록된 값을 읽게 됨을 보장한다.
- volatile은 주의해서 사용해야 한다.

### 78-4 잘못된 코드 - 동기화가 필요하다!
```java
private static volatile int nextSerialNumber = 0;

public static int generateSerialNumber() {
    return nextSerialNumber++;
}
```
- 이 메서드는 매번 고유한 값을 반환할 의도로 만들어졌다
- nextSerilalNumber라는 단 하나의 필드로 결정되는데, 원자적으로 접근할 수 있고 어떤 값이든 허용한다.
- 동기화하지 얺더라도 불변식을 보호할 수 있어 보인다.
- 하지만 이 역시 동기화 없이는 올바로 동작하지 않는다.
- 문제는 증가 연산자(++)이다.
- 이 연산자는 코드상으로는 하나지만 실제로는 nextSerialNumber 필드에 두 번 접근한다.
- 먼저 값을 읽고, 그런 다음 새로운 값을 저장하는 것이다.
- 만약 두 번째 스레드가 이 두 접근 사이를 비집고 들어와 값을 읽어가면 첫 번째 스레드와 똑같은 값을 돌려받게 된다.
- 프로그램이 잘못된 결과를 계산해내는 이런 오류를 안전 실패(safety failure)라고 한다.
- generateSerialNumber 메서드에 synchronized 한정자를 붙이면 이 문제가 해결된다.
- 동시에 호출해도 서로 간섭하지 않으며 이전 호출이 변경한 값을 읽게 된다는 뜻이다.
- 메서드에 synchronized를 붙였다면 NextSerialNumber 필드에서는 volatile을 제거해야한다?
- 이 메서드를 더 견고하게 하려면 Int 대신 long을 사용하거나 nextSerialNumber가 최댓값에 도달하면 예외를 던지게 하자.

### java.util.concurrent.atomic 패키지의 AtomicLong을 사용해보자.
- 이 패키지에는 락 없이도 스레드 안전한 프로그래밍을 지원하는 클래스들이 담겨있다.
- volatile은 동기화의 두 효과 중 통신 쪽만 지원하지만 이 패키지는 원자성(배타적 실행)까지 지원한다.
- 성능도 동기화 버전보다 우수하다.

### 코드 78-5 java.util.concurrent.atomic을 이용한 락-프리 동기화
```java
private static final AtomicLong nextSerialNum = new AtomicLong();

public static long generateSerialNumber() {
    return nextSerialNum.getAndIncrement();
}
```

```
문장 "메서드에 synchronized를 붙였다면 NextSerialNumber 필드에서는 volatile을 제거해야 한다"의 의미는 다음과 같습니다:

만약 어떤 객체의 메서드가 synchronized로 선언되어 있다면, 해당 객체의 필드에 대한 모든 접근은 자동으로 동기화됩니다. 이는 synchronized 메서드가 한 번에 하나의 스레드만 실행할 수 있도록 보장하기 때문입니다.
이 경우, 동일한 객체의 NextSerialNumber와 같은 필드에 volatile 키워드를 사용할 필요가 없습니다. 왜냐하면 synchronized 메서드를 통해 이미 필요한 모든 동기화와 메모리 가시성이 보장되기 때문입니다.
따라서, synchronized 메서드 내에서 사용되는 필드에 volatile을 사용하는 것은 중복적인 동기화로 간주되어, 불필요한 성능 저하를 야기할 수 있습니다.
```

### 가변 데이터는 단일 스레드에서만 쓰도록 하자. 
- 가장 좋은 방법은 애초에 가변 데이터를 공유하지 않는 것

```
핵심 정리
여러 스레드가 가변 데이터를 공유한다면 그 데이터를 읽고 쓰는 동작은 반드시 동기화 해야 한다.
배타적 실행은 필요 없고 스레드끼리의 통신만 필요하다면 volatile 한정자만으로 동기화할 수 있다.
```