---
layout: single
title:  "윤이나 팬페이지(1) 포토 섹션"
categories : "single-project"
tag : [JS, React]
---

# React 연습은 KLPGA 윤이나 덕질로!

React를 처음 배우면서 이 기술로 무엇인가 만들어보고 싶었던 찰나 2022 KLPGA 맥콜 모나파크 투어에서 멋진 샷을 보여준 윤이나 프로의 팬페이지를 만들어보고자 결심했다. 먼저 다른 골프 프로들의 홈페이지가 있나 찾아봤고, 세계적으로 유명한 맥길로이와 디셈보 선수의 페이지가 있어서 그 둘을 참고해서 만들어봤다.

[맥길로이](http://www.rorymcilroy.com/). <br>
[디샘보](https://brysondechambeau.com/).

앞으로 몇차례에 걸쳐서 블로그에 사용된 기술을 설명하고자 한다. 우선 첫 번째로 보여줄 것은 포토섹션이다.

<img src = '{{site.url}}/images/Ina_PhotoSectiongif.gif' width = '100%' />

이 움짤처럼 사진이 돌아가면서 생기는 것을 구현해보았다. 디샘보의 페이지에서 영감을 얻었는데, react라는 새로운 기술로 구현하려고 하니 쉽지만은 않았다.

# 사진을 어떻게 보여지게 할 것인가?

**!이 파트의 정보는 잘못되었습니다. 수정된 부분을 읽고 싶다면 아래의 포스팅으로 가주세요!** <br>
[윤이나 팬페이지(1-1) 포토 섹션 긴급수정]({{site.url}}/single-project/PhotoSectionFix/}).

이 질문이 기술 구현에 있어서 가장 중요했다. 내가 React를 이용해 생각해본 방법은 두 가지다. 먼저 채택되지 않은 방법은 3장의 사진을 모두 한 번에 랜더링 하고 시간 간격으로 사진의 css요소를 건드려서 하나씩 보이게 하는 방법이었다. 이 방법을 채택하지 않은 이유는 지금까지 내가 React를 공부한 범위 내에서는 미리 사진들을 모두 랜더링해도 style의 state값이 바뀌면, 어차피 전부 다시 랜더링이 된다. (DOM의 style을 변경하는 방법은 state를 이용해야하기 때문이다.)

그렇다면 시간 간격마다 한장씩 새로 랜더링 하는 것이랑 성능차이가 없을 것이라 생각했고, 시간 간격마다 사진 파일을 새로 랜더링 하는 방법을 채택했다. 그래서 이미지 컴포넌트를 이렇게 구성했다.

```js
export default function ImageSlide ({img, fade}) {
  
  return(
    <Images src={img} alt={'사진'} style={ fade ? {
      animationName : 'fadeIn',
      animationDuration: 2000,
      animationIterationCount : 'infinite',
      animationDelay : 0
    } :{animation : 'none'}  }></Images>
  )
}

```
일단 페이드 기능을 위한 style을 제어하는 props는 뒤에서 설명할 것이다. 이미지 컴포넌트는 부모 컴포넌트인 포토 섹션 컴포넌트에 랜더링된다. 

```js
import HiteProfile from "../images/hite_profile_crop-removebg.png"
import Pinkfront from "../images/pink_front_crop-removebg.png"
import WithBall from "../images/withball_crop-removebg.png"

export default function PhotoSection () {
  const photoArr = [HiteProfile, Pinkfront, WithBall];

  return (
    <ImageSlide img={photoArr[index]} fade={fade}/>
  ) 
}
```
포토 섹션 컴포넌트에서 3장의 사진을 import하고 이것으로 배열을 구성한다. 그리고 예상되겠지만, state를 통해 배열의 인덱스를 관리해서 시간 간격마다 다른 이미지를 랜더링하게 의도했다.

이제 중요한 것은 index를 state로 어떻게 관리했는지이다. 

# setInterval() in React

이 메소드는 생각보다 많이 써봤다. 하지만 리액트에서 사용하는 것은 바닐라 스크립트에서 쓰는 것이랑 난이도가 많이 달랐다. 왜냐하면 React는 변화를 감지할때마다 컴포넌트를 새로 랜더링한다. 그 뜻은 setInterval()에서 일정 간격으로 변화를 만들때마다 새로운 setInterval()이 랜더링 된다는 것이다. 시간이 조금 지나면 수십개의 setInterval()이 내 state를 맘대로 가지고 노는 것을 볼 수 있다. 

그래서 나는 uesEffect()를 통해서 setInterval()를 단 한번만 랜더링 시키고자 의도했다.

```js
  const [index, setIndex] = useState(0);
  useEffect(()=> {
    const interval = setInterval(() => {
      if (index < 3) {
       setIndex(index => (index + 1) % 3) 
      } 
    }, 2000)
    return () => clearInterval(interval)
  }, [] )
```
이 코든 비어있는 의존배열을 통해 단 한번만 setInterval()을 실행하게 해준다. 이것만으로도 사실 충분했다. 하지만 디샘보의 페이지를 내가 경험하면서 불편했던 것중 하나가 이미지 슬라이드를 멈추는 기능이 없다는 것이었다. 그래서 나는 pause기능을 넣고 싶었고 새로운 문제가 발생했다.

# setInterval()을 멈추는 방법

사실 setInterval() 멈추는 방법은 간단하다. clearInterval()을 사용하면 된다. 하지만 내가 구현한 기능은 pause이다. 즉 continue가 가능해야 한다. 하지만 clearInterval()을 써버리면 continue가 안된다. 여기서 나눈 두 가지 방식을 고민했다.

첫번째는 pause 상태에서 버튼을 다시 누르면 새로운 setInterval()을 만드는 것이다. 하지만 이것은 쉽지 않은 과정이다. 왜냐하면 onClick이벤트로  setInterval()을 발생시킨다고 가정해보자. 앞서 언급했듯이 React에서 setInterval()은 useEffect의 의존배열을 통해서 관리할 필요가 있다. 따라서 onClick이벤트가 받는 함수 내부에 useEffect를 위치시켜야 하는데, vs코드가 훅을 함수 안에 넣지 말라고 주의를 준다. 그리고 React 페이지에서도 js 함수 내에서 호출하면 컴포넌트 관리가 힘들다고 주의한다. 

그래서 생각한 방법이 delay를 state 값으로 바꾸는 것이었다. 예를 들어, delay 를 2000000000000정도로 바꾸면, 사실상 pause는 아니지만 UX에는 아무런 지장이 없을 것이다. 그리고 이와 관련해서 구글링을 해봤는데, 역시 사람 고민하는게 다 비슷하더라.  

```js
function useInterval(callback, delay) {
    const savedCallback = useRef(); 
  
    useEffect(() => {
      savedCallback.current = callback; 
    }, [callback]);
  
    useEffect(() => {
      function tick() {
        savedCallback.current(); 
      }
      if (delay !== null) {
        let id = setInterval(tick, delay); 
        return () =>  clearInterval(id); 
      }
    }, [delay]); 
  }
```
이 코드는 Dan이라는 개발자가 만든 코드이다. 이 코드는 useEffect의 의존배열에 delay를 넣어서 delay가 바뀔때마다 기존의 setInterval()을 클리어하고 새로운 setInterval()을 만든다. 

사족을 하나 써보자면, 이 코드를 나도 이용했다. 너무 고맙긴하지만 굳이 콜백함수나 ref까지 끌고올 필요가 있나 싶다. 왜냐하면 내 코드에서 콜백 함수가 바뀔일이 없기 때문이다. 여기서 가장 중요한 것은 delay를 관리하는 것이다. 

```js
  const [delay, setDelay] = useState(500);
  const [index, setIndex] = useState(0);

  useEffect(()=> {
    const interval = setInterval(()=> {
      setIndex(index => (index + 1) % 3)
    }, delay)
    return () => clearInterval(interval)
  }, [delay])
```
요런식으로 useEffect()를 구현하고

```js
  function change () {
    if (delay === 200000000) {
      setDelay(2000)
    } else if (delay === 2000) {
      setDelay(200000000)
    }
  }

  return (
    <button onClick={change}>pause</button>
  )
```
요런 식으로 delay를 관리하는 방식으로 구현해도 작동은 한다. 뭐 둘다 작동해서 뭐가 좋은진 모르겠는데, 아무튼 사족이었다. 

# 애니매이션 효과 넣기 (React에서 Dom Style 제어하기)

일단 pause 옆에 있는 오른쪽 버튼은 설명하지 않겠다. 그냥 index를 제어하는 이벤트를 넣으면 되니까. 그보다 나는 애니매이션 효과에 많은 노력을 했다. 바닐라스크립트에서는 참 쉬운 일인데, React는 style 제어하는게 참 껄그럽더라. 그래도 막상 해결하고보니까 쉬운것을 보니, React가 어렵다기보다는 신기술이어서 어색한 것이 맞는 것 같다. 

일단 React에서 Dom의 style을 제어하는 것은 역시 state를 통해서다

```js
const [fade, setFade] = useState(true);

return (
  <ImageSlide img={photoArr[index]} fade={fade}/>
)
```
fade state를 선언하고 이 값을 props를 통해 자식 컴포넌트한테 보낸다. 일단 fade를 제어하는 이유는 pause를 눌렀을 때, 사진은 멈춰있는데 애니매이션만 적용되어서 그렇다. 

``` js
export default function ImageSlide ({img, fade}) {
  
  return(
    <Imgages src={img} alt={'사진'} style={ fade ? {
      animationName : 'fadeIn',
      animationDuration: 2000,
      animationIterationCount : 'infinite',
      animationDelay : 0
    } :{animation : 'none'}  }></Imgages>
  )
}

```
아무튼 props로 받은 자식 컴포넌트에서 fade가 true이면 다음과 같은 스타일을 적용한다. 만약 false면 애니매이션을 끈다. 아 참, fade는 평상시에 true 이다가 pause를 누르면 false가 된다.

# 마치며

중요한 구현은 모두 설명한 것 같다. 아무래도 내 최애에 대한 덕질이다보니까 방망이 깎는 노인의 심정으로 css까지 열심히 했다. 그럼 이만.. 다음 포스팅에서 다른 구현도 설명해보겠다!