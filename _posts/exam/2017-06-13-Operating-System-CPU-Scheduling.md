---
layout: post
title: C# WPF CPU 스케줄링
published: True
category: 학교시험
tags: [c#, cs, csharp, wpf, cpu scheduling]
comments: true
---

# [C# WPF] CPU 스케줄링

2016년도 1학기 Android 교과목 과제

---

### 시작하기 전에

이 편의 기본 코드는 `CPU 스케줄링 기법 - 고정국`교수님의 자료 중 의사코드를 참고하여 작성하였습니다.

코드는 [Github - OperatingSystem](https://github.com/level120/OperatingSystem)에서 확인할 수 있습니다.

---

### 과제목표

| 최종목표 | 팀 목표 |
| --- | --- |
| 1. 사용자가 다양한 매개변수 설정 | 1. 간단한 구조의 시뮬레이션(UX/UI) |
| - 프로세스 도착, 서비스 시간 등 | 2. 한눈에 결과 확인이 가능한 차트 |
| 2. CPU 스케줄링 알고리즘 6종 구현 | 3. 처음 사용자를 위한 사용 설명서 |
| 3. 스케줄링 결과의 시각화 | 4. 문제 발생 시 빠른 수정이 가능하도록 그룹 설정 |
| 4. 결과 분석화면 제공 | 5. 각 계획에 할당된 기간 준수 |


### 개발환경

| 항목 | 내용 |
| :-----: | ----- |
| 사용언어 | C# WPF .Net Framework 4.5.1 |
| 코딩작업 | Visual Studio 2017 Community |
| 디자인 작업 | Blend for Visual Studio 2017 Community |
| 형상관리 | level120's Github |
| 최적화 | Sonar Qube v6.3.1 |
| 테스트 환경 | Windows 7, 10 |


### 역할분배

| 팀원 | 알고리즘 |
| :-----: | ----- |
| 12110066 | SJF, SRT, HRN, Priority, Round-Robin |
| 14110075 | FCFS |


### 수행일정

| 항목 | 1주차 | 2주차 | 3주차 | 4주차 | 5주차 |
| :---: | :---: | :---: | :---: | :---: | :---: |
| 설계 계획서 작성 | * | | | | |
| 알고리즘 구현 | | * | | | |
| 기능 점검 | | | * | | |
| UI 구현 | | | * | * | |
| 추가기능 | | | | * | |
| 최종 점검 | | | | | * |

---

### 구현과정

보고서로 대체

---

### 완성모습

| 화면모습 | 설명 |
| :-------: | ---- |
| ![Main View](/asset/img/cpu_opsys/1.PNG) | 좌측의 알고리즘 리스트, 상단의 프로세스 개수, 중앙의 결과 모습 |
| ![Main View](/asset/img/cpu_opsys/2.PNG) | 좌측에서 알고리즘 선택 후 프로세스 생성하면 다음과 같은 모습이 등장 |
| ![Main View](/asset/img/cpu_opsys/3.PNG) | FCFS 결과 모습 |
| ![Main View](/asset/img/cpu_opsys/4.PNG) | FCFS 프로세스 정보 고정, SJF 모습 |
| ![Main View](/asset/img/cpu_opsys/5.PNG) | FCFS 프로세스 정보 고정, SRT 모습 |
| ![Main View](/asset/img/cpu_opsys/6.PNG) | FCFS 프로세스 정보 고정, HRN 모습 |
| ![Main View](/asset/img/cpu_opsys/7.PNG) | FCFS 프로세스 정보 고정, 우선순위 모습(여기서만 우선순위 열 표시) |
| ![Main View](/asset/img/cpu_opsys/8.PNG) | FCFS 프로세스 정보 고정, 라운드 로빈 모습 |
| ![Main View](/asset/img/cpu_opsys/9.PNG) | 결과가 나온 뒤 프로세스를 누르면 자세한 정보 제공 |
| ![Main View](/asset/img/cpu_opsys/10.PNG) | 도움말 기능 포함 |
