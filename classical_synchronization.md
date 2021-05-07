# 전통적 동기화 예제

## 생산자-소비자 문제
* **생산자**가 데이터를 생산하면 **소비자**는 그것을 소비하는 형태이다.
* `High-Level 언어`에서 코드를 작성한 뒤 `Compiler`에서 `컴파일링`을 통해서    
  `Assembly Language`를 **생산**한다면 이를 컴퓨터/서버 등에서 `Executable Code`로 **소비**한다.    
* 여기서 해결해야 할 이슈는 **Bounded Buffer**가 있다.
    * 생산된 데이터는 버퍼에 일단 우선적으로 저장이 되며 (속도 차이 등)
    * 현실 시스템에서 **버퍼 크기**는 유한하기에 생산자-소비자 간 괴리가 발생할 수 있다.
    > 생산자는 버퍼가 가득 차면 더 넣을 수가 없고, 소비자는 버퍼가 비면 뺄 수 없다.

### 생산자-소비자 문제의 문제 예시
```java
class Buffer { # Buffer 클래스를 생성
    int[] buf; # 여기서는 stack으로
    int size; # size 변수
    int count;
    int in;
    int out;

Buffer (int size) {
    buf = new int[size];
    this.size = size;
    count = in = out = 0;
}

void insert (int item) { # 특정 아이템을 insert하는 기능
    while (count == size) { # 꽉 찰때까지
        buf[in] = item; # buf라는 stack의 in의 index에 item 삽입
        in = (in+1) % size # (in+1) % size로 순회하면서 들르기
        count ++; # 안의 개수는 늘어남
}

void remove() { 특정 out 위치의 item을 제거
    while (count == 0) { 안의 count가 0이 될 때까지
        int item = buf[out]; # buf stack에서 out위치의 item을 지정
        out = (out + 1) % size; # 순회
        count --; # 카운트 하나씩 감소
        return item; # 아이템을 return하고
```
### 실제 실행하는 main 스크립트의 예시
```java
# Producer 생산자 예시
class Producer extends Thread { # 스레드 생성
    Buffer b;
    int N;
    Producer (Buffer b, int N) {
        this.b = b, this.N = N
    }
    public void run() {
        for (int i = 0, i < N, i++)
                    b.insert(i);
    }
}
```
```java
# Consumer 소비자 예시
class Consumer extends Thread { # 스레드 생성
    Buffer b;
    int N;
    Producer (Buffer b, int N) {
        this.b = b, this.N = N
    }
    public void run() {
        for (int i = 0, i < N, i++)
                    item = b.remove();
    }
}
```
```
# main 함수의 예시
# 여기서는 문제 발생 (why? count, buf[]의 동시 업데이트 과정)
# 이를 해결하기 위해서는 count, buf[]에 대한 동시 업데이트가 필요하다
# 상호배타, 세마토 mutex.value = 1을 활용한다

class Test {
    public static void main(String[] arg) {
        buffer b = new Buffer(100);
        Producer p = new Producer(b, 10000);
        Consumer c = new Consumer(b, 10000);
        p.start();
        c.start();
        try {
               p.join();
               c.join();
        } catch (InterruptedException e) {}
        System.out.println("Number:" + b.count);
    }
}
```
### 임계구역 해결을 위해 코드를 추가
```java
void insert (int item) { # 특정 아이템을 insert하는 기능
    while (count == size) { # 꽉 찰때까지
        try {
        mutex.acquire();
        ######## critical area ##########
        buf[in] = item; # buf라는 stack의 in의 index에 item 삽입
        in = (in+1) % size # (in+1) % size로 순회하면서 들르기
        count ++; # 안의 개수는 늘어남
        ######## critical area ##########
        mutex.release();
        } catch (InterrruptedException e) {}
}

void remove() { # 특정 out 위치의 item을 제거
    while (count == 0) { # 안의 count가 0이 될 때까지
        try {
        mutex.acquire()
        ########## critical area ############
        int item = buf[out]; # buf stack에서 out위치의 item을 지정
        out = (out + 1) % size; # 순회
        count --; # 카운트 하나씩 감소
        ########## critical area ############
        mutex.release();
        return item; # 아이템을 return
        } catch (InterruptedException e) {}
        return -1; # 예외인 경우
```
```java
# 추가적으로 semaphore를 import해야 함!

import java.util.concurrent.Semaphore;

class Buffer { # Buffer 클래스를 생성
    int[] buf; # 여기서는 stack으로
    int size; # size 변수
    int count;
    int in;
    int out;
    Semaphore mutex; # Semaphore가 추가
    
Buffer (int size) {
    buf = new int[size];
    this.size = size;
    count = in = out = 0;
    mutex = new Semaphore(1); # 초깃값 1로 설정
}
```
### Busy-wait 문제를 해결
* 기존의 **mutex 싱글 세마포어**와 **무한 루프**는 `임계구역` 문제를 해결할 수 있다.
* 하지만 계속 무한 루프를 도는 과정에서 **심각한 속도의 손실** 발생 (활동을 하지 않음에도 계속 작동)
* 새로운 세마포어인 `full`, `empty`를 추가해서 다음과 같은 알고리즘으로 이를 최소화
> 1) 처음은 아무것도 없으므로 크기가 size인 버퍼를 준비한다.
> 2) `empty (초깃값은 size -- 몇개가 비었나?).acquire()`를 실행해서 생산자만 critical area로
> 3) 생산자는 produce를 실시하고 나오면서 `full (초깃값 0 -- 얼만큼 찼나?).release()`로 wakeup
> 4) 소비자는 `full.acquire()`를 통해 임계 구역으로 진입 후 소비를 하고 empty.release()로 생산자 깨움
```java
# 추가적으로 semaphore를 import해야 함!
# Buffer 클래스에 full, empty를 추가하는 걸로

import java.util.concurrent.Semaphore;

class Buffer { # Buffer 클래스를 생성
    int[] buf; # 여기서는 stack으로
    int size; # size 변수
    int count;
    int in;
    int out;
    Semaphore mutex, full, empty; # Semaphore가 추가
}

Buffer (int size) {
    buf = new int[size];
    this.size = size;
    count = in = out = 0;
    mutex = new Semaphore(1); # 초깃값 1로 설정
    full = new Semaphore(0); # full의 초깃값은 0
    empty = new Semaphore(size); # empty의 초깃값은 size
}
```
```java
void insert (int item) { # 특정 아이템을 insert하는 기능
        try {
            empty.acquire();
            mutex.acquire();
        ######## critical area ##########
            buf[in] = item; # buf라는 stack의 in의 index에 item 삽입
            in = (in+1) % size # (in+1) % size로 순회하면서 들르기
            count ++; # 안의 개수는 늘어남
        ######## critical area ##########
            mutex.release();
            full.release(); # 여기서 remove를 자극
        } catch (InterrruptedException e) {}
}

void remove() { # 특정 out 위치의 item을 제거
        try {
            full.acquire()
            mutex.acquire()
        ########## critical area ############
            int item = buf[out]; # buf stack에서 out위치의 item을 지정
            out = (out + 1) % size; # 순회
            count--; # 카운트 하나씩 감소
        ########## critical area ############
            mutex.release();
            empty.release();
            return item; # 아이템을 return
        } catch (InterruptedException e) {}
        return -1; # 예외인 경우
}
```


