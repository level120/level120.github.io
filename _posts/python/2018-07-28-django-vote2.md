---
layout: post
title: Django와 Ajax으로 좋아요! 형태의 투표 만들기(2)
published: True
category: Python
tags: [python, py, django]
comments: true
---

# 시작하기 전에

여기서부터는 `ajax` 비동기 통신만 다룹니다.

-----

# 1. 만들고자 하는 것

![투표 시스템](/asset/img/vote/1.gif)

위의 이미지와 같이 버튼을 누를 때 새로고침 없이 득표현황이 갱신되는 것을 만드려고 한다.

투표 버튼은 다른 커뮤니티의 `좋아요` 방식을 이용한다.

이를 위해 여기서는 `Jquery`의 `ajax`을 이용해 비동기 통신으로 해결한다.


-----

# 2. 작업순서(ajax 비동기 통신하기)

a) ajax 작성하기
b) post 전용 view 만들기
c) b를 url에 등록하기


-----

# 3. ajax 작성하기

1편에서 만든 `template`에 이어서 작성하며 `jquery`를 사용한다.

`ajax`은 두 개를 작성해야 하는데, 하나는 페이지 첫 진입 시 투표여부 확인이고 두 번째는 투표하기 버튼을 눌렀을 때 처리되는 내용이다.

여기서는 두 가지 모두 다 다룬다.

첫 번째 코드는 아래와 같으며, `csrf_token` 만 그대로 쓰고 나머지는 자신의 코드에 맞게 고친다(저 토큰이 없으면 데이터가 전달되지 않음).

```js
# VoteApp/vote.html
$(document).ready(function () {
    $('[name^=vote]').each(function () {
        let pk = $(this).attr('id');
        $.ajax({
            type: "POST",
            url: "{% url 'like_ready' %}",
            data: {
                'pk': pk,
                'csrfmiddlewaretoken': '{{ csrf_token }}'
            },
            dataType: "json",
            success: function(response) {
                $('#'+pk).text(' ' + response.message);
                if(dday>0){
                    $('#' + pk).attr('class', (response.message !== "투표취소 ") ? 'like btn btn-success glyphicon glyphicon-thumbs-up' : 'like btn btn-danger glyphicon glyphicon-thumbs-down');
                }
                else{
                    $('#' + pk).attr('class', (response.message !== "투표취소 ") ? 'like btn btn-success glyphicon glyphicon-thumbs-up disabled' : 'like btn btn-danger glyphicon glyphicon-thumbs-down disabled').attr('disabled', 'disabled');
                }
            },
            error: function(request, status, error) {
                console.log("code:" + request.status + "\n" + "message:" + request.responseText + "\n" + "error:" + error);
            }
        });
    })
});
```

버튼을 눌렀을 때 행동하는 코드는 아래와 같다.

로그인 한 사용자만 구분하기 위해 로그인 여부 구문을 함께 사용한다.

```js
# VoteApp/vote.html
{% if user.is_authenticated %}
$('.like').click(function() {
    let pk = $(this).attr('id');
    $.ajax({
        type: "POST",
        url: "{% url 'like' %}",
        data: {
            'pk': pk,
            'csrfmiddlewaretoken': '{{ csrf_token }}'
        },
        dataType: "json",
        success: function(response) {
            $('.pb'+pk).css('width', (response.likes_count/{{ all_member }})*100+'%');
            $('.pb'+pk+' span').text(response.likes_count+' / '+{{ all_member }});
            $('#'+pk).text(' ' + response.message).attr('class', (response.message!=="투표취소 ")?'like btn btn-success glyphicon glyphicon-thumbs-up':'like btn btn-danger glyphicon glyphicon-thumbs-down');
        },
        error: function(request, status, error) {
            console.log("code:" + request.status + "\n" + "message:" + request.responseText + "\n" + "error:" + error);
        }
    });
})

{% endif %}
```

이로서 `ajax` 부분은 끝났다.


-----

# 4. post 전용 view 만들기

조금 전 만든 `ajax`을 처리하는 부분이 이곳이다.

아래 코드에서 상태 메세지(`투표하기`, `투표취소`)를 전송하는 것은 비효율 적인 것 같다. 왜냐면 이건 서버가 처리할 내용은 아니기 때문..

이 코드는 처음 작성하는 거라 다른 분 예제를 참고했기에.. 다음에 비슷한 내용을 작성할 일이 생긴다면 이 점은 반드시 고려하자.

```py
#view.py
@login_required
@require_POST
def like(request):
    if request.method == 'POST':
        user = request.user
        name_id = request.POST.get('pk', None)
        obj = Like.objects.get(pk=name_id)

        if obj.likes.filter(id=user.id).exists():
            obj.likes.remove(user)
            message = '투표하기 '
        else:
            obj.likes.add(user)
            message = '투표취소 '

    context = {'likes_count': obj.total_likes, 'message': message}
    return HttpResponse(json.dumps(context), content_type='application/json')


@login_required
@require_POST
def like_ready(request):
    if request.method == 'POST':
        user = request.user
        name_id = request.POST.get('pk', None)
        obj = Like.objects.get(pk=name_id)

        if obj.likes.filter(id=user.id).exists():
            message = '투표취소 '
        else:
            message = '투표하기 '

    context = {'likes_count': obj.total_likes, 'message': message}
    return HttpResponse(json.dumps(context), content_type='application/json')
```

정확한 이유는 모르지만 상황에 따라선 `json`에러가 날 수 있다. 스택 오버플로우에 다양한 해결책이 있었지만 이들 중 내가 차용한 방법은 현재 나의 버전에 맞는 문법을 사용하도록 바꾼 것이다.

스택 오버플로우 등 다양한 곳에 설명된 django의 버전은 대부분 1.6미만.. 하지만 내 버전은 2.0이다. 따라서 예제라도 문법의 차이가 생긴 듯 하다.


-----

# 5. url 등록하기

이제 `ajax`으로 통신하는 `url`만 연결하면 된다.

```py
urlpatterns = [
     ...
     # Like & Dislike
     url(r'^like/$', VoteApp.views.like, name='like'),
     url(r'^like_ready/$', VoteApp.views.like_ready, name='like_ready'),
     ...
]
```

첫 파라미터는 `template`에서 `ajax`의 `url`부분과 반드시 일치해야 하며, 가운데 파라미터의 함수명이 반드시 `view`의 함수명과 일치해야 한다.


-----

# 끝! 서버를 다시 실행해 동작여부를 확인하자!

아! 추가로 유저를 저기 테이블에 등록해야 페이지에 정상적으로 나타난다.

이 부분은 여기에 기술하지 않았으므로 `admin` 페이지에서 수동으로 등록해서 확인하자.