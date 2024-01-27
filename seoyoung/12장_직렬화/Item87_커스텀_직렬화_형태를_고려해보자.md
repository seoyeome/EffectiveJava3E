## Item87. 커스텀 직렬화 형태를 고려해보라

### 괜찮다고 판단될 때만 기본 직렬화 형태를 사용하라
- 기본 직렬화 형태는 유연성, 성능, 정확성 측면에서 신중히 고민한 후 합당할 때만 사용해야 한다.

- 어떤 객체의 기본 직렬화 현태는 그 객체를 루트로 하는 객체 그래프의 물리적 모습을 나름 효율적으로 인코딩한다.
- 객체의 물리적 표현과 논리적 내용이 같다면 기본 직렬화 형태라도 무방하다.
```
Java의 기본 직렬화 메커니즘이 객체와 그 객체가 참조하는 다른 객체들(즉, 객체 그래프)의 전체 구조를 바이트 스트림으로 변환하는 과정이 비교적 효율적이라는 것을 의미합니다.

객체 그래프(Object Graph)
객체 그래프란 어떤 객체와 그 객체가 참조하는 다른 객체들, 그리고 그 객체들이 다시 참조하는 객체들까지를 포함하는 전체 구조를 말합니다. 각 객체는 그래프의 노드로, 객체 간의 참조는 그래프의 엣지로 표현됩니다.

기본 직렬화 과정
Java에서 객체를 직렬화할 때, Serializable 인터페이스를 구현한 객체는 자동으로 직렬화될 수 있습니다. 기본 직렬화 메커니즘은 객체의 상태(즉, 모든 필드의 값)를 포함합니다.

이때, 객체가 다른 객체를 참조하고 있다면, 참조된 객체들도 함께 직렬화됩니다. 이 과정은 참조된 객체들이 더 이상 새로운 객체를 참조하지 않을 때까지 재귀적으로 계속됩니다.

물리적 표현(Physical Representation)
객체의 물리적 표현은 객체가 메모리에 어떻게 저장되는지, 즉 객체의 실제 메모리 구조를 의미합니다. 이는 객체의 모든 필드(공개된 필드와 비공개된 필드 포함), 그리고 그 필드의 타입과 값을 포함합니다.

논리적 내용(Logical Content)
객체의 논리적 내용은 객체가 나타내는 정보의 의미, 즉 객체가 프로그램 내에서 어떤 역할을 하는지에 대한 것입니다. 예를 들어, Date 객체의 논리적 내용은 특정 날짜와 시간을 나타냅니다.

```

### 기본 직렬화 형태가 적합하다고 결정했더라도 불변식 보장과 보안을 위해 readObject 메서드를 제공해야 할 때가 많다.
- 필드가 모두 private 여도 문서화 주석을 달아야 한다.
- 직렬화 형태에 포함되는 공개 API는 반드시 문서화 해야 한다.
- 자바독에 알려주는 역할은 @serial 태그가 한다.

### 객체의 물리적 표현과 논리적 표현의 차이가 클 때 네 가지 문제
1. 공개 API가 현재의 내부 표현 방식에 영구히 묶인다.
2. 너무 많은 공간을 차지할 수 있다.
3. 시간이 너무 많이 걸릴 수 있다.
4. 스택 오버플로를 일으킬 수 있다.

### 코드87-2 기본 직렬화 형태에 적합하지 않은 클래스
```java
public final class StringList implements Serializable {
    private int size = 0;
    private Entry head = null;
    
    private static class Entry implements Serializable {
        String data;
        Entry next;
        Entry previous;
    }
    
    ... // 나머지 코드 생략
}
```
### 코드87-3 합리적인 커스텀 직렬화 형태를 갖춘 StringList
```java
public final class StringList implements Serializable {
    private transient int size = 0;
    private transient Entry head = null;
    
    private static class Entry {
        String data;
        Entry next;
        Entry previous;
    }
    
    // 지정한 문자열을 이 리스트에 추가한다.
    public final void add(String s) { ... }
    
    /**
     * 이 {@code StringList} 인스턴스를 직렬화한다.
     *
     * @serialData 이 리스트의 크기(포함된 문자열의 개수)를 기록한 후
     * ({@code int}), 이어서 모든 원소를(각각은 {@code String}) 순서대로 기록한다.
     */
    private void writeObject(ObjectOutputStream s) throws IOException {
        s.defaultWriteObject();
        s.writeInt(size);
        
        // 모든 원소를 올바른 순서로 기록한다.
        for (Entry e = head; e != null; e = e.next) {
            s.writeObject(e.data);
        }
    }
    
    private void readObject(ObjectInputStream s) throws IOException, ClassNotFoundException {
        s.defaultReadObject();
        int numElements = s.readInt();
        
        // 모든 원소를 읽어 이 리스트에 삽입한다.
        for (int i = 0; i < numElements; i++) {
            add((String) s.readObject());
        }
    }
    
    ... // 나머지 코드 생략
}
```
1번 코드와 2번 코드 사이의 주요 차이는 직렬화 방식에 있습니다. 이 차이점은 메모리 사용량과 성능에도 영향을 미칩니다.

1번 코드의 직렬화 방식
1번 코드에서는 StringList 클래스와 내부 정적 클래스 Entry 모두 Serializable 인터페이스를 구현합니다. 이 경우, 직렬화 과정에서 Java의 기본 직렬화 메커니즘이 사용됩니다. 즉, StringList 객체와 이 객체가 참조하는 Entry 객체들이 재귀적으로 직렬화됩니다. 이 때, 모든 Entry 객체와 그 객체가 참조하는 다음 Entry 객체, 그리고 문자열 데이터까지 모두 직렬화 대상이 됩니다.

이 방식의 문제점은 다음과 같습니다:

메모리 사용량: 모든 Entry 객체와 연결된 참조가 직렬화되기 때문에, 많은 양의 메모리를 사용할 수 있습니다.
성능 저하: 크고 복잡한 연결 리스트를 직렬화하는 데 시간이 오래 걸릴 수 있습니다.
2번 코드의 직렬화 방식
2번 코드에서는 transient 키워드를 사용하여 size 필드와 head 필드를 직렬화 대상에서 제외합니다. 또한, Entry 클래스는 Serializable을 구현하지 않습니다. 대신, writeObject와 readObject 메소드를 사용하여 직렬화와 역직렬화 과정을 명시적으로 정의합니다. 이 방식에서는 리스트의 크기와 각 Entry 객체가 포함하는 문자열 데이터만 직렬화됩니다.

이 방식의 장점은 다음과 같습니다:

메모리 사용량 감소: Entry 객체의 연결 관계가 직렬화되지 않기 때문에, 직렬화된 데이터의 크기가 크게 감소합니다.
성능 향상: 불필요한 객체 참조 정보를 직렬화하지 않기 때문에, 직렬화 및 역직렬화 과정이 더 빠릅니다.
결론
2번 코드는 직렬화 과정을 최적화하여 메모리 사용량을 줄이고 성능을 향상시킵니다. transient 키워드와 사용자 정의 직렬화 방법을 통해 불필요한 객체 정보를 직렬화 대상에서 제외함으로써, 데이터의 크기를 줄이고 처리 속도를 높입니다. 이러한 접근 방식은 특히 크고 복잡한 객체 그래프를 직렬화할 때 유용합니다.
```

### 해당 객체의 논리적 상태와 무관한 필드라고 확신할 때만 transient 한정자를 생략하라

### 어떤 직렬화 형탤르 택하든 직렬화 가능 클래스 모두에 직렬 버전 UID를 명시적으로 부여하자

```
핵심 정리
클래스를 직렬화하기로 했다면 어떤 직렬화 형태를 사용할지 심사숙고하기
자바의 기본 직렬화 형태는 객체를 직렬화한 결과가 해당 객체의 논리적 표현에 부합할 때만 사용하고, 그렇지 않으면 객체를 적절히 설명하는 커스텀 직렬화 형태를 고안하라.
직렬화 형태도 공개 메서드를 설계할 때에 준한느 시간을 들여 설계해야 한다.
한번 공개된 메서드는 향후 릴리스에서 제거할 수 없듯이, 직렬화 형태에 포함된 필드도 마음대로 제거할 수 없다.
직렬화 호환성을 유지하기 위해 영원히 지원해야 하는 것이다.
잘못된 직렬화 형태를 선택하면 해당 클래스의 복잡성과 성능에 영구히 부정적인 영향을 남긴다. 
```