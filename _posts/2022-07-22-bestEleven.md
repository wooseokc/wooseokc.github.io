---
layout: single
title:  "축구 포지션 만들기 - 리액트 리덕스 실습"
categories : "single-project"
tag : [JS, React, redux]
---

# 전역 상태 관리

리액트에서는 전역 상태 관리를 하는 것이 조금 까다롭다. 페이지를 컴포넌트단으로 분리해서 제작하기 때문에 컴포넌트가 다른 곳에서 같은 상태를 관리하고 싶다면 특별한 장치를 이용해야한다. 가장 기본적인 것은 'state 끌어올리기'일것이다. 근데 이것의 단점은 너무나도 명확하다. 상태 관리를 해야하는 컴포넌트다 한 단계 (부모-자식 관계)면 괜찮은데, 만약 여러단계를 넘어간다면 그거 하나하나 props로 넘기기 귀찮다. 그것을 해주려고 있는 것이 context api이다. context api를 통해 전역적으로 상태를 관리하고, 상태를 바꾸고 싶다면 reducer api를 이용해서 바꾸면 된다. 

# why redux?

그렇다면 굳이 왜 외부 라이브러리인 리덕스를 이용하는 이유가 있을까? 많은 이유가 있겠지만 성능이 그 중 하나이다. 리액트에서 렌더링을 효율적으로 하는 것은 성능에 매우 중요하다. redux는 컴포넌트의 상태에 변화가 있을 때만 해당 컴포넌트를 랜더링한다. 예를 들어 전역 상태 a,b가 각각 컴포넌트 A, B에 있다고 생각해보자. 컴포넌트 C에서 reducer를 이용해 b를 변경했다고 가정하면, context api에서는 A와 B가 모두 렌더링된다. 이것을 방지하려면 Provider를 따로 만들어야한다. 하지만 리덕스는 상태에 변화가 있는 경우에만 렌더링이 된다. 만약 C에서 reducer를 이용하더라도 a,b에 변화가 생기지 않는다면 A와 B는 다시 렌더링되지 않는다. 이 부분은 추후에 프로젝트를 통해 다시 보이도록 하겠다.

# 그래서 코드는 ?

일단 상위 컴포넌트인 app.js 아래 board.js와 input.js가 있다. input에서 reducer를 통해 전역 상태인 player Object를 관리한다. 사실 이정도 규모면 state 끌어올리기를 해도 되는데 그냥 연습해봤다.

```js
function reducer (currentState, action) {
  if (currentState === undefined) {
    return {
      player : {},
    }
  }
  const newState = { ...currentState };
  if (action.type === 'addPlayer') {
    if (action.payload !== '') {
      newState.player[action.payload[1]] = action.payload[0]
      console.log(newState.player)
      console.log(action.payload)
    }
  }
  if (action.type === 'deletePlayer') {
    delete newState.player[action.payload]
  }
  return newState;
}
const store = createStore(reducer)
```
일단 리듀서와 스토어를 app.js에 만든다.

```js
export default function Input (props) {
  const [index, setIndex] = useState(0)
  const dispatch = useDispatch()
  let ref = useRef()
  function buttonCLick () {
    const a = [ref.current.value, index]
    dispatch({type : 'addPlayer', payload : a}) 
    ref.current.value = ''
    setIndex(index => index +1)
  }
  function enterPress (e) {
    if (e.key === 'Enter') {
      buttonCLick(); 
    }
  }
  return (
    <>
      <input ref={ref} type={'text'} onKeyPress={enterPress}></input>
      <button onClick={buttonCLick} type={'submit'}></button>
    </>
  )
}
```
그리고 input.js 에서 선수 이름을 입력할때 그 값을 dispatch를 통해 리듀서로 보낸다.

```js
export default function Board(props) {
  const player = useSelector(state => Object.keys(state.player).length)
  const players = useSelector(state => state.player)
  let posX = 0;
  let posY = 0;
  const dispatch = useDispatch()

  function dragStartHandler (e) {
    const img = new Image();
    e.dataTransfer.setDragImage(img, 0, 0);
    posX = e.clientX;
    posY = e.clientY;
  }
  function dragHandler (e) {
    posY = e.clientY;
    posX = e.clientX;

    if (posX > 380 || posY > 610) {
      console.log('yeah')
      dispatch({type : 'deletePlayer', payload : e.target.id}) 
    }
    e.target.style.top = `${e.clientY-45}px`;
    e.target.style.left = `${e.clientX-35}px`;
  }

  const play = Object.entries(players).map((entry) => {
    console.log(`entry -> ${entry}`)
    return (
      <Players draggable id={entry[0]} key={entry[0]} onDrag={dragStartHandler} onDragEnd={dragHandler}> {entry[1]} </Players>
    )
  })
  return (
    <FootballField >
      {play}
    </FootballField>
  )
}
```
그리고 border.js 에는 전역 state를 받아와서 해당 state가 가지고 있는 player를 DOM으로 만들어서 렌더링한다.

# 결과물

 ![국대스쿼드 만들기]({{site.url}}/images/addplayer.gif)
 이렇게 대한민국 국가대표 스쿼를 만들수 있다. 그리고 제거하는 것도 구현했다. 누구나 국가대표가 되고 싶은 꿈이 있으니까 황의조를 보내고 나를 넣어보겠다. 

 ![나 넣기]({{site.url}}/images/delete.gif)

# 리덕스의 성능

board.js 코드를 보면 재밌는 코드가 한줄있다

```js
const player = useSelector(state => Object.keys(state.player).length)
```
이 코드인데 변수 'player'는 선언되나 사용되지 않는다. 그렇다면 player를 왜 선언해야할까. 아까 redux를 사용하는 이유에서 말했지만 redux는 state가 바뀌지 않으면 렌더링 되지 않는다. 그리고 board에서 사용하는 state는 state.player로 객체이다. 그리고 reducer를 보면 알 수 있지만, reducer는 state.player 라는 객체를 바꾸지 않는다. 그저 객체에 인자를 하나 추가할 뿐이다. 객체는 참조타입이다. 따라서 안에있는 요소가 바뀌더라고 객체 자체는 바뀌지 않는다.(같은 주소값을 지닌다는 뜻). 따라서 board를 state.player 라는 객체에 요소가 추가될때마다 렌더링 시키려면 state.player 라는 객체의 바뀌는 속성을 board 컴포넌트에서 state로 가져와야한다. 나는 객체의 크기를 가져왔다. 만약 위 코드를 주석처리한다면 reducer를 통해 state.player 객체에 아무리 많은 요소를 넣어도 board는 렌더링되지 않는다. 

input.js 와 board.js에 렌더링이 된다는 콘솔 요소를 넣고 실험을 해보겠다.

위 코드를 주석처리하면
 ![노렌더링]({{site.url}}/images/norender.gif)
다음과 같이 전혀 board가 렌더링 되지 않는다. state.player가 늘어나는데도 말이다.

하지만 다시 위 코드를 넣으면
 ![렌더yes]({{site.url}}/images/yesrender.gif)
다시 board가 렌더링되기 시작한다.

# 마치며 

기본 redux를 사용해서 프로젝트를 해봤다. 이제 toolkit도 배워서 써보고 싶은데 recoil이 더 좋아보인다. 다음에는 저 둘 중 하나를 써봐야지..!! 

# PS

이것은 redux랑은 상관 없는데 tmi를 하나 적자면, 원래 state.player를 배열로 만들었다. 그렇게 배열을 map으로 펴서 DOM을 만들었을때, 예를들어 배열이 [1,2,3,4]고 <div>1<div>, <div>2<div>, <div>3<div>, <div>4<div>가 만들어졌다고 해보자. 여기서 <div>3<div>의 스타일은 변경하고 배열에서 '3'을 splice 시키면 <div>3<div>의 스타일값이 <div>4<div>한테 간다. 변경된 style 값이 DOM을 직접건드리는게 아니라, map 함수의 몇번째 DOM을 간접적으로 바꾼다. 이 부분은 주의할 필요가 있다. 