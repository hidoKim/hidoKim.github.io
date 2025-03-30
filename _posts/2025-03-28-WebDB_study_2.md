---
layout: single
title: "[school] [WebDB_study] 웹 DB week4 정리"
categories: WebDB_study
tags: [school, WebDB_study, study]
toc: true
author_profile: false
---

# 웹 DB 프로그래밍 과목

## Node.js 프로그래밍

### 1. 자바스크립트와 Node.js

#### URL 이해하기

- http://opentutorials.org:3000/
- http://opentutorials.org:3000/main
- http://opentutorials.org:3000/main/side

  위 3개의 URL은 서로 다른요청, 요청의 대상은 웹서버이며 URL은 요청의 종류

- http://opentutorials.org:3000/main?id=HTML&page=12

  - http,https: 프로토콜(hyper test transfer protocol secure)
  - opentutorials: 호스트(도메인 네임)
  - 3000: 포트 번호(웹서버의 의미)
    - URL에 매핑되는 root 디렉토리가 홈 디렉토리
    - main.js를 실행 시킨 디렉토리가 홈 디렉토리가 됨
  - main, main/side: 다른 페이지를 불러오는 URL부분, 요청의 종류
  - ?: 이후에 나오는 것이 query string임을 알려줌
    - query string: 질의 단어, 조건 제시 기능
  - id=HTML&page=12: query string, **_&_**는 두 개의 변수를 연결

---

#### favicon.ico

- favorate icon의 약자
- /favicon.ico
  - 브라우저가 서버에게 자동으로 요청하는 URL -> 127.0.0.1:3000/favicon.ico

---

#### http 응답 상태 코드

- 클라이언트로부터의 HTTP요청이 성공적으로 완료되었는지 알려줌
- 응답은 5개의 그룹으로 나눔
  1. 정보제공 응답: 100 ~ 103
  2. 성공적인 응답: 200 ~ 208, 226
  3. 리다이렉트: 300 ~ 308
  4. 클라이언트 에러: 400 ~ 418, 421 ~ 424, 426, 428, 429, 431, 451
  5. 서버 에러: 500 ~ 508, 510, 511

---

#### 동적 페이지를 작성하기 위한 WAS와 HTML 간의 자료 주고 받는 방법

response 객체에 HTML문서를 전달해주는 방법

1. end 메소드에 직접 html 작성
2. end 메소드에 fs 모듈을 이용하여 html 파일의 내용을 읽어옴
3. template 언어 사용

WAS 프로그램 or 라우터(URL분류기)가 데이터를 HTML에 전달하면 HTML은 쿼리스트링을 전달한다.

---

### 2. Express

Node.js위에서 동작하는 웹 프레임워크

- 반복적으로 등장하는 기능을 처리할 때 더 적은 코드로 처리할 수 있도록 효율성 증가

  - 반복적인 일: URL파라미터를 통해 전달된 데이터로 작업
  - 정적인 자바스크립트 파일, 이미지 파일 등을 제공하는 기능, 로그인 기능, 보안 기능 등

- template 언어 적용
  - 보안, 안정성, 성능, 동시성을 DB가 제어함

---

#### Hello world

```js
const express = require("express");
// express 모듈을 import, const에 의해 express 변수는 값이 변하지 않음, http 모듈의 요청과 응답 객체에 추가 기능을 부여함.
const app = express();
// express() 함수에 의해 Application 객체를 app에 저장. 마치 http 모듈에서 반환된 객체를 createServer 메소드와 유사
app.get("/", (req, res) => res.send("Hello world"));
app.get("/author", (req, res) => res.send("/author"));
app.listen(3000, () => console.log("Example app listening on port 3000")); // 웹의 요청을 수신, 이벤트 루프
```

- get(path, callback): 사용자가 해당 경로를 요청하면 callback 실행
  - get 메소드는 라우팅 (URL 분류기)기능을 수행: 요청된 경로마다 응답해 주는 기능
  - 더이상 if else로 URL분류(라우팅) 할 필요 없음
- res.send(): express 에서 res객체에 send 메소드 추가, end 메소드와 같은 기능

---

#### ejs 사용 (template 언어 사용)

template 언어 문법
표기(ejs태그): 의미

- <%js 문법%>: 흐름 제어문
- <%= %>: 변수 값
- <%- %>: 변수 내용을 태그로 인식
- <%- include(view의 상대주소)%>: 다른 view파일을 불러 옴

---

##### ejs 작동 메커니즘

1. node 프로그램에서 html로 변수의 값을 전달하므로 변수의 이름과 개수가 일치해야 함
2. render 함수가 context 객체의 멤버 변수와 같은 home.ejs의 변수에 값을 넘겨주어 순수한 html을 생성
3. render의 세번째 인자인 콜백함수에 **_2._** 단계에서 생성된 html문서를 두번째 인자로 넘겨주고 콜백함수에서는 response 객체의 end메소드를 호출하여 HTML문서를 browser에서 전송한다.
   - rendering: ejs파일에 변수들의 값이나 프로그램들을 모두 컴파일 하여 완전한 html문서로 만드는 작업.
   - render(): rendering을 하는 메소드

```js
// main.js
app.get("./id", (req, res) => {
  var id = req.params.id;
  var context = { title: id };

  res.render("home", context, (err, html) => {
    res.end(html);
  });
});
app.get("/favicon.ico", (req, res) => res.writeHead(404));
app.listen(3000, () => console.log("Example app listening on port 3000"));
```

> node에서 title과 ejs에서 title이 일치

> id가 HTML이라고 가정

> context 객체의 title 멤버변수에 HTML저장

```html
<!-- home.ejs -->
<!doctype html>
<head>
    <title>WEB1 - <%=title%></title>
    <meta charset= 'utf-8'>
</head>
<body>
    <h1><a href="/">WEB</a></h1>
    <ol>
    <h2><%=title%></h2>
</body>
</html>
```

---
