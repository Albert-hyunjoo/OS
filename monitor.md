# 모니터

## 모니터에 대해서
* `모니터`는 세마포 다음 세대의 *high-level* 프로세스 동기화 도구
* 구조는 크게 `공유 자원`들과 `공유자원 접근 함수`, 그리고 2개의 `Queue`로 형성되어 있다.
* 공유자원 접근함수에는 **최대 1개**의 쓰레드만 진입이 가능하며,
* 두 개의 `Queue`는 각각 `배타 동기`와 `접근 동기` 목적으로 `프로세스`를 막거나 꺠운다.

### 자바 모니터와 접근 함수
* `배타동기` 구현 목적의 함수는 `synchronized` 형태로 한다.
* `조건동기` 구현 목적의 함수는 `wait()`, `notify()`, `notifyAll()`이다.
* 굳이 여러 개의 `메타포어`를 사용할 필요 없이 몇 개의 `모니터`로 자원 공유가 가능하다.

## 전통적 동기화 예제의 변형

### 예금 인출의 예제
#### 일반적인 해결법
```java
synchronized void deposit (int amt) {
    int temp = balance + amt;
    System.out.print("+");
    balance = temp;

synchronized void withdraw (int amt) {
    int temp = balance - amt;
    System.out.print("-");
    balance = temp;
}
int getBalance() {
    return balance;
}
```
#### 항상 입금 먼저 실시
```java
synchronized void deposit (int amt) {
    int temp = balance + amt;
    System.out.print("+");
    balance = temp;
    notify();    
}
synchronized void withdraw (int amt) {
    while (balance < 0) {
        try {
        wait();
        } catch InterruptedException (e) {}
    }
    int temp = balance - amt;
    System.out.print("-");
    balance = temp;
}
 
int getBalance() {
    return balance;
}
```
#### 항상 출금 먼저 실시
```java
synchronized void deposit (int amt) {
    while (balance <= 0) {
        try {
        wait();
        } catch InterruptedException (e) {}
    }
    int temp = balance + amt;
    System.out.print("+");
    balance = temp;   
}
synchronized void withdraw (int amt) {
    int temp = balance - amt;
    System.out.print("-");
    balance = temp;
    notify();
}
 
int getBalance() {
    return balance;
}
```
#### 번갈아가면서 하기 (p-c-p-c)
```java
boolean p_turn = true;
synchronized void deposit (int amt) {
    int temp = balance + amt;
    System.out.print("+");
    balance = temp;
    notify(); // wait하기 전에 child를 깨워서 돌도록 한다   
    p_turn = False; // p_turn이 false가 되어야 withdraw가 돈다
    try {
    wait();
    } catch InterrruptedException (e) {}
}

synchronized void withdraw (int amt) {
    while (p_turn) {
        try {
        wait();
        } catch InterruptedException (e) {}
    int temp = balance - amt;
    System.out.print("-");
    balance = temp;
    p_turn = True;
    notify();
}
 
int getBalance() {
    return balance;
}
```
### 배고픈 철학자 문제

```java
class Chopstick {
  private boolean inUse = false; // # inUse라는 변수를 생성
  synchronized void acquire() throws InterruptedException {
    while (inUse) // 젓가락이 사용중일 떄
      wait(); // wait()로 막은 다음
    inUse = true; // inUse는 True로 유지
  }
  synchronized void release() {
    inUse = false; // release()하면 inUse = false하고
    notify(); // 다른 철학자 thread에 notify(); 해서 깨우도록!
  }
}
  ```