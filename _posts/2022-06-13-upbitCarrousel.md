---
layout: single
title:  "upbit API를 이용해 코인 시세정보를 캐러셀 스타일로 만들기"
---

#fetch API를 통해 upbit 시세정보를 캐러셀 스타일로 만들기

일단 이 프로젝트는 크게 두 부분으로 나눌 수 있다. 첫번째는 캐러셀 슬라이드 만들기이고 두번째는 업비트 공개 api를 받아오는 것이다.

이 과제를 하게 된 이유는 캐러셀 슬라이드를 만드는 데 그냥 만들면 심심해서 캐러셀 요소안에 무엇인가 넣고 싶었고, 코인 시세를 넣게 되었다.

완성본은 대략 이러한 모습이다.
![20220619_131541](https://user-images.githubusercontent.com/99978225/174465667-61f93a52-03bb-45bf-bc6e-c9c143d679c7.png)

캐러셀 만드는 부분은 딱히 특별한 것은 없다. 기존의 캐러셀이랑 다르게 양쪽 끝에 있는 것을 클릭하면 두칸씩 이동하는 것 정도인데 딱히 특별할 것은 없다. 

``` js
function moveNext(event) {
  const $next = document.querySelector('.next');
  const $current = document.querySelector('.current');
  const $prev = document.querySelector('.prev');
  const $Pprev = document.querySelector('.Pprev');
  const $Nnext = document.querySelector('.Nnext');
  $item = event.target;
  const defaultClass = 'item';
  if ($item.className === 'item next') {
    $next.className = `${defaultClass} current`;
    $Nnext.className = `${defaultClass} next`;
    $current.className = `${defaultClass} prev`;
    $prev.className = `${defaultClass} Pprev`;
    $Pprev.className = `${defaultClass} Nnext`;
  } else if ($item.className === 'item Nnext') {
    $next.className = `${defaultClass} prev`;
    $Nnext.className = `${defaultClass} current`;
    $current.className = `${defaultClass} Pprev`;
    $prev.className = `${defaultClass} Nnext`;
    $Pprev.className = `${defaultClass} next`;
  } else if ($item.className === 'item prev') {
    $next.className = `${defaultClass} Nnext`;
    $Nnext.className = `${defaultClass} Pprev`;
    $current.className = `${defaultClass} next`;
    $prev.className = `${defaultClass} current`;
    $Pprev.className = `${defaultClass} prev`;
  } else if ($item.className === 'item Pprev') {
    $next.className = `${defaultClass} Pprev`;
    $Nnext.className = `${defaultClass} prev`;
    $current.className = `${defaultClass} Nnext`;
    $prev.className = `${defaultClass} next`;
    $Pprev.className = `${defaultClass} current`;
  }
}
```

코드를 보면 화면에 'click'이벤트가 발생했을때 그 이벤트가 캐러셀 요소이면 클릭 된 캐러셀을 중앙에 배치시키는 방법이다. 클릭이 일어날때마다 html의 클래스를 새로 바꾸는 방법으로 구현했다. 

두번째는 upbit api 받아오는 부분인데 이것도 upbit에서 워낙 친절하게 제공을 해줘서 특별한 것은 없다. 
https://api.upbit.com/v1/ticker?markets=KRW-BTC
위 api로 코인 시세를 받아올 수 있다. 

``` js
function getPrice() {
  // 비트코인
  function getBtc() {
    fetch('https://api.upbit.com/v1/ticker?markets=KRW-BTC')
      .then((res) => res.json())
      .then((data) => {
        // 가격 입력
        const price = data[0].trade_price.toLocaleString();
        $btcPrice.textContent = `${price}원`;
        // 증감률 입력
        const rate = (data[0].change_rate * 100).toFixed(2);
        $btcRate.textContent = `${rate}%`;
        // 색 결정
        const status = data[0].change;
        if (status === 'RISE') {
          $btc.querySelector('.coin-price').style.color = 'red';
        } else if (status === 'FALL') {
          $btc.querySelector('.coin-price').style.color = 'blue';
        } else {
          $btc.querySelector('.coin-price').style.color = 'black';
        }
      });
  }
  getBtc();
}
getPrice();
setInterval(() => getPrice(), 1000);
```
fetch api를 통해 ajax 통신으로 가격 받아오는 function을 만들고 이것을 setinterval을 통해 1초마다 한번 씩 받아오게 시켰다.
