---
layout: single
title:  "시간 멈추기 게임"
---

# 스탑워치 원리를 이용해 만든 시간 멈추기 게임

기존의 과제는 스탑워치를 만드는 것이지만 그냥 스톱워치는 너무 심심해서 게임을 추가해보았다.
이 프로젝트는 복잡한 것은 없고, 1. 타켓 설정하기 2. 사용자가 시간 멈추기. 3.결과 나타내기로 볼 수 있다.

일단 완성화면은
![20220619_140012](https://user-images.githubusercontent.com/99978225/174466867-3ae92c25-8927-4ea3-a319-f9b354285aad.png)

howtoplay에 마우스를 올리면 게임 방법이 나오고 play를 누르면 타켓 시간이 나온다.
![20220619_140025](https://user-images.githubusercontent.com/99978225/174467009-45f7f603-b39e-49bf-990b-f024fe0a1eec.png)
도전을 누르면 
![20220619_140038](https://user-images.githubusercontent.com/99978225/174467076-5ecc59ca-fe36-4150-a853-41a1938c9ab3.png)

결과는 다음과 같다 
![20220619_140056](https://user-images.githubusercontent.com/99978225/174467116-b71443ab-be42-4a40-a372-a2d6862e5372.png)

성공의 기준은 타겟을 기준으로 +- 0.5면 된다. (원래는 0.1초로 했는데 너무 어렵다)

일단 타겟 세팅은 초와 밀리초에 랜덤 숫자를 만드는 방법으로 설정했다.

``` js
const $mainBox = document.querySelector('.mainBox');
const $playButton = document.querySelector('.play')

function targetSetting () {
  let second = Math.floor(Math.random()*100)
  while(second >=10 || second <= 5) {
    second = Math.floor(Math.random()*100)
  }
  const milliSeconds = Math.floor(Math.random()*1000)
  return `${second} ${milliSeconds}`
}
```
그리고 타겟은 5초와 10초 사이로만 나오게 설정했다. 

두번째로 사용자의 시간 멈추기가 스톱 워치 기능을 사용한 부분인데, 

```js
    //타이머 on
    const startTime = new Date();

    // 10초 지나면 실격
    let interval = setInterval(()=> {
      const stopTime = new Date();
      let startSecond = startTime.getSeconds();
      let startmilli = startTime.getMilliseconds();
      let stopSecond = stopTime.getSeconds();
      let stopmilli = stopTime.getMilliseconds();
      if (startmilli > stopmilli) {
        stopSecond -= 1;
        stopmilli +=1000;
      }
      if (stopSecond < startSecond) {
        stopSecond += 60;
      }
      const gameSecond = stopSecond - startSecond
      const gameMilli =stopmilli -startmilli;
      if (gameSecond >= 10) {
        clearInterval(interval);
        timeOut();
      }
    },100)


    //타이머 off 
    const $stopButton = document.querySelector('.stop');
    function gameStop () {
      // 타임 stop
      clearInterval(interval);
      const stopTime = new Date();
      let startSecond = startTime.getSeconds();
      let startmilli = startTime.getMilliseconds();
      let stopSecond = stopTime.getSeconds();
      let stopmilli = stopTime.getMilliseconds();
      if (startmilli > stopmilli) {
        stopSecond -= 1;
        stopmilli +=1000;
      }
      if (stopSecond < startSecond) {
        stopSecond += 60;
      }
      const gameSecond = stopSecond - startSecond
      const gameMilli =stopmilli -startmilli;

      // 결과 결정
      let targetSum = Number(timeArr[0]*1000) + Number(timeArr[1]);
      let timeSum = (gameSecond*1000) + gameMilli;

      let gap = Math.abs(targetSum-timeSum)

      if (gap <=500) {
        success(gameSecond, gameMilli);
      }
      else {
        fail(gameSecond, gameMilli)
      }
    }
    $stopButton.addEventListener('click', gameStop)
  }
```
기본적으로 원리는 도전을 시작할때 new Date()를 설정하고 멈춰! 버튼을 눌렀을 때 새로운 Date 객체를 만들어 이 둘의 차이를 표현하는 방식이다. 딱히 특별한 것은 없다. 굳이 찾자면 스톱워치랑 다르게 이 프로젝트에서는 두 Date 객체의 차를 구해야 해서 넘김을 처리해줄 필요가 있다.  

```js
  if (startmilli > stopmilli) {
        stopSecond -= 1;
        stopmilli +=1000;
      }
      if (stopSecond < startSecond) {
        stopSecond += 60;
      }
```
요런 부분이다.

마무리하며
---------

이번 과제를 하면서 ui 디자인 하는 것이 참 어렵다는 것을 느꼈다. 색을 어떻게 써야 싼마이(?)가 안나는지 감이 안온다.... 그리고 js 코드 모듈 분리를 안해놔서 보기가 좀 어렵다. 다음 과제부터는 분리해서 작성해보도록 노력해보겠다. 
