---
layout: single
title:  "업비트 선물거래 기능 구현"
categories : "small-talk"
tag : [Typescript, React]
---

# 선물거래 기능 구현과 차트 구현 

사실 이 글은 single-project 에서 다뤄야 적절하지만 간단하게 small-tack에서 다루고 넘어가려고한다. 왜냐하면 이 부분이 언뜻보면 상당히 어려워보이지만 사실 어려울 것이 없다. 기술보다는 알고리즘 이해도가 중요하고 또 그렇게 어려운 수준도 아니어서, 간단하게 얘기하고 이 부분은 넘어가려고 한다. 

# 거래 기능 구현

거래 기능은 진짜 너무 간단하다. 이 앱은 setInterval로 api호출을 하면서 upbit에서 코인 시세를 받아온다. 그리고 주문 버튼을 누르는 그 순간의 코인 가격을 하나의 스테이트에 저장한다. 그리고 현재 코인 가격과 스테이트에 저장된 가격의 괴리를 구하고 거기에 레버리지를 곱한다. 끝이다.

# 차트 구현

이 부분이 안해본 사람은 좀 복잡하다고 느낄 것이다. 하지만 막상 해보면 간단하다. 일단 내가 구현한 방식은 Candle DOM의 스타일 속성을 변수를 통해서 정한 것이다. 차트에 총 100개의 캔들이 그려지는데, 일단 먼저 해야하는 것은 100개 캔들의 최댓값과 최솟값을 구해야한다. 이 부분은 Math.max()를 쓰거나 반복문을 돌리거나 취향것 하면 된다. 이렇게 최대 최소를 구하는 이유는 캔들의 크기를 비례적으로 만들어야하기 때문이다. 일단 최대값에서 최솟값을 뺀 totalSize란 값을 구하고 현 캔들의 시작가와 거래가의 차이를 구한다. 예를 들어 totalSize가 100이고 시작가와 거래가의 차이가 10일때, 이 캔들의 길이는 차트 컨테이너의 10분의 1이 되면 된다. 그리고 만약 시작가가 거래가 보다 크면 빨간색으로, 반대면 파란색으로 만들면된다.  어렵게 들릴 수 있는데 알고리즘 문제를 좀 풀어봤다면 이것은 정말 간단한 문제이다. 