---
layout: single
title:  "윤이나 팬페이지(3) 헤더 섹션- ref 포워딩을 통한 형제 컴포넌트 간의 스크롤 이벤트"
categories : "single-project"
tag : [JS, React, ref]
---

# ref 포워딩을 이용해보자!

  스크롤 이벤트는 이전 포스팅에서 다뤘듯이 이제 큰 문제가 아니다. 요긴한 커스텀 훅도 있기에 말이다. 하지만 이번에 해야 할 것은 조금 더 심화학습이다. 왜냐하면 하나의 컴포넌트가 반응해야하는 대상이 다른 컴포넌트에 있는 DOM 요소이기 때문이다. 이전 포스팅에서는 Carrre.js 컴포넌트 안에서 만들어진 DOM요소를 해당 컴포넌트 안에서 만든 intersectionObserver 객체를 통해 관측했다면, 이번 구현은 App.js 하위에 있는 PhotoSection.js 컴포넌트에서 만든 DOM을 형제 컴포넌트인 Header.js 컴포넌트에서 관리해야한다.

![ref 포워딩 짤]({{site.url}}/images/header_background.gif)

  결과물은 다음과 같은데 기가막히다. 원래 헤더랑 Career 컨텐츠랑 겹쳐보일때 UX가 너무 안좋았다. 그래서 Career가 보일떄는 배경을 주어서 Career 컨텐츠가 안보이게 하고 싶었다. 이를 구현하기 위해선 컴포넌트를 뛰어넘는 상호작용이 필요했다.

  일단 React에서 DOM 정보를 가져오는 가장 좋은 방법은 useRef 훅을 사용하는 것이다. 그리고 이번에는 형제 컴포넌트의 DOM을 가져와야하기 때문에 ref 상속을 하기위한 refForward를 사용할 것이다. 

# 어떻게 구조를 만들어야할까?

ref는 끌어올리기가 안되는 것으로 알고있다. 그래서 PhotoSection.js 컴포넌트에 있는 DOM 정보를 가져오겠다고 PhotoSection.js 컴포넌트에 ref 객체를 만들면 아무것도 못한다. ref는 상속만이 가능하기에 최상위 컴포넌트인 App.js에서 ref 객체를 생성한다.

```js

function App() {
  const ForHeader = useRef();
}
```

그리고 이 ref를 PhotoSection 과 Header 에 상속한다

```js
const PhotoSection = forwardRef((props, ref) => {

  return (
    <PS ref={ref}><PS/>
  )
})
``` 
PhotoSection은 forwardRef를 통해 App에서 온 ref를 받고 받은 ref의 current 값을 PS DOM으로 설정한다. PS는 PhotoSection의 준말로 핑크 배경이 있는 DOM을 나타낸다.  ref의 current 값이 PS가 설정되면 해당 ref의 Header로 상속된 current 값도 같아진다.

```js
const Header = forwardRef((props, ref) => {
  const [white, setWhite] = useState(false);
  function makeBackGround ([entry]) {
    if (!entry.isIntersecting) {
      setWhite(true)
    } else {
      setWhite(false);
    }
  }
  useEffect(() => {
    let observer;
    const current = ref.current

    if (current) {
      observer = new makeBackGround(makeBackGround, {threshold: 0.00001})
      observer.observe(current)
    }
  }, [])
  return (
      <HeaderSection style={white ? {background : 'white'} : {backgound : ''}}>
      <HeaderSection/>
  )
})
```
Header에 상속된 ref값은 <PS>이다. 그리고 위 코드를 보면 알수 있듯이 <PS>를 window가 아주 조금이라도 관측하면 white 값이 false가 되고 관측이 아예 없으면 true가 된다. 일단 ref가 넘어오면 해당 ref의 current를 'current' 변수에 담는다. 다시말해 current에는 ref.current느 <PS> DOM이다. 이 DOM이 대략 0.001%만이라도 관측되었을때 callback이 생긴다. callback인 makeBackGround에서는 makeBackGround가 리턴한 entry 배열을 받아오는데, 배열에는 하나의 객체뿐이다. 그리고 해당 객체에는 isIntersecting 밸류가 있고, 0.001%만이라도 관측되었을때는 true 아니면 false이다. 쉽게 생각해서 window에 핑크 배경이 있으면 true고 없으면 false이다. 그래서 그 상황에 맞게 isIntersecting이 false이면, 즉 핑크 배경이 있으면 배경색이 없고, true면 배경색이 생긴다.


# 역시 내가 사용하는 코드는 이해를 하고 사용해야 한다.!!

사실 이 부분에서 커스텀 훅을 이해하고 사용하는 것의 중요성이 부각되었다고 생각한다.  이전 포스팅에 사용한 커스텀 훅은 ref를 자체적으로 만들지 파라미터로 받아오지 않는다. 만약 훅을 이해하지 못하고 그냥 썼다면 ref를 다른 컴포넌트에서 받아올 때, 그 경우를 해결하지 못할 것이다. 

```js
const useScrollCount = (callback, threshold) => {

    let dom = useRef();
    const handleScroll = useCallback(([entry]) => {
      if(entry.isIntersecting) {
          callback()
      }
    }, []);
    
    useEffect(() => {
      let observer;
      const { current } = dom;
      if (current) {
        observer = new IntersectionObserver(handleScroll, { threshold: threshold });
        observer.observe(current);

        
        return () => observer && observer.disconnect();
      };
    }, [handleScroll, threshold]);
    
      return {
      refCount: dom,
    };
  }
```



