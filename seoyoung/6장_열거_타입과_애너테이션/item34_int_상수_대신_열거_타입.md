## Item 34. int 상수 대신 열거 타입을 사용하라

열거(enum) 타입은 일정 개수의 상수 값을 정의한 다음, 그 외의 값은 허용하지 않는 타입이다.


### 정수 열거 패턴(int enum pattern)
```java
public static final int APPLE_FUJI = 0;
public static final int APPLE_PIPPIN = 1;
public static final int APPLE_GRANNY_SMITH = 2;

public static final int ORANGE_NAVEL = 0;
public static final int ORANGE_TEMPLE = 1;
public static final int ORANGE_BLOOD = 2;
```
- 정수 열거 패턴 기법에는 단점이 많다.
- 타입 안전을 보장할 수 없고, 표현력도 좋지 않다.
- 열거 패턴의 단점을 보완한 것이 바로 열거 타입이다.

```java
public enum Apple { FUJI, PIPPIN, GRANNY_SMITH }
public enum Orange { NAVEL, TEMPLE, BLOOD }
```
- 열거 타입 자체는 클래스이며, 상수 하나당 자신의 인스턴스를 하나씩 만들어 public static final 필드로 공개한다.
- 클라이언트가 인스턴스를 직접 생성하거나 확장 할 수 없으니 인스턴스들은 딱 하나씩 존재한다.
- 열거 타입은 컴파일타임 타입 안정성을 제공한다.

### 열거 타입 메서드, 필드 추가
```java
public enum Planet {
    MERCURY(3.302e+23, 2.439e6),
    VENUS(4.869e+24, 6.052e6),
    EARTH(5.975e+24, 6.378e6),
    MARS(6.419e+23, 3.393e6),
    JUPITER(1.899e+27, 7.149e7),
    SATURN(5.685e+26, 6.027e7),
    URANUS(8.683e+25, 2.556e7),
    NEPTUNE(1.024e+26, 2.477e7);
    
    private final double mass; // 질량(단위: 킬로그램)
    private final double radius; // 반지름(단위: 미터)
    private final double surfaceGravity; // 표면중력(단위: m / s^2)
    
    // 중력상수(단위: m^3 / kg s^2)
    private static final double G = 6.67300E-11;
    
    // 생성자
    Planet(double mass, double radius) {
        this.mass = mass;
        this.radius = radius;
        surfaceGravity = G * mass / (radius * radius);
    }
    
    public double mass() { return mass; }
    public double radius() { return radius; }
    public double surfaceGravity() { return surfaceGravity; }
    
    public double surfaceWeight(double mass) {
        return mass * surfaceGravity; // F = ma
    }
}
```
- 열거 타입 상수 각각을 특정 데이터와 연결지으려면 생성자에서 데이터를 받아 인스턴스 필드에 저장하면 된다.