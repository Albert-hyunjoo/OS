# 프로세스의 관리 및 CPU 스케쥴링

## 전 주차의 복습
* `I/O 바운드`와 `CPU 바운드`는 각각 I/O를 소환하거나 CPU를 쓰는 정도이다.
* 즉, `job scheduler`는 `Main Memory`에서 두 바운드 사이의 균형을 유지해야 한다.
* 프로세스가 멈추거나 아무 일도 하지 않을 때, `medium-term scheduler`는 `swap-in`이나 `swap-out`을 통해서 CPU 효율을 관리한다.

## Context Switching (문맥전환)
* `문맥 전환 (context switching)`은 CPU가 `Ready Queue`안에 있는 프로세스를 **메인 메모리에서 실행시키는 순서**이다.
* `dispatcher`는 `scheduler`에서 정해진 순서대로 프로세스를 실질적으로 실행하는 방식이다.
    * 기존의 프로세스에 대한 **정보를 저장**하고 -- 다시 돌아가야 하니깐!
    * 다음 넘어갈 프로세스로 **정보를 변환**한다 (`NMU`, 프로세스 상태)
    * `dispatcher`는 `Context Switching Overhead`를 막기 위해 `assembly`같은 low-level 언어로 code
    
## CPU 스케쥴링 (CPU Scheduling)
* 스케쥴링은 크게 `선점`과 `비선점`으로 나뉘게 되는데, 그 차이는 **중간 개입 가능 여부**이다.
* CPU 스케쥴링의 평가 기준은 여러 가지로, 대표적인 것은 다음과 같다:
    * `CPU Utilization`: CPU의 이용률 (단위는 % -- 높을 수록 효율적)
    * `Throughout`: 시간당 처리율 (단위는 /s -- 높은 수록 효율적)
    * `Turnaround Time` : `Ready Queue`에서 배정 후에 끝나기 까지의 시간 (단위는 s)
    * `Waiting Time` : 대기 시간 (단위는 s)
    * `Response Time` : 어떤 요청 후의 응답 시간 (interactive system, 단위는 s)
    
### FCFS (First-Come, First-Served)

* 선착순 방식으로, Main Memory에 들어온 순으로 처리한다.
* 가장 간단하고 공정한 방식이지만, **항상 효율적인 것은 아니다.**
* 만약, P1 (24ms), P2 (3ms), P3(3ms)가 메인 메모리가 들어왔다고 가정할 때,

|프로세스|걸린 시간 (Bursted Time)|대기 시간|
|:---:|:---:|:---:|
|P1|24|0|
|P2|3|24|
|P3|3|27|
|Elapsed Time|**30**`3+3+24`|
|AWT| |**17**`(0+24+27)/3`

|프로세스|걸린 시간 (Bursted Time)|대기 시간|
|:---:|:---:|:---:|
|P3|3|0|
|P2|3|3|
|P1|24|6|
|Elapsed Time|**30**`3+3+24`|
|AWT| |**3**`(0+3+6)/3`

* `호위 효과 (Convoy Effect)`를 경계해야 한다!
  * `BT` 높은 프로세스가 **먼저 들어오면** 뒤의 프로세스의 **대기 시간이 길어**지는 현상
* `FCFS`는 **비선점적인 스케쥴링**으로, 하나가 다 끝나야 그 다음이 실행된다.

`
`
  