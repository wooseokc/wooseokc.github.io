---
layout: single
title:  "윤이나 팬페이지(4) 뉴스 섹션과 배포"
categories : "single-project"
tag : [JS, React, ref]
---

# 클라이언트에서 외부 api가져오기 + cors 에러

 저번에 업비트 오픈 api에서 크립토 시세 정보를 가져온적이 있었는데, 그 당시에는 아무런 어려움이 없었다. 하지만 이번에 네이버 오픈 api를 이용해 뉴스 정보를 가져오려고 하자, 콘솔창에 시뻘건 cors 에러가 뜨기 시작했다. 일단 cors에러가 무엇인지 알아야헸다.

 CORS는 Cross-Origin Resource Sharing의 약자이다. 초보 개발자 입장에서는 매우 화나는 존재였지만, 머리 좋은 사람들이 만들어놓은 것이기에 분명 필요한 존재일 것이다. 내가 이해한바로는 cors가 필요한 이유는 보안을 위해서이다. 내가 이해한 것을 바탕으로 예시를 만들어보자면...

 내가 만약 인스타그램에 로그인을 해놓은 상태로 실수로 이상한 사이트(www.weird.com)에 접속했다고 가정해보자. 그 이상한 사이트는 쿠키에 저장된 인증정보를 이용해서 인스타그램에 이상한 글을 올리라는 post 요청을 보낸다고 가정해보자. 그러면 나는 내 의지와 상관없이 sns에 이상한 글이 등록되게 된다. 이것 막기위해 존재하는 것이 CORS라고 보면된다. 현재 인스타그램은 브라우저 쿠키에 저장된 내 인증정보를 통해 로그인이 되어있다. 그리고 이상태에서 내가 글을 쓰면 내 클라이언트에서 post요청이 인스타그램 서버로 보내지고 인스타그램 서버에 내 글이 저장될 것이다.(이 부분은 확실치 않다...) 추가적으로 로그인이 되어있기 때문에 아마 post 요청의 헤더에 클라이언트 id와 password가 실려서 보내지지 않을까 싶다. 하지만 이 부분만으로 보안이 이뤄진다면, www.weird.com 에서 다운받은 자바스크립트가 인스타그램에 보내는 post 요청도 받아지게 된다. 왜냐하면 브라우저 쿠키에 내 인증정보가 있으니까. 이것을 막기 위해 있는 것이 CORS에 대한 제한이다. 인스타그램 서버와 정보를 교환하려면 클라이언트의 도메인 출처가 서버와 같아야한다. 예를 들어 내가 인스타그램에 로그인하면 클라이언트의 주소는 www.instagram.com 이다. 근데 만약 www.weird.com에서 인스타그램 서버로 post 요청을 받으면 Cross-Origin Resource Sharing 제한이 걸려있기 떄문에 에러가 발생하게 된다. 따라서 이것을 해결하려면 Cross-Origin Resource Sharing 제한을 풀거나 같은 출처로 요청을 하거나 아니면 브라우저가 아닌 백엔드 쪽에서 요청하면 된다.

# HOW TO SOLVE?

 위 세가지 방법 중에서 첫 번째는 내가 할 수가 없다. 저거는 네이버만이 가능한 것이기에 제외한다. 그럼 두번째와 세번째가 있다. 처음에는 두번째 방법으로 하려고 했다. 프록시 서버를 이용해서 도메인을 우회해서 바꾸는 방법인데, 이 방법은 나중에 배포를 했을 때 안통한다고 한다. 그래서 나는 세번째 방법으로 접근했다. 백엔드는 CORS제한이 안걸리는 이유는 간단하다. CORS제한은 브라우저에서 실행되는 시스템이기 때문이다.

 따라서 방법은 간단하다. 백엔드 코드를 이용해서 서버를 만든다. 그리고 해당 서버에서 네이버에 get 요청을 보낸다. 서버가 네이버에서 받은 정보를 다시 send한다. 중요한 것은 localpost:8080 과 localpost :3000 도 출처를 교차하는 것이기 CORS 제한을 풀어야한다. 그리고 서버에서 정보를 클라이언트에서 받아온다. 

 ```js
var express = require('express');
var app = express();
const cors = require('cors');

app.use(cors({
  origin: '*'
}));

var client_id = '=';
var client_secret = '=';

app.get('/blog', function (req, res) {
  var api_url = 'https://openapi.naver.com/v1/search/news?sort=sim&display=5&query=' + encodeURI('윤이나'); // json 결과
  var request = require('request');
  var options = {
      url: api_url,
      headers: {'X-Naver-Client-Id':client_id, 'X-Naver-Client-Secret': client_secret}
   };
  request.get(options, function (error, response, body) {
    if (!error && response.statusCode == 200) {
      res.writeHead(200, {'Content-Type': 'text/json;charset=utf-8'});
      res.end(body);
    } else {
      res.status(response.statusCode).end();
      console.log('error = ' + response.statusCode);
    }
  });
});

app.listen(8080, function () {
  console.log('app listening on port 8080!');
});

 ```

 이것이 백엔드 서버이다. 사실 내가 한건 없다. 네이버에서 백엔드 코드를 친절하게 제공해준다... (네이버 ♥) 하지만 이렇게 해도 클라이언트에서 불러올때 서버랑 출처가 달라지기에 

 ```js
app.use(cors({
  origin: '*'
}));
 ```
 이거는 내가 추가했다. 
 이렇게 하면 다음과 같은 결과가 나온다.

 ![apicall]({{site.url}}/images/news_section_apicall_1.png)

 그리고 이 정보는 모든 출처에서 접근이 가능하다. 그래서 클라이언트에서 이렇게 접근한다.

 ```js
const News = forwardRef((props, ref) => {
  const [newArr, setNewsArr] = useState([])
  async function getNews() {
    await axios('http://localhost:8080/blog').then(res => {
      // const obj = JSON.parse(res.data.body)
      setNewsArr(res.data.items)
      console.log(res.data.items)
      
    }) 
  }
  useEffect(() => {
    getNews()    
  // eslint-disable-next-line react-hooks/exhaustive-deps
  }, [])

  const NewsList = newArr.map((index) => (
    <NewsItem href={index.link} target={"_blank"} rel={'noopener'} key={index.title}>
      <NewsTitle dangerouslySetInnerHTML={ {__html: index.title} }></NewsTitle>
      <NewsDes dangerouslySetInnerHTML={ {__html: index.description} }></NewsDes>
    </NewsItem>

  ))


  return (
    <><TitleBox>News</TitleBox>
    <NewsBox ref={ref}>
      {NewsList}
    </NewsBox></>
  )
});
 ```

이렇게 하면 다음과 같은 결과가 나온다.

 ![뉴스 섹션]({{site.url}}/images/news_section_1.png)

# 이제 다 했다!! 배포!!!

 마지막으로 골머리를 썩히던 CORS 문제를 해결하고 배포만이 남았다. 나는 firebase를 통해 배포했다. 하지만 여기서 문제가 생겼다. api를 받아오려면 서버를 만들어서 받아와야하는데.... 서버를 firebase에서 만드려면 Cloud Function을 사용해야한다. 근데 이 기능이 basic 요금제로는 안된다고 한다... 허허 요금제를 올릴 것인가.... 아직 마음의 준비가 안되었기에.... 일단 보류........

 그렇게 firebase를 통한 반쪽짜리 웹 배포가 성공했다...

 [윤이나 팬페이지](https://yooninaweb.web.app/).

# 지금까지 느낀점

 처음으로 제작부터 배포까지 하면서 정신없이 만들었다. 매 순간이 위기였는데 구글 선생님과 함께 잘 이겨낸 것 같다..ㅎㅎ 그나저나 뭔가 더 잘 만들고 싶은데,, 뭘 넣어야할지 그리고 어떻게 만들어야 이뻐보이는지를 모르겠다.. 일단 배포를 한 만큼 아이디어가 생길때마다 추가하려고 한다. 