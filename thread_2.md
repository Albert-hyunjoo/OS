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
```latex
x^2 + y^2 = 3
```