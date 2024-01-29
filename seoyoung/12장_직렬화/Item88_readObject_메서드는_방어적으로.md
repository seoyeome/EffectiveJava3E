## Itme88. readObject 메서드는 방어적으로 작성하라

### 코드 88-1 방어적 복사를 사용하는 불변 클래스
```java
public final class Period {
    private final Date start;
    private final Date end;

    /**
     * @param start 시작 시각
     * @param end 종료 시각. 시작 시각보다 뒤여야 한다.
     * @throws IllegalArgumentException 시작 시각이 종료 시각보다 늦을 때 발생한다.
     * @throws NullPointerException start나 end가 null이면 발생한다.
     */
    public Period(Date start, Date end) {
        this.start = new Date(start.getTime());
        this.end = new Date(end.getTime());

        if (this.start.compareTo(this.end) > 0)
            throw new IllegalArgumentException(start + "가 " + end + "보다 늦다.");
    }

    public Date start() {
        return new Date(start.getTime());
    }

    public Date end() {
        return new Date(end.getTime());
    }

    public String toString() {
        return start + " - " + end;
    }

    // ... // 나머지 코드 생략
}
```
- 이 클래스를 직렬화하기로 결정해서 기본 직렬화 형태를 사용해도 되지만, Implements Serializable을 추가하면 이 클래스의 주요한 불변식을 보장하지 못하게 된다.
- readObject 메서드가 실질적으로 또 다른 public 생성자이기 때문...?
- readObject 메서드에서도 인수가 유효한지 검사해야 하고 필요하다면 매개변수를 방어적으로 복사해야 한다.
- readObject가 이 작업을 제대로 수행하지 못하면 공격자는 아주 손쉽게 해당 클래스의 불변식을 꺠뜨릴 수 있다.
- readObject는 매개변수로 바이트 스트림을 받는 생성자라 할 수 있다
- 보통의 경우 바이트 스트림은 정상적으로 생성된 인스턴스를 직렬화해 만들어진다.
- 하지만 불변식을 깨뜨릴 의도로 임의 생성한 바이트 스트림을 건네면 문제가 생긴다.
- 정상적인 생성자로는 만들어낼 수 없는 객체를 생성해낼 수 있기 때문이다.
- 단순히 Period 클래스 선언에 Implements Serialzable만 추가하면 종료 시작이 시작 시각보다 앞서는 Period 인스턴스가 만들어지게 된다........

### 해결 방법
- Period의 readObject 메서드가 defaultReadObject를 호출한 다음 역직렬화된 객체가 유효한지 검사해야 한다
- 유효성 검사에 실패하면 InvalidObjectException을 던져야 한다.

### 유효성 검사를 해도 문제가 있다
- 바이트 스트림 끝에 private Date 필드로의 참조를 추가하면 가변 Period 인스턴스를 만들어낼 수 있다.
- 공격자는 ObjectInputStream에서 Period 인스턴스를 읽은 후 스트림 끝에 추가된 이 악의적인 객체 참조를 읽어 Period 객체의 내부 정보를 얻을 수 있다. -> 불변이 아니다.

### 코드 88-4 가변 공격의 예
```java
public class MutablePeriod {
    // Period 인스턴스
    public final Period period;
    
    // 시작 시각 필드 - 외부에서 접근할 수 없어야 한다.
    public final Date start;
    
    // 종료 시각 필드 - 외부에서 접근할 수 없어야 한다.
    public final Date end;
    
    public MutablePeriod() {
        try {
            ByteArrayOutputStream bos = new ByteArrayOutputStream();
            ObjectOutputStream out = new ObjectOutputStream(bos);
            
            // 유효한 Period 인스턴스를 직렬화한다.
            out.writeObject(new Period(new Date(), new Date()));
            
            /*
             * 악의적인 '이전 객체 참조', 즉 내부 Date 필드로의 참조를 추가한다.
             * 상세 내용은 자바 객체 직렬화 명세의 6.4절을 참고하자.
             */
            byte[] ref = {0x71, 0, 0x7e, 0, 5}; // Ref #5
            bos.write(ref); // 시작 시각 필드
            ref[4] = 4; // Ref #4
            bos.write(ref); // 종료(end) 필드
            
            // Period 역직렬화 후 Date 참조를 '훔친다'.
            ObjectInputStream in = new ObjectInputStream(new ByteArrayInputStream(bos.toByteArray()));
            period = (Period) in.readObject();
            start = (Date) in.readObject();
            end = (Date) in.readObject();
        } catch (IOException | ClassNotFoundException e) {
            throw new AssertionError(e);
        }
    }
}
```

### 실제 공격이 이루어지는 모습
```java
public static void main(String[] args) {
    MutablePeriod mp = new MutablePeriod();
    Period p = mp.period;
    Date pEnd = mp.end;
    
    // 시간을 되돌리자!
    pEnd.setYear(78);
    System.out.println(p);
    
    // 60년대로 회귀!
    pEnd.setYear(69);
    System.out.println(p);
}
```

### 객체를 역직렬화할 때는 클라이언트가 소유해서는 안 되는 객체 참조를 갖는 필드를 모두 반드시 방어적으로 복사해야 한다.
- readObject에서는 불변 클래스 안의 모든 private 가변 요소를 방어적으로 복사해야 한다.

### 코드 88-5 방어적 복사와 유효성 검사를 수행하는 readObject 메서드
```java
private void readObject(ObjectInputStream s) throws IOException, ClassNotFoundException {
    s.defaultReadObject();
    
    // 가변 요소들을 방어적으로 복사한다.
    start = new Date(start.getTime());
    end = new Date(end.getTime());
    
    // 불변식이 만족되는지 검사한다.
    if (start.compareTo(end) > 0)
        throw new InvalidObjectException(start + "가 " + end + "보다 늦다.");
}
```

```
핵심 정리
readObject 메서드를 작성할 때는 언제나 public 생성자를 작성하는 자세로 임해야 한다.
readObject는 어떤 바이트 스트림이 넘어오더라도 유효한 인스턴스를 만들어내야한다.
바이트 스트림이 진짜 직렬화된 인스턴스라고 가정해서는 안된다.
이번 아이템에서는 기본 직렬화 형태를 사용한 클래스를 예로 들었지만 커스텀 직렬화를 사용하더라도 모든 문제가 그대로 발생할 수 있다.
readObject 메서드를 작성하는 지침
1. private이어야 하는 객체 참조 필드는 각 필드가 가리키는 객체를 방어적으로 복사하라. 불변 클래스 내의 가변 요소가 여기 속한다.
2. 모든 불변식을 검사하여 어긋나는 게 발견되면 InvalidObjectException을 던진다. 방어적 복사 다음에는 반드시 불변식 검사가 뒤따라야 한다.
3. 역직렬화 후 객체 그래프 전체의 유효성을 검사해야 한다면 ObjectInputValidation 인터페이스를 사용하라.
4. 직접적이든 간접적이든, 재정의할 수 있는 메서드는 호출하지 말자
```