---
layout: post
title: Android에서 Firebase를 이용해 Push 알림을 받아보자 - 파이썬 서버 편
published: True
category: Python
tags: [python, android, python, fcm, firebase]
comments: true
---

## 시작하기 전에

이 편의 기본 Base code는 [Noverish's Blog](http://noverish.me/blog/Android-Kotlin-Firebase-Push-Notification/)를 참고하였습니다.

---

### 1. FCM 알림 전송 방법

제가 참고한 블로그를 기준으로 4개 설명이 되어 있는데 그 중 오류가 있어 다시 정리합니다.

1. Firebase Admin ~~Node.js~~ SDK 사용하기
2. FCM HTTP v1 API (새로 나온 HTTP Request 방법)
3. Legacy HTTP protocol (예전에 사용하던 HTTP Request 방법)
4. Legacy XMPP Protocol

첫 번째 부분은 `Node.js`만 언급이 되어 있는데, 사실 4가지를 지원합니다(`Node.js`, `Java`, `Python`, `Go`)

여기서 저는 1번의 `Python` 방법과 2번을 사용합니다.


---

### 2. FCM 설정

![Create Project Img](/asset/img/kotlin_noti/21.PNG)

* 이전과 다른 페이지 입니다. [Google Cloud Console](https://console.cloud.google.com/)
* 여기서 `프로젝트 ID`를 기억하세요.
* 이후 좌측에 있는 `API 및 서비스`의 `대시보드`로 진입합니다.


![Create Project Img](/asset/img/kotlin_noti/22.PNG)

* 상단의 `API 및 서비스 사용 설정`을 누릅니다.


![Create Project Img](/asset/img/kotlin_noti/23.PNG)

* 검색창에 `firebase cloud messaging`으로 검색한 뒤 `Firebase Cloud Messaging API`라고 적힌 것으로 들어갑니다.


![Create Project Img](/asset/img/kotlin_noti/24.PNG)

* 만약 여기에 `API 사용 설정됨`이 안되어 있다면 `관리` 버튼이 `사용`으로 되어 있을 겁니다.
* 그 버튼이 `사용 설정`으로 바꿔줍니다. `관리`인 분도 누르셔서 `대시보드` 진입까지 확인합니다.


![Create Project Img](/asset/img/kotlin_noti/25.PNG)

* `대시보드` 진입 확인


---

### 3. Firebase Admin SDK 사용하기

![Create Project Img](/asset/img/kotlin_noti/26.PNG)

* `FCM Console`의 설정에서 `서비스 계정`으로 진입합니다.
* `Firebase Admin SDK` 화면에서 해당 플랫폼 선택 후 `새 비공개 키 생성`을 눌러줍니다.


![Create Project Img](/asset/img/kotlin_noti/27.PNG)

* 키 생성한 자료는 안내문대로 공개 저장소에 놔두지 맙시다.


![Create Project Img](/asset/img/kotlin_noti/28.PNG)

* 이제 서버를 만져야 합니다. 저는 테스트 겸이라 `Windows 10` 내에 있는 `Ubuntu`를 사용했습니다.
* `apt-get install python-pip python-dev python python3-pip python3-dev python3`가 기본적으로 설치되어 있어야 합니다.
* `pip install --upgrade google-api-python-client`를 입력해 설치합니다(`HTTP v1` 방식 사용 시 설치).
* `pip install requests`를 입력해 설치합니다(`HTTP v1` 방식 사용 시 설치).
* `pip install firebase-admin`을 입력해 설치합니다(`Firebase Admin SDK` 사용 시 설치).


---

### 4. 메세지 날리기

#### HTTP v1 방식

```py
#!/usr/bin/python
#-*- coding: utf-8 -*-
import requests
import json

from oauth2client.service_account import ServiceAccountCredentials

scopes = ['https://www.googleapis.com/auth/firebase.messaging']

def _get_access_token():
    credentials = ServiceAccountCredentials.from_json_keyfile_name('service-account.json', scopes)
    access_token_info = credentials.get_access_token()
    return access_token_info.access_token


project_id = "PROJECT_ID"
device_token = "ANDROID_CLIENT_TOKEN"

url = "https://fcm.googleapis.com/v1/projects/" + project_id + "/messages:send HTTP/1.1"

payload = {
    "message": {
        "token":device_token,
        "notification": {
            "body": "This is an FCM notification message!",
            "title": "FCM Message",
        }
    }
}

headers = {
  "Authorization": "Bearer " + _get_access_token(),
  "Content-Type": "application/json; UTF-8",
}

r = requests.post(url, data=json.dumps(payload), headers=headers)
print r.content
```

* `HTTP v1` 방식이므로 `header`와 `body`로 나눠져 갑니다.
* 채워야 할 곳은 4곳이며 이 중 필수인 곳은 `project_id`, `device_token` 입니다.
* `project_id`는 좀 전에 외워두라고 한 그 값이며, `device_token`은 안드로이드 앱의 `token`값 입니다.
* 이 방식을 이용 할 경우, 스마트폰이 놀고 있을 경우 알림이 수신되더라고 소리, 진동, 화면 켜짐이 나타나지 않습니다.
* `payload` 부분을 수정하면 된다고 하는데 아직 해결책을 찾기 못했으며, 심지어 이 방식은 단일 기기 송신 방식 입니다. 즉, 이 파일 하나로 한 개의 `Client` 장치에만 수신됩니다.

---

#### Firebase Admin SDK 방식

```py
#!/usr/bin/python
#-*- coding: utf-8 -*-
import firebase_admin
from firebase_admin import credentials
from firebase_admin import messaging
from firebase_admin import datetime

cred = credentials.Certificate('service-account.json')
default_app = firebase_admin.initialize_app(cred)

# This registration token comes from the client FCM SDKs.
#registration_token = 'ANDROID_CLIENT_TOKEN'
topic = 'capstone'

# See documentation on defining a message payload.
message = messaging.Message(
    android=messaging.AndroidConfig(
        ttl=datetime.timedelta(seconds=3600),
        priority='normal',
        notification=messaging.AndroidNotification(
            title='알림인데',
            body='백그라운드 자비 좀',
            icon='',
            color='#f45342',
            sound='default'
        ),
    ),
    data={
        'score': '850',
        'time': '2:45',
    },
    webpush=messaging.WebpushConfig(
        notification=messaging.WebpushNotification(
            title='웹 알림',
            body='여긴 어떨까',
            icon='',
        ),
    ),
    topic=topic
    #token=registration_token
)

# Send a message to the device corresponding to the provided
# registration token.
response = messaging.send(message)
# Response is a message ID string.
print 'Successfully sent message:', response
```

* 이 방식도 `HTTP v1` 방식처럼 단일 기기 대상으로 송신하려면 `topic`을 지우고 `registration_token`, `token`을 활성화 시켜주면 됩니다.
* 이 방식은 좀 전의 `새 비공개 키 생성`시 받았던 파일에 모든 정보가 들어있으므로, 여기서 따로 설정할 것은 없습니다.
* 또한 안드로이드, 웹 동시에 전달이 가능합니다.
* 특히 `sound='default'` 되어있는 이곳을 지우면 백그라운드에서 알림이 수신되더라도 소리, 진동, 화면켜짐이 나타나지 않습니다.


---

### 5. 결과 확인

![Create Project Img](/asset/img/kotlin_noti/34.PNG)

* 파일을 실행시켜 봅니다.
* 위 사진은 `HTTP v1` 방식으로 전송한 모습이며, 실패 시 에러코드와 함께 에러내용이 나타납니다.
* `Firebase Admin SDK` 방식으로 전송할 경우, 성공 시 성공 메세지가 나타납니다.