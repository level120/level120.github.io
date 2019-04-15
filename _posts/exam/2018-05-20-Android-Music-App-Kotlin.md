---
layout: post
title: 안드로이드 뮤직 플레이어 - Kotlin
published: true
category: Univ.
tags: [android, kotlin, music pleayer, music app]
comments: true
---

# Android Music App(Kotlin)

2018년도 1학기 Android 교과목 과제

---

### 시작하기 전에

이 편의 기본 코드는 [tkddlf4209 블로그 - [Android] 안드로이드 Mp3 플레이어 만들기 / MediaPlayer](http://blog.naver.com/PostView.nhn?blogId=tkddlf4209&logNo=220746210643&categoryNo=41&parentCategoryNo=0&viewDate=&currentPage=1&postListTopCurrentPage=1&from=postView)를 참고하여 작성하였습니다.
참고한 기본 코드는 `Java`로 작성하였고 여기는 이를 `Kotlin`으로 변경하였으며 추가 혹은 일부코드 변경이 있습니다.

* 리스트에서 앨범 이미지 표시 시 `Bitmap`에서 `Glide`로 변경

코드는 [Github = Android_MusicPlayer](https://github.com/level120/Android_MusicPlayer)에서 확인할 수 있습니다.

---

### 동작환경

* Android 5.0 롤리팝 이상

### 구현내용

* 음원목록
* 재생화면(재생, 일시정지, 이전곡, 다음곡, 플레이지점 선택가능)


### 파일구조

4개의 소스파일(`*.kt`)과 2개의 레이아웃(`*.xml`)로 구성되어 있습니다.

| Name | Files | | | |
| :-----------: | :-----: | :----: | :----: | :----: |
| Kotlin Code | `MainActivity.kt` | `MusicActivity.kt` | `MusicAdapter.kt` | `MusicType.kt` |
| Layoyut File | `activity_main.xml` | `activity_music.xml` | `itemlist_item.xml` | - |

---

### 구현과정

보고서로 대체

---

### 완성모습

| 플레이리스트 | 재생화면 |
| :-------: | :----: |
| ![Playlist](https://github.com/level120/Android_MusicPlayer/raw/master/asset/1.png) | ![Playlist](https://github.com/level120/Android_MusicPlayer/raw/master/asset/2.png) |