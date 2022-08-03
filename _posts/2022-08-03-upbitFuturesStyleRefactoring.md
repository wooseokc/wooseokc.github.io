---
layout: single
title:  "업비트 선물거래 프로젝트 스타일 컴포넌트 리팩토링"
categories : "single-project"
tag : [Typescript, React, redux-toolkit]
---

# 리팩토링 계기

어제 우연히 유튜브에서 Html Css에 관련된 영상을 보았다. 이 영상의 내용은 html css를 성의있게 작성하라는 것인데, 그 이유는 개발자 본인이 편하기 때문이다. 나중에 협업을 하거나 보수를 할때 개떡같은 css는 알아보기가 힘들어 수정하기 힘들기 때문! 그래서 업비트 선물거래 코드의 스타일 컴포넌트를 봤는데 유투브 영상에서 말하는 하지 말아야 하는 것들을 꽤나 많이 하고 있었다.


# 1. 인라인 스타일 넣기

``` js
    <HeaderSection>
      <Logo></Logo>
      <Nav>
        <NavItem hove={false}>거래소</NavItem>
        <NavItem hove={false}>입출금</NavItem>
        <NavItem hove={false}>투자내역</NavItem>
        <NavItem hove={false}>코인동향</NavItem>
        <NavItem hove={false}>스테이킹</NavItem>
        <NavItem hove={false} style={{position: 'absolute', left : 625, bottom: 19}}>NFT</NavItem>
        <NavItem hove={false} style={{position: 'relative', left : 58}}>고객센터</NavItem>
        <NavItem style={{color : '#fff', position: 'relative', left : 55}} hove={def}>선물거래</NavItem>
      </Nav>
    </HeaderSection>
```

예를 들어 이런 부분인데.. 어차피 반응형을 의도하지도 않았고 좀 조잡한 세밀한 설정이기에 그냥 인라인으로 때려넣었었다. 하지만 인라인으로 설정을 하면 나중에 인라인으로 설정한 것을 깜빡했을 때 큰 문제가 생긴다. 그래서 전부 props로 넘겨서 스타일 컴포넌트에서 처리하게 할 것이다. 

```js
    <HeaderSection>
      <Logo></Logo>
      <Nav>
        <NavItem hove={false}>거래소</NavItem>
        <NavItem hove={false}>입출금</NavItem>
        <NavItem hove={false}>투자내역</NavItem>
        <NavItem hove={false}>코인동향</NavItem>
        <NavItem hove={false}>스테이킹</NavItem>
        <NavItem hove={false} character='NFT'>NFT</NavItem>
        <NavItem hove={false} character='고객센터'>고객센터</NavItem>
        <NavItem hove={def}  character="선물거래">선물거래</NavItem>
      </Nav>
    </HeaderSection>
```
아주 깔끔하다! 

스타일 컴포넌트의 경우
```js
export const NavItem = styled.div<{hove : boolean, character? : string}>`
  white-space : nowrap;
  font-size: 15px;
  font-weight: 400;
  margin-left : 35px;
  cursor: pointer;

  ${(props) => (props.character === 'NFT' && {position : 'absolute' , left : 625, bottom: 19})}
  ${(props) => (props.character === '고객센터' && {position: 'relative', left : 58})}
  ${(props) => (props.character === '선물거래' && {color : '#fff', position: 'relative', left : 55})}
  &:hover {
    font-weight: ${(props) => (props.hove === true ? '400' : '700')}
`
```
이렇게 했다. props받는 부분은 깔끔하지만 일반 속성을 정의하는 부분은 너무 논리적이지 못하고 중구난방이다. 나는 이부분도 상당히 중요하다고 본다. 왜냐하면 어떤 속성은 상속받는 것이 있고 아닌 것이 있는데, 사이즈 속성과 배치 속성 같은 것이 뒤죽박죽이면 무엇이 상속받았고 아닌지 파악하기 힘들다. 만약 모든 스타일 컴포넌트에 사이즈 속성 > 배치 속성 순으로 작성되어 있을때 사이즈 속성이 없다면 사이즈 속성은 부모컴포넌트에서 상속받는 다는 생각을 쉽게 할 수 있다. 그래서 나는 css 속성을 두 가지로 분류(1차속성과 2차속성)했다. 1차 속성은 DOM자체가 지니는 속성(사이즈 컬러 hover 등등) 2차 속성은 상대적인 위치를 정하는 속성(postion display margin padding) 등등

```js
export const NavItem = styled.div<{hove : boolean, character? : string}>`
  font-size: 15px;
  font-weight: 400;
  
  cursor: pointer;

  margin-left : 35px;
  white-space : nowrap;

  ${(props) => (props.character === 'NFT' && {position : 'absolute' , left : 625, bottom: 19})}
  ${(props) => (props.character === '고객센터' && {position: 'relative', left : 58})}
  ${(props) => (props.character === '선물거래' && {color : '#fff', position: 'relative', left : 55})}
  &:hover {
    font-weight: ${(props) => (props.hove === true ? '400' : '700')}
`
```
이렇게 수정하면 딱 보는 순간 NavItem은 사이즈를 부모에게서 상속받거나 내용물 크기에 따라 자동으로 정해지게 의도되었다는 것을 쉽게 알 수 있다. 
