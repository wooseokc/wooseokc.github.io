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
