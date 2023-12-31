---
title: "웹팩으로 리액트 시작하기 4 - dev, prod 환경 분리하기"
categories:
  - 프로그래밍
  - webpack
tags:
  - Webpack with React
image:
  thumbnail: /assets/images/webpack.png
  path: /assets/images/webpack.png
---

지금까지 하나의 파일(webpack.config.js)에서 환경(dev, prod)에 대한 웹팩을 설정하였다. 환경에 따른 분리가 필요하면 삼항연사자를 사용한다. 아직은 파일이 크지 않아 삼항연산자도 충분하다. 하지만 앞으로 설정이 많아지고 복잡해지면 삼항연산자로 인한 환경 구분은 상당히 피로감을 줄 수 있다. 때문에 환경에 따라 파일을 분리하는 것이 효율적이다. **웹팩으로 리액트 시작하기** 네 번째 파트에서는 환경에 분리하여 웹팩을 설정하는 과정을 다룬다.

다음의 글은 웹팩 공식문서를 참고하여 작성했다.

<a href="https://webpack.kr/guides/production/" target="blank">Production | 웹팩</a>

# 1. 파일 분리하기

다음의 파일을 새롭게 생성한다. 이때 파일의 경로는 기존 `webpack.config.js`의 경로와 동일하다.

- `webpack.common.js`: dev, prod 환경에서 공통 설정
- `webpack.dev.js`: dev 환경에서의 설정
- `webpack.prod.js`: prod 환경에서의 설정

<br/>

# 2. webpack-merge 유틸리티

dev환경에서의 웹팩 빌드는 `webpack.dev.js`파일과 `webpack.common.js`파일을 함께 사용한다. 때문에 이 둘을 합쳐주는 기능이 필요하다. 이를 쉽게 도와주는 유틸리티가 `webpack-merge`라이브러리이다. 이를 설치한다.

```bash
yarn add -D webpack-merge
```

<a href="https://github.com/survivejs/webpack-merge" target="blank">webpack merge</a>

<br/>

# 3. webpack.common.js

먼저 두 환경에서의 공통 웹펙 설정이다.

```jsx
const path = require("path");
const HtmlWebpackPlugin = require("html-webpack-plugin");

// 객체를 내보내는 것이 아니라 함수를 내보낸다. 공식문서에는 객체를 내보낸다.
module.exports = () => {
  return {
    entry: "./src/index.tsx",
    output: {
      filename: "bundle.js",
      path: path.resolve(__dirname, "dist"),
      clean: true,
      publicPath: "/",
    },
    resolve: {
      extensions: [".ts", ".tsx", ".js", ".jsx"],
      alias: {
        // ...
      },
    },
    module: {
      rules: [
        {
          test: /\.css$/,
          use: ["style-loader", "css-loader"],
        },
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
        {
          test: /\.(png|svg|jpg|jpeg|gif)$/i,
          type: "asset/resource",
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

<br/>

# 4. webpack.dev.js, webpack.prod.js

dev환경에서만 필요한 웹팩 설정이다.

```jsx
// webpack.dev.js

const { merge } = require("webpack-merge");
const common = require("./webpack.common.js");
const Dotenv = require("dotenv-webpack");

module.exports = merge(common(), {
  mode: "development",
  devtool: "eval-source-map",
  devServer: {
    port: 3000,
    historyApiFallback: true,
    hot: true,
  },
  plugins: [
    new Dotenv({
      path: "./env-submodule/.env.development",
    }),
  ],
});
```

prod환경에서만 필요한 웹팩 설정이다.

```jsx
// webpack.prod.js

const { merge } = require("webpack-merge");
const common = require("./webpack.common.js");
const Dotenv = require("dotenv-webpack");

module.exports = merge(common(), {
  mode: "production",
  devtool: "source-map",
  plugins: [
    new Dotenv({
      path: "./env-submodule/.env.production",
    }),
  ],
});
```

`mode`, `devtool`, `Dotenv의 path`의 값은 삼항연산자로 구분했지만 파일을 나누었기 때문에 나누지 않고 dev환경일 때의 값을 넣어주면 된다. 또한 `devServer`는 prod환경에서는 필요 없기 때문에 설정하지 않는다.

위 두 파일을 살펴보면 `webpack-merge`유틸리티에서 제공하는 `merge`함수를 사용하여 `webpack.common.js`파일과 각각의 환경 파일을 통합하는 것을 알 수 있다. 이때 주의해야 할 점은 다음과 같다.

1. `webpack.common.js`에서 함수를 내보낼 경우: `common()`로 함수를 실행하여 객체를 가져온다.
2. `webpack.common.js`에서 객체를 내보낼 경우: `common`으로 바로 객체를 가져온다.

<br/>

# 5. NPM Scripts

마지막으로 환경에 따라 `scripts`의 명령어를 다르게 설정해야 한다. 다음과 같이 수정한다.

```json
{
  // ...
  "scripts": {
    "prod": "NODE_ENV=production webpack server --open --config webpack.prod.js",
    "dev": "NODE_ENV=development webpack server --open --config webpack.dev.js",
    "build": "yarn test && NODE_ENV=production webpack --config webpack.prod.js"
  }
  // ...
}
```
