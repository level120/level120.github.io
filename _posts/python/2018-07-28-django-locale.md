---
layout: post
title: Django locale 에러에 관한 고찰
published: True
category: Python
tags: [python, py, django, locale]
comments: true
---

# locale 에러, 왜 뜨는 걸까? (추측)

개발용 pc에서는 시간, 날짜 등 `locale` 문제(에러 아님)를 해결하기 위해 아래와 같은 코드를 썻었다.

```py
import locale
locale.setlocale(locale.LC_CTYPE, 'korean')
```

잘 된다.

하지만 문제는 이게 배포할 때 생긴다.

실제 운영하는 곳은 `Ubuntu 16.04 LTS`을 사용하고 있다.

그런데 여기에 바로 올리는 것이 아니라 `docker` 서비스 위의 `container`로 돌아간다.

여기서 실행만 하면 `locale` 에러가 뜬다..

문제는 개발pc가 두 대인데, 하나는 윈도우고 하나는 맥이다. 그런데 맥에서도 같은 에러가 뜬다..

구글링부터 자체 분석, 여러가지 솔루션을 도입했지만 에러는 해결되지가 않았다.

그러던 중 우연히 위에 있는 코드를 주석처리했는데 에러없이 잘 돌아간다..

그런데 이게 개발pc에선 주석처리 상태론 에러가 뜬다..

이로서 알아낸 정보는 다음과 같다.


-----

# 알아낸 정보? (확실하지 않음)

개발용 pc 중 한 대는 `windows 10 64bit korean`이고 배포하려는 서버는 `Ubuntu 16.04 LTS English`이므로 차이가 생기는데 이게 주 원인이지 않을까 생각된다(맥이 `unix` 기반, 리눅스가 `unix base`인 걸 감안할 때 같은 원인이지 않을까).

`ubuntu`에서 한글을 적용하면 문제가 많이 발생하기 때문에 애당초 영어버전으로 설치했고 이후에 한글 `locale`을 설정했지만 이것이 적용되지가 않았으며, 이는 `docker container`에도 영향을 끼친 것으로 생각된다.

따라서, 윈도우에서 작업할 땐 `locale` 코드를 써주고, 리눅스나 맥으로 작업할 땐 `locale` 코드를 주석처리한 뒤 작업한다.