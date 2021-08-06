공식 [문서](https://babeljs.io/docs/en/)를 재구성한 내용입니다.

---

# Babel이 뭔가요?

Babel은 JS 컴파일러다. 이는 주로 ECMAScript6+ 코드를 이전 버전의 JS로 변환하여 구형 브라우저 및 환경에서 동작하도록 해준다. 아래는 Babel이 해주는 주된 역할이다.

- Syntax 변환
- Target 환경에 존재하지 않는 기능에 대한 폴리필 추가 (`@babel/polyfill`)
- 소스 코드 변경 (codemods)
- 그 외 등등...

```js
// Babel Input: ES2015 arrow function
[1, 2, 3].map((n) => n + 1);

// Babel Output: ES5 equivalent
[1, 2, 3].map(function (n) {
  return n + 1;
});
```

## 어떻게 쓸 수 있을까?

### ES6+ 사양의 프로젝트

Babel은 최신 버전의 JS 사용을 구문 변환을 통해 지원해준다. Babel이 제공하는 플러그인들을 통해, 브라우저가 지원하지 않는 사양의 문법까지 사용할 수 있도록 한다.

### JSX와 React

Babel은 마찬가지로 JSX 구문도 변환할 수 있다.

```bash
npm install --save-dev @babel/preset-react
```

### Type Annotations (Flow & TypeScript)

Babel은 타입 주석(Type Annotation)을 제거할 수 있다. 사실, Babel 자체는 타입체킹을 수행하지 않으며, 타입 체킹을 위해서는 별도로 Flow나 TypeScript를 사용해야 함을 명심하자.

```bash
npm install --save-dev @babel/preset-flow
```

```bash
npm install --save-dev @babel/preset-typescript
```

### Pluggable

Babel은 여러 플러그인들로 구성되어 있다. 존재하는 플러그인 혹은 본인이 직접 작성한 플러그인을 통해 자신만의 구문 변환 파이프라인을 구성할 수 있다. 더 쉽게는 preset을 만들거나 사용해서 일련의 플러그인들을 사용해도 된다.

```
// 사실, 플러그인은 그냥 함수일 뿐이다.
export default function ({types: t}) {
  return {
    visitor: {
      Identifier(path) {
        let name = path.node.name; // reverse the name: JavaScript -> tpircSavaJ
        path.node.name = name.split('').reverse().join('');
      }
    }
  };
}
```

### Debuggable

**Souce map**은 컴파일된 코드를 쉽게 디버깅할 수 있도록 도와준다.

### Spec Compliant (규격 준수)

Babel은 가능한 ECMAScript 표준을 준수하려고 한다. 성능이 좀 떨어지더라도 표준 준수를 위해 더 구체적인 옵션을 가질 수 있다.

### Compact (압축)

Babel은 용량이 큰 런타임에 의존하지 않고 최대한 작은 양의 코드를 사용하려고 한다.

이는 상황에 따라 이루어지기 어려울 수도 있으며, 가독성, 파일 크기, 속도에 대한 규격을 준수하도록 하는 구체적인 변환에 대한 "느슨한" 옵션들이 존재한다.

# Usage Guide

일반적으로 Babel을 사용하는 케이스처럼, ES2015+ 문법들을 현재 브라우저에 적합한 사양으로 변환하고자 한다.

이러한 작업은 문법을 새로운 형태로 작성하고, 없는 기능에 대한 폴리필을 추가함으로써 이루어질 수 있다.

이에 대한 전반적인 과정은 아래와 같다.

## 일단 훑어보기

1. 패키지 인스톨

```sh
npm install --save-dev @babel/core @babel/cli @babel/preset-env
npm install --save @babel/polyfill
```

2. 프로젝트 루트에 `babel.config.json` (`v7.8.0` 이상 요구) config 파일 생성

```json
{
  "presets": [
    [
      "@babel/env",
      {
        "targets": {
          "edge": "17",
          "firefox": "60",
          "chrome": "67",
          "safari": "11.1"
        },
        "useBuiltIns": "usage",
        "corejs": "3.6.5"
      }
    ]
  ]
}
```

위의 설정은 하나의 예시일 뿐이고, 본인이 지원하고자 하는 브라우저 스펙에 따라 이를 적절히 설정해주어야 한다.

[여기](https://babeljs.io/docs/en/babel-preset-env)에서 `@babel/preset-env`가 보유한 옵션들을 확인하자.

`v7.8.0` 미만 버전의 Babel에서는 대신에 `babel.config.js`를 사용할 수 있다.

```js
const presets = [
  [
    '@babel/env',
    {
      targets: {
        edge: '17',
        firefox: '60',
        chrome: '67',
        safari: '11.1',
      },
      useBuiltIns: 'usage',
      corejs: '3.6.4',
    },
  ],
];

module.exports = { presets };
```

3. 이후 `src` 디렉토리에 있는 모든 코드에 대해 컴파일링을 수행하여 `lib`에 작성하고자 하는 경우 아래와 같이 cli를 이용할 수 있다.

```sh
./node_modules/.bin/babel src --out-dir lib
```

npm@5.2.0 이후부터 `./node_modules/.bin/babel`은 `npx babel`로 대체될 수 있다.

## CLI 기초

모든 Babel모듈들은 `@babel` 이라는 이름 하에 여러 개의 npm 패키지들로 나누어져 있다. (v7 이후)

이렇게 각각의 모듈로 디자인되어 있는 덕분에에, 각각의 상황에 적절하게 사용할 수 있다.

### Core Library

Babel의 핵심 기능들은 `@babel/core` 모듈에 위치해 있다.

```sh
npm install --save-dev @babel/core
```

이를 직접 JS 상에서 사용할 수 있다.

```js
const babel = require('@babel/core');

babel.transformSync('code', optionsObject);
```

### CLI 툴

`@babel/cli`는 터미널을 통해서 babel을 사용할 수 있게 해주는 툴이다. 아래와 같이 설치한다.

```sh
npm install --save-dev @babel/core @babel/cli
```

그리고 아래와 같이 사용한다.

```sh
./node_modules/.bin/babel src --out-dir lib
```

위 명령어는 `src` 디렉토리 내에 위치한 모든 JS 파일들을 파싱하여 지정된 모든 변환 작업들을 수행한다. 이후 각각의 파일들은 `lib` 디렉토리에 위치한다.

현재까지는 아무런 변환 작업을 지정해주지 않았기 때문에, 출력된 코드는 입력과 동일하게 될 것이다.

CLI 툴이 어떤 옵션들을 보유하고 있는지에 대해 알고 싶다면 `--help`를 이용하자.

## Plugins & Presets

Babel에서의 모든 변환들은 **Plugin**(이하 플러그인)을 통해서 이루어집니다. 이는 하나의 작은 JS 프로그램인데, Babel에게 코드를 어떤 식으로 변환해야 하는지 지시해주는 역할을 한다.

심지어 플러그인은 본인이 직접 작성할 수도 있다.

`@babel/plugin-transform-arrow-functions` 플러그인을 적용하는 간단한 예시를 확인해보자.

```sh
npm install --save-dev @babel/plugin-transform-arrow-functions

./node_modules/.bin/babel src --out-dir lib --plugins=@babel/plugin-transform-arrow-functions
```

이후, 변환된 코드는 아래와 같아진다.

```js
const fn = () => 1;

// converted to

var fn = function fn() {
  return 1;
};
```

기본적으로는 이렇게 하나의 플러그인을 통해 변환을 수행할 수 있지만, 이렇게 플러그인을 하나하나씩 추가하는 것은 다소 귀찮아 보인다.

이런 경우에 사용할 수 있는 것이 **Preset**(이하 프리셋)이며, 이는 플러그인의 묶음이라고 이해할 수 있다.

플러그인과 마찬가지로, 이러한 프리셋 역시 자신이 원하는 플러그인들을 임의로 지정해 만들어낼 수 있다.

Babel에서 자주 사용되는 preset으로는 `env`가 있다.

```sh
npm install --save-dev @babel/preset-env

./node_modules/.bin/babel src --out-dir lib --presets=@babel/env
```

별 다른 설정을 하지 않더라도, `preset-env`는 모던 JS를 지원하기 위한 모든 플러그인들을 추가한다.

물론 프리셋 역시 옵션을 지정할 수 있으며, 이는 CLI 상에서 지정하는 것이 번거롭기에 아래처럼 config 파일을 생성하는 방식을 많이 이용한다.

## Plugins & Presets

CLI를 통해 모든 옵션을 지정하기보다는 별도의 설정 파일을 만드는 방식이 종종 사용된다.

본인이 원하는 형태에 따라 작성해야 할 Configuration 파일 형태들이 조금씩 달라질 수 있다.

상세 설정에 관련해서는 여기 [문서](https://babeljs.io/docs/en/configuration)를 참고하도록 하자.

v7.8.0 이상에서는 일반적으로 `babel.config.json`을 생성한다.

```json
{
  "presets": [
    [
      "@babel/env",
      {
        "targets": {
          "edge": "17",
          "firefox": "60",
          "chrome": "67",
          "safari": "11.1"
        }
      }
    ]
  ]
}
```

이제 `env` 프리셋은 위에서 우리가 지정한 target 브라우저들에서 지원하지 않는 기능들에 대한 플러그인들만을 사용할 것이다.

이로써 문법 변환에 대해서는 모두 알아봤다.

`v7.4.0` 미만의 버전에 대해서는 `@babel/polyfill` 모듈이 별도로 사용되지만, 이들 내용이 `core-js/stable`과 `regenerator-runtime/runtime` 플러그인에 각각 추가되었으므로 별도로 사용할 필요가 없게 되었다.

혹시나 `v.7.4.0` 미만의 Babel을 사용해야 하는데, polyfill이 필요한 경우에는 여기 [문서](https://babeljs.io/docs/en/usage#polyfill)를 따로 찾아보자.
