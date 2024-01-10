## 2. 생성자에 매개변수가 많다면 빌더를 고려하라.
### 점층적 생성자 패턴(telescoping constructor pattern)
- 필수 매개변수만 받는 생성자, 필수 매개변수와 선택 매개변수 1개를 받는 생성자, 선택 매개변수를 전부 다 받는 생성자까지 늘려가는 방식

- <span style="color:#F46666">단점</span> : 매개변수 개수가 많아지면 클라이언트 코드를 작성하거나 읽기 어렵다.
### 자바빈즈 패턴(JavaBeans pattern)
- 매개변수가 없는 생성자로 객체를 만든 후, 세터(setter) 메서드들을 호출해 원하는 매개변수의 값을 설정하는 방식
- <span style="color:#F46666">단점</span> : 객체 하나를 만들려면 메서드를 여러 개 호출해야 하고, 객체가 완전히 생성되기 전까지는 일관성이 무너진 상태에 놓이게 된다.
```java
// 예시
public class Person {
    private String name;
    private int age;

    public Person() { }

    public void setName(String name) {
        this.name = name;
    }

    public void setAge(int age) {
        this.age = age;
    }
}
```
```java
Person person = new Person();
person.setName("Alice");
// 이 시점에서는 age 속성이 초기화되지 않음.
```
### 빌더 패턴(Builder pattern)
- 필수 매개변수만으로 생성자를 호출해 객체를 만든다.
- 빌더 패턴은 계층적으로 설계된 클래스와 함께 쓰기에 좋다.
- 객체를 만들기 전 빌더부터 만들어야 한다.
- 매개변수가 4개 이상은 되어야 값어치를 한다.
