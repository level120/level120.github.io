---
layout: post
title: Android에서 Firebase를 이용해 Push 알림을 받아보자 - 안드로이드 편
published: True
category: Python
tags: [python, android, kotlin, fcm, firebase]
comments: true
---

## 시작하기 전에

이 편은 다른 블로그를 참고해도 전혀 다를 것이 없으며, 기본 Base code는 [Noverish's Blog](http://noverish.me/blog/Android-Kotlin-Firebase-Push-Notification/)를 참고하였습니다.

---

### 1. 프로젝트 생성

![Create Project Img](/asset/img/kotlin_noti/1.png)

* `Application name`에 자신이 만드는 이름 설정합니다.
* `Company domain` 가급적이면 `domain` 의미를 찾아봐서 설정할 것을 권장합니다. 또한 알림 서비스를 찾을 때 키워드로도 쓰입니다.
* `Project location` 원하는 프로젝트 위치를 잡아줍니다.
* `Include Kotlin support`를 반드시 체크해 줍니다(여기서는 `Kotlin` 문법을 다룹니다).


![Create Project Img](/asset/img/kotlin_noti/2.png)

* `Phone and Tablet`에서 만들어질 앱의 안드로이드 최소버전을 지정해 줍니다(호환성).


![Create Project Img](/asset/img/kotlin_noti/3.png)

* 여기서는 편하게 `Empty Activity`로 설정하였습니다.


![Create Project Img](/asset/img/kotlin_noti/4.png)

* 완료


---

### 2. Firebase 프로젝트 생성

여기서는 기존 프로젝트에 추가하는 방향으로 설명합니다.
처음부터 시작하려면 [Noverish's Blog](http://noverish.me/blog/Android-Kotlin-Firebase-Push-Notification)를 참고하기 바랍니다.


![Create Project Img](/asset/img/kotlin_noti/5.png)

* [Firebase Console](https://console.firebase.google.com)에서 프로젝트 선택 후 `설정`으로 진입합니다.
* `google-services.json` 파일을 내려받습니다.


---

### 3. Android에서 Firebase 설정

![Create Project Img](/asset/img/kotlin_noti/6.png)

* 다운받은 파일을 `(Project_Root)/app` 아래로 이동합니다.


![Create Project Img](/asset/img/kotlin_noti/7.png)

* 좌측 화면을 `Project`로 바꾼 다음 위 그림과 같이 `google-services.json`이 올바르게 들어왔는지 확인합니다.


![Create Project Img](/asset/img/kotlin_noti/36.png)

* 아래에 있는 `gradle/build.gradle`을 열어 `classpath`를 추가합니다.
* 작성일 기준 최신버전은 `classpath 'com.google.gms:google-services:3.2.0'` 입니다.


![Create Project Img](/asset/img/kotlin_noti/9.png)

* 위쪽에 있는 `app/build.gradle`을 열어 `firebase-messaging`과 `apply` 구문을 추가합니다.
* 작성일 기준 최신버전은 `implementation 'com.google.firebase:firebase-messaging:11.8.0'`, `apply plugin: 'com.google.gmc:google-services'` 입니다.
* **[주의]** `SDK Manager`를 이용하거나 다른 들로그를 참고하면 `implementation 'com.google.firebase:firebase-core:15.0.0'`을 추가하라고 합니다만, 작성하게 될 경우 모든 버전이 꼬여버리는 점과 나중에 작성할 일부 `class` 코드들에 빨간 줄이 그이는 문제점이 있습니다. 따라서 이 코드를 확실이 알기 전까지는 작성하지 않는 것이 좋습니다.
* **[주의]** 이제 `compile` 키워드는 사용하지 않습니다. `implementation`으로 대체 되었습니다.
* 우측 상단의 `Sync Now`를 눌러 적용합니다.


---

### 4. Android에서 Firebase 용 코드 작성

![Create Project Img](/asset/img/kotlin_noti/10.png)

* `MainActivity`와 같은 위치에 코틀린용 파일을 새로 만듭니다(클래스 명 : `MyFirebaseInstanceIDService.kt`)
* `To Do ...` 아래에 `token`이 발급 받은 후 `DB`로 발급받은 `token`을 전달하는 코드를 추가할 수 있습니다.
* `onTokenRefresh()`는 키가 생성되는 경우에만 동작하며, 이러한 경우로는 앱 재설치가 해당될 수 있습니다.

```kotlin
import com.google.firebase.iid.FirebaseInstanceId
import com.google.firebase.iid.FirebaseInstanceIdService

class MyFirebaseInstanceIDService : FirebaseInstanceIdService() {
    override fun onTokenRefresh() {
        val refreshedToken = FirebaseInstanceId.getInstance().token
        println("Refreshed token: " + refreshedToken!!)
                                   
        // To Do Develop Code, example sending token
    }
}
```


![Create Project Img](/asset/img/kotlin_noti/11.png)
* `MainActivity`와 같은 위치에 코틀린용 파일을 새로 만듭니다(클래스 명 : `MyFirebaseMessagingService.kt`)
* `Notification` 수신 시 `onMessageReceived()`로 실행하게 됩니다.
* `sendNotification()`에서 알림 옵션을 자세하게 수정할 수 있습니다.
* **[주의]** `Forground`에서는 소리, 진동, 알림 다 잘 되는데 `Background`에서만 안될 수 있습니다. 이 문제를 해결하려고 아래 코드에 `PowerManager` 등을 추가하는 블로그들이 많은데 `Firebase Admin SDK`를 이용할 경우 작성할 필요가 없습니다. 자세한 내용은 서버편에서 다룹니다.

```kotlin
import android.app.NotificationManager
import android.content.Context
import android.media.RingtoneManager
import android.support.v4.app.NotificationCompat
import com.google.firebase.messaging.FirebaseMessagingService
import com.google.firebase.messaging.RemoteMessage

class MyFirebaseMessagingService : FirebaseMessagingService() {
    override fun onMessageReceived(remoteMessage: RemoteMessage?) {
        if (remoteMessage!!.notification != null) {
            sendNotification(remoteMessage.notification!!.body!!)
        }
    }

    private fun sendNotification(messageBody: String) {
        val defaultSoundUri = RingtoneManager.getDefaultUri(RingtoneManager.TYPE_NOTIFICATION)
        val notificationBuilder = NotificationCompat.Builder(this)
                .setSmallIcon(R.mipmap.ic_launcher_round)
                .setContentTitle("FCM Message")
                .setContentText(messageBody)
                .setSound(defaultSoundUri)

        val notificationManager = getSystemService(Context.NOTIFICATION_SERVICE) as NotificationManager
        notificationManager.notify(0, notificationBuilder.build())
    }
}
```


![Create Project Img](/asset/img/kotlin_noti/12.png)

* `AndroidManifest.xml`에서 다음 코드를 추가합니다.

```xml
<service
    android:name=".MyFirebaseMessagingService">
    <intent-filter>
        <action android:name="com.google.firebase.MESSAGING_EVENT"/>
    </intent-filter>
</service>

<service
    android:name=".MyFirebaseInstanceIDService">
    <intent-filter>
        <action android:name="com.google.firebase.INSTANCE_ID_EVENT"/>
    </intent-filter>
</service>
```

---

### 5. 선택사항과 실행

![Create Project Img](/asset/img/kotlin_noti/35.png)

* `MainActivity`에 아무것도 추가하지 않아도 동작합니다.
* `FirebaseMessaging.getInstance().subscribeToTopic("TOPIC_NAME")`을 지정하면 `token`없이 `TOPIC_NAME`로 발송 된 메세지 수신이 가능합니다.
* `logcat`에서도 `token`이 찍히지만 굳이 다시 알아내고 싶다면 `FirebaseInstanceId.getInstance().token`를 사용하면 됩니다.
