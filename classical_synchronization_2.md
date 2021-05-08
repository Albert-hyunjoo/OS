# 전통적 동기화 문제

## Readers-Writers Problem
* 공통 데이터 베이스에 접근할 때 생기는 동기화 문제에 해당한다.
* `Readers`는 **데이터베이스에 접근**하지만 **데이터는 건드리지 않으며**,     
  `Writer`는 **데이터베이스에 접근**하면서도 **데이터를 건드리는 경우**에 해당한다.
* 두 개를 모두 기존의 `세마포어` 형태로 제한하는 것은 매우 비효율적이므로 다른 방향으로 접근
> Read, Writing 섹션의 경우는 반드시 `임계 구역` 이내에서 벌어질 수 있게 하되,    
> `Writer`는 상호배타를 보장하지만, reader는 굳이 보장하지 않아도 된다.    
> `Writer`가 작성 중일 때는 `Reader`나 다른 `Writer`가 접근할 수 없으며,    
> 반대로 `Reader`가 접근 시에는 다른 `Reader`는 허용하되, `Writer`의 접근을 막는다.

### 구현의 예시
```java
# 뮤텍스는 총 2개를 구현한다
# 첫번째 뮤텍스는 reader가 writer가 활동하는지 여부를 검사하는 세마포어
# 두번째 세마포어는 writer가 하는 도중 다른 writer가 접근하는 것을 막는 세마포어

semaphore rw_mutex = 1; # writing 중 read 접근 방지
semaphore mutex = 1; # writing중 writing 접근 방지
int read_count = 0; # reader count

## 저자의 구조 ##

do {
    wait(rw_mutex); # 여기서 임계 구역에 진입
    ...
    /* 쓰기 작업을 여기서 수행한다*/
    ...
    signal(rw_mutex) # 작업 종료 후 rw_mutex에 lock 반납
}

## 독자의 구조 ##

do {
    wait(mutex); # read_count는 1개씩만 증가해야!
    read_count ++; # read_count가 1 증가한다
    if (read_count == 1) { # 만약 read_count가 증가했으면
        wait(rw_mutex); # 임계 구역 안에 writer 여부를 확인
    }
    signal(mutex); # 일단 read_count+1 됏으므로 mutex lock 반환 및 다른 reader 접근
    /* 읽기 작업을 수행한다 */
    wait(mutex) # read_count-1 위해 다시 lock
    read_count --;
    if (read_count == 0) { # reader가 없으면
        signal(rw_mutex);  # writer에 reader가 없다고 알림
    }
    signal(mutex); # lock을 반환한다
}
```
## 식사하는 철학자 문제 (Dining Philosopher)
* 5명의 철학자가 원탁에서 밥을 먹거나 생각을 하는 경우만 고려한다.
* 식사를 하기 위해서는 2개의 젓가락이 필요하며, 젓가락 한 개씩 철학자 사이에 있다.
* 문제는 양쪽의 철학자가 동시에 한 젓가락을 잡으면 영원히 식사를 할 수 없다는 점이다 (데드락)

### 데드락이란?
* 프로세스는 실행을 위해서 **여러 자원**을 필요로 한다 (프로세스가 CPU 점유 등)
* 문제는 **2+ 개**의 프로세스가 **하나의 자원**을 얻기 위해 **서로가 무한히 대기**를 하는 경우
> 데드락이 일어나는 **4가지 필요조건**은 다음과 같다 :    
> _하지만 이 4가지가 충족된다고 반드시 데드락이 일어나는 건 아니다._    
> 1) Mutual Exclusion (상호배타)    
> 2) Hold and Wait (보유 및 대기)    
> 3) No Preemption (비선점)
> 4) Circular Wait (환형대기)
* 어떤 자원이 어떤 프로세스에 할당되었는지는 `자원 할당도`로 확인할 수 있다.
* `자원 할당도` 상에서 원이 만들어지면 (**환형대기**) **교착상태의 필요조건**으로 짐작할 수 있다.

### 식사하는 철학자 문제 코드 예시
```java
import java.util.concurrent.Semaphore;

class Philosopher extends Thread {
      int id;
      Semaphore lstick, rstick;
  
      Philosopher (int id, Semaphore lstick, Semaphore rstick) {
          this.id = id;
          this.lstick = lstick;
          this.rstick = rstick;
}

public void run() { # 여기서 변경한다
    try {
    while (true) {
        # 만약 철학자의 id가 짝수면 lstick, rstick 순으로
        # id가 홀수면 lstick, rstick
        if (id % 2 == 0) {
            lstick.acquire();
            rstick.acquire();
        }
        else { 
            rstick.acquire();
            lstick.acquire();
        }
        eating();
        lstick.release();
        rstick.release();
        thinking();
    } catch (InterruptedException e) {}
}
```
