## Item64.객체는 인터페이스를 사용해 참조하라

### 적합한 인터페이스만 있다면 매개변수뿐 아니라 반환값, 변수, 필드를 전부 인터페이스 타입으로 선언하라.
- 객체의 실제 클래스를 사용해야 할 상황은 '오직' 생성자로 생성할 때뿐이다.

AnalysisTargetDocument
FileAnalysisTargetDocument, UrlAnalysisTargetDocument << AnalysisTargetDocument

```java
//좋은 예. 인터페이스를 타입으로 사용했다.

Set<Son> sonSet = new LinkedHashSet<>();
```

```java
//나쁜 예. 클래스를 타입으로 사용했다.
LinkedHashSet<Son> sonSet = new LinkedHashSet<>();
```

### 인터페이스를 타입으로 사용하는 습관을 길러두면 프로그램이 훨씬 유연해질 것이다.

### HashMap
- 순서 보장 없음 : "HashMap"은 키-값 쌍을 저장할 때 순서를 보장하지 않습니다. 즉, 맵에 요소를 추가한 순서와 순회할 때의 순서가 다를 수 있습니다.
- 속도가 더 빠름 : 일반적으로 'HashMap'은 'LinkedHashMap'보다 약간 더 빠른 접근 속도를 가집니다.

### LinkedHashMap
- 삽입 순서 보장 : 'LinkedHashMap'은 요소를 추가한 순서대로 순회합니다. 따라서 순서가 중요하다면 'LinkedHashMap'을 사용하는 것이 좋습니다.
- 약간 느린 접근 속도 : 'LinkedHashMap'은 삽입 순서를 유지하기 위해 추가적인 데이터 구조를 사용하기 때문에 'HashMap'보다 약간 느린 접근 속도를 가집니다.

### EntrySet을 이용하여 'for'문을 돌릴 계획이라면, 순회 순서가 중요하다면 'LinkedHashMap'을 사용하는 것이 좋습니다.
### 순회 순서가 중요하지 않고 성능 최적화에 더 중점을 두고 싶다면 'HashMap'을 사용하는 것이 좋습니다.

### 적합한 인터페이스가 없는 부류
1. 클래스 기반으로 작성된 프레임 워크가 제공하는 객체들
- ex) java.io 패키지의 OutputStream, InputStream, Reader, Writer 등

2. 인터페이스에는 없는 특별한 메소드를 제공하는 클래스들
- ex) PriorityQueue : Queue 인터페이스에는 없는 comparator 메서드를 제공한다. 클래스 타입을 직접 사용하는 경우는 이런 추가 메서드를 꼭 사용해야 하는 경우로 최소화해야한다.
