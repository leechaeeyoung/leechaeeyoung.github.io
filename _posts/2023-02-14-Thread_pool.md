---
layout: post
title: "Thread pool(스레드 풀)"
categories: [java]
tags: [Thread pool,Java,Server]
description: thread pool에 대해 알아보기
date: 2023-02-14
---

## Thread Pool (스레드 풀)
 - `thread pool`: 작업처링 사용되는 스레드를 제한된 개수만큼 정해놓고, 작업큐에 들어오는 작업을 하나씩 스레드가 맡아 처리하는 방식
 #### 스레드 풀을 사용하는 이유?
 -  프로그램 성능 저하를 방지하기 위해
 -  다수의 사용자 요청처리를 위해

### Thead Pool(스레드 풀) 생성
 `java.util.concurrent` 패키지에서 `Executors`클래스를 제공
 | 초기 스레드 수 | 코어 스레드 수 | 최대 스레드 수 |
 |------------|:-----------|:------------|
 |ExecutorService객체가 생성될 때 기본적으로 생성되는 스레드 수| 스레드 증가 후 사용되지 않는 스레드를 풀에서 제거할 대 최소한 유지할 수| 스레드풀에서 관리하는 최대 스레드 수|

 - Executors클래스의 두가지 메소드로 스레드풀 생성 가능하다.

 |method| 초기 스레드 수 | 코어 스레드 수 | 최대 스레드 수 |
 |------|:-----------|:------------|:-----------|
 |newCachedThreadPool()|0개|0개|Integer.MAX_VALUE|
 |newFixedThreadPool(int nThreads)|0개|nThreads|nThreads|

  - **CachedThreadPool()**
    - Thread 개수 > 작업 개수 -> new 스레드 -> 종료
    - 생성된 스레드가 60초간 아무일 하지 않으면 제거
    ```java
    ExecutorService executor = Executors.newChachedThreadPool();
    ```

  - **FixedThreadPool**
    - 존재하는 Thread 개수 < 작업량 -> Thread 생성
    - method가 놀아도 제제 x
    ```java
    ExecutorService executor = Executors.newFixedThreadPool(100); 
    // 스레드 풀 생성(스레드 수 100개)
    ```

  - **FixedThreadPool**
    - 직접 스레드 개수 설정 가능
    ```java
    ExecutorService myThread = new ThreadPoolExecutor(
        3, // 코어 스레드 수
        100, // 최대 스레드 수
        120, // 최대 유효시간
        TimeUnit.SECONDS  // 최대 유효시간 단위
        new SynchronousQueue<>()  // 작업 큐
    )
    ```
 - - -
 ### Thread Pool(스레드 풀) 종료
  스레드 풀에 속한 스레드는 main 스레드가 종료되어도 작업 처리위해 계속 실행상태로 남아있다.

  - **shutdown()**: void 리턴 타입, 현재 + 작업 큐에 대기하는 스레드 모두 종료
  - **shutdownNow()**: List 리턴 타입, 현재 스레드 작업중지 후 종료, 리턴 값 = 작업큐 스레드
  - **awaitTermination(long timeout, TimeUnit unit)**: boolean 리턴 타입, shutdown() 이후 시간 안 처리시 true, 아니면 false출력

  - - -
### Thread Pool(스레드 풀)에 작업 처리 요청
  - **Runnable - Callable, 작업의 단위**
    - 스레드 풀에 작업 처리 요청을 위해 `Runnable`, `Callable`인터페이스를 구현한 후 ExecutorService에 전달
    - 작업완료: Runnable - return값 x, Callable - return 값 o
 
  - **작업요청**
   - `execute()`: 작업처리 결과 반환x, 예외발생시 스레드 종료+제거, 다른 작업처리 위해 새로운 스레드 생성됨
   - `submit()`: 작업처리 결과 반환o, 예외발생시 다음작업위해 재사용, 주로 사용하는 작업메소드


### 참고
 <https://limkydev.tistory.com/55>
 <https://steady-coding.tistory.com/548>
 <https://hudi.blog/java-thread-pool/>
