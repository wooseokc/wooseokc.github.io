---
layout: single
title:  "업비트 선물거래 - 서버와의 통신 방법"
categories : "single-project"
tag : [Typescript, React, websocket ,redux-toolkit]
---

# 시세 정보를 서버에서 받아오는 방법

클라이언트에서 백엔드 서버와 데이터를 주고 받는 방법은 rest api를 사용하거나 websocket을 사용하는 것이다. 그 중에서 자산의 시세 데이터를 받아올 때 사용해야 하는 방법은 무조건 websocket이다. 왜냐하면 rest api는 클라이언트가 요청을 보내야 서버가 응답하는데, 매 순간 시세 정보를 받아오려면 거의 뭐 10ms마다 요청을 보내야한다. 이것은 매우 효율적인 방법이 아니다. websocket을 이용하면 클라이언트에서 소켓에 접속했을 때, 데이터가 바뀔 때마다 서버에서 알아서 데이터를 보내준다.

# REST API에서 Websocket으로!

이렇게 websocket이 적절한데, 그렇다면 왜 기존에 나는 Get 요청을 interver api를 이용해서 반복적으로 보냈었나? 왜냐하면 websocket이 있는지도 몰랐다. 그래도 알게 된 후에 언젠간 고쳐야지 하고 마음 먹고 있다가 드디어 고쳤다.


```js
const dispatch = useAppDispatch()

  useInterval(() => {
    dispatch(coinApiCall())
  }, 500)
```
기존의 코드는 이렇게 0.5초마다 get 요청을 업비트에게 날렸다. (업비트야 미안해~)
저 coinApiCall()은 비동기 데이터를 전역적으로 redux로 관리하기 위해 createAsyncThunk로 만들어논 함수다. 이전 포스팅에 나와있다.

아무튼 이 코드를 web소켓으로 바꿨다. 
```js
  const dispatch = useAppDispatch()
  // 소켓에서 받은 데이터를 redux로 관리하기 위한 코드
  const coin = useAppSelector((state) => state.coin.now);
  // btc와 eth중 지금 무슨 코인의 정보를 받아와야하는 지 알아야함

  useEffect(() => {
    const webSocket = new WebSocket('wss://api.upbit.com/websocket/v1');
    webSocket.binaryType = 'arraybuffer';
    webSocket.onopen = () => {
      const str = [{"ticket":"test"},{"type":"ticker","codes":[`KRW-${coin}`]}]
      webSocket.send(JSON.stringify(str))
      console.log('connect')
    }

    webSocket.onmessage = (evt) => {
      let enc = new TextDecoder("utf-8");
      let arr = new Uint8Array(evt.data);
      let data = JSON.parse(enc.decode(arr));
      dispatch(changePrice(data))
    }

    return () => {
      webSocket.close()
    }
  }, [coin])
```
일단 가장 중요했던 부분은 소켓의 생명주기이다. BTC에서 ETH로 바꿨을 때 소켓 해제를 적절하게 해주지 않으면 시세가 뒤죽박죽이 된다. 그 외에 어려웠던 점은 소켓으로 upbit가 쏘는 데이터가 상당히 파악하기 힘든 형식이었다. 그래서 구글링을 해본 결과 디코딩을 해야함을 알았고 디코딩 과정을 추가했더니 관리하기 쉬운 데이터가 되었다.

# PS
차트에는 websocket을 사용하는 것이 적절하지 않다. 그리고 upbit에서 차트 데이터를 주는 소켓을 열어주지도 않는다.

# 결론

사실 버린 프로젝트라 고칠까 말까 고민을 많이 했는데, 막상 해결하니까 마음이 상당히 가볍다. 굿!