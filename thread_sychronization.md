# 프로세스 (쓰레드) 동기화

## 임계구역 문제 발생
* 여러 `쓰레드`가 하나의 `공통변수`에서 **변수 변경/파일 작성/테이블 변경**할 떄 생기는 문제
* `high-Level` 레벨에서 `low-level` 레벨로 (**어셈블리**) 변경하는 과정에서 커맨드 사이에 **스위칭** 발생
* 그 과정에서 공통 변수에 여러번 접근하면 `데이터의 일관성`을 헤칠 우려가 발생
* 이를 해결하려면 `임계구간`에서 **상호배타**와 **유한된 진행/대기시간**이 보장되어야 한다.

## 프로세스의 동기화 (Sychronization)
* `임계구역 문제`를 해결함으로써 여러 번의 Trial에도 답이 바뀌지 않도록 한다.
* **쓰레드의 동기화**를 통해 프로세스의 실행 순서를 원하는 대로 제어하도록 해야 한다.
* `Busy Wait` 문제를 해결한다, 즉 다중 프로세스 (스레드)가 이용 권한을 획득하는 과정의 delay가 없도록!

## 대표적인 동기화 도구
* 동기화 도구에는 `세마포어`, `모니터` 등이 있다.
* `세마포`는 동기화 문제 해결을 위한 소프트웨어 도구
* `세마포`는 **정수형 변수**와 **두 개의 동작**으로 구성되어 있다.

### 세마포의 구조
```java
class Semaphore {
        int value; ## 정수형
           Semaphore (int value) {
           ...
           }
           void acquire() { ## Proberen (test) 
           }
           void release() { ## Verhogen (increment)
           }
}
```
### 세마포의 작동원리 (1): 일반적 사용 (Mutual Exclusion)
```java
# acquire()는 다음의 원리로 작동한다
# semaphore에서 정수 값 n은 임계구역이 진입할 수 있는 쓰레드의 수 (1 가정)
# 만약 acquire()가 작동되면 value는 -1이 되므로 0 -> 임계구역으로 진입
# 그 다음 들어오는 쓰레드는 정수 (0)-1 = -1이므로 semaphore의 queue에 갖힌다.
void acquire() {
        value --;
        if (value < 0) {
                add this process/thread to list;
                block;
        }
}
```
```java
# 임계구역 안의 process가 작업이 완료되면 release()를 실행시킨다.
# 이 경우에는 -1+1=0이 되므로 세마포어 큐에서 쓰레드를 제거한 다음에 
# wakeup으로 임계구역으로 진입해 임무를 수행한다.
void release() {
        value ++;
        if (value <= 0) {
                remove a process P from list;
                wakeup P;
        }
} 
```
### 세마포의 작동원리 (2): Ordering
|P1|P2|
|:---:|:---:|
|x|`sem.acquire()`|
|S1;|S2;|
|`sem.release()`|x|
(목적은 S1를 무조건 우선적으로 실행하도록 하는 것)
> (*sem.value = 0으로 가정한다.*)
> 1) **S1**이 자연스럽게 실행된다
> 2) `sem.acquire()`가 실행되면서 *0-1=-1*이므로 **S2**는 실행되지 않는다.
> 3) **S1**이 실행된 이후에 `sem.release()`가 실행되면서 *-1+1=0*이 되면서
> 4) S2가 `wakeup`되면서 자연스럽게 **S1->S2** 방식으로 설명이 된다.