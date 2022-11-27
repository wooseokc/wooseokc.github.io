---
layout: single
title:  "업비트 선물거래 - 함수형 프로그래밍"
categories : "single-project"
tag : [Typescript, React]
---

# 순수 함수로 코딩하기

기존에 내가 작성했던 레거시 코드를 보면 함수 컴포넌트를 사용하지만 함수형 프로그래밍 방법론을 전혀 이해하지 못한 것을 볼 수 있다.
```js
export default function Chart () {
  async function apicall() {
    return await axios.get(`https://api.upbit.com/v1/candles/minutes/${unit}?market=KRW-${coin}&count=100`).then(res => {
      setChartArr(res.data.reverse());
    });
  } 
  return ()
}
```
위 코드에서 보면 알 수 있듯이 Chart 컴포넌트가 받는 인자는 없으며, 컴포넌트 안에서 서버에 데이터 호출을 요청한다. 따라서 함수의 결과 값이 함수의 인자가 아닌 업비트 서버에 따라 달라진다. 이것은 당연히 함수형 프로그래밍이 아닌 함수를 사용하는 절차적 프로그래밍이다.

나는 이 컴포넌트를 순수 함수로 만들기 위해 다음과 같이 수정했다.

```js
interface ChartData {
  time : number,
  open : number,
  trade : number,
  high : number,
  low : number,
  volume : number,
  candle_date_time_kst? : string,
}

interface importData {
  data : ChartData[],
  count : number,
  from : number,
  setCount : React.Dispatch<React.SetStateAction<number>>
  setFrom : React.Dispatch<React.SetStateAction<number>>
}

export default function Chart (props : importData) {

  return()
}
```
위 코드에서는 Chart 컴포넌트에 props로 전달된 차트 데이터 배열을 그냥 시각화해주는 함수로 나타난다. props로 전달되는 데이터 배열이 달라지면 그 데이터에 맞춰 Chart 컴포넌트는 데이터를 시각화해줄 뿐이다. Chart컴포넌트는 data 배열을 전혀 건드리지 않고 부모 컴포넌트에서 전달받는 데이터를 그릴 뿐이다.

# Why?

사실 이 어플리케이션에서는 이러한 함수형 프로그래밍의 장점이 살아나지 않는다. 왜냐하면 차트를 한번에 하나만 띄우기 때문이다. 하지만 동시에 비트코인과 이더리움 차트를 동시에 띄운다고 가정해보자. 레거시의 방법에선 ChartBTC 컴포넌트와 ChartETH 컴포넌트를 따로 만들어서 각각 컴포넌트에서 api 호출을 하고 차트를 그려야한다. 상당히 비효율적이다.

반면 함수형 프로그래밍에서는 Chart 컴포넌트에 한번은 BTC 데이터를 한번은 ETH 데이터를 넣어주면된다. 

추가적으로 차트의 유연함이 좋아진다. 이번에 리팩토링하면서 새로 추가된 기능인 확대/축소 + 차트 이동 기능이 쉽게 가능해진다. 

# 차트의 기능 - 확대/축소 + 이동

앞서 설명했던 Chart 컴포넌트의 순수 함수 작업은 다른 작업들을 수월하게 해주었다.

레거시 코드를 작성할 때 원래 차트의 확대/축소 그리고 이동 기능을 만들려고 했었다. 하지만 레거시 코드에서 확대와 축소는 이 기능을 동작할 때마다 api호출을 다시 해야했고, 차트의 이동 기능은 상당히 까다롭다. (업비트에서 한번에 200개의 캔들만 제공해준다. 만약 레거시의 방법에서 하려면 차트 컴포넌트에서 데이터 배열 state를 만들고 계속해서 api 요청을 하면서 과거의 차트를 이 state에 붙여나가면서 동시에 현재 렌더링시키는 데이터 배열의 위치를 또 state로 저장해야 한다. 불가능한 것은 아니지만 Chart 컴포넌트가 너무나 복잡해진다.) 

반면 함수형 컴포넌트의 방식으로 코딩을 하면 너무나도 간결하다. 부모 컴포넌트에서 데이터 배열을 계산하고 그 배열을 Chart 컴포넌트로 넘기면 Chart컴포넌트는 그저 받은 데이터 배열을 시각화한다. 이렇게 하면 정말 간다하게 차트에 다양한 기능들을 추가할 수 있다.


