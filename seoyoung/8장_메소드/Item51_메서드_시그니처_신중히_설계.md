## 아이템 51. 메서드 시그니처를 신중히 설계하라

### 메서드 이름을 신중히 짓자
- 항상 표준 명명 규칙을 따라야한다.
- 같은 패키지에 속한 다른 이름들과 일관되게 짓는게 최우선 목표이다.
- 개발자 커뮤니티에서 널리 받아들여지는 이름을 사용
- 긴 이름을 피하자
- API 가이드 참조

### 편의 메서드를 너무 많이 ㅁ만들지 말자
- 메서드가 너무 많은 클래스는 익히고, 사용하고, 문서화하고, 테스트하고, 유지보수하기 어렵다.
- 클래스나 인터페이스는 자신의 각 기능을 완벽히 수행하는 메서드로 제공해야 한다.
- 아주 자주 쓰일 경우에만 별도의 약칭 메서드를 두자
- 확신이 서지 않으면 만들지 말자

### 매개변수 목록은 짧게 유지하자
- 4개 이하가 좋다.
- 같은 타입의 매개변수 여러 개가 연달아 나오는 경우가 특히 해롭다.

### 긴 매개변수 목록을 짧게 줄여주는 기술 세 가지
- 여러 메서드로 쪼갠다.
- 매개변수 여러 개를 묶어주는 도우미 클래스를 만든다.
- 객체 생성에 사용한 빌더 패턴을 메서드 호출에 응용한다.

### 매개변수의 타입으로는 클래스보다는 인터페이스가 더 낫다