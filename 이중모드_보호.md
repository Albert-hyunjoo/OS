# 이중모드 (Dual Mode)

## 이중모드의 위험성
* 한 사람이 여러 컴퓨터를 이용하거나, 여러 사람이 한 컴퓨터를 이용하는 모드
* 만약 한 사람이 실수한다면, 전체 시스템에 위험이 갈 수 있음 (i.e. STOP, HALT)
* 이를 방지하기 위해 **사용자 모드 (User)** 와 **관리자 모드 (Supervisor)** 로 나뉘게 됨
* STOP, HALT, TIMER 등 직접적인 명령은 Previlege Mode에서만 실행 가능

## 이중모드의 사용에 대해서
* 레지스터에는 32, 64 비트 등으로 기록이 되어 있는데, 여기에 *Monitor Bit*가 추가
* OS는 모니터 비트를 체크해서 사용자 모드인지, 관리자 모드인지를 구별 (Flag로 구별)
* 소프트웨어나 하드웨어가 인터럽트를 실행하면, Monitor Bit를 체크해서 관리자 모드에서 실행
    * HDD에 사용자 프로그램이 데이터를 **입력하거나 저장하는 경우**가 대표적인 예
* 사용자 소프트웨어가 특정 HDD에 접근하려면, 무조건적으로 OS의 ISR를 거쳐야 (Software Interrupter)
* 만약 ISR이 잘못된 명령으로 판단한다면, (i.e. 유저 모드에서의 STOP) 즉시 프로그램을 kill하는 형식으로 보호
* 운영체제를 부팅하거나 인터럽트가 발생할 때마다 사용자 모드와 관리자 모드를 왔다갔다 한다.
    * 운영체제 -- 관리자, 사용자 프로그램 -- 사용자, 인터럽트 -- 관리자, 끝나면 -- 사용

# 하드웨어 보호 (Hardware Protection)

## 입출력 장치 보호 (I/O protection)
* 사용자가 타 사용자의 **입출력 장치에 관여**하는 경우를 방지 **(프린터, HDD)**
* 이를 방지하기 위해 **IN/OUT을 Previleged Command로 간주**하여, **운영체제가 이를 대행**
  * 소프트웨어가 이를 요청하는 경우에는 Software Interrupter로 진행한다.
* 만약 사용자가 입출력 명령을 직접 내린 경우는, **Previleged Instruction Violation**

## 메모리 보호 (Memory Protection)
* 사용자 프로그램이 **타 프로그램의 정보 및 메모리**를 읽고 쓰거나, **OS의 ISR를 변경**하는 경우
* 이 경우에는 **MMU (Memory Management Unit)**를 두어서, 메모리 침범을 감시한다.
  * MMU는 프로그램 별로 **Base와 Limit**을 지정해서 소프트웨어가 **그 범위 안의 메모리만 접근 가능**하도록 강제하는 역할을 한다.
    [MMU 자세한 설명에 대해서 -- MMU의 원리와 BASE, LIMIT ](https://jhnyang.tistory.com/247)
* MMU의 설정 및 적용은 **특권 명령 (Previlege)** 으로 적용되며, OS만 이를 설정할 수 있다.

* 사용자 프로그램이 이를 거부한 경우에는 **Segment Violation**으로 간주

