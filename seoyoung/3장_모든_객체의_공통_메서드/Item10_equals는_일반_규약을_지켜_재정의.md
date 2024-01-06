# 3장. 모든 객체의 공통 메서드

- Object에서 final이 아닌 메서드(equals, hashCode, toString, clone, finalize) 는 모두 재정의(overriding)를 염두에 두고 설계된 것이라 재정의 시 지켜야 하는 일반 규약이 명확히 정의되어 있다.
- final이 아닌 Object 메서드들을 언제 어떻게 재정의해야 하는지를 알아보자.

## 10. equals는 일반 규약을 지켜 재정의하라

- equals 메서드는 재정의하기 쉬워 보이지만 곳곳에 함정이 도사리고 있어서 자칫하면 끔찍한 결과를 초래..

### [재정의하지 않는 상황]
- 각 인스턴스가 본질적으로 고유하다.
  * 값을 표현하는 게 아니라 동작하는 개체를 표현하는 클래스가 여기 해당한다.
  * Thread가 좋은 예로, Object의 equals 메서드는 이러한 클래스에 딱 맞게 구현되었다.


- 인스턴스의 '논리적 동치성(logical equality)'을 검사할 일이 없다.
  - 예컨대 <span style="color:#F46666;">java.util.regex.Pattern</span>은 equals를 재정의해서 두 Pattern의 인스턴스가 같은 정규표현식을 나타내는지를 검사하는, 즉 논리적 동치성을 검사하는 방법도 있다.

- 상위 클래스에서 재정의한 equals가 하위 클래스에도 딱 들어맞는다.
  - 대부분의 Set 구현체는 AbstractSet이 구현한 equals를 상속받아 쓰고, List 구현제들은 AbstractList로 부터, Map 구현체들은 Map 구현체들은 AbstractMap으로부터 상속받아 그대로 쓴다.

- 클래스가 private이거나 package-private이고 equals 메서드를 호출할 일이 없다.
  - equals 메서드가 호출되는 일이 없다면 재정의할 필요가 없다.

### [재정의해야 하는 상황]
- 객체 식별성(object identity; 두 객체가 물리적으로 같은가)이 아니라 논리적 동치성을 확인해야 하는데, 상위 클래스의 equals가 논리적 동치성을 비교하도록 재정의하지 않았을 때
- 주로 값 클래스들이 여기 해당한다.
- 값 클래스란 Integer와 String처럼 값을 표현하는 클래스를 말한다.
- 두 값 객체를 equals로 비교하는 프로그래머는 객체가 같은지가 아니라 값이 같은지를 알고 싶어 할 것.

```
- 반사성(reflexivity): null이 아닌 모든 참조 값 x에 대해, x.equals(x)는 ture다.
- 대칭성(symmetry): null이 아닌 모든 참조 값 x, y에 대해, x.equals(y)가 true면 y.equals(x)도 true다.
- 추이성(transitivity): null이 아닌 모든 참조 값 x, y, z에 대해, x.equals(y)가 true이고 y.equals(z)도 true면 x.equals(z)도 true다.
- 일관성(consistency): null이 아닌 모든 참조 값 x, y에 대해, x.equals(y)를 반복해서 호출하면 항상 true를 반환하거나 항상 false를 반환한다.
- null-아님: null이 아닌 모든 참조 값 x에 대해, x.equals(null)은 false다.
```
### 양질의 equals 메서드 구현 방법
1. == 연산자를 사용해 입력이 자기 자신의 참조인지 확인한다.
2. instanceof 연산자로 입력이 올바른 타입인지 확인한다.
3. 입력을 올바른 타입으로 형변환한다.
4. 입력 객체와 자기 자신의 대응되는 '핵심' 필드들이 모두 일치하는지 하나씩 검사한다.

