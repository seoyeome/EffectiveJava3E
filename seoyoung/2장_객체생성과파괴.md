# 2장 - 객체생성과 파괴
- - -
## 1. 생성자 대신 정적 팩터리 메서드를 고려하라
- 클래스는 생성자와 별도로 정적 팩터리 메서드를 제공할 수 있다.
  * 팩터리 메서드 : 클래스의 인스턴스를 반환한느 단순한 정적 메서드.

```
 예시 
 public static Boolean valueOf(boolean b) {
     return b ? Boolean.TRUE : Boolean.FALSE;
 }
```
### 1.1 정적 팩터리 메서드 장점

- 이름을 가질 수 있다.
  * 생성자는 클래스 이름과 같아야 하지만, 팩터리 메서드는 이름을 자유롭게 지을 수 있다.
  * 생성자는 어떤 객체를 생성하는지 정확히 알 수 없지만, 팩터리 메서드는 이름만 잘 지으면 반환될 객체의 특성을 쉽게 묘사할 수 있다.
  * ex) BigInteger.probablePrime
- 호출될 때마다 인스턴스를 새로 생성하지 않아도 된다.
  * ex) Boolean.valueOf(boolean) 메서드는 객체를 아예 생성하지 않는다. 
  * 플라이웨이트 패턴
    + 소프트웨어 디자인 패턴 중 하나, 주로 메모리 사용량을 줄이는 데 사용된다.
    + 한 번 생성된 객체는 다시 사용되고, 필요할 때마다 새로운 객체를 생성하는 대신 기존 객체를 재사용
```java
//Flightweight Pattern 예시
import java.util.HashMap;
import java.util.Map;

class Character {
private final char value;

    public Character(char value) {
        this.value = value;
    }

    public void print() {
        System.out.print(value);
    }
}

// Flyweight 팩토리를 나타내는 클래스
class CharacterFactory {
private Map<Character, Character> characters = new HashMap<>();

    public Character getCharacter(char characterValue) {
        Character character = characters.get(characterValue);
        if (character == null) {
            character = new Character(characterValue);
            characters.put(characterValue, character);
        }
        return character;
    }
}

// 클라이언트 코드
public class Main {
public static void main(String[] args) {
CharacterFactory factory = new CharacterFactory();

        String document = "Hello, flyweight pattern!";
        for (char c : document.toCharArray()) {
            Character character = factory.getCharacter(c);
            character.print();
        }
    }
}
```
- 반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다.
  * 반환할 객체의 클래스를 자유롭게 선택할 수 있게 하는 엄청난 유연성을 준다.
  * 반환할 객체의 클래스를 API에 공개하지 않아도 된다.
  * API를 작게 유지할 수 있다.
```java
public abstract class Shape {
    // ... 다양한 추상 메서드와 구현 ...
    
    public static Shape createCircle(double radius) {
        return new Circle(radius);
    }

    public static Shape createRectangle(double width, double height) {
        return new Rectangle(width, height);
    }
}

class Circle extends Shape {
    double radius;

    Circle(double radius) {
        this.radius = radius;
    }

    // ... Circle 관련 메서드와 구현 ...
}

class Rectangle extends Shape {
    double width, height;

    Rectangle(double width, double height) {
        this.width = width;
        this.height = height;
    }

    // ... Rectangle 관련 메서드와 구현 ...
}
```
```
//사용 예시
Shape circle = Shape.createCircle(10);
```
- 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다.
  * 반환 타입의 하위 타입이기만 하면 어떤 클래스의 객체를 반환하든 상관없다.
  * 반환할 객체의 클래스가 public이 아니어도 된다.
  * 반환할 객체의 클래스를 변경해도 클라이언트 측에서는 코드를 수정할 필요가 없다.
  * ex) EnumSet 클래스는 원소가 64개 이하면 RegularEnumSet 클래스의 인스턴스를, 65개 이상이면 JumboEnumSet 클래스의 인스턴스를 반환한다.

- 정적 팩터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다.
  * ex) JDBC

- - -
1. **서비스 인터페이스**:
```java
public interface PaymentService {
    void pay(int amount);
}
```

2. **서비스 제공자 인터페이스**:
```java
public interface PaymentServiceProvider {
    PaymentService createService();
}
```

3. **제공자 등록 및 서비스 접근 API**:
```java
public class PaymentServiceRegistry {
    private static Map<String, PaymentServiceProvider> providers = new HashMap<>();

    public static void registerProvider(String name, PaymentServiceProvider provider) {
        providers.put(name, provider);
    }

    public static PaymentService getService(String name) {
        PaymentServiceProvider provider = providers.get(name);
        if (provider == null) {
            throw new IllegalArgumentException("No provider registered with name: " + name);
        }
        return provider.createService();
    }
}
```

4. **제공자 구현 예시**:

신용카드 결제 제공자:
```java
public class CreditCardPaymentServiceProvider implements PaymentServiceProvider {
    @Override
    public PaymentService createService() {
        return new CreditCardPaymentService();
    }
}

class CreditCardPaymentService implements PaymentService {
    @Override
    public void pay(int amount) {
        System.out.println("Paid " + amount + " using Credit Card.");
    }
}
```

휴대폰 결제 제공자:
```java
public class MobilePaymentServiceProvider implements PaymentServiceProvider {
    @Override
    public PaymentService createService() {
        return new MobilePaymentService();
    }
}

class MobilePaymentService implements PaymentService {
    @Override
    public void pay(int amount) {
        System.out.println("Paid " + amount + " using Mobile Payment.");
    }
}
```

5. **사용 예시**:
```java
public class Main {
    public static void main(String[] args) {
        // 제공자 등록
        PaymentServiceRegistry.registerProvider("creditcard", new CreditCardPaymentServiceProvider());
        PaymentServiceRegistry.registerProvider("mobile", new MobilePaymentServiceProvider());

        // 서비스 사용
        PaymentService creditCardService = PaymentServiceRegistry.getService("creditcard");
        creditCardService.pay(100);

        PaymentService mobileService = PaymentServiceRegistry.getService("mobile");
        mobileService.pay(200);
    }
}
```

### 1.2 정적 팩터리 메서드 단점
- 상속을 하려면 public이나 protected 생성자가 필요하니 정적 팩터리 메서드만 제공하면 하위 클래스를 만들 수 없다.
- 정적 팩터리 메서드는 프로그래머가 찾기 어렵다.
  * 생성자처럼 API 설명에 명확히 드러나지 않으니 사용자는 정적 팩터리 메서드 방식 클래스를 인스턴스화할 방법을 알아내야 한다.
```
//정적 팩터리 메서드 명명 방식

from : 매개변수를 하나 받아서 해당 타입의 인스턴스를 반환하는 형변환 메서드
of : 여러 매개변수를 받아 적합한 타입의 인스턴스를 반환하는 집계 메서드
valueOf : from과 of의 더 자세한 버전
instance 혹은 getInstance : 매개변수로 명시한 인스턴스를 반환하지만, 같은 인스턴스임을 보장하지 않는다.
create 혹은 newInstance : instance 혹은 getInstance와 같지만, 매번 새로운 인스턴스를 생성해 반환함을 보장한다.
getType : getInstance와 같으나, 생성할 클래스가 아닌 다른 클래스에 팩터리 메서드를 정의할 때 쓴다.
newType : newInstance와 같으나, 생성할 클래스가 아닌 다른 클래스에 팩터리 메서드를 정의할 때 쓴다.
type : getType과 newType의 간결한 버전
```