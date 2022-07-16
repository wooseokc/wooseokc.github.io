---
layout: single
title:  "윤이나 팬페이지(1-1) 포토섹션 성능 개선 (긴급 수정!)"
categories : "single-project"
tag : [JS, React, ref]
---

# 성능 결함을 발견하다!

[윤이나 팬페이지(1) 포토 섹션]({{site.url}}/single-project/Ina_PhotoSection/}).

이 프로젝트 첫번째 포스팅에서 나는 '사진 이벤트를 어떻게 구현할까'라는 질문을 던졌다. 하지만 위 포스팅에서 내렸던 답은 틀렸다. 위 포스팅에서 나는 react는 변화가 있을때마다 랜더링을 하기 때문에 세 장의 사진을 한번에 랜더링하고 특정 시간마다 style값을 바꾸는 방법과 특정시간마다 한 장의 사진을 랜더링 하는 방법에 성능차이가 없다고 생각했다. 왜냐하면 세 장의 사진을 처음에 모두 렌더링해도 스타일값이 바뀔때마다 또 렌더링이 일어나기 때문이다. 하지만 이 추측은 틀렸다.

![네트워크 문제]({{site.url}}/images/image_networks.gif)

위 짤은 특정시간마다 사진을 랜더링하는 것이고, 이것을 보면 사진이 바뀔때마다 네트워크 통신이 이뤄지는 것을 볼 수 있다. 이렇게 매번 네트워크 통신이 이뤄지는 코드는 아래와 같다. 

```js
import HiteProfile from "../images/hite_profile_crop-removebg.png"
import Pinkfront from "../images/pink_front_crop-removebg.png"
import WithBall from "../images/withball_crop-removebg.png"
const function PhotoSection () {
  const photoArr = [HiteProfile, Pinkfront, WithBall];

  return (
    <ImageSlide img={photoArr[index]} fade={fade}/>
  )
}
```
이렇게 만들면 index가 바뀔때마다 네트워크는 사진파일을 새로 불러온다. 내가 잘못 생각했던 것은 배열에 이미지 파일을 모두 불러왔다고 생각했다. 하지만 생각해보면 브라우저는 이미지 DOM을 새로 렌더링하려면 서버에서 다시 가져오는 방법밖에는 없을 것이다. 따라서 나는 한번 가져온 이미지 컴포넌트를 지우는 것이 아니라 css를 이용해 보이지만 않게 만들어야 성능에서 더 좋은 퍼포먼스를 낼 수 있다고 생각했다. 그렇게 코드 전체에 대한 대규모 공사가 이루어졌다.

컴포넌트 구성이 App - PhotoSection - ImageSlide 이고, img DOM은 ImageSlide에 있기 때문에 인라인으로 스타일을 주기 위해서는 거의 모든 코드를 PhotoSection에서 ImageSlide 로 옮겨야 했다. 

```js
  <Imgages src={photoArr[0]} style={index === 0 ? {visibility : 'visible'} : {visibility : 'hidden'}} fade={fade}></Imgages>
      <Imgages src={photoArr[1]} style={index === 1 ? {visibility : 'visible'} : {visibility : 'hidden'}} fade={fade}></Imgages>
      <Imgages src={photoArr[2]} style={index === 2 ? {visibility : 'visible'} : {visibility : 'hidden'}} fade={fade}></Imgages>

```
그렇게 탄생한 코드이다. 블로깅을 위헤 코드를 풀었고, map을 이용하면 더 쉽게 만들 수 있다. 이렇게 되면 style 속성이 바뀔때 렌더링이 일어나지만, 그 부분에서만 렌더링이 일어나기에 사진 파일을 새로 불러오지 않는다. 그리고 결과적으로도 index가 바뀔때마다 네트워크 통신도 발생하지 않았다.

#마무리

이 문제를 발견하게된 이유는 윤이나 프로에 대한 네이버 뉴스 api를 받아오는 구현을 하고 싶었다. 그 과정에서 CORS라는 어마어마한 산을 만났고, 네트워크 창에서 뭐가 문제인지 탐색을 하고 있었다. 그런데 갑자기 사진을 index 값이 바뀔때마다 새로 받아오는 것이었다. 성능에는 큰 문제가 없었지만, 그래도 더 좋은 구현이 하고 싶었고 그렇게 대규모 공사가 이뤄졌다.