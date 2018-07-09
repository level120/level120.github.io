---
layout: post
title: JavaScript 정보보안 S-DES
published: True
category: 학교시험
tags: [js, javascript, html, html5, sdes]
comments: true
---

# [JS] 정보보안 S-DES

2017년도 2학기 정보보안 교과목 과제

---

### 시작하기 전에

이 편의 기본 코드는 `S-DES - 추영열`교수님의 자료 중 의사코드를 참고하여 작성하였습니다.

이곳에 등장하는 값의 랜덤배치는 `자료에 나와있는 순서를 따를 것`이라는 조건에 따라 고정값을 사용하였습니다.

추가적으로 쓸데없는 비동기 연출을 나타내느라 코드가 좋지는 않습니다(특히 `js/func.js`).

이러한 연출이 필요없다면 코드를 훨씬 간결하게 나타낼 수 있지만 제출용(시뮬레이터용)이므로 수정하지 않았습니다.

* 코드는 [Github - Information_Secuity](https://github.com/level120/Infomation_Security)에서 확인할 수 있습니다.
* 완성모습은 [여기](https://level120.github.io/Infomation_Security)에서 확인할 수 있습니다.

---

### 개발환경

| 항목 | 내용 |
| :-----: | ----- |
| 사용언어 | HTML5, CSS3, JavaScript |
| 디자인 | Bootstrap |
| 테스트 환경 | Chrome, Firefox, Edge, Whale에서 구동확인, IE11(미지원) |

---

### 구현과정

단순 DES를 만드는 과정은 키 값을 먼저 지정하고 지정된 절차에 따라 평문을 암호문으로 바꿔주는 과정이다.

따라서 크게 3부분으로 나뉘며 이는 각각 다음과 같다.

* 키 만들기
* 암호문 만들기
* 복호문 만들기

이 때 암호문은 평문과 1:1로 대응해야 한다(`평문길이 == 암호문길이`).

따라서 위와 같은 순서로 개발을 진행한다.

![Fig 3.1](/asset/img/sdes/1.png)

---

#### 키 만들기

![Fig 3.2](/asset/img/sdes/2.png)

키는 10-bit로 이루어지기 때문에 `0b0000000000` ~ `0b1111111111`의 숫자까지만 가능하도록 미리 제한한다.

(10진수로 나타내면 `0` ~ `1023`까지이다)

![0b1111111111](/asset/img/sdes/3.png)

이렇게 들어온 10-bit의 키 값은 특정 순서로 비트의 자리를 바꾼다.

아래 코드는 나중에 나올 8-bit와 4-bit에서 똑같은 작업을 수행하기 떄문에 하나로 묶어놓은 코드이다.


```js
// Usage : bit count, item(10=key, 8=char), item length
function p(cnt, item, bit) {
    var bitset = [];
    bitset.length = bit;

    for (var i = 0; i < bitset.length; i++) {
        bitset[i] = (item >> (bitset.length - (i + 1))) & 0x1;
    }
    // console.log((cnt==10?('bitset10 : '):cnt==8?('bitset8 : '):('bitset4 : ')) + bitset);

    var list = (cnt == 10 ? [2, 4, 1, 6, 3, 9, 0, 8, 7, 5] :
        cnt == 8 ? [5, 2, 6, 3, 7, 4, 9, 8] : [1, 3, 2, 0]);
    var result = 0;

    for (i = 0; i < cnt; i++) {
        result |= bitset[list[i]];
        if (i != (cnt - 1)) {
            result = result << 1;
        }
    }
    return result;
}
```

이렇게 얻은 키 값을 `5-bit`로 나누어 `1-bit`씩 왼쪽으로 옮겨야 한다(`LS`는 `Left Shift`의 줄임말).

아래 코드는 나중에 나올 `LS-2`인 경우를 위해 하나로 묶어 나타낸 것이며 칸수 구분을 `scnt` 변수를 통해 조절한다.

```js
// Usage : shift count, bit size, item(5-bit)
function ls(scnt, bsize, item) {
    if (scnt == 1) {
        return ((item << 1) | (item >> (bsize - 1))) & 0x1F;
    }

    if (scnt == 2) {
        return ((item << 2) | ((item >> (bsize - 1)) << 1) | ((item >> (bsize - 2)) & 0x1)) & 0x1F;
    }
}
```

이렇게 나온 값을 다시 `p` 함수를 이용하여 `8-bit`만 선택하면 `k1`이 된다.

`k2`를 구하려면 위의 `LS-2`를 실행한 뒤 `p` 함수를 이용하려 `8-bit`만 선택하면 된다.

---

#### 암호문 만들기

대망의 키 만들기가 끝났지만 아직 넘어야 할 과제가 두 개 더 남았다.

암호문을 만들기 위해서는 먼저 평문의 자리를 바꿔야 한다.

문제는 여기인데, `ASCII`를 사용하는 영어 혹은 숫자의 경우에는 문자 하나당 `1-byte`로 문제가 없지만,

이러한 방식이 아닌 한국어, 일본어, 중국어 등을 사용하게 되면 문자 하나당 `2-byte`가 된다.

따라서 이를 제어하지 못한다면 영어와 숫자만 사용하도록 해야 한다.

(이 과제에는 이러한 내용이 적용되어 있지만 본 문서에서는 기술하지 않음, 참고 `js/func.js` -  [`line no.222`](https://github.com/level120/Infomation_Security/blob/master/js/func.js#L222))

아래 코드는 하나의 문자(`1-byte`)를 입력받으면 `4-bit`씩 나누어 자리를 교환한다.

또한 나중에 나올 역함수를 위해 `inverter`를 포함시킨 내용이다.

```js
// Usage : is inverter, char(8-bit)
function ip(isInv, item) {
    var bitset = [];
    bitset.length = 8;

    for (var i = 0; i < bitset.length; i++) {
        bitset[i] = (item >> (bitset.length - (i + 1))) & 0x1;
    }
    // console.log('bitset8S : ' + bitset);

    var list = (!isInv ? [1, 5, 2, 0, 3, 7, 4, 6] : [3, 0, 2, 4, 6, 1, 7, 5]);
    var result = 0;

    for (i = 0; i < bitset.length; i++) {
        result |= bitset[list[i]];
        if (i != (bitset.length - 1)) {
            result = result << 1;
        }
    }
    return result;
}
```

다음은 순열과 치환 함수의 조합은 `f`를 다룬다.

여기서 `L`은 앞서 나온 값의 왼쪽 `4-bit`이고 `R`은 앞서 나온 값의 오른쪽 `4-bit`이다.

`SK`는 `Sub Key`의 약자로 처음에 구했던 키 값 `k1`이다.

이 때 함수 `F(R, SK)`는 확장, 순열작업으로 여기에 나오는 `S-Box(s0, s1)`은 고정 값으로 처리한다.

아래 코드는 `F(R, SK)`를 처리하는 함수이며 `return` 부에서 해당 값을 확인 후 `p`과정을 통해 최종값이 반환되는 구조이다.

```js
// Usage : right bit(4-bit), sub key
function F(r, sk) {
    var s0 = [
                [1, 0, 3, 2],
                [3, 2, 1, 0],
                [0, 2, 1, 3],
                [3, 1, 3, 2]
            ];
    var s1 = [
                [0, 1, 2, 3],
                [2, 0, 1, 3],
                [3, 0, 1, 0],
                [2, 1, 0, 3]
            ];

    var bitsetR = [];
    var bitsetSK = [];
    bitsetR.length = 4;
    bitsetSK.length = 8;

    for (var i = 0; i < 4; i++) {
        bitsetR[i] = (r >> (4 - (i + 1))) & 0x1;
    }
    for (i = 0; i < 8; i++) {
        bitsetSK[i] = (sk >> (8 - (i + 1))) & 0x1;
    }
    // console.log('bitset4ef : ' + bitsetR);
    // console.log('bitset8sk : ' + bitsetSK);

    var list = [3, 0, 1, 2, 1, 2, 3, 0];
    var ep = 0;

    for (i = 0; i < 8; i++) {
        ep |= bitsetR[list[i]];
        if (i != 7) {
            ep = ep << 1;
        }
    }

    ep = ep ^ sk;
    return p(4, ((s0[(ep & 0x80) >> 6 | (ep & 0x10) >> 4][(ep & 0x40) >> 5 | (ep & 0x20) >> 5]) << 2) | (s1[(ep & 8) >> 2 | (ep & 1)][(ep & 4) >> 1 | (ep & 2) >> 1]), 4);
}
```

다음은 `fk`과정이다. 순서로 따지면 위의 함수를 아래 코드에서 호출한다.

```js
// Usage : item(8-bit), sub key
function fk(item, sk) {
    var l = item >> 4;
    var r = item & 0xF;

    return ((l ^ F(r, sk)) << 4) | r;
}
```

다음은 이러한 과정을 거치고 나온 값은 스위치(`SW`)하는 과정이다.

이는 단순히 좌측 `4-bit`와 우측 `4-bit`를 교환하기만 하면 된다.

아래 코드는 단순 스위치 과정이다.

```js
// Usage : item(8-bit)
function sw(item) {
    return ((item & 0xF) << 4) | (item >> 4);
}
```

이제 함수 정의는 모두 끝났으며 다시 `fk` 호출한다. 이 때 사용되는 `sk`는 `k2`이다.

마지막으로 `ip inverter`과정만 거치면 암호문이 완성된다.

---

#### 복호문 만들기

복호문을 만드는 방법은 암호문을 만드는 방법과 똑같다.

다른점은 평문 대신 암호문을 입력하고, `k1`과 `k2`의 순서가 바뀐다는 것 뿐이다.

---

### 결과

![테스트 결과](/asset/img/sdes/4.png)

위의 그림과 같이 암/복호화가 잘 이루어졌다.

중간에 주석되어 있는 코드를 해제하면 개발자 도구에서 로그값이 찍히는 것도 확인할 수 있다.

---

#### 추가 - 문자 혹은 숫자를 이진수로 바꾸는 방법

`1-byte` 문자나 숫자를 이진수로 나타내야 할 경우가 있다.

아래 코드는 이를 해결해주는 코드이다.

(이 때 `cnt`는 비트의 개수이며 보통 `1-byte`당 `8-bit`이다)

```js
// Usage : bit count, item(n-bit)
function getbit(cnt, item) {
    var bitset = [];
    bitset.length = cnt;

    for (var i = 0; i < bitset.length; i++) {
        bitset[i] = (item >> (bitset.length - (i + 1))) & 0x1;
    }
    var result = '';
    for (var i in bitset) {
        result += bitset[i];
    }
    return result;
}
```