---
layout: post
title: Django와 Ajax으로 좋아요! 형태의 투표 만들기(1)
published: True
category: Python
tags: [python, py, django]
comments: true
---

# 시작하기 전에

Django 기본 앱과 Bootstrap에 대해 설치 및 기본적인 개념은 하지 않습니다.

또한 css 코드 역시 설명하지 않습니다.

** [중요!] 로그인 된 사용자가 아니면 동작하지 않습니다. **

사용한 django 버전 : `django >=1.9, <2`
사용한 django app 이름 : `VoteApp`

-----

# 1. 만들고자 하는 것

![투표 시스템](/asset/img/vote/1.gif)

위의 이미지와 같이 버튼을 누를 때 새로고침 없이 득표현황이 갱신되는 것을 만드려고 한다.

투표 버튼은 다른 커뮤니티의 `좋아요` 방식을 이용한다.

이를 위해 여기서는 `Jquery`의 `ajax`을 이용해 비동기 통신으로 해결한다.


-----

# 2. 작업순서

a) model 만들기
b) view 만들기, url 연결하기
c) template 만들기, html 만들기
d) ajax 비동기 통신


-----

# 3. model 만들기

저의 경우에는 `후보자 이름`, `후보자 소개`, `후보자 여부`, `수상이력` 들을 알기 위해 아래처럼 작성했으나,

```py
# models.py
from django.db import models

# Create your models here.
from django.contrib.auth.models import User

class Like(models.Model):
    name_id = models.ForeignKey(User, on_delete=models.CASCADE)
    name = models.CharField(max_length=8, db_column='이름')
    description = models.CharField(max_length=30, db_column='소개')
    isVote = models.BooleanField(default=False, db_column='후보')
    prize = models.BooleanField(db_column='우승이력', default=False)
    likes = models.ManyToManyField(User, null=True, blank=True, related_name='likes')

    @property
    def total_likes(self):
        return self.likes.count()

    def save(self, *args, **kwargs):
        super(Like, self).save(*args, **kwargs)
```

꼭 필요한 요소는 아래와 같으므로 아래에 있는 속성들만 꼭 있으면 된다.

```py
# models.py
from django.db import models

# Create your models here.
from django.contrib.auth.models import User

class Like(models.Model):
    name_id = models.ForeignKey(User, on_delete=models.CASCADE)
    likes = models.ManyToManyField(User, null=True, blank=True, related_name='likes')

    @property
    def total_likes(self):
        return self.likes.count()
```

여기서의 `name_id`는 후보자를 구분하는 `id`가 되기 때문에 `User` 모델과 연결한다.

`likes`는 `ManyToManyField`이므로 득표수를 알려면 그 타이밍 마다 뒤의 속성까지 다 적어줘야 하는 불편함이 있다.

이러한 문제해결을 위해 `total_likes`를 `@property`로 추가해야 한다.


-----

# 4. view 만들기

조금전에 만든 `model`을 `view`에서 `template`으로 넘겨줘야 우리가 보는 화면에 띄울 수가 있다.

여기서는 그러한 작업을 진행한다.

아래 코드는 `view.py`의 코드 중 일부이다.

```py
# views.py
from .models import Like

def vote(request):
    likes = Like.objects.filter(isVote=True)
    return render(
        request,
        "VoteApp/vote.html",
        {
           'players' : likes,
           'all_member' : User.objects.count()-1
        }
    )
```

조금전에 정의한 `Like`을 쓰기 위해선 모델에서 가져와야 한다.

이 부분이 `from .models import Like`이며 이 문장 이후로 `Like`를 원하는 때에 가져다 쓸 수가 있다.

이 페이지를 띄우기 위한 함수를 `vote`로 정의하였고 `likes`에 `Like` 모델을 가져와서 담는다.

이 때 후보자만 가져오도록 `isVote=True`로 설정하였으나... 구축해본 결과 저건 만들 필요가 없었던 것 같다.

왜냐면 저 테이블에 들어가는 건 후보자만 들어가기 때문..

어쨋든 이것을 감안하면 저 문장을 다음과 같이 바꿀 수도 있다.

`Like.objects.all()`

`VoteApp/vote.html` 여기는 자신의 `html`과 맞추면 된다.

`players : likes`는 `likes`를 `template`으로 넘길 때 `players`라는 이름으로 넘긴다는 뜻이다.

`all_member`에 카운트 값을 `-1`한 이유는 관리자 계정을 제외하기 위함이다. 기본적으로 `django`의 시스템에는 관리자 계정도 포함되어 있어 사용자만 카운팅 하기 위해서는 이를 제외해야 한다.


-----

### url 추가하기

자신의 앱 최상단의 `urls.py`에 저 내용을 등록해야 한다.

아래와 같이 하나만 추가하면 된다.

```py
# urls.py
import VoteApp.views

urlpatterns = [
    ...
    url(r'^vote$', VoteApp.views.vote, name='vote'),
    ...
]
```

가운데 인자인 `VoteApp.views.vote`에는 조금 전 작성한 함수명과 반드시 일치해야 한다.


-----

# 5. template 만들기

`template` 만들 때 `view`에서 넘겨준 `players`를 이용한다.

즉, `players`를 통해 후보자 전체를 문서에 출력하는 것이다.

이렇게 작성된 코드는 아래와 같다.

만약 `model`에서 필수코드만 썻다면 아래 코드에서 다음 두 라인만 제외하면 된다.

```html
<td style="vertical-align:middle; text-align:center"><span class = 'user-green'>{{ player.name }}</span></td>
<td style="vertical-align:middle" class="click-this">{{ player.description }}</td>
```

```html
# VoteApp/vote.html
<table style="font-size: 13px" class="table table-striped table-bordered sortable-table clickable-table" id="problemset">
    <thead>
        <tr>
            <th style="text-align:center;width:6%" data-sort="int">No.</th>
            <th style="text-align:center;width:12%" data-sort="string">이름</th>
            <th style="text-align:center;width:40%" data-sort="string">설명</th>
            <th style="text-align:center;width:35%">득표현황</th>
            <th style="text-align:center;width:8%">투표여부</th>
        </tr>
    </thead>
    <tbody>
        {% for player in players %}
        <tr>
            <td style="vertical-align:middle; text-align:center">{{ player.id }}</td>
            <td style="vertical-align:middle; text-align:center"><span class = 'user-green'>{{ player.name }}</span></td>
            <td style="vertical-align:middle" class="click-this">{{ player.description }}</td>
            <td style="vertical-align:middle;width:35%">
                <div class="progress progress-striped active" style="margin-bottom: 0px;">
                    {% with percent=100 %}
                    <div class="progress-bar pb{{ player.id }}" style="width: {{ player.total_likes|div:all_member|mul:percent }}%;"><span style="vertical-align:middle">{{ player.total_likes }} / {{ all_member }}</span></div>
                    {#<div class="progress-bar progress-bar-success" style="width: 100%;">1</div>#}
                    {% endwith %}
                </div>
            </td>
            <td style="vertical-align:middle"><button type="button" id="{{ player.id }}" name="vote{{ player.id }}" class="like btn btn-success glyphicon glyphicon-thumbs-up"> 투표하기</button></td>
        </tr>
        {% endfor %}
    </tbody>
</table>
```

중간에 있는 `progress-bar`의 단위를 조심하자. 단위 실수하면 `px`단위로 적용된다.
```html
style="width: {{ player.total_likes|div:all_member|mul:percent }}%;"
```

이러면 반복문을 돌면서 후보자 계정만큼 만들어지게 된다.


그러고 한 번 켜보자. 페이지가 완성이 되었을 것이다.

하지만 동작은 하지 않는데 이는 다음 편에서 다루고자 한다.
