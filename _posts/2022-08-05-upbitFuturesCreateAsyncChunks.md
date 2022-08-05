---
layout: single
title:  "업비트 선물거래 - 전역상태 관리에 대한 고민. Solution 1 (createAsyncThunk)"
categories : "single-project"
tag : [Typescript, React, redux-toolkit]
---

# createAsyncThunk 구현 (리팩토링)

이 작업이 쉬워보이면서도 생각보다 껄끄러운 작업이었다. 아예 처음부터 새로하는 것이면 괜찮은데 기존에 완성된 프로젝트의 상태(변수)들을 잘 연결하는 과정이 중요했기에 무턱대도 코딩을 할 수가 없었다. 그나마 다행인 것은 전역으로 관리가 되어있는 상태를 디루는 컴포넌트는 어차피 store에서 값을 가져오기에 store값만 잘 세팅하면 되었다. 조금 골치가 아픈 부분은 api 호출을 하는 컴포넌트였는데, 이 부분도 컴포넌트 자체에서 설정하던 state를 store에서 가져오게 함으로써 해결 할 수 있었다.

# 기본 골자 변경

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
기존의 프로젝트는 store로 InfoSector만을 감쌌다. 하지만 이제 api 호출을 이 컴포넌트에서 해야 하기에 store를 더 높은 컴포넌트로 옮겼다. 사실 이럴 필요는 없지만, 코드를 짜는 과정에서 뭔가 store와 dispatch를 분리하는 것이 더 깔끔해보였다. 

그렇게 store를 최상위인 app 컴포넌트로 옮기고
```js
function App() {
  return (
    <> {
      window.innerWidth > 600 ? 
      <>
        <GlobalStyle />
        <Provider store={store}>
          <Header />
          <InfoSector/>
        </Provider>
      
      </> :
      <div>Please use Upbit futures in PC</div>
    }
    </>
  );
}
```
infoSector에는 dispatch를 줬다. 

```js
export default function InfoSector () {
  const dispatch = useAppDispatch()
  useInterval(() => {
    dispatch(coinApiCall())
  }, 500)

  return (
    <>
      <InfoSection>
        <InfoHeader></InfoHeader>
        <CoinInfo></CoinInfo>
        <Chart></Chart>
      </InfoSection>
      <Order></Order>
    </>
  )
}
```
이렇게 dispatch로 비동기 함수인 coinApiCall()을 보낼 수 있는 이유는 reducer를 slice에서 만들때 createAsyncThunk를 이용했기 때문이다.

```js
let nowCoin = 'BTC'

const coinApiCall = createAsyncThunk(
  'coinName/apiCall',
  async () => {
    const res = await axios.get(`https://api.upbit.com/v1/ticker?markets=KRW-${nowCoin}`)
    return res.data[0]
  }
)

const initialState : coinState = { now : 'BTC', price : {} }

const coinSlice = createSlice({
  name : 'coinName',
  initialState : initialState,
  reducers: {
    changeCoin : (state, action: PayloadAction<string>) => {
      state.now = action.payload;
      nowCoin = action.payload;
    }
  },
  extraReducers : (builder) => {
    builder.addCase(coinApiCall.fulfilled, (state, action : PayloadAction<object>) => {
      state.price = action.payload;
    })
  }
})
```
슬라이스를 만들면서 한가지 골치가 아팠던 것이 api 호출 url에 전역 변수가 들어가는 것이었다. 예를 들어 BTC와 ETH에 따라 시세를 호출하는 url이 달라진다. 처음에는 coinSlice의 state에 접근하려고 했는데 당연히 불가능했다. 고민을 하다가 코인이름 변수는 jsx에 랜더링 될 일도 없고 그냥 let으로 선언을 하고 reducer가 전역 state를 바꿀때 코인 이름도 바꾸면 쉽게 해결 할 수 있다는 것을 생각해냈다. 그렇게 해결!

# 결론

기능에는 아무런 변화가 없지만... 뭐 스스로 코드가 깔끔해졌다고 생각하고있다..ㅎ;;