## 3. private 생성자나 열거 타입으로 싱글턴임을 보증하라
- 싱글턴이란 인스턴스를 오직 하나만 생성할 수 있는 클래스를 말함
- 클래스를 싱글턴으로 만들면 이를 사용하는 클라이언트를 테스트하기가 어려워짐

### 싱글턴을 만드는 방식
1. public static 멤버가 final 필드인 방식
    - ```java
      public class Elvis {
          public static final Elvis INSTANCE = new Elvis();
          private Elvis() { ... }
          public void leaveTheBuilding() { ... }
      }
      ```
    - private 생성자는 public static final 필드인 Elvis.INSTANCE를 초기화할 때 딱 한번만 호출됨
    - public이나 protected 생성자가 없으므로 Elvis 클래스가 초기화될 때 만들어진 인스턴스가 전체 시스템에서 하나뿐임이 보장됨
    - 리플렉션 API인 AccessibleObject.setAccessible을 사용해 private 생성자를 호출할 수 있음
        - 이를 방지하려면 생성자를 수정하여 두 번째 객체가 생성되려 할 때 예외를 던지게 하면 됨
    - 리플렉션을 통한 공격을 방어하려면 생성자를 수정하여 두 번째 객체가 생성되려 할 때 예외를 던지게 하면 됨

2. 정적 팩터리 메서드를 public static 멤버로 제공하는 방식
    - ```java
      public class Elvis {
          private static final Elvis INSTANCE = new Elvis();
          private Elvis() { ... }
          public static Elvis getInstance() { return INSTANCE; }
          public void leaveTheBuilding() { ... }
      }
      ```
    - API를 바꾸지 않고도 싱글턴이 아니게 변경할 수 있음
    - 정적 팩터리를 제네릭 싱글턴 팩터리로 만들 수 있음
    - 정적 팩터리의 메서드 참조를 공급자(supplier)로 사용할 수 있음
   ```
   Q. Elvis::getInstance를 Supplier<Elvis>로 사용한다는게 뭔가요?
   ```
   
   ```java
   class Elvis {
       private static final Elvis INSTANCE = new Elvis();
   
       private Elvis() { }
   
       public static Elvis getInstance() {
           return INSTANCE;
       }
   
   }
   
   public class Main {
       public static void main(String[] args) {
           Supplier<Elvis> elvisSupplier = Elvis::getInstance;
           Elvis elvis = elvisSupplier.get();
       }
   }
   ```
    - public 필드 방식과 비슷하지만, 더 간결하고, 추가 노력 없이 직렬화할 수 있고, 심지어 아주 복잡한 직렬화 상황이나 리플렉션 공격에서도 제2의 인스턴스가 생기는 일을 완벽히 막아줌
    - 대부분 상황에서는 원소가 하나뿐인 열거 타입이 싱글턴을 만드는 가장 좋은 방법임