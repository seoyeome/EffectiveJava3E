## 아이템 50. 적시에 방어적 복사본을 만들라

- 클라이언트가 불변식을 깨드리려 혈안이 되어있다고 가정하고 방어적으로 프로그래밍해야 한다.
- Date는 낡은 API 이므로 새로운 코드를 작성할 때 더 이상 사용하면 안된다.

```java
public period (Date start, Date end) {
    if (start.compareTo(end) > 0) {
        throw new IllegalArgumentException(start + "가 " + end + "보다 늦다.");
    }
    this.start = start;
    this.end = end;
}
public Date start() {
    return start;
}
public Date end() {
    return end;
}
//나머지 코드 생략
```
```java
Date start = new Date();
Date end = new Date();
Period p = new Period(start, end);
end.setYear(78); // p의 내부를 수정했다!
```
- Period 인스턴스 내부의 Date 필드들은 Period 인스턴스를 통해 접근할 수 없지만, Date는 가변이므로 클라이언트가 얼마든지 수정할 수 있다.
- 매개변수의 유효성을 검사하기 전에 방어적 복사본을 만들면 위의 문제를 해결할 수 있다.
- 매개변수가 제3자에 의해 확장될 수 있는 타입이라면 방어적 복사본을 만들 때 clone을 사용해서는 안된다.

### 방어적 복사에는 성능 저하가 따른다.

### 호출자에서 해당 매개변수나 반환값을 수정하지 말아야 함을 명확히 문서화하자.

