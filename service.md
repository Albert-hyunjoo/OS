## CPU 프로텍션
* 한 프로세스가 CPU 자원을 독점하는 경우 `while n = 1`
* 이 경우에는 `Timer` 를 활용해서 한 프로그램이 자원 독점하는 걸 막는다.
* `Timer Interrupt`로 인해서 `ISR`로 넘어간 경우에는 OS가 이를 체크한다.

# 운영체제의 서비스

## 프로세스의 관리
* 프로세스는 **현재 메모리 내**에서 돌아가는 프로그램의 통칭이다.
* 프로세스의 **주요한 기능**은 다음과 같다 :
    * `creation` 과 `delete`: 프로세스의 생성과 소멸
    * `suspend`와 `resume`: 프로세스의 활동 및 일시정지
    * `IPC`: 프로세스 간의 통신
    * `synchronization`: 프로세스 간의 동기화 -- 순서의 배열
    * `deadlock handling`: 교착 상태의 처리

## 메인 메모리 관리
*  주기억 장치의 **주요한 기능**은 다음과 같다:  
    * `(de)allocation`: 프로세스의 메모리 공간 할당 및 회수    
    * 메모리가 어느 프로세스에 할당되었는지 추적 및 감시를 실시
    * `virtual memory`: 실제 메인 메모리를 크게 쓸 수 있는 방법

## 메인 메모리 관리
*  주기억 장치의 **주요한 기능**은 다음과 같다:
    * `(de)allocation`: 프로세스의 메모리 공간 할당 및 회수
    * 메모리가 어느 프로세스에 할당되었는지 추적 및 감시를 실시
    * `virtual memory`: 실제 메인 메모리를 크게 쓸 수 있는 방법 (MM는 SM에 비해 용량 SMALL)

## 파일 관리
* HDD같은 보조 기억 장치는 보통 Track, sector를 head가 읽는 형식
* 이 디스크를 파일이라는 논리적 관계로 볼 수 있게 해주는 역할
* 파일 관리자로서의 OS의 역할은 다음과 같다:
    * `file creation and deletion`: 파일의 생성 및 삭제
    * 디렉토리 (혹은 폴더) 의 생성 및 삭제 
    * 기본적인 동작 지원 : `open`, `close`, `read`, `write`, `create`, `delete`
    * track/sector 간의 매핑 `mapping`
    * 백업의 관리

## 보조기억 장치의 관리
* 하드 디스크, 플래시 메모리 등의 용량 관리
* 보조기억 장치의 관리자로서 OS의 역할은 다음과 같다:
    * 빈 공간의 관리 (free space management)
    * 저장 공간의 할당 (storage allocation)
    * 디스크 스케쥴링 (disk scheduling) -- 헤드 최적화 등

## I/O (입출력) 관리
* 장치의 드라이브 (device drivers)
* 입출력 장치의 성능 향상
    * `buffering`: 메모리에다가 문서를 놓고 오기
    * `caching`: `buffering`과 유사
    * `spooling`: 보조 기억 장치 (HDD)를 중간에 놓아서 cpu가 다른 작업이 가능하도록

## 시스템 콜 (System Call)
* 소프트웨어나 하드웨어가 **OS 서비스**를 받기 위한 호츌
* 주요한 시스템 콜 (System Call)은 다음과 같다 :
    * process: `end`, `abort`, `load`, `execute`...
    * memory: `allocate`, `free`
    * file: `create`, `delete`, `open`, `close`....
    * device: `request`, `release`, `read`, `write`....
    * information: `get/set time`, `get/set time data`
    * allocation: `socket`, `send`, `receive`
