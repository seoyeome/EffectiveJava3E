## Item80. 스레드보다는 실행자, 태스크, 스트림을 애용하라.

- 작업 큐가 필요 없어지면 클라이언트는 큐에 중단을 요청할 수 있고, 큐는 남아 있는 작업을 마저 완료한 후 스스로 종료한다.
- 더 이상은 이런 코드를 작성하지 않아도 된다?
```java
// 작업 큐를 생성하는 방법
ExecutorService exec = Executors.newSingleThreadExecutor();
//실행자에 실행할 태스크를 넘기는 방법
exec.execute(runnable);
//실행자를 우아하게 종료시키는 방법
exec.shutdown();
```

### 실행자 서비스의 주요 기능
- 특정 태스크가 완료되기를 기다린다.
- 태스크 모음 중 아무것 하나(invokeAny) 또는 모든 태스크(invokeAll)가 완료되기를 기다린다.
- 실행자 서비스가 종료하기를 기다린다.(awaitTermination 메서드)
- 완료된 태스크들의 결과를 차례로 받는다.(ExecutorCompletionService 이용)
- 태스크를 특정 시간에 혹은 주기적으로 실행하게 한다.(ScheduledThreadPoolExecutor 이용)

### 실행자 서비스의 종류
- ThreadPoolExecutor -> 직접 스레드 풀을 관리할 수 있다.
- newCachedThreadPool -> 무거운 프로덕션 서버에는 좋지 못하다, 요청받은 태스크들이 큐에 쌓이지 않고 즉시 스레드에 위임돼 실행된다. 가용한 스레드가 없다면 새로 하나를 생성한다.
- newFixedThreadPool -> 크기가 고정된 스레드 풀을 생성한다. 큐에 작업이 가득 차면 풀에 스레드가 추가되지 않는다.

포크 조인 풀,,,,,,,