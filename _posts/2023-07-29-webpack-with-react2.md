---
title: "웹팩으로 리액트 시작하기 2 - 절대 경로"
categories:
  - 프로그래밍
  - webpack
tags:
  - Webpack with React
image:
  thumbnail: /assets/images/webpack.png
  path: /assets/images/webpack.png
---

**웹팩으로 리액트 시작하기**의 두 번째 파트에서는 `eslint`, `tsconfig`, `webpack`, 설정을 통해 `절대 경로`를 적용하는 방법을 다룬다.

# 1. eslint 설정

`eslint` 을 아직 설치를 하지 않았으니 이를 먼저 설치한다. 또한 `import`에 관한 플러그인을 함께 설치한다.

```bash
yarn add -D eslint eslint-plugin-import
```

이후 간단한 명령어를 통해 `eslint`설정 파일을 생성한다. 다음의 명령어를 실행하면 각자가 원하는 `eslint`을 설정할 수 있다. 해당 과정을 통해 여러 가지의 플러그인이 생성되고 `eslint`설정 파일이 루트경로에 생성된다.

```bash
npx eslint --init
```

이제 생성된 `eslint` 설정 파일에서 다음과 같은 설정을 추가한다.

- `import` 플러그인 추가
- `import/order` 규칙 추가

```json
{
  // ...
  "plugins": ["@typescript-eslint", "react", "import"], // `import` 플러그인 추가
  "rules": {
    "import/order": [
      // `import/order` 규칙 추가
      "error",
      {
        "groups": [
          "builtin",
          "external",
          "internal",
          ["parent", "sibling"],
          "index",
          "unknown"
        ],
        "pathGroups": [
          {
            "pattern": "react*,react*/**",
            "group": "external",
            "position": "before"
          },
          {
            "pattern": "@Components/**/*",
            "group": "internal",
            "position": "after"
          }
          // ...
        ],
        "newlines-between": "always",
        "alphabetize": {
          "order": "asc"
        }
      }
    ]
  }
}
```

`import/order` 규칙에 대한 자세한 내용은 아래의 글을 참고

[import/order github](https://github.com/import-js/eslint-plugin-import/blob/main/docs/rules/order.md)

<br/>

# 2. tsconfig.json 설정

`tsconfig.json`에 파일 경로에 대한 옵션을 추가한다.

```json
{
  "compilerOptions": {
    // ...
    "baseUrl": "./src",
    "paths": {
      "@Components/*": ["./components/*"]
      // ...
    }
    // ...
  }
}
```

- `baseUrl`: 기본 `url`을 설정한다. 위 설정에서는 `src`폴더 내부에 있는 파일을 기준으로 한다.
- `paths`: 내가 만들고자 하는 절대 경로를 `key`, 실제 경로를 `value`로 한다.

위 설정에서는 `tsconfig.json` 파일의 위치를 기준으로 ‘./src/components’ 이후의 파일을 ‘@Components’ 을 붙여 import을 한다는 뜻이다. 만약 `components` 폴더 안에 또 폴더가 있고 그 안에서 파일을 import을 하려고 하면 다음과 같이 수정을 하면 된다.

```json
{
  "compilerOptions": {
    // ...
    "baseUrl": "./src",
    "paths": {
      "@Components/*": ["./components/**/*"]
      // ...
    }
    // ...
  }
}
```

위와 같이 수정하면 ‘./src/components’이후 폴더가 오더라도 절대 경로를 사용할 수 있다.

만약 `tsconfig.json`에서 경로 관련된 옵션을 다른 파일로 분리하고 싶으면 다음과 같이 하면 된다.

```json
// tsconfig.json

{
  "extends": "./tsconfig.paths.json",
  "compilerOptions": {
    // ...
  }
}
```

```json
// tsconfig.paths.json

{
  "compilerOptions": {
    "baseUrl": "./src",
    "paths": {
      "@Components/*": ["./components/*"]
      // ...
    }
  }
}
```

<br/>

# 3. 절대 경로 사용하기

`components` 폴더 내에 Button 컴포넌트를 만들고 App.js에서 불러와 보자.

```tsx
// src/components/Button.tsx

const Button = () => {
  return <button>버튼</button>;
};

export default Button;
```

```tsx
// src/App.tsx

import Button from "@Components/Button";

const App = () => {
  return (
    <div>
      <Button />
    </div>
  );
};

export default App;
```

절대 경로를 사용하면 잘 불러와지는 것을 볼 수 있다. 하지만 `yarn dev`로 서버를 실행시키면 다음과 같이 컴파일 오류가 나타난다. 그 이유는 웹팩에 절대경로를 알려주지 않았기 때문이다.

<div style="text-align: center"> <img src="https://github.com/haru-study/haru-study.github.io/blob/main/_posts/img/path_error.png?raw=true"> </div>

<br/>

# 4. webpack 설정하기

`webpack`의 `resolve`속성에 `alias`를 추가한다.

```javascript
// webpack.config.js

module.exports = () => {
  const isDevelopment = process.env.NODE_ENV !== "production";

  return {
    // ...
    resolve: {
      extensions: [".ts", ".tsx", ".js", ".jsx"],
      alias: {
        "@Components": path.resolve(__dirname, "src/components"),
        // ...
      },
    },
    // ...
  };
};
```

이후 서버를 종료하고 다시 시작하면 `Button`컴포넌트가 정상적으로 불러오는 것을 확인할 수 있다.
