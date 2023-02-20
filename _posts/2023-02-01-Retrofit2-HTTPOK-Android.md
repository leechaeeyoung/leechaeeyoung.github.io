---
layout: post
title: "Retrofit2/ HTTPOK 차이"
categories: [Server]
tags: [Retrofit2,HTTPOK,HTTP,Server]
description: Retrofit2와 HTTPOK 차이 자세히 살펴보기 
date: 2023-02-01
---

# Retrofit 과 OKHTTP 차이
<br>

### 구조화된 URL, 매개변수 구성
#### Retrofit

   ```java
    Retrofit.Builder()
        .addConverterFactory(GsonConverterFactory.create())
        .baseUrl(BASE_URL)
        .build()
   ```
   - API 인터페이스에서 모든 쿼리 매개변수 정의
   ```java
   @GET(BASE_URL)
   fun hitCountCheckCall( @Query(PARAM_ACTION) action : String,
                      @Query(PARAM_FORMAT) format : String,
                      @Query(PARAM_LIST) list : String,
                      @Query(PARAM_SRSEARCH) srsearch : String)
    : Call<Model.Result>
   ```
   - 네트워크 호출하려면 API에 값을 제공하기만 하면된다
   ```java
   wikiApiServe.hitCountCheckCall(
            VALUE_QUERY, VALUE_JSON, VALUE_SEARCH, searchString
   )
   ```

#### OKHTTP
- 수동으로 요청해야함
```java
val request = Request.Builder().url("$BASE_URL$BASE_PATH/?"+
    "$PARAM_ACTION=$VALUE_QUERY&$PAARAM_FORMAT=$VALUE_JSON&"+
    "$PARAM_LIST=$VALUE_SEARCH&$PARAM_SRSEARCH=$searchString")
    .build()
```
- - -

## 결과처리
#### Retrofit
 - 통신 성공 시 객체의 바디를 분리 과정으로 변환 없이 바로 실행
  ```java
  Call?.enqueue(
    object : Callback<Model. Result>{
        override fun onFailure(call: Call<Model. Result>, t: Throwable){
            ...
        }
        override fun onResponse(call: Call<Model. Result>,
            response: Response<Model.Result>){
                response.body()?.let 
            }
        )
    }
  ```

### OKHTTP
 - 반환된 JSON 개체를 데이터 클래스로 바로 변환하지 못해, 별도의 처리 필요함
  ```java
  client.newCall(request).enqueue(
  object : okhttp3.Callback {
    override fun onFailure(call: okhttp3.Call, e: IOException) {
         ...
    }

    override fun onResponse(call: okhttp3.Call, 
          response: okhttp3.Response) {
            response.body()?.let { 
               val result = Gson().fromJson(it.string(), 
                            Model.Result::class.java)
               useResult(result) 
            }
        }
    )
  ```
  - - - 

## 바로 결과가 반환된 책임 바로 호출가능한지
#### Retrofit
 - 결과 반환시 메인스레드에 전달되기 때문에 자동으로 UI관련 결과값 사용가능
    ```kotlin
    override fun onResponse(call: CAll<Model.Result>,
            response: Response<Model.Result>{
                Toast.makeText(...).show()
            })
    ```

#### OKHTTP
 - enqueue 사용시 네트워크 호출이 자동으로 백그라운드에서 수행, 결과값 메인스레드 사용위해 runOnUiThread 없어짐
    ```kotlin
    override fun onResponse(call: okhttp3.Call,
        response: okhttp3.Response){
        runOnUiThread{
            Toast.makeText(...).show()
        }
    }
    ```
