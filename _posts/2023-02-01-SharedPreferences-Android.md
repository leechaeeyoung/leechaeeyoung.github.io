---
layout: post
title: "[Android] SharedPreferences"
categories: [Android]
tags: [Android, Kotlin, SharedPreferences]
description: android studio with kotlin, SharedPreferences에 관해 자세히 살펴보기 
date: 2023-02-01
---

# SharedPreferences
 - `sharedPreferences`: 데이터를 장치의 파일에 key,value로 저장
 -  파일에 대한 기본 설정 데이터를 읽고 쓰고 관리 가능하다
- 저장된 인스턴스 상태의 데이터는 세션 활동간 유지
- 사용자 세션간 유지 가능, 앱을 재부팅해도 sharedPreferences는 유지된다.

### 실습
- EditText에서 text입력 후 화면을 닫았다 열어도 text가 그대로 유지될 수 있도록하는 실습


1. viewBinding 사용: `build.gragle`설정
    {% highlight kotlin %}
   android{ 
     //뷰 바인딩 viewBinding{
    enable = true
     }
    }
    {% endhighlight %}

2. `SaveData메소드`: 객체를 edit()사용해 수정모드사용, put메소드롤 key값 지정 후 apply()로 데이터 저장
    {% highlight kotlin %}
    private fun saveData(){
        val pref = getSharedPreferences("pref",0)
        val edit = pref.edit()  // 수정모드
        // 두번째는 데이터 존재하지 않을 때 대치값 
        edit.putString("name",binding.etHello.text.toString())
        edit.apply()    // 저장
    }
    {% endhighlight %}

3. `loadData메소드`: getShared~메소드를 통해 key값 지정
    {% highlight kotlin %}
    private fun loadData(){
        val pref = getSharedPreferences("pref",0)
        // 첫번째 인자 key, 두번째 인재는 존재하지 않을 경우의 값
        binding.etHello.setText(pref.getString("name",""))
    }
    {% endhighlight %}

4.`Destory()`: 프로그램 종료시점
    {% highlight kotlin %}
    override fun onDestroy(){
        super.onDestroy()
        //저장된 데이터를 저장
        saveData()
    }
    {% endhighlight %}