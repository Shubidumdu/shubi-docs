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
