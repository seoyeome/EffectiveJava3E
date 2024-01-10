## 아이템 45.스트림은 주의해서 사용하라

### 스트림 API
1. 스트림(stream)
2. 스트림 파이프라인(stream pipeline)
- 각 중간 연산은 스트림을 어떠한 방식으로 변환한다. 원소에 함수를 적용하거나 특정 조건을 만족 못하는 원소를 걸러낼 수 있다.

### 스트림은 잘못 사용하면 읽기 어렵고 유지보수도 힘들어진다.

### 스트림을 적절히 활용하면 깔끔하고 명료해진다.

```java
public class Anagrams {
    public static void main(String[] args) throws IOException {
        Path dictionary = Paths.get(args[0]);
        int minGroupSize = Integer.parseInt(args[1]);
        
        try (Stream<String> words = Files.lines(dictionary)) {
            words.collect(groupingBy(word -> alphabetize(word)))
                .values().stream()
                .filter(group -> group.size() >= minGroupSize)
                .forEach(g -> System.out.println(g.size() + ": " + g));
        }
    }
}
```

### 람다에서는 매개변수 이름을 잘 지어야 스트림 파이프라인의 가독성이 유지된다.
### char 값들을 처리할 때는 스트림을 삼가는 편이 낫다
### 기존 코드는 스트림을 사용하도록 리팩터링하되, 새 코드가 더 나아 보일 때만 반영!!!

- 코드 블록에서는 범위 안의 지역변수를 읽고 수정할 수 있다.
- 하지만 람다에서는 final 이거나 사실상 final인 변수만 읽을 수 있고, 지역변수를 수정하는건 불가능하다. -> 나 이거 몰라서 이번에 에러나씀
- 코드 블록에서는 Return 문을 사용해 메서드에서 빠져나가거나, break나 continue 문으로 블록 바깥의 반복문을 종료하거나 반복을 건너 뛸 수 있다.