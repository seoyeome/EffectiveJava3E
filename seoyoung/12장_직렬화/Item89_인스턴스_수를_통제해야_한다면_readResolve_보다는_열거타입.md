## Item89. 인스턴스 수를 통제해야 한다면 readResolve보다는 열거 타입을 사용하라

```java
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();
    private Elvis() { ... }

    public void leaveTheBuilding() { ... }
}
```
- 선언에 implements Serializable을 추가하는 순간 더 이상 싱글턴이 아니게 된다.
- 기본 직렬화를 쓰지 않더라도, 명시적인 readObject를 제거하더라도 소용없다.
- 어떤 readObject를 사용하든 이 클래스가 초기화될 때 만들어진 인스턴스와는 별개인 인스턴스를 반환하게 된다.
- readResolve 기능을 이용하면 readObject가 만들어낸 인스턴슬르다른 것으로 대체할 수 있다.
- 역직렬화한 객체의 클래스가 readResolve 메서드를 적절히 정의해뒀다면, 역직렬화 후 새로 생성된 객체를 인수로 이 메서드가 호출되고, 이 메서드가 반환한 객체 참조가 새로 생성된 객체를 대신해 반환된다.
- readResolve 메서드는 역직렬화된 객체를 다른 객체로 대체할 수 있는 기능을 제공

```java
//인스턴스 통제를 위한 readResolve - 개선의 여지가 있다!
private Object readRsolve() {
    // 진짜 Elvis를 반환하고, 가짜 Elvis는 가비지 컬렉터에 맡긴다.
    return INSTANCE;
}
```
- 역직렬화한 객체는 무시하고 클래스 초기화 때 만들어진 Elvis 인스턴스를 반환한다.
- Elvis 인스턴스의 직렬화 형태는 아무런 실 데이터를 가질 이유가 없으니 모든 인스턴스 필드를 transient로 선언해야 한다.
### readResolve를 인스턴스 통제 목적으로 사용한다면 객체 참조 타입 인스턴스 필드는 모두 transient로 선언해야 한다.
```
transient 키워드는 자바에서 직렬화 과정에 영향을 주는 키워드입니다. 클래스의 특정 필드를 직렬화 과정에서 제외하고 싶을 때 transient 키워드를 사용합니다. 이는 필드가 직렬화 대상에서 제외되며, 따라서 그 필드의 데이터는 객체가 직렬화될 때 저장되거나 전송되지 않습니다.
```