---
title: "프론트엔드 성능 베이스캠프 1 - 요청 크기 줄이기"
categories:
  - 프로그래밍
  - 성능
tags:
  - 우아한테크코스
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

`Intrinsic size`을 살펴보면 hero 이미지의 원본 사이즈(해상도)를 확인할 수 있다. 하지만 `4100 x 2735 px` 만큼 필요하지 않다.

때문에 해상도를 줄이기 위해 다음의 사이트를 이용하였다.

[resizepixel](https://www.resizepixel.com/ko?status=1)

이미지 비율을 그대로 두고 사이즈를 `2000 x 1333px`으로 줄였다. 결과는 다음과 같다.

<img src="/assets/images/performance/change-hero-intrinsic-size.png">

벌써 이미지 사이즈가 확 줄어든 것을 확인할 수 있다.

## 2-3. png보단 jpg

format을 변경하여 이미지 사이즈를 더 줄여보자. jpg이 png와 비교했을 때 용량이 훨씬 작고 사진과 같은 색상이 많은 이미지에 어울린다.

이번에도 유용한 사이트의 도움을 받아 쉽게 변경할 수 있다.

[PNG to JPG](https://png2jpg.com/ko/)

결과는 다음과 같다.

<img src="/assets/images/performance/convert-format-hero-image.png">

이처럼 이미지의 원본 사이즈를 줄이고 format만 변경하여도 이미지 크기는 많이 줄어든다.

|      기준      | 이미즈 크기 |
| :------------: | :---------: |
|      원본      |   10.7MB    |
| 해상도 낮춘 후 |    2.4MB    |
| format 변경 후 |    219kM    |

## 2-4. 이미지를 webp로 변한하기

앞선 과정을 통해 이미지크기가 많이 줄어들었다. 하지만 요구사항(120kB미만)은 아직 충족시키지 못하였다. 이를 webp를 활용하여 해결해보자.

`web`이미지를 사용한다면 일반적으로 파일 크기가 25~35% 감소한다. 명령어를 활용하여 jpg형태의 이미지를 webp이미지로 변경해보자.

먼저 기본적인 명령어 사용방법은 다음과 같다.

```bash
$ cwebp -q {품질(0~100)} {원본 파일 경로} -o {변경될 webp 파일 경로}
```

다음은 실제로 프로젝트에서 사용한 명령어이다.

```bash
$ cwebp -q 40 src/assets/images/hero.jpg -o src/assets/images/hero.webp
```

webp로 변환하여 이미지 크기를 확인한 결과 109kB로 줄어든 것을 확인할 수 있다.

<img src="/assets/images/performance/convert-hero-webp.png">

다음으로 webp롤 사용하여 이미지를 화면에 띄어보자. 이를 위해 `picture` 태그를 활용해야 한다.

```tsx
// ...
const MyComponents = () => {
  return (
    <picture>
      <source
        className={styles.heroImage}
        type="image/webp"
        srcSet={heroImageWebp}
      />
      <img className={styles.heroImage} src={heroImage} alt="hero image" />
    </picture>
  );
  // ...
};
```

`webp`를 지원하는 브라우저인 경우 `webp`이미지를 보여주고 그렇지 않은 브라우저인 경우엔 `jpg`이미지를 보여준다. 다음이 webp를 적용한 결과이다.

<img src="/assets/images/performance/set-hero-webp.png">

또한 다음과 같이 화면 너비에 따라 크기가 다른 이미지를 각각 불러올 수도 있다.

```tsx
const MyComponents = () => {
  return (
    <picture>
      <source
        src={heroImage}
        className={styles.heroImage}
        type="image/webp"
        srcSet={`${heroImageSmallWebp} 700w, ${heroImageLargeWebp} 2000w`}
      />
      <img className={styles.heroImage} src={heroImage}></img>
    </picture>
  );
  // ...
};
```

## 2-5. 결론 및 최종 결과

이미지 최적화를 하기 위해 코드적으로 많은 부분을 수정했기 보다는 이미지 포멧, 형식을 바꾸어 이미지의 크기(용량)자체를 줄였다. 지금까지 거쳐온 과정과 이에 따른 결과를 표로 나타내면 다음과 같다.

|      기준      | 이미즈 크기 |
| :------------: | :---------: |
|      원본      |   10.7MB    |
| 해상도 낮춘 후 |    2.4MB    |
| format 변경 후 |    219kM    |
| webp로 변경 후 |    109kM    |

이미 해상도 변경, format 변경을 가능하게 해주는 사이트는 검색만 해도 많이 있다. 때문에 어렵지 않았고 webp도 사이트에서 변경을 할 수 있지만 [명령줄을 사용하여 WebP 이미지 만들기](https://web.dev/codelab-serve-images-webp/) 에서 소개하는 `cwebp` 명령어를 사용하였다. `picture` 태그에 대한 설명도 함께 있어 큰 어려움이 없이 이미지 최적화를 마무리 할 수 있었다.

다음은 `이미지 크기 줄이기` 작업을 진행한 커밋이다.

<a href="https://github.com/nlom0218/new-perf-basecamp/commit/b3af11c4b4b8ee1bfcdd1199c22b223cc5f3b598" target="_blank">refactor: 2. 이미지 크기 줄이기</a>

<br />

# 3. 애니메이션 GIF를 비디오로 대체하기

애니메이션 GIF는 용량이 크다. 때문에 mp4, webm과 같은 파일 형식의 비디오로 변환하여 페이지를 더 빠르게 로드한다면 성능은 더욱 좋아질 것이다.

<img src="/assets/images/performance/before-gif-file.png">

## 3-1. ffempeg 설치

미션에서 애니메이션 GIF를 비디오(mp4, webm)으로 변환하기 위해 `ffempeg`명령어를 사용했다. `ffempeg`명령어를 사용하기 위해선 이를 설치해야 하는데, 설치부터 쉽지 않은 과정이었다. 설치만 매끄럽게 되었다면 훨씬 간편하고 빠르게 작업을 마무리 했었을 것이다.

기본적인 설치 명령어는 다음과 같다.

```bash
$ brew install ffmpeg
```

하지만 이 과정에서 설치 오류가 뜬다면 최신 릴리즈 버전으로 설치를 해야 하는데, 이때에는 다음과 같은 명령어를 입력해야 한다.

```bash
$ brew install ffmpeg --HEAD
```

## 3-2. mp4 비디오로 변환하기

우선 `mp4`로 변환을 해보자. `ffempeg`명령어를 사용하여 `gif`을 `mp4`로 변환하기 위해선 다음과 같이 명령어를 실행해야 한다.

```bash
$ ffmpeg -i {원본 gif파일 경로} -b:v 0 -crf 25 -f mp4 -vcodec libx264 -pix_fmt yuv420p {변환될 mp4 파일 경로}
```

위 명령어에서는 `libx264` 인코더를 사용하는데 이는 320x240픽셀과 같이 크기가 짝수인 파일에서만 작동한다. 때문에 `gif`의 크기가 홀수인 경우엔 다음과 같이 "crop=trunc(iw/2)*2:trunc(ih/2)*2"을 명령어에 추가해야 한다.

```bash
$ ffmpeg -i {원본 gif파일 경로} -vf "crop=trunc(iw/2)*2:trunc(ih/2)*2" -b:v 0 -crf 25 -f mp4 -vcodec libx264 -pix_fmt yuv420p {변환될 mp4 파일 경로}
```

이를 바탕으로 실제 프로젝트에 사용해보자.

```bash
$ ffmpeg -i src/assets/images/find.gif -vf "crop=trunc(iw/2)*2:trunc(ih/2)*2" -b:v 0 -crf 25 -f mp4 -vcodec libx264 -pix_fmt yuv420p src/assets/images/find.mp4
$ ffmpeg -i src/assets/images/free.gif -vf "crop=trunc(iw/2)*2:trunc(ih/2)*2" -b:v 0 -crf 25 -f mp4 -vcodec libx264 -pix_fmt yuv420p src/assets/images/free.mp4
$ ffmpeg -i src/assets/images/trending.gif -vf "crop=trunc(iw/2)*2:trunc(ih/2)*2" -b:v 0 -crf 25 -f mp4 -vcodec libx264 -pix_fmt yuv420p src/assets/images/trending.mp4
```

성공적으로 변환된 것을 확인할 수 있다.

<img src="/assets/images/performance/convert-mp4.png">

다음 표는 `gif`와 `mp4`의 파일 크기를 비교한 것이다.

| 이름\파일형식 |  gif  |  mp4  |
| :-----------: | :---: | :---: |
|     find      |  2MB  | 218kB |
|     free      | 1.7MB | 126kB |
|   trending    | 1.3MB | 111kB |

## 3-3. webm 비디오로 변환하기

이번에는 webm 비디오로 변화를 해보자. `mp4`는 1999년부터 사용되었지만 `webm`은 비교적 최근인 2010년에 출시된 새로운 파일 형식이다. `webm`이 `mp4`와 비교했을 때 파일 크기가 작지만 모든 브라우저가 지원을 하는 것이 아니므로 두개의 형식의 비디오를 모두 만들어 사용하는 것이 좋다.

다음은 `ffempeg`명령어를 사용하여 `webm`으로 변환하는 명령어이다.

```bash
$ ffmpeg -i {원본 gif파일 경로} -c vp9 -b:v 0 -crf 41 {변환될 webm 파일 경로}
```

바로 프로젝트에 적용해보자.

```bash
$ ffmpeg -i src/assets/images/find.gif -c vp9 -b:v 0 -crf 41 src/assets/images/find.webm
$ ffmpeg -i src/assets/images/free.gif -c vp9 -b:v 0 -crf 41 src/assets/images/trending.webm
$ ffmpeg -i src/assets/images/trending.gif -c vp9 -b:v 0 -crf 41 src/assets/images/trending.webm
```

`webm`으로 성공적으로 변환되었고 `mp4`보다 더 작아진 파일 크기를 확인할 수 있다.

<img src="/assets/images/performance/convert-webm.png">

| 이름\파일형식 |  gif  |  mp4  | webm  |
| :-----------: | :---: | :---: | :---: |
|     find      |  2MB  | 218kB | 159kB |
|     free      | 1.7MB | 126kB | 105kB |
|   trending    | 1.3MB | 111kB | 84kB  |

## 3-4. 코드에 적용하기

다음은 `gif`파일을 바탕으로 렌더링을 하는 기존코드이다.

```tsx
const FeatureItem = ({ title, imageSrc }: FeatureItemProps) => {
  return (
    <div className={styles.featureItem}>
      <img className={styles.featureImage} src={imageSrc} />
      <div className={styles.featureTitleBg}></div>
      <h4 className={styles.featureTitle}>{title}</h4>
    </div>
  );
};
```

우선 `img`태그를 `video`태그로 바꾸어보자. 이때 `video`태그에는 다음과 같은 속성을 추가하여 `gif`처럼 보이게 할 수 있다.

- autoPlay: 자동 재생
- loop: 무한 반복
- muted: 음소거
- playsInline: 인라인으로 재생(전체 화면이 아닌)

```tsx
const FeatureItem = ({ title, imageSrc }: FeatureItemProps) => {
  return (
    <div className={styles.featureItem}>
      <video className={styles.featureImage} autoPlay loop muted playsInline>
        <h4 className={styles.featureTitle}>{title}</h4>
      </video>
    </div>
  );
};
```

다음 작업은 `source`태그를 통해 브라우저가 지원할 수 있는 비디오 파일을 추가한다. 먼저 `webm` 그 다음은 `mp4`이다.

```tsx
const FeatureItem = ({ title, webmSrc, mp4Src }: FeatureItemProps) => {
  return (
    <div className={styles.featureItem}>
      <video className={styles.featureImage} autoPlay loop muted playsInline>
        <source src={webmSrc} type="video/webm" />
        <source src={mp4Src} type="video/mp4" />
        <h4 className={styles.featureTitle}>{title}</h4>
      </video>
    </div>
  );
};
```

## 3-5. 결론 및 최종 결과

성공적으로 `gif`을 `webm`으로 변환하여 파일 크기를 줄였다.

<img src="/assets/images/performance/webm-size.png">

`애니메이션 GIF를 비디오로 대체하기`작업을 위해 참고한 글이다.

[애니메이션 GIF를 비디오로 대체하여 페이지를 더 빠르게 로드](https://web.dev/replace-gifs-with-videos/#create-mpeg-videos)

다음은 `애니메이션 GIF를 비디오로 대체하기` 작업을 진행한 커밋이다.

<a href="https://github.com/nlom0218/new-perf-basecamp/commit/fc5d6e797042dd8f291217cffd6c685fa1f6a89a" target="_blank">refactor: 3. 애니메이션 GIF를 비디오로 대체하기</a>
