---
layout: post
title: "[Android] Network"
categories: [Android]
tags: [Android, Kotlin, Internet]
description: android studio with kotlin, 네트워크 작업에 관해 자세히 살펴보기 
date: 2023-02-20
---

## Network 작업 실행
 ### 1) 네트워크에 연결
  애플리케이션에서 네트워크 작업하려면 매니페스트에 권한을 포함
  ```xml
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
  ```

  - **저장소로 네트워크 작업 캡슐화과정**
    ```kotlin
        interface UserService {
            @GET("/users/{id}")
            suspend fun getUser(@Path("id") id: String): User
        }
    ```
    네트워크 작업에 필요한 http 메소드, URL, 인수 및 응답 유형을 지정하는 인터페이스
    <br>

    ```kotlin
    class UserRepository constructor(private val userService: UserService) {
        suspend fun getUserById(id: String): User {
            return userService.getUser(id)
        }
    }
    ```
    데이터 저장 방식에 관한 변경사항을 저장소 클래스와 분리, `UserRepo`클래스에서 네트워크 작업은 실제로
    트리거되지 않는다. `코루틴, enqueue()`를 시용해 구현 가능하다.
    <br>

  - **구성 변경사항 유지(화면 회전, 재시동)**
    - `ViewModel`사용: 수명 주기고려, UI데이터 저장 관리
    ```kotlin
    class MainViewModel constructor(
            savedStateHandle: SavedStateHandle,
            userRepository: UserRepository
            ) : ViewModel() {
            private val userId: String = savedStateHandle["uid"] ?:
                throw IllegalArgumentException("Missing user ID")

            private val _user = MutableLiveData<User>()
            val user = _user as LiveData<User>

            init {
                viewModelScope.launch {
                    try {
                        // Calling the repository is safe as it will move execution off
                        // the main thread
                        val user = userRepository.getUserById(userId)
                        _user.value = user
                    } catch (error: Exception) {
                        // show error message to user
                    }

                }
            }
        }
    ```
- - -

 ### 2) 네트워크 사용 관리
 네트워크 리소스 사용을 세밀하게 제어하는 애플리케이션 작성 방법
  -많은 네트워크 작업: 데이터사용패턴(동기화빈도, wifi연결때 다운로드, 로밍 중 데이터사용여부)을 사용자가 제어할 수 있도록 사용자 설정을 제공해야함

 - **기기의 네트워크 연결 확인**
  와이파이 혹은 네트워크 연결을 사용하는 것에 중점
   - 네트워크 연결을 확인해주는 클래스
     - `ConnectivityManager`: 네트워크 연결 상태에 대한 쿼리에 답, 네트워크 연결 변경 시 알림
     - `Networkinfo`: 특정 유형(wifi,mobile)의 네트워크 인터페이스 상태 설명
     - wifi및 모바일 네트워크 연결 테스트 코드(네트워크 사용 가능여부)
     ```kotlin
     private const val DEBUG_TAG = "NetworkStatusEx"
     ...
     val connMgr = getSystemService(Context.CONNECTIVITY_SERVICE) as ConnectivityManager
     var isWifiConn: Boolean = false
     var isMobileConn: Boolean = false
     connMgr.allNetworks.forEach{ network -> connMgr.allNetworkInfo(network).apply{
        if(type==ConnectivityManager.TYPE_WIFI){
            iswifiConn = isWifiConn or isConnected
        }
        if(type==ConnectivityManager.TYPE_MOBILE){
            isMobileConn = isMobileConn or isConnected
        }
     }
     Log.d(DEBUG_TAG,"wifi connected: $isWifiConn)
     Log.d(DEBUG_TAG,"mobile connected: $isMobileConn)
     }
     ```

     - wifi및 모바일 네트워크 연결 테스트 코드(네트워크 인터페이스가 사용 가능여부)
        - `getActivityNetworkInfo()`: 처음 연결된 네트워크 인터페이스를 찾아 networkInfo 인스턴스 반환, null 반환
            ```kotlin
            fun isOnline(): Boolean{
                val connMgr = getSystemService(Context.CONNECTIVITY_SERVICE) as ConnectivityManager
                val networkInfo: NetworkInfo? = connMgr.activeNetworkInfo
                return networkInfo?.isConnected == true
            }
            ```
 - **네트워크 사용 관리**
  네트워크 엑세스 및 네트워크 사용관리를 지원하는 앱 작성 시 매니페스트에 올바른 권한과 인텐트 필터가 있어야한다.
   - manifest.xml 권한에 포함
     - `android.permission.INTERNET`: 애플리케이션에서 네트워크 소켓 열 수 있도록함
     - `android.permission.ACCESS_NETWORK_STATE`: 애플리케이션에서 네트워크에 관한 정보에 엑세스할 수 있게함
     - `android.intent.action.MANAGE_NETWORK_USAGE`: 데이터 사용량 제어 옵션을 제공
     
 - **환경설정 활동 구현**
  `SettingsActivity`는 `PreferenceActivity`의 서브 클래스, 사용자 지정 환경설정 제공
    - xml피드 항목에 관한 요약을 표시할지, 항목의 링크만 표시할지 설정
    - 네트워크 연결 가능할때, xml피드 다운로드? wifi 사용할때만 다운할지 설정
    - 사용자가 환경설정을 변경하면 `onSharedPreferenceChanged()`를 실행해 `refreshDisplay`를 참으로 설정
    ```kotlin
    class SettingsAcivity: PreferenceActivity(), OnSharedPreferenceChangeListener{
        override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        // load xml preference file
        addPreferencesFromResource(R.xml.preferences)
        }

        override fun onResume(){
            super.onResume()

            // register a listener whenever a key changes
            preferenceScreen?.sharedPreferences?.registerOnSharedPreferenceChangeListener(this)
        }

        override fun onPause(){
            super.onPause()

            // unnecessary system overhead
            preferenceScreen?.sharedPreferences?.unregisterOnSharedPreferenceChangeListener(this)
        }

        override fun onSharedPreferenceChanged(sharedPreferences: SharedPrefernces, key: String){
            // sets refreshDisplay to true so that when the user return to the main
            NetworkActivity.refreshDisplay = true
        }
    }