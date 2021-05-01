# 쓰레드 (2)

## 자바 스레드의 예시
* `맥`을 만드는 방법을 `java`를 통해 표현하면 다음과 같은 기능을 구현한다.
* `java.lang.Thread`를 통해서 `java` 내에서 쓰레드를 생성한다.
> `public void run()`: 새로운 맥이 흐르는 곳 (치환)    
> `void start()`: 쓰레드의 시작을 요청    
> `void join()`: 쓰레드가 마치기를 기다린다    
> `static void sleep()`: 쓰레드 잠자기 (ms)

## `java.lang.Thread`
* 쓰레드가 `thread.run()`을 통해서 `run()` 메소드를 실행하고 `run()`를 **치환**한다.
```java
class MyThread *extends* Thread {
    public void run() { // 여기서 치환이 발생한다.
    ...
    }
   }
```
## 프로세스의 동기화
* 동기화는 프로세스 (스레드) 작동 과정에서 **데이터의 일관성**을 유지하기 위해 행한다.
* 프로세스는 *독립적*이거나 *상호적*이며, 대부분의 프로세스는 *상호적*이다.
* 프로세스 간에 **상호작용**하거나 **자원을 공유**함으로써 다양한 **프로세스**가 작동된다.
* 프로세스의 동기화는 데이터의 *Consistency*를 유지하기 위한 concurrent한 과정이다.
* 다양한 프로세스가 **공통된 데이터**에 접근하는 과정을 **순서화** 시켜놓는다.
```java
class BankAccount { # 은행 계정을 만든다
    int balance; # balance를 만들고
    void deposit(int amount) { # deposit 기능을 추가
        balance = balance + amount;
    }
    void withdraw(int amount) { # withdraw 기능을 추가
        balance = balance - amount;
    }
    int getBalance() { # balance값을 출력
        return balance;
    }
   }

# child thread로 생성된다

class Child extends Thread { # extend로 thread 생성
    BankAccount b;
    Child (BankAccount b) {
        this.b = b;
    }
    public void run() {
        for (int i = 0; i < 100; i ++)
            b.withdraw(1000);
    }
   }

# Parent thread로 생성된다
# 여기서도 extend를 통해서 생성한다

class Parent extends Thread { # extend로 thread 생성
    BankAccount b;
    Parent (BankAccount b) {
        this.b = b;
    }
    public void run() {
        for (int i = 0; i < 100; i ++)
            b.deposit(1000);
    }
   }

# main test 함수를 실행한다
# 만약 deposit이나 withdraw 과정에서 시간 지연 목적으로 temp라는 공통변수에 같이 접근하면
# 여기서 공통변수에 따른 동시 업데이트 문제가 발생하여 올바른 결과가 나오지 않는다.

class Test { # extend로 thread 생성
    public state void main(String[] args)
    throws InterruptedException {
        BankAccount b = new BankAccount();
        Parent p = new Parent(b);
        Child c = new Child(b);
        p.start();
        c.start();
        p.join();
        c.join();
        System.out.println(
        ....));
    }
   }
```
