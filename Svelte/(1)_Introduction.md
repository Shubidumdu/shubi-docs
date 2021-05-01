# Introduction

## Svelte란?

Svelte는 React, Vue와 같이 유연하게 상호작용 가능한 UI를 구성하기 위한 JS 프레임워크다.

단, 한가지 중요한 차이가 있는데, Svelte가 런타임 시점에 코드를 해석하는 것이 아니라 **빌드 과정**을 거친다는 점이다. 즉, 프레임워크의 추상화에 따른 성능적인 비용이 없고, 최초 로딩에 있어 부담이 덜하다.

Svelte에서 애플리케이션은 하나 이상의 컴포넌트들로 구성된다. 컴포넌트는 HTML / CSS / JS를 하나로 재사용 가능하게 묶은 코드 블럭이며, 이는 `.svelte` 확장자 파일로 관리된다.

## Data 추가

```svelte
<script>
  let name = 'world';
</script>

<h1>Hello {name}!</h1>
```

## 동적 어트리뷰트

```svelte
<script>
  let src = 'some-image.png';
</script>

<img src={src} alt="A man dances.">
```

## 스타일링

```svelte
<style>
  p {
		color: purple;
		font-family: 'Comic Sans MS', cursive;
		font-size: 2em;
	}
</style>

<p>This is a paragraph.</p>
```

## 중첩 (Nested) 컴포넌트

```svelte
<script>
	import Nested from './Nested.svelte';
</script>

<p>This is a paragraph.</p>
<Nested/>
```

## HTML 태그

JS에서 일반적인 string을 HTML 태그로서 삽입하고자 할 때 사용한다.

```svelte
<script>
	let string = `this string contains some <strong>HTML!!!</strong>`;
</script>

<p>{@html string}</p>
```

### **주의!**

> Svelte는 `{@html ...}` 내에 작성된 내용을 DOM에 추가하기 전에 그 어떤 처리도 하지 않는다. 다시 말해, 신뢰할 수 없는 출처를 통해 해당 기능을 적용하고자 하는 경우, 이에 대한 이스케이프를 직접 처리해주는 것이 매우 중요하다. 그렇지 않은 경우 XSS 공격의 위험이 있다.

## 프로젝트 세팅

Svelte는 Rollup이나 Webpack과 같은 빌드 툴과 함께 사용할 수 있다.

또, VS Code 상에서 Svelte 익스텐션을 설치하여 IDE 상의 피드백을 받을 수 있다.

Webpack을 통한 세팅이 완료가 되면, `svelte-loader`가 각각의 컴포넌트들을 JS 클래스로 변환한다. 해당 컴포넌트들은 `new` 키워드로 아래와 같이 사용할 수 있다.

```js
import App from './App.svelte';

const app = new App({
  target: document.body,
  props: {
    // we'll learn about props later
    answer: 42,
  },
});
```
