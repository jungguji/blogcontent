---
layout: posts
title: Android Button tag AppCompat Error
date: 2021-05-03 21:54:53
tags:
    - Android
---

## 서론

어쩌다보니 학부생 시절 가장 싫어해 마지않던 안드로이드를 다시 공부하게 되었는데, 간단한 화면 하나 만들고도 실행이 안되서 삽질한 내용을 정리한다.
* * *

## 발생한 에러

```java
2021-05-03 22:06:26.170 18752-18752/com.jgji.memo E/AndroidRuntime: FATAL EXCEPTION: main
    Process: com.jgji.memo, PID: 18752
    java.lang.RuntimeException: Unable to start activity ComponentInfo{com.jgji.memo/com.jgji.memo.MainActivity}: android.view.InflateException: Binary XML file line #33 in com.jgji.memo:layout/activity_main: Binary XML file line #33 in com.jgji.memo:layout/activity_main: Error inflating class Button
        at android.app.ActivityThread.performLaunchActivity(ActivityThread.java:3449)
        at android.app.ActivityThread.handleLaunchActivity(ActivityThread.java:3601)
        at android.app.servertransaction.LaunchActivityItem.execute(LaunchActivityItem.java:85)
        at android.app.servertransaction.TransactionExecutor.executeCallbacks(TransactionExecutor.java:135)
        at android.app.servertransaction.TransactionExecutor.execute(TransactionExecutor.java:95)
        at android.app.ActivityThread$H.handleMessage(ActivityThread.java:2066)
        at android.os.Handler.dispatchMessage(Handler.java:106)
        at android.os.Looper.loop(Looper.java:223)
        at android.app.ActivityThread.main(ActivityThread.java:7656)
        at java.lang.reflect.Method.invoke(Native Method)
        at com.android.internal.os.RuntimeInit$MethodAndArgsCaller.run(RuntimeInit.java:592)
        at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:947)
     Caused by: android.view.InflateException: Binary XML file line #33 in com.jgji.memo:layout/activity_main: Binary XML file line #33 in com.jgji.memo:layout/activity_main: Error inflating class Button
     Caused by: android.view.InflateException: Binary XML file line #33 in com.jgji.memo:layout/activity_main: Error inflating class Button
     Caused by: java.lang.IllegalArgumentException: The style on this component requires your app theme to be Theme.AppCompat (or a descendant).
        at com.google.android.material.internal.ThemeEnforcement.checkTheme(ThemeEnforcement.java:243)
        at com.google.android.material.internal.ThemeEnforcement.checkAppCompatTheme(ThemeEnforcement.java:213)
        at com.google.android.material.internal.ThemeEnforcement.checkCompatibleTheme(ThemeEnforcement.java:148)
        at com.google.android.material.internal.ThemeEnforcement.obtainStyledAttributes(ThemeEnforcement.java:76)
        at com.google.android.material.button.MaterialButton.<init>(MaterialButton.java:229)
        at com.google.android.material.button.MaterialButton.<init>(MaterialButton.java:220)
        at com.google.android.material.theme.MaterialComponentsViewInflater.createButton(MaterialComponentsViewInflater.java:43)
        at androidx.appcompat.app.AppCompatViewInflater.createView(AppCompatViewInflater.java:123)
        at androidx.appcompat.app.AppCompatDelegateImpl.createView(AppCompatDelegateImpl.java:1551)
        at androidx.appcompat.app.AppCompatDelegateImpl.onCreateView(AppCompatDelegateImpl.java:1602)
        at android.view.LayoutInflater.tryCreateView(LayoutInflater.java:1059)
        at android.view.LayoutInflater.createViewFromTag(LayoutInflater.java:995)
        at android.view.LayoutInflater.createViewFromTag(LayoutInflater.java:959)
        at android.view.LayoutInflater.rInflate(LayoutInflater.java:1121)
        at android.view.LayoutInflater.rInflateChildren(LayoutInflater.java:1082)
        at android.view.LayoutInflater.inflate(LayoutInflater.java:680)
        at android.view.LayoutInflater.inflate(LayoutInflater.java:532)
        at android.view.LayoutInflater.inflate(LayoutInflater.java:479)
        at androidx.appcompat.app.AppCompatDelegateImpl.setContentView(AppCompatDelegateImpl.java:696)
        at androidx.appcompat.app.AppCompatActivity.setContentView(AppCompatActivity.java:170)
        at com.jgji.memo.MainActivity.onCreate(MainActivity.java:27)
        at android.app.Activity.performCreate(Activity.java:8000)
        at android.app.Activity.performCreate(Activity.java:7984)
        at android.app.Instrumentation.callActivityOnCreate(Instrumentation.java:1309)
        at android.app.ActivityThread.performLaunchActivity(ActivityThread.java:3422)
        at android.app.ActivityThread.handleLaunchActivity(ActivityThread.java:3601)
        at android.app.servertransaction.LaunchActivityItem.execute(LaunchActivityItem.java:85)
```

## 해결

결론부터 바로 말하면

```java

...

import androidx.appcompat.app.AppCompatActivity;

...

@SuppressLint(value = "StaticFieldLeak")
public class MainActivity extends AppCompatActivity {

    private MemoDatabase db;
    private List<MemoEntity> memoList;
    private RecyclerView recyclerView;
    private MemoDAO memoDAO;

...
```

되어 있던 코드를 [AppCompatActivity](https://developer.android.com/reference/androidx/appcompat/app/AppCompatActivity) 말고 [Activity](https://developer.android.com/reference/android/app/Activity.html?hl=ko)를 상속 받도록 변경해서 해결 했다.

```java
...

import android.app.Activity;

...

@SuppressLint(value = "StaticFieldLeak")
public class MainActivity extends Activity {

    private MemoDatabase db;
    private List<MemoEntity> memoList;
    private RecyclerView recyclerView;
    private MemoDAO memoDAO;

...
```

* * *

## 여담

둘의 정확한 차이는 모르겠지만, AppCompatActivity가 이전 안드로이드버전까지 커버해주는 클래스이고, Activity는 최신버전만 지원한다고 하는데... 이 문제랑 무슨 상관이 있는지 필자의 좁은 식견으로는 알 수가 없었기에 문제 해결에 도움을 받은 스택오버플로우의 글을 링크하며 글을 마무리 한다.  [You need to use a Theme.AppCompat theme (or descendant) with this activity](https://stackoverflow.com/questions/21814825/you-need-to-use-a-theme-appcompat-theme-or-descendant-with-this-activity)
* * *

## 참고 사이트

- [https://developer.android.com/reference/androidx/appcompat/app/AppCompatActivity](https://developer.android.com/reference/androidx/appcompat/app/AppCompatActivity)
- [https://developer.android.com/reference/android/app/Activity.html?hl=ko](https://developer.android.com/reference/android/app/Activity.html?hl=ko)
- [https://stackoverflow.com/questions/21814825/you-need-to-use-a-theme-appcompat-theme-or-descendant-with-this-activity](https://stackoverflow.com/questions/21814825/you-need-to-use-a-theme-appcompat-theme-or-descendant-with-this-activity)
