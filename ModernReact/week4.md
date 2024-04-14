## ESLint
- ESLint는 자바스크립트에서 사용할 수 있는 정적 코드 분석 도구이다.
### ESLint의 코드분석
1. 코드를 문자열로 읽는다
2. parser로 코드를 구조화한다
    - 코드를 구조화하는 과정에서 사용하는 parser로 기본값으로 espree를 사용한다
    - espree 이외의 parser는 [이곳](https://astexplorer.net)
3. 구조화한 트리를 AST라 하며, 구조화된 트리를 기준으로 각종 규칙과 대조한다.
4. 규칙과 대조해서 위반한 코드를 알리거나, 수정한다.
- 찾아보니까 parser별로 eslint에 큰 변화가 생기는 건 아닌 듯 하다.
### eslint 관련 패키지
- **eslint-plugin**: 규칙을 모아놓는 패키지
- **eslint-config**: plugin을 묶어서 한 세트로 제공하는 패키지.
- **eslint-config-airbnb**: 리액트 기반 프로젝트에서 사용하는 가장 유명한 config. google이나 naver 대비 압도적인 다운로드 수를 자랑한다.
- **@titicaca/triple-config-kit**: airbnb를 기반으로 하지 않고 유용한 규칙을 모두 제공한다. 제공하는 규칙에 테스트 코드가 존재한다.
- **eslint-config-next**: next 관련 eslint-config 패키지. JSX 뿐만 아니라 HTML 코드도 정적 분석 대상으로 제공하며 web vital을 분석해 제공한다.
### eslint 규칙 만들기
```js
module.exports = {
  rules: {
    'no-restricted-imports': [
      'error',
      {
        // paths에 금지시킬 모듈을 추가한다.
        paths: [
          {
            name: 'react',
            importNames: ['default'],
            message: "import React from 'react'는 react 17부터 더 이상 필요하지 않습니다. 필요한 것만 react로부터 import해서 사용해 주세요.",
          },
        ],
      },
      {
        name: 'lodash',
        message: 'lodash는 CommonJS로 작성돼 있어 트리셰이킹이 되지 않아 번들 사이즈를 크게 합니다. lodash/* 형식으로 import 해주세요.'
      }
    ],
  },
}
```
message 기능이 신기해서 가져왔다.
### eslint에 새로운 규칙 추가하기
1. 규칙 생성
```
module.exports = {
  meta: { // 규칙과 관련된 정보를 나타내는 필드
    type: 'suggestion',
    docs: {
      description: 'disallow use of the new Date()',
      recommended: false,
    },
    fixable: 'code',
    scheme: [],
    messages: {
      message: 'new Date()는 클라이언트에서 실행 시 해당 기기의 시간에 의존적이라 정확하지 않습니다. 현재 시간이 필요하다면 ServerDate()를 사용해 주세요.',
    },
  },
  create: function (context) {
    return {
      NewExpression: function (node) { // new 생성자 사용시  ESLint가 실행되도록
        if (node.callee.name === 'Date' && node.arguments.length === 0)  {
          context.report({
            node: node,
            massageId: 'message',
            fix: function (fixer) {
              return fixer.replaceText(node, 'ServerDate()')
            },
          })
        }
      },
    }
  },
}
```
2. 패키지 구성
```shell
yo eslint:plugin
yo eslint:rule
# rule ID를 물어볼 때 no-new-date를 추가한다
```
3. rules/no-new-date.js 파일을 열고 작성한 규칙을 붙여넣고, docs에는 규칙을 위한 설명을 작성한다. tests에는 테스트 코드를 작성한다.
### Prettier 충돌
- 규칙을 처음부터 prettier와 충돌되지 않도록 작성하는 방법
- js나 ts는 eslint가 감지하고, 그외엔 prettier가 감지하도록 설정하는 방법
  이렇게 2가지가 있지만 나는 prettier를 기반으로 eslint가 작동하도록 해두었다.
## Front 혼자서 Cors를 해결하는 방법
최근에 Apache Superset 솔루션을 커스터마이징하는 작업을 진행하면서 많은 것을 배우고 있다. 그 중에 가장 최근에 있었던 문제와 해결 방법에 대해 이야기 하고자 한다.
### 요구사항
클라이언트 측에서 전달받은 요구사항은 다음과 같았다.
- Superset에서 제공하는 차트를 다른 페이지에서 보고 싶다.
### 문제점
가장 큰 문제점은 역시 cors였다. cors를 해결하는 방법은 크게 3가지가 있는데
- front 단에서 proxy 설정을 통해 우회하거나
- back 에서 예외처리를 해주거나
- 같은 도메인을 사용하거나
  이렇게 있다. 하지만 cors의 근본적인 원인을 생각한다면 다른 해결방법도 있다. cors는 프론트엔드에서 다른 도메인의 백엔드로 api 요청을 보낼때 chrome에서 차단하는 정책이다. 그리고 이것은 백 to 백에선 발생하지 않는다. 이 경우 또 하나의 해결책이 있다.
- 같은 도메인을 가진 백엔드에서 프론트의 요청을 대신 요청해주는 것이다.
### 해결방법
먼저 새롭게 프로젝트를 만들었고 express 서버와 client를 workspace로 묶어서 만들었다. 그리고 express 서버에 다음과 같은 코드를 추가했다.
```js
import express from 'express';
import fetch from 'node-fetch';
import cors from 'cors';

const PORT = 3001;
const app = express();

const id = "430b5226-ec14-4f33-8a61-c481aeaf2f84";
const domain = "http://localhost:8088/api/v1/";

app.use(cors());
app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});

async function fetchAccessToken() {
  const url = "/security/login"
  try {
    const body = {
      username: "admin",
      password: "admin",
      provider: "db",
      refresh: true,
    };

    const response = await fetch(`${domain}${url}`,
      {
        method: "POST",
        body: JSON.stringify(body),
        headers: {
          "Content-Type": "application/json",
        },
      }
    );

    const jsonResponse = await response.json();
    return jsonResponse.access_token ?? '';
  } catch (e) {
    console.error("fetchAccessToken Error:", e);
  }
}

async function fetchCsrfToken() {
  const accessToken = await fetchAccessToken();

  const url = "/security/csrf_token/"

  try {
    const response = await fetch(`${domain}${url}`,
      {
        method: "GET",
        headers: {
          "Content-Type": "application/json",
          Authorization: `Bearer ${accessToken}`,
        },
      }
    );

    const jsonResponse = await response.json();
    return {
      access_token: accessToken,
      csrf_token: jsonResponse.result
    };
  } catch (e) {
    console.error("fetchAccessToken", e);
  }
}

async function fetchGuestToken() {
  const {
    csrf_token,
    access_token
  } = await fetchCsrfToken();

  const url = "/security/guest_token"

  try {
    const body = {
      resources: [
        {
          type: "dashboard",
          id: id,
        },
      ],
      rls: [],
      user: {
        username: "guest",
        first_name: "Guest",
        last_name: "User",
      },
    }
    const response = await fetch(
      `${domain}${url}`,
      {
        method: "POST",
        body: JSON.stringify(body),
        headers: {
          "Content-Type": "application/json",
          "accept": "application/json",
          Authorization: `Bearer ${access_token}`,
          'X-CSRFToken': `csrf ${csrf_token}`,
        },
      }
    )
    const jsonResponse = await response.json()
    return jsonResponse?.token
  } catch (error) {
    console.error("fetchGuestToken", error)
  }
}

app.get("/guest-token", async (req, res) => {
  const token = await fetchGuestToken()
  console.info("response Data:", token);
  res.json(token)
})
```
위 코드는 슈퍼셋 서버로부터 게스트 토큰을 얻어오는 api다.
```tsx
import './App.css'
import { useState } from 'react';

function App() {
  const [token, setToken] = useState('');

  const onClick = async () => {
    const token = await fetch('guest-token');
    return setToken(await token.text() ?? '');
  }

  return (
    <div>
      <button onClick={onClick}>login</button>
    </div>
  )
}

export default App;
```
속도는 직접 요청하는 것보단 느릴 순 있어도, 확실하게 작동한다.