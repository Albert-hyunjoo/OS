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
        return item; # 아이템을 return
        ########## critical area ############
        mutex.release()
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
