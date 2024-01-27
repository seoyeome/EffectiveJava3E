## Item83. 지연초기화는 신중히 사용하라

- 지연초기화는 필드의 초기화 시점을 그 값이 처음 필요할 때까지 늦추는 기법이다.
- 값이 전혀 쓰이지 않으면 초기화도 결코 일어나지 않는다.
- 정적 필드와 인스턴스 필드 모두에 사용할 수 있다.
- 지연 초기화는 주로 최적화 용도로 쓰이지만, 클래스와 인스턴스 초기화 때 발생하는 위험한 순환 문제를 해결하는 효과도 있다.

### 지연 초기화는 "필요할 때까지는 하지 말라"
- 클래스 혹은 인스턴스 생성 시의 초기화 비용은 줄지만 그 대신 지연 초기화하는 필드에 접근하는 비용은 커진다.
- 지연 초기화하려는 필드들 중 결국 초기화가 이뤄지는 비율에 따라, 실제 초기화에 드는 비용에 따라, 초기화된 각 필드를 얼마나 빈번히 호출하느냐에 따라 지연 초기화가 실제로는 성능을 느려지게 할 수 있다.

### 지연 초기화가 필요한 경우
- 해당 클래스의 인스턴스 중 그 필드를 사용하는 인스턴스의 비율이 낮은 반면, 그 필드를 초기화하는 비용이 크다면 지연 초기화가 제 역할을 해줄 것이다.
- 정말 그런지 확인하는 방법 : 지연 초기화 적용 전후의 성능을 측정해보아야함

### 멀티스레드 환경에서는 지연 초기화를 하기가 까다롭다.
- 지연 초기화하는 필드를 둘 이상의 스레드가 공유한다면 어떤 형태로든 반드시 동기화해야 한다.

### 대부분의 상황에서 일반적인 초기화가 지연 초기화보다 낫다.

### 코드 83-1 인스턴스 필드를 초기화하는 일반적인 방법
```java
private final FieldType field = computeFieldValue();
```
- 지연 초기화가 초기화 순환성을 깨뜨릴 것 같으면 syncronized를 단 접근자를 사용하자.

### 코드83-2 인스턴스 필드의 지연 초기화 - synchronized 접근자 방식
```java
private FieldType field;

private synchronized FieldType getField() {
    if (field == null) {
        field = computeFieldValue();
    }
    return field;
}
```

### 성능 때문에 정적 필드를 지연 초기화해야 한다면 지연 초기화 홀더 클래스 관용구를 사용하자.
```java
public class Singleton {
    private Singleton() {}

    private static class Holder {
        private static final Singleton INSTANCE = new Singleton();
    }

    public static Singleton getInstance() {
        return Holder.INSTANCE;
    }
}
```
- 이 코드에서 Singleton 클래스의 getInstance 메소드가 호출될 때까지 Singleton 인스턴스는 생성되지 않습니다. getInstance 메소드가 호출되면, Holder 클래스가 로드되고, 그 때 Singleton 인스턴스가 생성됩니다. 이 방식은 지연 초기화와 스레드 안전성을 동시에 제공합니다.

### 성능 때문에 인스턴스 필드를 지연 초기화해야 한다면 이중검사 관용구를 사용하라.

### 코드83-4 인스턴스 필드 지연 초기화용 이중검사 관용구

```
이중 검사 잠금(Double-Check Locking) 관용구에서 volatile로 필드를 선언하는 것은 멀티스레드 환경에서 객체의 안전한 초기화를 보장하기 위한 중요한 부분입니다. volatile 키워드의 사용과 관련된 의미를 설명하겠습니다:

메모리 가시성 보장: 자바에서 volatile 키워드는 필드가 여러 스레드에 의해 공유될 때 메모리 가시성을 보장합니다. volatile 필드의 값이 변경되면, 그 변경 사항이 즉시 다른 스레드에게도 보여집니다. 이는 한 스레드에서 수정한 필드의 값을 다른 스레드가 항상 볼 수 있게 합니다.

재배치 방지: 컴파일러와 프로세서는 성능 최적화를 위해 명령어의 순서를 재배치할 수 있습니다. volatile 키워드는 필드에 대한 연산 순서를 재배치하지 않도록 보장합니다. 이는 멀티스레드 환경에서 중요한 안정성을 제공합니다.

이중 검사 잠금과의 관계: 이중 검사 잠금 관용구는 객체의 초기화를 지연시키고, 해당 객체가 필요할 때만 초기화를 수행하는 방법입니다. 이 패턴은 객체의 초기화 여부를 먼저 검사한 후(첫 번째 검사), 객체가 초기화되지 않았다면 동기화 블록으로 진입하여 다시 한 번 초기화 여부를 검사합니다(두 번째 검사). volatile 필드 사용은 이 과정에서 중요합니다. 왜냐하면 객체의 초기화 상태가 모든 스레드에게 정확히 반영되어야 하기 때문입니다.
```
```java
private volatile FieldType field;

private FieldType getField() {
    FieldType result = field;
    if (result != null) { // 첫 번째 검사 (락 사용 안 함)
        return result;
    }

    synchronized(this) {
        if (field == null) { // 두 번째 검사 (락 사용)
            field = computeFieldValue();
        }
        return field;
    }
}
```
### 코드83-5 단일검사 관용구 - 초기화가 중복해서 일어날 수 있다
```java
private volatile FieldType field;

private FieldType getField() {
    FieldType result = field;
    if (result == null) {
        field = result = computeFieldValue();
    }
    return result;
}
```
- 보통은 거의 쓰이지 않는다.

```
핵심 정리
대부분의 필드는 지연시키지 말고 곧바로 초기화해야 한다.
성능 때문에 혹은 위험한 초기화 순환을 막기 위해 꼭 지연 초기화를 써야 한다면 올바른 지연 초기화 기법을 사용하자.
인스턴스 필드에는 이중검사 관용구를, 정적 필드에는 지연 초기화 홀더 클래스 관용구를 사용하자.
반복해 초기화해도 괜찮은 인스턴스 필드에는 단일검사 관용구도 고려 대상이다.
```