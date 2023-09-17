---
title: "프론트엔드 성능 베이스캠프 1 - 요청 크기 줄이기"
categories:
  - 프로그래밍
  - 성능
tags:
  - 프론트엔드 성능 베이스캠프
image:
  thumbnail: /assets/images/thumbnail/performance.jpg
  path: /assets/images/thumbnail/performance.jpg
---

`프론트엔드 성능 베이스캠프`는 우아한테크코스 레벨4 `웹 성능 미션`을 바탕으로 마주한 문제를 해결한 과정을 다룬다. 그 중 첫 번째는 `요청 크기 줄이기`이다.

요청 크기라고 하면 다음과 같은 것들을 생각할 수 있다.

1. 소스코드
2. 이미지와 같은 assets 파일
3. 폰트

이 중 `소스코드`와 `이미지와 같은 assets 파일`의 크기를 줄여보도록 한다.

> 성능 테스트에 사용되고 개선되는 코드는 [여기에서](https://github.com/nlom0218/new-perf-basecamp) 확인할 수 있다.

# 1. 소스코드 크기 줄이기

## 1-1. 기존 소스코드 크기

서비스를 실행하게 되면 `bundle.js`가 불러와진다. `bundle.js` 크기를 비교하기 전 `package.json`의 scripts를 다음과 같이 수정하자.

```json
{
  "scripts": {
    // ...
    "serve:dev": "webpack serve --mode=development", // 수정한 스크립트, `dev` 환경
    "serve:prod": "webpack serve --mode=production", // 추가한 스크립트, `prod` 환경
    // ...
    "deploy": "npm run build:prod && npx gh-pages -d dist" // 기존 스크립트, `github-pages` 배포 환경
  }
}
```

여러 환경에서의 `bundle.js`의 크기를 확인하면 다음과 같다.

1. dev 환경에서의 `bundle.js` 크기 - 2.8MB
   <img src="/assets/images/performance/bundle-dev.png">

2. prod 환경에서의 `bundle.js` 크기 - 1.6MB
   <img src="/assets/images/performance/bundle-prod.png">

3. github-pages 환경에서의 `bundle.js` 크기 - 284kB
   <img src="/assets/images/performance/bundle-github-pages.png">

각각의 환경에 왜 `bundle.js` 크기의 차이가 있을까? 그 이유는 webpack의 v4 이상 부터는 각각의 환경(dev, prod)에 따라 최적화가 진행된다. 즉, `prod` 환경으로 빌드를 하게 되면 별다른 webpack 설정 없이 자동으로 최적화가 진행된다.

다음은 웹팩문서에서 설명하고 있는 `mode 옵션`(dev, prod, none)에 대한 설명이다.

| 옵션        | 설명                                                                                                                                                                                                                                                           |
| ----------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| development | DefinePlugin의 process.env.NODE_ENV를 development로 설정합니다. 모듈과 청크에 유용한 이름을 사용할 수 있습니다.                                                                                                                                                |
| production  | DefinePlugin의 process.env.NODE_ENV를 production으로 설정합니다. 모듈과 청크, FlagDependencyUsagePlugin, FlagIncludedChunksPlugin, ModuleConcatenationPlugin, NoEmitOnErrorsPlugin, TerserPlugin 등에 대해 결정적 망글이름(mangled name)을 사용할 수 있습니다. |
| none        | 기본 최적화 옵션에서 제외                                                                                                                                                                                                                                      |

<a href="https://webpack.kr/configuration/mode/#usage" target="_blank">Webpack | Mode</a>

그리고 `github-pages`에 배포를 하게 된다면 `bundle.js`은 `gzip` 형태로 인코딩되어 불러와지기 때문에 소스 크기는 더욱 작아지게 된다.

<img src="/assets/images/performance/bundle-github-pages-gzip.png">

## 1-2. optimization.minimize 적용하기

`optimization`의 `minimize` 속성은 `TerserPlugin` 또는 `optimization.minimizer`에 지정된 플로그인을 사용하여 번들을 최소화한다.

webpack의 `optimization` 설정을 보면 다음과 같다.

```javascript
module.exports = {
  // ...
  optimization: {
    minimize: false,
  },
};
```

`minimize` 속성을 `true`로 수정한 후 다시 빌드를 해보자.

```javascript
module.exports = {
  // ...
  optimization: {
    minimize: true,
  },
};
```

결과는 다음과 같다.

1. dev 환경: 2.8MB -> 1.3MB
2. prod 환경: 1.6MB -> 446kB

별다른 설정을 하지 않아도 `bundle.js` 크기가 크게 줄어든 것을 확인할 수 있다. 즉, 단순히 `minimize` 속성을 `true`만 해주어도 기본적인 최적화가 일어난다.

여기서 의문점이 들었다. 아예 `optimization` 설정을 지우게 된다면 어떻게 될까? 결과는 다음과 같다.

1. dev 환경: 2.8MB
2. prod 환경: 446kB

여기까지의 내용을 정리하자면 `prod` 환경에서는 `optimization` 설정을 굳이 하지 않아도 기본적인 최적화가 이루어지고 `dev` 환경에서는 `optimization`의 `minimize` 속성을 `true`로 해주어야 최적화가 이루어진다. 즉, `optimization` 설정을 통해 추가 최적화를 설정하지 않는다면 `optimization` 설정은 하지 않아도 된다.(`dev` 환경에서의 최적화는 굳이 필요 없으니까)

## 1-3. CSS 최적화와 optimization.minimizer의 기본값 유지

만약 `CssMinimizerPlugin`을 사용하여 css를 최적화를 추가 진행한다면 `optimization`의 `minimizer` 속성을 다음과 같이 할 수 있을 것이다.

```javascript
module.exports = {
  // ...
  optimization: {
    minimize: true, // 제거 가능. 단, false로 할 경우 최적화는 진행되지 않음.
    minimizer: [new CssMinimizerPlugin()],
  },
};
```

위와 같은 경우는 `minimizer`을 사용하므로 `minimize`가 `true`가 된다. 즉 `minimize` 속성을 제거해도된다. 단, 위에서도 설명했듯이 `dev` 환경에는 다르게 적용된다. `minimize`가 `true`이어야만 최적화가 이루어진다.

어쨌든 위 설정에 따른 결과는 어떨까? 다음과 같다.

1. dev 환경: 2.8MB
2. prod 환경: 1.6MB

다시 `bundle.js`의 크기가 이전으로 돌아간 이유는 무엇일까? 이는 `minimizer` 속성에서 기본 값을 제거했기 때문이다. 만약 추가적인 최적화를 하기 위해 플러그인을 사용한다면 기본 값을 유지하는 구문이 필요하다. 다음은 `minimizer` 속성의 기본 값을 유지하
는 코드와 `minimizer` 속성의 기본 값이다.

```javascript
// `minimizer` 속성의 기본 값
[
  {
    apply: (compiler) => {
      // Lazy load the Terser plugin
      const TerserPlugin = require("terser-webpack-plugin");
      new TerserPlugin({
        terserOptions: {
          compress: {
            passes: 2,
          },
        },
      }).apply(compiler);
    },
  },
];
```

```javascript
// `minimizer` 속성의 기본 값을 유지하기 위한 코드
module.exports = {
  // ...
  optimization: {
    minimize: true,
    minimizer: ["...", new CssMinimizerPlugin()],
  },
};
```

`'...'`을 추가해야 기본 값을 유지한다.

결과는 다음과 같다.

1. dev 환경: 1.3MB
2. prod 환경: 446kB

## 1-4. 결론 및 최종 결과

현재 `CssMinimizerPlugin`을 사용하여 css 최적화를 진행을 해도 변화가 없기 때문에 설정하지 않았다. 또한 `optimization` 설정을 추가하지 않아도 충분히 `bundle.js`의 크기가 줄어들기 때문에 `optimization` 설정 자체를 제거하였다.

만약 최적화에 대한 추가적인 설정이 필요하다면 아래의 링크를 참고하면 된다.

<a href="https://webpack.kr/configuration/optimization/" target="_blank">Webpack | Optimization</a>
<br>
<a href="https://webpack.kr/plugins/terser-webpack-plugin/" target="_blank">Webpack | TerserWebpackPlugin</a>
<br>
<a href="https://webpack.kr/plugins/css-minimizer-webpack-plugin/" target="_blank">Webpack | CssMinimizerWebpackPlugin</a>

다음은 `github-pages`에서의 `bundle.js` 소스 크기이다. 이전보다 확 줄어든 것을 확인할 수 있다.

284kB -> 84.8kB

<img src="/assets/images/performance/bundle-github-pages-imporve-size.png">

다음은 `소스코드 크기 줄이기` 작업을 진행한 커밋이다.

<a href="https://github.com/nlom0218/new-perf-basecamp/commit/d96978eb3ac7465b61038869b05e353a369583c9" target="_blank">refactor: 1. 소스코드 크기 줄이기</a>

# 2. 이미지 크기 줄이기

이미지 크기를 줄이는 방법은 다양하다. 외부 사이트를 이용하여 직접 이미지 크기를 줄일 수 있고 명령어를 통해 이미지 크기와 더불어 확장자도 변환시킬 수 있다. 또한 웹팩 플러그인을 활용할 수도 있다. 아래에서는 내가 미션에서 적용했던 방법에 대한 설명이다.

## 2-1. 기존 이미지 크기

<img src="/assets/images/performance/assets-pre-per.png">

기존 이미지 크기를 살펴보면 `hero.png`의 Size는 10.7MB로 굉장히 크다. 더불어 `gif`의 Size도 줄여야 할 필요가 느껴진다.

## 2-2. 너무나 큰 원본 사이즈

<img src="/assets/images/performance/hero-intrinsic-size.png">

`Intrinsic size`을 살펴보면 hero 이미지의 원본 사이즈를 확인할 수 있다. 하지만 `4100 x 2735 px` 만큼 필요하지 않다.
