---
layout: post
title: 교내 LG 휘센 중앙제어 조작하기
published: True
category: Python
tags: [IP Scanning]
comments: true
---

# 시작하기 전에

이 포스트는 [이미 알려진 취약점](http://www.dailysecu.com/?mod=news&act=articleView&idxno=7508)에 대한 설명입니다.


-----

# 난이도 : ★☆☆☆☆


-----

# 준비물

[Angry IP Scanner](https://angryip.org/)
[Java(JRE)](https://java.com/ko/download/)
[WHOIS](https://xn--c79as89aj0e29b77z.xn--3e0b707e/kor/whois/whois.jsp)

-----

# 시도

1. 학교 ip를 알아낸다(강의실, 연구실, 쉼터 등 어느곳이든).
2. 위의 `WHOIS`에 접속해 방금 알아낸 ip를 입력한다.
3. ![ip img](/asset/img/ip/1.png)
4. ip 주소가 할당된 대역을 확인한다(여기는 `x.x.76.0` ~ `x.x.93.255`)
5. `Angry IP Scanner`를 실행하여 해당 대역을 검색한다.
6. 웹 서버가 `Boa`인 것을 찾아 해당 주소로 접속한다.
7. ![v-net img](/asset/img/ip/2.gif)
8. 위 그림을 눌러 실행파일을 내려받은 후 실행한다.
9. ![](/asset/img/ip/3.png)
10. 뭔가 막혔다. 이것은 java 보안 설정 때문이므로 `제어판` -> `java`를 예외 등록을 해 줘야 한다.
11. ![](/asset/img/ip/4.png)
12. 위 그림과 같이 예외 설정을 한 뒤 다시 `acp.jnlp`를 실행한다.
13. ![](/asset/img/ip/5.png)
14. 컴퓨터에 따라 시간이 걸리지만 위와 같은 화면이 뜬다. 여기서부터는 관리자의 보안 개념에 따라 다르지만 초기 비밀번호 `digital21`을 입력해본다(권고사항은 비밀번호 변경이지만 일부 학교에서는 그렇지 않은 듯 하다).
15. ![](/asset/img/ip/6.png)
16. 접속 성공! 되면 학교 보안을 탓합시다.



-----

# 결과

학교의 치명적인 보안 시스템을 발견 했지만, 담당자가 하루에 3번씩 모니터링을 하는 모양인 듯 하다.

올해까지 다른 보안문제를 찾아보고 이상있으면 같이 보고하고자 한다.