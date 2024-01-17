## Item79. 과도한 동기화는 피하라

### 응답 불가와 안전 실패를 피하려면 동기화 메서드나 동기화 블록 안에서는 제어를 절대로 클라이언트에 양도하면 안된다.
- 동기화된 영역 안에서는 재정의할 수 있는 메서드는 호출하면 안되며, 클라이언트가 넘겨준 함수 객체를 호출해서도 안된다.
- 동기화된 영역을 포함한 클래스 관점에서는 이런 메서드는 모두 바깥 세상에서 온 외계인........
- 그 메서드가 무슨 일을 할지 알지 못하며 통제할 수 없다.
- 동기화된 영역은 예외를 일으키거나, 교착상태에 빠지거나, 데이터를 훼손할 수도 있다.

### 79-1 잘못된 코드. 동기화 블록 안에서 외계인 메서드를 호출한다.
```java
public class ObservableSet<E> extends ForwardingSet<E> {
    public ObservableSet(Set<E> set) {
        super(set);
    }

    private final List<SetObserver<E>> observers = new ArrayList<>();

    public void addObserver(SetObserver<E> observer) {
        synchronized (observers) {
            observers.add(observer);
        }
    }

    public boolean removeObserver(SetObserver<E> observer) {
        synchronized (observers) {
            return observers.remove(observer);
        }
    }

    private void notifyElementAdded(E element) {
        synchronized (observers) {
            for (SetObserver<E> observer : observers) {
                observer.added(this, element);
            }
        }
    }

    @Override
    public boolean add(E element) {
        boolean added = super.add(element);
        if (added) {
            notifyElementAdded(element);
        }
        return added;
    }

    @Override
    public boolean addAll(Collection<? extends E> c) {
        boolean result = false;
        for (E element : c) {
            result |= add(element); // notifyElementAdded를 호출한다.
        }
        return result;
    }
}
```
### 은행 계좌 예시
```java
public class BankAccount {
    private double balance;

    public BankAccount(double balance) {
        this.balance = balance;
    }

    // 동기화된 입금 메서드
    public synchronized void deposit(double amount) {
        balance += amount;
    }

    // 동기화된 출금 메서드
    public synchronized void withdraw(double amount) {
        balance -= amount;
    }

    // 동기화된 잔액 조회 메서드
    public synchronized double getBalance() {
        return balance;
    }

    // 동기화된 이체 메서드
    public synchronized void transfer(BankAccount otherAccount, double amount) {
        this.withdraw(amount);
        otherAccount.deposit(amount); // 여기서 문제 발생 가능
    }
}
```
이전의 은행 계좌 예시에서 문제가 되는 시나리오는 다음과 같습니다:

1. 스레드 A가 BankAccount 객체의 transfer 메서드를 호출합니다. 
2. transfer 메서드 내에서 this.withdraw(amount)를 호출하는 동안, 스레드 A는 해당 객체에 대한 잠금을 보유합니다. 
3. transfer 메서드가 계속해서 otherAccount.deposit(amount)를 호출하려고 합니다. 
4. 만약 스레드 B가 이미 otherAccount 객체에 대한 잠금을 보유하고 있고, this 계좌의 withdraw나 transfer 메서드를 호출하려고 한다면, 스레드 A와 스레드 B는 서로가 보유한 잠금을 기다리게 됩니다. 
5. 이러한 상황은 교착 상태(deadlock)로 이어질 수 있으며, 이는 각 스레드가 서로의 실행을 영원히 기다리게 되는 상태를 의미합니다.

```
핵심 정리
교착상태와 데이터 훼손을 피하려면 동기화 영역 안에서 외계인 메서드를 절대 호출하지말자.
일반화해 이야기하면, 동기화 영역 안에서의 작업은 최소한으로 줄이자.
가변클래스를 설계할 때는 스스로 동기화해야 할지 고민하자.
멀티코어 세상인 지금은 과도한 동기화를 피하는 게 과거 어느 때보다 중요하다.
합당한 이유가 있을 때만 내부에서 동기화하고, 동기화했는지 여부를 문서에 명확히 밝히자.
```