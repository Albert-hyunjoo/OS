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
        signal(rw_mutex); # writer에 reader가 없다고 알림
    signal(mutex); # lock을 반환한다
```

