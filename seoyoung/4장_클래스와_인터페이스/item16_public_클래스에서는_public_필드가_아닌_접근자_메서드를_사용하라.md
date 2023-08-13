## 16. public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라

### 퇴보한 클래스
```java
class Point {
    public double x;
    public double y;
}
```
### 패키지 바깥에서 접근할 수 있는 클래스라면 접근자를 제공
```java
class Point {
    private double x;
    private double y;

    public Point(double x, double y) {
        this.x = x;
        this.y = y;
    }

    public double getX() { return x; }
    public double getY() { return y; }

    public void setX(double x) { this.x = x; }
    public void setY(double y) { this.y = y; }
}
```

```
public 클래스는 절대 가변 필드를 직접 노출해서는 안 된다.
불변 필드라면 노출해도 덜 위험하지만 완전히 안심할 수는 없다.
하지만 package-private 클래스나 private 중첩 클래스에서는 종종(불변이든 가변이든) 필드를 노출하는 편이 나을 때도 있다.
```