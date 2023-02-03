---
layout: post
title: "Retrofit이란?"
categories: [Server]
tags: [Retrofit2,HTTP,Server]
description: Retrofit 자세히 살펴보기 
date: 2023-02-01
---

# Retrofit?
- RESTAPI 통신을 위해 구현된 라이브러리
- AsyncTask 없이 Background Thread에서 실행 -> callback 통해 Main Thread에서 UI 업데이트

### Retrofit의 장점
 - `빠른성능`: OKHTTP는 AsyncTask사용해 성능이 떨어짐
 - `간단한 구현`: 반복된 작업을 라이브러리에 넘겨 처리
   - HttpUrlConnection의 connection / input & outputstream / URL encoding 할당 작업 필요 x
   - okHttp의 Query String, Request/ Response 반복작업 필요 x

### Retrofit의 3가지 구성요소
 - `DTO(POJO)`: 'Data Transfer Object', 'Plain Old Java Object'형태의 모델, JSON 변환에 사용(`JSON 형태 모델 클래스`)
 - `interface`: 사용할 HTTP CRUD 동작(메소드)를 정의한 인터페이스
   - `CRUD`: Create, Read, Update, Delete
   - `HTTP Method`: GET, POST, PUT, DELETE
- `Retrofit.Builder 클래스`: interface를 사용할 인스턴스(baseUrl(URL)/Converter(변환기) 설정)

### 사용방법

1. `build.gradle`에 의존성 추가
    ```
    implementation 'com.squareup.retrofit2:retrofit:2.7.2'
    implementation 'com.squareup.retrofit2:converter-gson:2.7.2'
    implementation 'com.google.code.gson:gson:2.8.7'
    ```
2. `Manifest` 권한설정 추가
    ```xml
    <uses-permission android:name="android.permission.INTERNET"/>
    <uses-permission 
    android:name="android.permission.ACCESS_NETWORK_STATE"/>
    ```
3. JSON 형태 모델 클래스를 생성
    ```java
    data class GithubRepo (@SerializedName("name") val name: String)
    ```
4. 인터페이스 정의
    ```java
    interface GithubClient{
        @GET("/users/{user}/repos")
        fun reposForUser(@Path("user") user: String): List<GitHubRepo>
    }
    ```
5. Retrofit 인스턴스 생성
    ```java
    class MainActivity: AppCompatActivity(){
        override fun onCreate(savedInstanceState: Bundle?){
            super.onCreate(savedInstanceState)
            setContentView(R.layout.activity_main)

            // Request
            val builder: Retrofit.Builder = Retrofit.Builder()
                .baseUrl("http://api.github.com/")
                .addConverterFactory(GsonConverterFactory.create())
            
            val retrofit: Retrofit = builder.build()  // Retrofit 객체
            val client: GithubClient = retrofit.create(GithubClient::class.java)
            val call: Call<List<GithubRepo>> = client.reposForUser("leechaeeyoung") // 서버연동 시작

            // 서버로부터 response할때마다 enqueue메소드 실행
            call.enqueue(object: Callback<List<GithubRepo>>){
                override fun onFailure(call: Call<List<GitHubRepo>>, t: Throwable){
                    Log.e("debugTest","error:(${t.message})")
                }
                // response
                override fun onResponse(call: Call<List<GithubRepo>>, response: Response<List<GithubRepo>>){
                    val repos: List<GithubRepo>? = response.body()
                    var reposStr = ""

                    repos?.forEach{ it ->
                        reposStr += "$it\n" }
                    textView.text = reposStr // 화면에 띄우기
                }
            }
        }
    }
    ```
    