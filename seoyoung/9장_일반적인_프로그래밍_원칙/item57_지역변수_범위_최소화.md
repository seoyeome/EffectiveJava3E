## Item57. 지역변수의 범위를 최소화하라

### 지역변수의 유효 범위를 최소로 줄이면 코드 가독성과 유지보수성이 높아지고 오류 가능성이 낮아진다.
### 지역변수의 범위를 줄이는 가장 강력한 기법은 역시 '가장 처음 쓰일 때 선언하기'다.
### 거의 모든 지역변수는 선언과 동시에 초기화해야 한다.
- try-catch문은 이 규칙에서 예외다. try 블록에서 선언된 변수는 try 블록에서만 사용해야 한다.

### 코드57-1 컬렉션이나 배열을 순회하는 권장 관용구
```java
for (Element e : c) {
    ... // e로 무언가를 한다.
}
```
### 코드57-2 반복자가 필요할 때의 관용구
```java
for (Iterator<Element> i = c.iterator(); i.hasNext(); ) {
    Element e = i.next();
    ... // e와 i로 무언가를 한다.
}
```

### for 문을 사용하면 복사 붙여넣기 오류를 컴파일 타임에 잡아준다.
```java
for ( Iteratof<Element> i = c.iterator(); i.hasNext(); ) {
    for (Iterator<Element> j = c.iterator(); j.hasNext(); ) {
        Element e = i.next();
        ... // e와 j로 무언가를 한다.
    }
}
```
```java
for ( Iterator<Element> i2 = c2.iterator(); i.hasNext(); ) {
    Element e2 = i2.next();
    ... // e와 e2로 무언가를 한다.
}
```

- for 문 변수 유효 범위가 for 문 범위와 일치하여 똑같은 이름의 변수를 여러 반복문에서 써도 서로 아무런 영향을 주지 않는다.
- for문은 wihle 문보다 짧아서 가독성이 좋다
- 지역변수 범위를 최소화하는 마지막 방법 : 메서드를 작게 유지하고 한 가지 기능에 집중하는 것