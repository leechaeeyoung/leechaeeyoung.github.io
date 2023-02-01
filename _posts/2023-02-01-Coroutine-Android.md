---
layout: post
title: "[Android] Coroutine"
categories: [Android]
tags: [Android, Kotlin, Coroutine]
description: android studio with kotlin, Coroutine에 관해 자세히 살펴보기 
date: 2023-02-01
---

# Coroutine

### 코루틴
 - 협력형 멀티태스킹 제공
 - 동시성 프로그래밍 지원
 - 비동기 처리 쉬움

#### 협력형 멀티태스킹

  - 연속으로 표시된 상태를 일부 제어권을 넘겨주거나 다른 코루틴이 작업 완료때까지 기다려야하는 시기
  - 흔히 모든 작업이 완료될때까지 기다려야하는 SubRoutine과 비교
  <img width="686" alt="스크린샷 2023-02-01 오후 1 30 36" src="https://user-images.githubusercontent.com/94501397/215950051-7e15a7c1-a539-4313-83df-f88eb90e9cae.png">
  - 작업루틴을 잠시 중단, 시작하기 위해 사용되는 키워드: `suspend`

  <br>

#### 동시성 프로그래밍 지원
 - 함수를 빠져나오고 진입하고 다시시작하는 것을 반복해 동시에 수행하는 것처럼 보여지는 프로그래밍기법
 - 동시에 작업 처리를 원한다면 `async함수 `사용
   - `async scope 함수`:
     - 동시에 수행 가능하게함

<br>

#### 비동기 처리
 - 특정 로직의 실행 끝날때까지 기다리지 않고 나머지 코드를 먼저 실행하는 처리 방식
  ```kotlin
  // 일반 callback 함수
    fun goCompany(person: Person){
        val 철수 = person

        wakeUp(잠든 철수){ 깨어난 철수 ->
            takeShower(깨어난 철수){ 씻는 철수 ->
                putOnDress(씻는철수){ 옷입는 철수 ->
                    getOnBus(옷입은 철수).....
    }}}}
  ```
  ```java
  // coroutine 사용
    suspend goCompany(person: Person){
        val 잠든 철수 = person
        try{
            val 깨어난 철수 = wakeUp(잠든 철수)
            val 씻는 철수 = takeShower(깨어난 철수)
            val 옷입는 철수 = putOnDress(씻는 철수)
            ...

            출근한 철수.doWork()
        } catch (e: Exception){
            실패했을때 처리
        }
    }
  ```
- 만약 wakeUp이 실행 후 goCompany를 빠져나가도 wakeUp함수 일이 끝날 시 goCompany로 돌아올 수 있다. (바동기 처리 장점)
 - - -

 ### 코틀린 함수
  - `Job`: 최소 가능한 작업단위 함수
    - `job.join()`: job완료될 때까지 기다림
    - `job.cancel()`: job을 취소
<br>

  - `Dispatchers`: 코루틴 실행에 사용하는 스레드를 결정
    - `Dispachers.Default`: 오래걸리는 작업할때 공유된 백그라운드 스레드 풀 사용
    - `Dispachers.IO`: 파일, 네트워크, API 콜 같은 상황에서 사용
    - `Dispachers.Main`: 메인스레드 작업에 사용
<br>  

- `GlobalScope`: 앱의 생명주기와 함께 동작, 실행 도중 별도의 생명주기 관리 필요없음(시작~종료 실행되는 코루틴에 적합)
- `CoroutineScope()`: 버튼눌러 다운로드, 서버에서 이미지 ... 필요할때 사용,종료가능함
- `ViewModelScope`: jetpack 아키텍처 뷰모델 사용시 ViewModel 인스턴스 사용위해 제공되는 스코프
<br> 

##### 코루틴 빌더
 - `launch`: 결과를 호출한쪽에 반환X, suspend가 아닌 일반함수 안에서 suspend 호출 or 코루틴 결과 필요없을 때 사용 
   - 상태관리 가능
 - `async`: 병행으로 실행되는 코루틴에 사용
   - 상태관리 + 결과 반환
 - `withContext`: IO사용시 사용됨, 부모코루틴의해 사용하던 컨텍스트와 다른 컨텍스트에서 코루틴 실행 가능