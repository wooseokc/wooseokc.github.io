---
layout: single
title:  "업비트 선물거래 - 전역상태 관리에 대한 고민"
categories : "single-project"
tag : [Typescript, React, redux-toolkit]
---

# 리덕스 툴킷을 이용한 전역 상태 관리 (API 호출)

redux의 상태를 업데이트하는 reducer는 순수 함수이다. 이는 외부의 의존하는 api 호출 함수는 reducer가 될 수 없다는 뜻이다. 이 문제를 해결하기 위해 redux에서는 middleware를 설정해서 디스패치된 api 호출 함수를 그대로 reducer에 도달하기 전에 중간에 호출을 하고 결과를 reducer로 보내는 방식이다. 나는 이 아이디어를 듣고 '그냥 api호출을 한 데이터를 dispatch 하면 되는거 아니야?' 라는 생각이 들었다. 쉽게 말해서 미들웨어를 쓰면 dispatch > api호출 > reducer에 호출 데이터 전송 순이라면, 내 아이디어는 api 호출 > 호출 데이터 dispatch > reducer 순인 것이다. 그래서 코드를 다음과 같이 짰다.

```js
  async function apicall() {
    await axios.get(`https://api.upbit.com/v1/ticker?markets=KRW-${coin}`).then(res => {
      setCoinPrice(res.data[0].trade_price)
      dispatch(changePrice(res.data[0]))
    })
  } 
```
일단 이 함수를 보기 전에 이 프로젝트의 계층 관계를 파악할 필요가 있다. 이 웹은 크게 header와 infoSector로 이뤄져있다. 그리고 코인과 주문 정보가 담긴 부분이 infoSector이다. 

```js
export default function InfoSector () {

  return (
    <Provider store={store}>
      <InfoSection>
        <InfoHeader></InfoHeader>
        <CoinInfo></CoinInfo>
        <Chart></Chart>
      </InfoSection>
      <Order></Order>
    </Provider>
  )
}
```
infosector 안에는 order와 infoSection이 있다. 거래 기능이 order 컴포넌트 차트를 비롯한 코인 정보가 infoSection 컴포넌트에 있다. 여기서 코인 정보에 대한 api호출은 order 컴포넌트와 chart 컴포넌트에서 발생한다. 두곳에서 진행하는 이유는 캔들 정보와 시세 정보를 동시에 주는 api를 업비트에서 제공하지 않기 때문이다(반드시 api호출을 두번 해야함!). 아무튼 chart 컴포넌트에서 호출하는 캔들 정보는 해당 컴포넌트에서만 쓰기에 전역 관리를 할 필요가 없다. 하지만 order 컴포넌트에서 호출하는 시세 정보는 infoSector에서 전역으로 관리된다. 

자 다시 order 컴포넌트에 있는 api 호출 함수를 보자.
```js
  async function apicall() {
    await axios.get(`https://api.upbit.com/v1/ticker?markets=KRW-${coin}`).then(res => {
      setCoinPrice(res.data[0].trade_price)
      dispatch(changePrice(res.data[0]))
    })
  } 
  useInterval(apicall, 500)
```
order 컴포넌트에서는 useInterval 이라는 커스텀 훅을 통해 0.5초마다 시세 정보를 받아온다. 그리고 dispatch를 통해 이 시세 정보를 전역적으로 관리한다. 이렇게 하면 dispatch 하는 것이 비동기 함수가 아니기에 middleware가 없어도 된다. 그리고 기능적으로도 아무 문제가 없다. (내가 봤을 때 성능적으로도 차이가 없다. 왜냐하면 api 호출을 middleware를 이용해 reducer 쪽에서 해도 호출하는 횟수는 같다.) 사실 이렇게 아무 문제를 인지하지 못하고 넘어갈뻔 했다.

# createAsyncThunk 와 query

사실 위 처럼 작성하는 코드의 문제점을 처음 인식한 것은 query를 공부하면서다. 이 과정에서 createAsyncThunk를 알게 되었는데  createAsyncThunk는 RTK에 내장된 api로 redux에서 비동기 형태의 dispatch를 할때 생기는 귀찮음을 해결하기 위해 생겼다고 한다. 처음에는 그냥 위에서 내가 한 것처럼 컴포넌트에서 api 호출을 하고 dispatch 하면되는데 굳이 createAsyncThunk를? 이란 생각이었다. 근데 어제 css 리팩토링 하면서 깨달음을 얻었다. 

# api 호출이 많다면? 컴포넌트가 많다면?

내 프로젝트에는 api 호출이 단 두개이다. 그리고 컴포넌트도 매우 적다. 따라서 코인 시세를 호출하는 함수를 쉽게 찾을 수 있다. 하지만 만약 api 호출 함수가 10개가 넘고, 컴포넌트가 100개가 넘어간다면? 나도 찾기 힘들고 특히 협업할 때 팀원들은 아주 고역에 빠질 것이다. 만약 api 호출을 createAsyncThunk reducer에 넣어놓으면 이 스토어가 관리하는 컴포넌트들에서 api 호출을 이런 것들이 있구나 쉽게 파악할 수 있을 것이다.

PS 근데 구현이 쉽지 않다.... 구현을 하고 다음 블로그를 작성하기로...