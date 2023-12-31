---
title: "웹팩으로 리액트 시작하기 1"
categories:
  - 프로그래밍
  - webpack
tags:
  - Webpack with React
image:
  thumbnail: /assets/images/webpack.png
  path: /assets/images/webpack.png
---

리액트를 시작하기 위해 `Create-React-App(CRA)` 혹은 `Vite`을 많이 사용한다. 이러한 도구는 `webpack`설정 없이 편하게 리액트를 시작할 수 있도록 도와준다. 때문에 많은 개발자들이 사용하고 있는 도구이다. 하지만 단점도 분명 존재한다. `webpack`설정을 추가하거나 수정하는 것이 매우 번거롭다는 것이다. 이러한 단점 때문에 `webpack`설정을 도와주는 다양한 라이브러리가 등장하였다. `craco`, `react-app-alias`, `react-app-rewired` 등이 그것이다. 충분히 좋은 라이브러리들이지만 처음부터 `webpack`을 직접 설정을 했다면 번거로운 작업은 하지 않아도 된다. 또한 프론트엔드 개발자로서 `webpack`은 학습해야 하는 개념이기 때문에 직접 `webpack`을 설정하여 리액트를 시작하는 것은 매력적인 과정이라고 생각한다.

**웹팩으로 리액트 시작하기**는 `하루 스터디`서비스를 개발하면서 마주한 `react` 시작 및 `webpack`설정에 대한 글을 다룬다. 그 중 첫 번째 파트에서는 필요한 라이브러리 설치 및 간단한 웹팩 설정과 기본 파일 구조를 추가하는 것을 다룬다. 아래의 내용으로도 리액트 서버를 열 수 있다.

# 1. React + Typescript

가장 먼저 해야할 것은 `react`을 설치하는 것이다. 추가로 `typescript`을 사용하기 때문에 `typescript`도 함께 설치한다.

```shell
yarn add react react-dom
yarn add -D typescript @types/react @types/react-dom
```

이후 tsconfig.json을 만들기 위해 다음의 명령어를 입력한다.

```shell
yarn tsc --init
```

tsconfig.json이 만들어졌으면 다음과 같이 하나의 옵션을 수정한다.

```json
{
  "compilerOptions": {
    // ...
    "jsx": "react-jsx"
    // ...
  }
}
```

<br/>

# 2. webpack 설치 및 설정

`webpack` 설정을 위한 라이브러리를 설치해야 한다. 다음과 같은 라이브러리를 설치한다.

- `webpack`: webpack 라이브러리
- `webpack-cli`: webpack을 명령어로 실행하기 위한 라이브러리
- `webpack-dev-server`: webpack을 사용하여 서버를 열기 위한 라이브러리, 로컬에서 개발하기 위한 테스트 서버
- `ts-loader`: webpack을 위한 타입스크립트 loader
- `html-webpack-plugin`: webpack 번들을 제공하는 HTML 파일 생성을 단순화

```shell
yarn add -D webpack webpack-cli webpack-dev-server html-webpack-plugin ts-loader
```

이후 webpack.config.js 파일을 만들고 다음과 같이 작성한다.

```javascript
const path = require("path");
const HtmlWebpackPlugin = require("html-webpack-plugin");

module.exports = () => {
  const isDevelopment = process.env.NODE_ENV !== "production";

  return {
    entry: "./src/index.tsx",
    output: {
      filename: "bundle.js",
      path: path.resolve(__dirname, "dist"),
      clean: true,
    },
    resolve: {
      extensions: [".ts", ".tsx", ".js", ".jsx"],
    },
    devServer: {
      port: 3000,
      hot: true,
    },
    devtool: isDevelopment ? "eval-source-map" : "source-map",
    module: {
      rules: [
        {
          test: /\.(ts|tsx)$/,
          use: {
            loader: "ts-loader",
            options: {
              configFile: path.resolve(__dirname, "tsconfig.json"),
            },
          },
          exclude: /node_modules/,
        },
      ],
    },
    plugins: [
      new HtmlWebpackPlugin({
        template: "public/index.html",
      }),
    ],
  };
};
```

- entry: 다른 모듈을 사용하고 있는 최상위 모듈을 명시
- output
  - path: 번들된 하나의 모듈이 위치하게 될 주소
  - filename: 번들된 하나의 모듈 이름
  - clean: 이전 빌드 결과를 완전히 삭제하는 여부
- resolve
  - extensions: 탐색할 모듈의 확장자를 지정, 사용자가 import 할 때 확장자를 생략할 수 있도록 도움
- devServer: `webpack-dev-server(dev-server)`의 동작에 영향을 미치는 옵션
  - port: 요청을 수신할 포트 번호
  - hot: webpack의 `Hot Module Replacement`기능을 활성화 여부
- devtool: 개발을 용이하게 하기 위해 소스맵을 제공하는 옵션
  - eval-source-map: 고품질 소스맵을 포함한 개발 빌드를 위해 추천하는 옵션
  - source-map: 고품질 소스맵을 포함한 프로덕션 빌드를 위해 추천하는 옵션
- module: loader 규칙 설정, loader를 사용하면 javascript를 넘어 모든 정적 리소스를 번들링 할 수 있음
- plugins: 번들을 최적화, 에셋을 관리하고 환경 변수 주입등과 같은 광범위한 작업을 수행하는 규칙을 설정

<br/>

# 3. package.json script 작성

`webpack`으로 로컬 개발 서버를 열기 위해 `script`을 추가해야 한다.

```json
{
  // ...
  "scripts": {
    "prod": "NODE_ENV=production webpack serve",
    "dev": "NODE_ENV=development webpack serve",
    "build": "NODE_ENV=production webpack"
  }
  // ...
}
```

<br/>

# 4. 기본 파일 작성

## public/index.html

```html
<!DOCTYPE html>
<html lang="ko">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Hello React</title>
  </head>
  <body>
    <div id="root"></div>
  </body>
</html>
```

## src/index.tsx

```tsx
import { createRoot } from "react-dom/client";
import App from "./App";

const container = document.getElementById("root");
const root = createRoot(container!);

root.render(<App />);
```

## src/App.tsx

```tsx
const App = () => {
  return <div>Hello React</div>;
};

export default App;
```

# 5. webpack server 실행

```shell
yarn dev
```
